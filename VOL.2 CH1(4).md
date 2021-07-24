
#### 프로토타입과 스코프

1. 싱글톤 빈
- 기본적으로 스프링의 빈은 싱글톤 > 상태 값을 저장하는 인스턴스 변수는 두지 않는다.

2. 프로토타입 빈
- 프로토타입 스코프는 컨테이너에게 빈을 요청할 떄마다 새로운 오브젝트를 생성해준다.
- 프로토타입 스코프를 갖는 빈은 DI를 통해 컨테이너 밖으로 전달된 후에는 더이상 스프링이 관리하지 않는다.
- 빈 오브젝트의 관리는 DI 받은 오브젝트에 달려있다.
- 매번 새로운 오브젝트가 필요하면서 DI를 통해 다른 빈을 사용할수 있어야할 때 사용한다.
- 주로 new 키워드를 대신하기 위해 타용. DL 방식으로 사용해야 한다.
    ```java
        @Scope("prototype")
        static class PrototypeBean{}
    ```

3. DL 방식을 코드에서 사용하는 방법
- ApplicationContext, BeanFactory를 DI 받은 뒤 getBean()메소드로 빈을 가져오는 방법
- ApplicationContext를 DI 받아서 getBean()으로 빈을 가져오는 방식으로 동작하는 팩토리를 만들어 이를 사용하는 방법(Controller > ObjectFactory, ObjectFactoryCreatingFactoryBean > ApplicationContext)
- DL 방식으로 가져올 빈을 리턴하는임의의 메소드를 정의한 인터페이스를 만든뒤 ServiceLocatorFactoryBean 사용

```java
    // ObjectFactoryCreatingBean
    @Configuration
    public class ObjectFactoryBeanConfig{
        @Bean
        public ObjectFactoryCreatingFactoryBean serviceRequestFactory(){
            ObjectFactoryCreatingFactoryBean bean = new ObjectFactoryCreatingFactoryBean();
            bean.setTargetBeanName("serviceRequest");
            return bean;
        }
    }
```

``` xml
    <!-- ServiceLocatorFactoryBean-->
    <bean class="org.springframework.beans.factory.config.ServiceLocatorFactoryBean">
        <!-- ServiceRequestFactory는 getServiceFactory 메소드가 있는 인터페이스-->
        <property name="serviceLocatorInterface" value=".. ServiceRequestFactory"/>
    </bean>>
```

``` java
    @Resource
    private ObjectFactory<ServiceRequest> serviceRequestFactoryOF;
    @Autowired
    ServiceRequestFactory serviceRequestFactorySF;

    public void serviceRequestUse(HttpRequest request){
        ServiceRequest serviceRequestOF = this.serviceRequestFactoryOF.getObject();
        ServiceRequest serviceRequestSF = this.serviceRequestFactorySF.getServiceFactory();
    }
```

- 메소드 주입 사용 : 일정한 규칙을 따르는 추상 메소드를 작성해두면 새로운 프로토타입 빈을 가져오는 기능을 담당하는 메소드를 런타임에 추가해주는 기술

``` xml
    <bean id="serviceRequestController" class="...ServiceRequestController">
        <!-- 스프링이 구현해줄 추상 메소드, 가져올 빈의 이름-->
        <lookup-method name="getServiceRequest" bean="serviceRequest">
    </bean>
```

``` java
    abstract public ServiceRequest getServiceRequest();

    public void serviceRequestUse(HttpRequest request){
        ServiceRequest serviceRequest = this.getServiceRequest();
    }
```
- @Inject + Provider<T> : ObjectFactory와 비슷. 빈을 등록해주지 않아도 된다.

``` java
    @Inject
    Provider<ServiceRequest> serviceRequestProvider;

    public void serviceRequestUse(HttpRequest request){
        ServiceRequest serviceRequest = this.serviceRequestProvider.get();
    }
```

