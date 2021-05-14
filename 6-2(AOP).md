### ProxyFactoryBean

- 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈
- 순수하게 프록시를 생성하는 작업만을 담당
- 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다. (타깃 오브젝트에 대한 정보도 제공받는다.)
- 새로운 부가기능을 추가할 때마다 프록시와 프록시 팩토리 빈을 추가하지 않아도 된다.
- 독립적이고, 여러 프록시가 공유할 수 있는 어드바이스와 포인트컷으로 확장 기능을 분리할 수 있다.

```java
@Test
public void proxyFactoryBean(){
    // 인터페이스 타입 전달할 필요 없음.
    ProxyFactoryBean pfBean = new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());
    pfBean.addAdvice(new UppercaseAdvice()); // 여러개의 MethodInterceptor 추가 가능
}
```

### Advice

- 타깃 오브젝트에 적용하는 부가기능을 담은 오브젝트 (부가기능 제공)
- InvocationHandler와 다르게 타깃과 메소드 선정 알고리즘 코드에 의존하지 않는다. 
- 직접 타깃을 호출하지 않는다. 
- 부가기능 재사용 가능
- Advice : 템플릿, MethodInvocation : 콜백

```java 

// 타깃 오브젝트에 종속 x
public class UppercaseAdvice implements MethodInterceptor {
    @Override
    // MethodInvocation는 타킷 오브젝트의 메소드를 실행할 수 있다. 타깃 오브젝트의 정보를 가지고 있다.-
    public Object invoke(MethodInvocation invocation) throws Throwable {
        String ret = (String)invocation.proceed(); // 타킷 메소드 호출 (콜백 오브젝트)
        return ret.toUpperCase();
    }
}
```

### Pointcut

- 메소드 선정 알고리즘을 담은 오브젝트
- 클라이언트 -> 프록시 메소드 호출 -> 포인트 컷에서 부가기능 부여할 메소드인지 체크 -> 어드바이스 호출
- 어드바이저 : 포인트컷 + 어드바이스

``` java
    NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
    pointcut.setMappedName("sayH*"); // sayH로 시작하는 메소드에만 부가기능 적용
    pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UppercaseAdvice()));
```

### BeanPostProcessor

- 스프링 빈 오브젝트로 만들어지고 난 후에, 빈 오브젝트를 다시 가공할 수 있다.
- DefaultAdvisorAutoProxyCreator 등록되어 있을 경우 : 빈 생성 -> 빈 후 처리기 -> 어드바이저 포인트 컷 정보를 확인해 프록시 적용 대상인지 확인 (클래스 확인 후 메소드 확인) -> 적용 대상이면 프록시 생성기에 프록시를 만들게 함 -> 프록시에 어드바이저 연결 -> 실제 빈 대신 프록시 빈 전달
- 직접 타겟과 어드바이저를 설정해주지 않아도 된다.

``` xml
    <!-- 빈 후처리기 -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

    <!-- 클래스 이름이 ServiceImpl로 끝나면 프록시 적용 -->
    <bean id="transactionPointcut" class="toby.spring.study.service.NameMatchClassMethodPointcut">
        <property name="mappedClassName" value="*ServiceImpl"/>
        <property name="mappedName" value="upgrade*"/>
    </bean>

    <!-- 만들어진 프록시의 upgrade로 시작하는 메소드에 transactionAdvice 연결 -->
    <bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
        <property name="advice" ref="transactionAdvice"/>
        <property name="pointcut" ref="transactionPointcut"/>
    </bean>

    <bean id="transactionAdvice" class="toby.spring.study.service.TransactionAdvice">
        <property name="transactionManager" ref = "transactionManager"/>
    </bean>
```

### 포인트컷 표현식

- 표현식 언어를 사용해서 포인트컷을 작성할 수 있도록 하는 방법
- 문법구조
    - execution() : 메소드의 시그니처 비교, 클래스 이름에 적용되는 패턴은 타입 패턴이다.
    - execution([접근 제한자 패턴] 리턴 값 타입패턴 [ 패키지 클래스 타입패턴.] 메소드 이름패턴 (파라미터 타입패턴 | "..",...) [throws 예외 패턴])
    - bean() : 빈의 이름으로 비교 (ex. bean(*Service) - 아이디가 Service로 끝나는 모든 빈)
    - annotation() : 애노테이션이 적용되어 있는 것으로 보고 메소드를 선정

### AOP

- 애플리케이션의 핵심적인 기능에서 부가적인 기능을 분리해서 모듈로 만들어 설계하고 개발하는 방법 ( 특정한 관점을 기준으로 바라볼 수 있게 해준다는 의미에서 관점 지향 프로그래밍이라고도 한다. )
- 프록시 AOP : 어드바이스가 적용되는 객체를 프록시로 만들어서 DI로 연결된 빈 사이에 적용해 타기의 메소드 호출 과정에 참여해서 부가기능을 제공한다.
- 주요 용어
    - 타깃 : 부가기능을 부여할 대상
    - 어드바이스 : 타깃에게 제공할 부가기능을 담은 모듈
    - 조인 포인트 : 어드바이스가 적용될 수 있는 위치(타깃 오브젝트가 구현한 인터페이스의 모든 메소드)
    - 포인트컷 : 어드바이스를 적용할 조인 포인트를 선별하는 작업 또는 기능을 정의한 모듈
    - 프록시 : 클라이언트와 타깃 사이에서 부가기능을 제공하는 오브젝트
    - 어드바이저 : 포인트컷 + 어드바이스 객체
    - Aspect : AOP의 기본 모듈 (어드바이저는 단순한 Aspect라 볼 수 있다.)
```

* * *
책 : 토비의 스프링 3.1 Vol.1 스프링의 이해와 원리 (저 이일민)
