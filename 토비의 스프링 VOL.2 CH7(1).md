#### 테스트 프레임워크

빈으로 사용된다.
- 다른 빈에 의해 DI 돼서 사용되는 서비스라는 의미
- 다른 빈이나 정보에 의존한다.

스프링 학습하기
- 스프링 API 문서로 구현 인터페이스와 프로퍼티를 분석한다.
- 빈 클래스의 프로퍼티 중 인터페이스 타입의 프로퍼티는 해당 구현 클래스의 확장 포인트다.

LazyConnectionDataSourceProxy
- 트랜잭션 매니저와 DataSource 사이에서 DB 커넥션 생성을 최대한 지연시켜준다.
- Statement를 만들어 최초로 SQL을 실행하는 시점에 커넥션을 생성하고 그 전에는 가상의 프록시 Connection을 돌려준다.

AbstractRoutingDataSource
- 다중 DataSource에 대한 라우팅을 제공한다.
- 룩업 키와 DataSource맵을 정의해야한다.

``` xml
<bean id="dataSourceRouter" class="...ReadOnlyRoutingDataSource">
    <property name="targetDataSources">
        <map>
            <!-- 현재 트랜잭션의 속성이 쓰기 가능한 트랜잭션이면 masterDataSource사용 -->
            <entry key="READWRITE" value-ref="masterDataSource"> 
            <!-- 현재 트랜잭션의 속성이 읽기전용 트랜잭션이면 readOnlyDataSource사용 -->
            <entry key="READONLY" value-ref="readOnlyDataSource">
        </map>
    </property>
    <property name="defaultTargetDataSource" ref="masterDataSource">
</bean>
```

#### IoC 컨테이너 DI

- BeanPostProcessor
    - postProcessBeforeInitialization() 메소드로 초기화 메소드가 호출되기 전에 실행될 로직을 구현할 수 있다.
    - 빈 오브젝트를 바꿔치기하거나 기본 설정과 다른 기능을 부여할 수 있다.
    - 자동 프록시 생성기도 BeanPostProcessor를 구현한 빈이다.

- BeanFactoryPostProcessor
    - 빈 팩토리에 대한 후처리를 가능하게 하는 인터페이스.
    - 등록된 빈의 메타정보 자체를 조작할 수 있다.
    - ConfigurationClassPostProcessor가 @Configuration과 @Bean을 이용해서 새로운 빈을 추가해준다.

- Marshaller/Unmarshaller
    - Marshaller : Object to XML, Unmarshaller : XML to Object
    - OXM 어댑터 빈은 Marshaller, Unmarshaller 둘 다 구현하기 때문에 동일한 빈 레퍼런스를 주입한다.
    - JAXB,JiBX,XMLBeans OXM 어댑터 빈은 태그가 제공된다.

#### 리모팅

- 원격 시스템과 스프링 애플리케이션이 연동해서 동작하게 해주는 기술

- 익스포터
    - 원격 요청을 받아 서비스 빈에 요청을 전달해주는 빈
    - HTTP 요청을 전달받으면 해석한 후 등록된 인터페이스를 이용해 서비스 빈을 호출한다.

```xml
    <!-- /remoting/userservice URL로 요청이 들어오면 userService 빈 호출-->
    <bean name="/remoting/userservice" class="..HttpInvokerServiceExporter">
        <property name="service" ref="userService"/>
        <property name="serviceInterface" ref="..UserService"/>
    </bean>
```

- 프록시 
    - 원격 시스템에 있는 오브젝트를 대신해서 클라이언트 오브젝트의 호출을 받고, 이를 원격 오브젝트에 전송해서 결과를 가져온 뒤 클라이언트 오브젝트에 돌려주는 역할을 하는 빈 오브젝트

- RESTful 서비스 템플릿
    - 원격 RESTful 서비스 사이트를 이용해 결과를 가져오는 기능만 제공한다.
    - 템플릿/콜백 방식의 템플릿을 사용한다.

``` java
    //RestTemplate  
    String result = template.getObject("서비스URL패턴",String.class, "파라미터");
```

#### 태스트 실행과 스케줄링

- 태스크 : 독립적인 스레드 안에서 동작하도록 만들어진 오브젝트
- TaskExecutor : 태스크를 실행하도록 만들어진 추상화된 인터페이스  
- TaskScheduler : 태스크를 일정한 간격 또는 시간 기준에 따라 실행되도록 스케줄링 기능을 제공하는 추상화된 서비스 인터페이스
- @Scheduled : 태스크 역할을 맡을 메소드에 부여해서 스케줄이 적용되게 해준다.

```xml
    <task:executor id="executor" pool-size="5"> <!-- TaskExecutor 빈 등록 -->
    <task:schduler id="scheduler"> <!--TaskScheduler 타입의 빈 등록-->
    <!--아래 두 태그로 일반 빈 메소드를 태스크로 활용할 수 있고, 여러개의 스케줄을 등록할 수 있다.-->
    <task:scheduled-tasks>
        <task:scheduled/>
    </task:scheduled-tasks>
```

#### Reference
* * *
책 : 토비의 스프링 3.1 Vol.2 스프링의 기술과 선택 (저 이일민)
