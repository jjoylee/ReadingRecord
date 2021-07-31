
#### 빈의 종류
    
- 애플리케이션 로직 빈 : 애플리케이션 로직을 담고 있는 주요 클래스의 오브젝트 빈
- 애플리케이션 인프라 빈 : 애플리케이션의 로직을 담당하지는 않지만 애플리케이션의 로직 빈을 지원한다. (ex.DataSource)
- 컨테이너 인프라 빈 : 스프링 컨테이너의 기능을 확장해서 빈의 등록과 생성 등의 작업에 참여하는 빈. 주로 전용 태그를 통해 등록. (ex.DefaultAdvisorAutoProxyCreator, annotation-config 태그)

참고 : \<context:component-scan> 태그는 빈 스캐너 기능을 제공하고 \<component:annotation-config>에 의해 등록되는 빈들도 등록한다.

#### IoC/DI 설정 방법의 발전

- 스프링 1.x : XML을 이용한 빈 등록방법 사용
- 스프링 2.0 : 스키마와 네임스페이스를 가진 전용 태그 제공
- 스프링 2.5 : 빈 자동등록 방식과 애노테이션 기반의 의존관계 설정 방법이 등장. 애플리케이션 로직 빈의 설정 메타정보는 애노테이션을 이용해 자바 코드로 작성
- 스프링 3.0 : 자바 코드를 이용해 빈 설정정보, 설정 코드를 만들 수 있다. 컨테이너 인프라 빈은 XML을 이용해서 등록
- 스프링 3.1 : 컨테이너 인프라 빈도 자바 코드로 등록할 수 있다.

#### 자바 코드를 이용한 컨테이너 인프라 빈 등록

- @ComponentScan 
    - \<context:component-scan>, 스테레오타입 애노테이션이 붙은 빈을 자동으로 스캔해서 등록
    - basePackageClasses=마커클래스, 인터페이스 : 마커 클래스나 인터페이스의 패키지가 빈 스캐닝 기준 패키지가 된다.
    - excludeFilters로 스캔 대상에서 제외할 클래스를 설정할 수 있다.
- @Import
    - @Configuration 클래스를 빈 메타정보에 추가할 때 사용
- @ImportResource
    - XML이 꼭 필요한 빈 설정만 별도의 파일로 작성한 뒤, @Configuration 클래스에서 @ImportResource로 XML파일의 빈 설정을 가져올 수 있다.
- @EnableTransactionManagement
    - \<tx:annotation-driven/>
    - AOP 관련 빈 등록
- AnnotationConfigWebAplicationContext는 \<context:annotation-config/>이 등록해주는 빈을 기본적으로 추가해준다.

#### 런타임 환경 추상화와 프로파일

- 환경에 맞게 빈의 설정정보를 달라지게 만든다.
1. 메타정보를 담은 XML이나 클래스를 따로 준비한다. 
    - 개발이나 유지보수가 계속 진행되면서 설정정보를 관리하는 것이 번거롭다는 단점이 있다.
2. 환경에 따라 달라지는 정보를 담은 프로퍼티 파일을 활용한다.
    - 환경에 따라 달라지는 외부 정보만 프로퍼티 파일에 두고 읽어서 사용한다.
    - 환경에 따라 빈 메타정보  자체가 바뀌는 경우에는 해결할 수 없다.
    - 프로퍼티 : 키와 그에 대응되는 값의 쌍
3. 런타임 환경 추상화를 이용한다.
    - 환경에 따라 프로파일과 프로퍼티 소스가 다르게 설정된 Environment 오브젝트(런타임 환경 오브젝트) 사용
    - 런타임 환결 = 프로파일 + 프로퍼티 소스
    - 환경에 따라 다르게 구성되는 빈들을 다른 프로파일 안에 정의한다. 애플리케이션 컨텍스트가 시작될 때 지정된 프로파일의 빈만 생성되게 한다.    
    - 프로파일 지정 : @Configuration 클래스는 @Profile(프로파일 이름)로 지정할 수 있다.
    - 사용할 프로파일 지정 : 해당 프로파일을 활성 프로파일로 만들어준다. 
    ``` xml
    <!--서블릿 컨텍스트 파라미터로 활성 프로파일 지정-->
    <context-param>
        <param-name>spring.profiles.active</param-name>
        <param-value>spring-test</param-value>
    </context-param>
    ```

    ``` xml
        <!--profile 조건에 따라 정의된 빈을 사용하지 말지 결정-->
        <beans profile="spring-test">
            <jdbc:embedded-database id="dataSource" type="HSQL">
                <jdbc:script location="schema.sql"/>
            </jdbc:embedded-database>
        </beans>
    ```
#### 프로퍼티 종류

1. 환경변수
    - OS의 환경변수
    - 자바에서는 System.getEnv() 메소드로, 스프링에서는 systemEnvironment 빈으로 프로퍼티 맵을 가져올 수 있다.
2. 시스템 프로퍼티
    - JVM 레벨에 정의된 프로퍼티. JVM 관련 정보 등이 시스템 프로퍼티로 등록된다.
    - 자바에서는 System.getProperties(), 스프링에서는 systemProperties 빈으로 접근
3. JNDI
    - 하나의 애플리케이션에만 프로퍼티를 지정하고 싶을 때 사용
4. 서블릿 컨텍스트 파라미터
    - web.xml에 서블릿 컨텍스트 초기 파라미터를 프로퍼티로 사용할 수 있다.
5. 서블릿 컨픽 파라미터

#### 프로퍼티 소스 사용하기

1. Environment 사용하기

``` java

    @Autowired Environment env;
    private String adminEmail;

    @PostConstruct
    public void init(){
        this.adminEmail = env.getProperty("admin.email");
    }

```

2. PropertySourceConfigurerPlaceholder와 \<context:property-placeholder> 사용하기
- PropertySourceConfigurerPlaceholder 빈을 등록해야 @Value를 사용할 수 있다.(static 메소드로 등록해야한다.)
- PropertySourceConfigurerPlaceholder : 환경 오브젝트에 통합된 프로퍼티 소스에서 값을 가져와 ${} 치환자의 값을 바꿔준다. (프로퍼티 파일 지정 x)

``` java
    @Value("${admin.email}") private String adminEmail;
```
3. @PropertySource로 프로퍼티 파일을 프로퍼티 소스로 등록하기

``` java
    @Congifuration
    @PropertySource("database.properties")
    public class AppConfig
```
