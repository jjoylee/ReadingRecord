
#### JPA

- 영속성 관리와 ORM을 위한 표준 기술
- ORM : 오브젝트를 가지고 정보를 다루면 RDB에 적절한 형태로 변환, RDB에 저장된 정보를 자바 오브젝트가 다루기 쉬운 형태로 변환.
- JPA 퍼시스턴트 컨텍스에 접근하고 엔티티 인스턴스를 관리하려면 EntityManager를 구현한 오브젝트가 필요하다.
- EntityManager는 애플리케이션 또는 컨테이너가 관리한다.

#### EntityManagerFactory 빈 등록하기

1. LocalEntityManagerFactoryBean을 사용해서 EntityManagerFactory를 빈으로 등록한다.
- PersistentProvider 자동 감지 기능을 이용해 프로바이더를 찾는다.
- persistence.xml에 담긴 퍼시스턴스 유닛의 정보를 활용해 EntityManagerFactory를 생성한다.
- 스프링의 DataSource를 사용할 수 없어서 실전에서는 잘 사용하지 않는다.

``` xml
    <bean id="emf" class="org.springframework.orm.jpa.LocalEntityManagerFactoryBean"></bean>
```

2. JavaEE5 서버가 제공하는 EntityManagerFactory사용
- JNDI를 통해 EntityManager와 EntityManagerFactory를 제공받는다.

``` xml
    <jee:jndi-lookup id="emf" jndi-name="persistence/myPersistenceUnit"/>
```

3. LocalContainerEntityManagerFactoryBean

- 스프링이 직접 제공하는 컨테이너 관리 EntityManager를 위한 EntityManagerFactory를 만들어준다.

``` xml
    <bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- EntityManager 방식에서는 컨테이너가 제공하는트랜재견 매니저가 반드시 필요하다.-->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="emf">
    </bean>
```

``` java
// 자바 오브젝트와 RDB 테이블 사이의 매핑과 변환을 위한 정보 정의
@Entity // Member 클래스를 JPA가 관리하는 엔티티 오브젝트를 지정한다.
public class Member{
    @Id
    int id;

    @Column(length=100) // 길이를 100자로 제한한다.
    String name;

    @Column(nullable=false)
    double point
    // ... 수정자, 접근자
}
```

- persistenceUnitName : 퍼시스턴스 유닛의 이름 지정
- persistenceXmlLocation : persistence.xml 파일을 만들어서 파일 위치를 지정해준다.
- jpaProperties, jpaPropertyMap : EntityManagerFactory를 위한 프로퍼티를 지정할 때 사용한다.

``` xml
<properties>
    <property name="eclipselink.weaving" value="false">
</properties>
```

- jpaVendorAdaptor : JAP 구현 벤더별로 프로퍼티나 설정을 다르게 지정할 수 있다. (EclipseLink, Hibernate 등)

``` xml
<property name="jpaVendorAdapter">
    <bean class="org.springframework.orm.jpa.vendor.EclipseLinkJpaVendorAdapter">
        <property name="showSql" value="true">
    </bean>
</property>
```
- loadtimeWeaver : JPA는 자바 코드로 만들어진 엔티티 클래스의 바이트코드를 직접 조작해서 확장된 기능을 추가한다. 런타임 시에 클래스를 로딩하면서 기능을 추가하는 것을 loadtime weaving이라 한다. loadtimeWeaver로 로드타임 위빙을 어떤 방식으로 할지 지정한다.

#### JPA DAO에서 EntityManager 사용하기

1. JpaTemplate 사용하기
- 템플릿의 메소드와 콜백 사용
- persist() : 저장 / find() : 조회

``` java
public class MemberTemplateDao{
    private JpaTemplate jpaTemplate;

    @Autowired
    public void init(EntityManagerFactory emf){
        jpaTemplate = new JpaTemplate(emf);
    }
}
```