4. 스코프
- 요청 스코프 
    - 웹 요청 안에서 만들어지고 해당 요청이 끝날 때 제거된다. 요청별로 독립적인 빈이 만들어진다.
    - 애플리케이션 코드에서 생성한 정보를 서비스나 인터셉터 등에 전달할 떄 사용.

- 세션 스코프, 글로벌세션 스코프
    - HTTP 세션과 같은 존재 범위를 갖는 빈으로 만들어주는 스코프
    - 로그인 정보나 사용자별 선택옵션을 저장하기에 유용하다.
    - 세션 스코프를 사용해서 HTTP 세션에저장되는 정보를 모든 계층에서 안전하게 이용할 수 있다.

- 애플리케이션 스코프
    - 서블릿 컨텍스트에 저장되는 빈 오브젝트
    - 싱글톤 스코프와 비슷하지만 존재범위가 다른 경우가 있다.

- 스코프 빈 사용방법
- 스코프 빈은 DL 방식으로 사용해야 한다.
- 직접 스코프 빈을 DI하는 대신 스코프 빈에 대한 프록시를 DI한다.

```java
    //@Scope(value="session",proxyMode=ScopeProxyMode.TARGET_CLASS) > 프록시를 이용해 DI
    @Scope("session")
    public class LoginUser{
        String loginId;
        String name;
    }

    public class LoginService{
        @Autowired
        Provider<LoginUser> loginUserProvider;
        @Autowired
        LoginUser loginUserProxy;

        public void login(Login login){
            LoginUser loginUser = loginUserProvider.get();
            loginUser.setLoginId(login.getId());
            this.loginUser.setLoginId(login.getId());
        }
    }
```

5. 빈 설정 메타정보

- 빈 식별자 : id, name
    - @Compnent("빈 이름"), @Named("빈 이름") : 애노테이션에서의 빈이름 설정. 보통 클래스 이름에서 첫 글자만 소문자랑 바꾼다 
    - @Bean 메소드를 이용하면 메소드 이름이 빈 이름이다. @Bean("빈 이름") 형식으로 지정할 수 있다.

- 초기화 메소드 : 빈 오브젝트가 생성되고 DI 작업을 마친 다음에 실행되는 메소드
    - 초기화 콜백 인스턴스 : InitializingBean 인터페이스를 사용해서 빈을 작성하는 방법. afterPropertiesSet() 메소드로 초기화
    - init-method 지정 : bean 태그에 init-method 애트리뷰트로 초기화 작업을 수행할 메소드 이름을 지정한다. 코드에서는 @Bean(init-method="메소드 이름") 처럼 사용한다.
    - @PostConstruct : 초기화를 담당할 메소드에 @PostConstruce 부여

- 제거 메소드 : 컨테이너가 종료될 때 호출. 빈이 사용한 리소스를 반환하거나 종료 전 처리해야하는 작업 수행.
    - 제거 콜백 인스턴스 : DisposableBean 인터페이스 구현. destroy() 구현
    - destory-method : 빈 태그에 destory-method에 제거 메소드 지정
    - @PreDestory

6. 팩토리 빈과 팩토리 메소드
- 팩토리 빈 : 생성자 대신 오브젝트를 생성해주는 코드의 도움을 받아서 빈 오브젝트를 생성하는 것. 
- FactoryBean 인터페이스 : 다이나믹 프록시를 빈으로 등록하기 위해 FactoryBean을 구현해서 다이내믹 프록시를 생성하는 getObject() 를 구현하고 팩토리 빈으로 등록해서 사용
- 스태틱 팩토리 메소드 : 클래스의 스태틱 메소드를 호출해서 인스턴스를 생성하는 방법. bean 태그의 factory-method에 스태틱 메소드 지정
- 인스턴스 팩토리 메소드 : 오브젝트의 인스턴스 메소드를 이용해 빈 오브젝트 생성. bean 태그의 factory-bean, factory-method에 빈, 메소드 설정
- @Bean 메소드
