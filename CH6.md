
### 프록시 패턴 이용해서 트랜잭션 코드 분리하기

- DI 적용해서 비즈니스 로직과 트랜잭션 코드 분리
- Client(UserServiceTest) -> UserServiceTx(프록시 / 트랜잭션 처리 로직) -> UserServiceImpl(타킷 / 비즈니스 로직)
- UserServiceTx에서 트랜잭션 경계설정 작업을 하고 실제 비즈니스 로직은 UserServiceImpl에서 담당
- 핵심 기능 코드와 부가기능 코드 분리, 부가기능을 통해 핵심기능 이용

### 프록시 패턴

- 클라이언트에게 실제 타깃 오브젝트 대신 프록시를 넘겨주는 것.
- 프록시 : 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것
- 클라이언트가 프록시의 메소드를 통해 타깃을 사용하려고 하면 프록시가 타깃 오브젝트를 생성하고 요청을 위임해준다.
- 타깃의 기능 자체에는 관여하지 않으면서 접근하는 방법을 제어해주는 프록시를 이용하는 것.
- 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
- 지정된 요청에 대해서 부가기능을 수행한다.
- 객체에 추가적인 기능을 동적으로 유연하게 첨가
- 장점 : 비즈니스 로직을 담당하는 UserServiceImp는 트랜잭션을 신경쓰지 않아도 된다. 그리고 테스트를 더 쉽게 만들 수 있다.
- 단점 : 타깃의 인터페이스를 모두 구현해야하고, 위임하는 코드를 작성하기 번거롭다. 부가기능 코드가 중복될 가능성이 많다.

``` java 
public class UserServiceTx implements UserService{
    UserService userService; // UserServiceImpl 주입
    PlatformTransactionManager transactionManager;
    // 코드들
    public void upgradeLevels(){
        // 부가기능 수행
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        try{
            userService.upgradeLevels(); // 핵심 기능 호출, 위임
            this.transactionManager.commit(status);
        }
        // 코드들
    }
}
```

``` java
    UserServiceTx userServiceTx = new UserServiceTx(); // 데코레이터, 트랜잭션 경계설정 기능 부여
    UserServiceImpl userServiceImpl = new UserServiceImpl(); // 타깃
    userServiceTx.setUserService(userServiceImpl); 
```

### 다이내믹 프록시

- 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트.
- 타깃의 인터페이스와 같은 타입으로 만들어진다. 
- 인터페이스를 모두 구현해서 클래스를 정의할 필요가 없다.
- 부가기능은 `InvocationHandler`를 구현한 오브젝트에 담는다.
- InvocationHandler 구현 오브젝트가 리플렉션을 이용해 위임 코드를 만든다.
- 장점 : 인터페이스의 메소드가 늘어나도 코드를 추가하지 않아도 된다. 타깃의 종류에 상관없이도 적용이 가능하다.
- 단점 : 다이내믹 프록시 오브젝트는 일반적인 스프링 빈으로는 등록할 방법이 없다.

```java
private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    try{
        Object ret = method.invoke(target,args); // 타깃 메소드 실행, 위임
        this.transactionManager.commit(status); // 부가기능
        return ret;
    } catch (InvocationTargetException e){
        this.transactionManager.rollback(status);
        throw e.getTargetException();
    }
}
```

### 프록시 팩토리 빈

- 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈
- 스프링은 FactoryBean 인터페이스를 구현한 클래스가 빈의 클래스로 지정되면, 팩토리 빈 클래스의 오브젝트의 getObject() 메소드를 이용해 오브젝트를 가져오고, 이를 빈 오브젝트로 사용한다.
- 프록시 팩토리 빈을 사용하면 다이내믹 프록시 오브젝트를 스프링의 빈으로 만들어 줄 수 있다.
- 장점 : 재사용이 가능하다.
- 단점 : 한번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 것을 불가능하다. 빈 설정 중복되어 설정파일이 복잡해진다.

``` java
public class TxProxyFactoryBean implements FactoryBean<Object> {
    // 코드
    @Override
    public Object getObject() throws Exception {
        TransactionHandler txHandler = new TransactionHandler();
        txHandler.setTarget(target);
        txHandler.setTransactionManager(transactionManager);
        txHandler.setPattern(pattern);
        // 다이내믹 프록시 오브젝트 생성
        return Proxy.newProxyInstance(getClass().getClassLoader(), new Class[] { serviceInterface }, txHandler);
    }
}
```