2. 애플리케이션 코드가 관리하는 EntityManager를 이용한다.
- DI대신 @PersistenceUnit을 사용할 수 있다.

``` java
public class MemberTemplateDao{
    @Autowired EntityManagerFactory emf;

    public void addMember(Member member){
        EntityMember em = emf.createEntityManager();
        em.getTransaction().begin();
        em.persist(new Member(1,"Spring",2.7));
        em.getTransaction().commit();
    }
}
```

3. EntityManager와 @PersistenceContext 사용
- EntityManager 제공받아서 사용
- JPA @PersistenceContext 어노테이션을 사용해서 EntityManager를 주입받는다.
- @PersistenceContext로 주입받는 EntityManager는 현재 진행 중인 트랜잭션에 연결되는 퍼시스턴스 컨텍스트를 갖는 일종의 프록시다.
- 트랜잭션마다 다른 EntityManager 오브젝트 사용

``` java
public class MemberTemplateDao{
    @PersistenceContext EntityManager em;

    public void addMember(Member member){
        em.persist();
    }
}
```

4. EntityManager와 확장된 펀시스턴스 컨텍스트 사용하기
- @PersistenceContext(type=PersistenceContextType.EXTENDED) 사용
- 확장된 스코프를 갖는 EntityManager를 만듬. 상태유지 세션빈에 바인딩
- 상태를 가진 세션빈이나 스코프 빈에만 사용

#### JPA 예외변환 AOP

- AOP를 이용해서 JPA 예외를 스프링 예외로 전환해주는 부가기능 추가하기

1. DAO 클래스에 @Repository 애노테이션 부여하기 
    - AOP를 이용한 예외 기능 변환 기능을 부가할 빈으로 선정
2. PersistenceExceptionTranslationPostProcessor을 빈으로 등록한다.
    - AOP 어드바이스를 적용해주는 후처리기

#### 하이버네이트 

- 오픈소스 ORM 프레임워크
- SessionFactory : JPA의 EntityManagerFactory 같은 역할.
- SessionFactory를 빈으로 등록하기 위한 팩토리 빈이 필요하다.

##### 팩토리 빈 등록

1. LocalSessionFactoryBean : DataSource를 이용해서 트랜잭션 메니저와 연동할 수 있도록 설정된 SesionFactory를 만들어 주는 팩토리 빈(설정파일, 매핑파일 필요)
    - mappingLocations : 매핑파일 정보 등록
    - hibernateProperties : 하이버네이트 설정 값들을 지정할 수 있다.

``` xml
    <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="springbook/learningtest/spring/hibernate/hibernate.cfg.xml"/>
    </bean>
```
2. AnnotationSessionFactoryBean : 클래스에 애노ㅔ이션을 부여하고 이를 매핑정보로 사용한다.
    - annotatedClasses : 매핑 애노테이션에 부여된 클래스 목록을 지정할 수 있다.
    - packagesToScan : 자동스캔용 패키지를 지정해서 엔티티 클래스 등록(@Entity)

##### 트랜잭션 매니저

1. Hibernate TransactionManager

``` xml
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
```
2. JtaTranscationManager
- 여러 개의 DB에 대한 작업을 하나의 트랜잭션으로 묶을 때 사용

#####  Session을 사용하는 방법

1. HibernateTemplate
    - SessionFactory를 생성자로 제공해서 만든다. 
    - hibernate = new HibernateTemplate(sessionFactory)
    - execute()와 HibernateCallback 콜백 오브젝트를 사용해서 진행 중인 트랜잭션과 동기화된 Session을 사용할 수 있다.
2. SessionFactory.getCurrentSession()
    - 현재 트랜잭션에연결되어있는 Session을 돌려준다.
    - 하이버네이트 API를 사용하지만 트랜잭션 동기화 기능과 연동된다.
    - 예외처리를 위해서는 JPA와 동일하게 @Repository 와 PersistenceExceptionTranslationPostProcessor 빈이 필요하다.
