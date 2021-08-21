#### 스프링 트랜잭션 서비스

- 트랜잭션 추상화 : 트랜잭션 서비스의 종류나 환경이 바뀌더라도 트랜잭션 코드는 그대로 유지할 수 있다.
- 트랜잭션 동기화 : 트랜잭션을 일정 범위 안에서 유지해주고, 어디서든 자유롭게 접근할 수 있게 해준다.

#### PlatformTransactionManager 

- 트랜잭션 추상화 핵심 인터페이스
- 트랜잭션 경계를 지정하는데 사용한다. (getTransaction())
- 시작과 종료를 트랜잭션 전파 기법을 이용해 조합하고 확장할 수 있다.
- TransactionStatus : 참여하고 있는 트랜잭션의 ID와 구분정보를 담고 있다.

구현 클래스

1. DataSourceTransactionManager
- 위의 트랜잭션 매니저를 사용하려면 트랜잭션을 적용할 DataSource를 스프링의 빈으로 등록해야한다.
- DataSourceUtils.getConnection(DataSource)로 트랜잭션 매니저가 관리하는 Connection을 가져올 수 있다.
- 내부에서 Connection과 트랜잭션 작업을 알아서 처리해주는 템플릿을 사용하는 방법이 제일 좋다.

``` xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

2. JpaTransactionManager
- JPA를 사용하는 DAO에 사용한다.
- LocalContainerEntityManagerFactoryBean을 프로퍼티로 등록해줘야한다.

3. HibernateTransactionManager
- 하이버네이트 DAO에서 사용한다.
- SessionFactory 타입의 빈을 프로퍼티로 넣어주어야 한다.

4. JmsTransactionManager, CciTransactionManager

5. JtaTransactionManager
- 하나 이상의 DB 또는 트랜잭션 리소스가 참여하는 글로벌 트랜잭션을 적용할때 JTA 사용
- JtaTransactionManager 빈을 등록해서 사용한다.

#### 트랜잭션 경계설정 전략

1. 코드에 의한 프로그램적인 방법
- PlatformTransactionManager 메소드 또는 TransactionTemplate을 이용한다.
- 주로 선언적 트랜잭션 방식을 사용한다.
- PlatformTransactionManager의 getTransaction()으로 트랜잭션 상태를 확인할 수 있다.

2. AOP를 이용한 선언적인 방법 (aop 스키마 태그, tx 스키파 태그 사용)
- 코드에 영향을 주지 않고 일괄적으로 트랜잭션을 적용하고 변경할 수 있다는 장점이 있다.( 메소드명으로 패턴지정 등 )
- 데코레이터 패턴을 적용한 트랜잭션 프록시 빈을 사용한다.
- 포인트컷은 기본적으로 인터페이스에 적용된다.

```xml
    <!-- 어드바이스-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attribute>
            <tx:method name="*"/>
        </tx:attribute>
    </tx:advice>

    <!-- 어드바이저 -->
    <aop:config>
        <!--포인트 컷-->
        <aop:pointcut id="txPointcut" expression="execution(* *..MemberDao.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>
```

3. AOP를 이용한 선언적인 방법 (@Transactionl 애노테이션을 이용)
- <tx:annotation-driven \/> (@EnableTransactionManager)
- (트앤잭션 대상으로 지정) 
- @Transactional 우선순위 : 클래스의 메소드 > 클래스 > 인터페이스 메소드 > 인터페이스
- 세밀하게 트랜젝션 속성을 부여하기에 좋다.

4. 인터페이스를 구현하지 않은 클래스에 트랜잭션 적용하기(레거시 코드 등)
- 클래스 프록시 모드 사용하기
- 클래스 프록시는 상속이 불가능한 final 클래스에는 적용할 수 없다.
- 모든 public 메소드에 트랜잭션이 적용된다.

```xml
    <!-- aop/tx-->
    <aop:config proxy-target-class="true">
        <aop:pointcut id="txPointcut" expression="execution(* *..MemberDao.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>

    <!--@Transactional -->
    <tx:annotation-driven proxy-target-class="true"/> 
```

5. 추가설명 : Proxy AOP
- 타킷 오브젝트의 자기 호출에는 AOP가 적용되지 않는다.
- 해결방법 
    1. AopContext.currentProxy() : 현재 진행 중인 프록시를 가져올 수 있다.(사용 권장 x)
    2. 프록시 AOP 대신 AspectJ AOP를 사용 : 프록시 대신 클래스 바이트 코드를 직접 변경하기 때문에 자기 호출에도 AOP 적용 

#### 트랜잭션 설정

1. 트랜잭션 전파 : propagation
- 트랜잭션을 시작하거나 기존 트랜잭션에 참여하는 방법을 결정한다.
- REQUIRED : 미리 시작된 트랜잭션이 있으면 참여하고 없으면 새로 시작한다.
- SUPPORTS : 이미 시작된 트랜잭션이 있으면 참여하고 없으면 트랜잭션 없이 진행한다.
- MANDATORY : 이미 시작된 트랜잭션이 있으면 참여하고 없으면 예외를 발생시킨다.
- REQUIRES_NEW : 항상 새로운 트랜잭션을 시작한다.
- NOT_SUPPORTED : 트랜잭션을 사용하지 않게 한다.
- NEVER : 트랜잭션을 사용하지 않게 강제한다. 진행 중인 트랜잭션이 있으면 예외 발생.
- NESTED : 이미 진행중인 트랜잭션이 있으면 중첩 트랜잭션을 시작한다. (트랜잭션 안의 트랜잭션)

2. 트랜잭션 격리수준 : isolation
- 동시에 여러 트랜잭션이 진행될 때 작업결과를 다른 트랜잭션에 어떻게 노출할지 결정
- DEFAULT : DB의 디폴트 설정(READ_COMMITTED)
- READ_UNCOMMITTED : 하나의 트랜잭션이 커밋되기 전에 변화가 다른 트랜잭션에 그대로 노출(데이터 일관성 낮음)
- READ_COMMITTED : 가장 많이 사용. 다른 트랜잭션이 커밋하지 않은 정보는 읽을 수 없다.
- REPEATABLE_READ : 하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정 못하게 막는다.
- SERIALIZABLE : 트랜잭션을 순차적으로 진행. 성능이 가장 떨어진다.

3. 트랜잭션 제한시간 : timeout 
- 트랜잭션에 제한시간 지정

4. 읽기전용 트랜잭션 : read-only, readOnly
- 트랜잭션을 읽기 전용으로 설정
- INSERT, UPDATE, DELETE 작업 진행하면 예외 발생

5. 트랜잭션 롤백 예외 : rollback-for, rollbackFor, rollbackForClassName
- 원래 체크예외가 발생해도 트랜잭션은 작업한 내용을 커밋한다.
- 위의 엘리먼트, 어트리뷰트로 예외를 지정하면 체크예외도 롤백 대상으로 삼을 수 있다.
- rollback-for, rollbackForClassName ="예외이름"
- rollbackForClassName=예외클래스
- no-rollback-for, noRolllbackFor, noRollbackForClassNmae은 반대로 런타임 예외가 발생해도 커밋 대상으로 지정할 수 있다.

#### 데이터 엑세스 기술

- 하나의 트랜잭션 매니저가 여러 개의 데이터 액세스 기술의 트랜잭션 기능을 지원해 줄 수 있다.
- DataSourceTransactionManager : JDBC와 iBatis 두가지 기술을 함께 사용할 수 있다.
- JpaTransactionManager : EntityManagerFactory가 사용하는 DataSource로 트랜잭션 동기화 (JDBC, iBatis, JPA)
- HibernateTransactionManager : SessionFactory가 사용하는 DataSource로 트랜잭션 동기화 (JDBC, iBatis, 하이버네이트)
- ORM과 비 ORM DAO를 함꼐 사용할 때 주의할 점
    - JPA나 하이버네이트 같은 경우 쿼리를 바로 실행시키지 않고 실제 DB로 등록하는 것을 지연시키는 기법을 사용한다.(캐싱)
    - 바로 쿼리하는 JDBC와 함께 사용해야할 때 예상하는 결과와 다르게 나올 수 있다.
    - 저장이나 수정 작업 후 캐시 내용을 강제로 DB로 보내주는 flush() 메소드를 사용해서 위와 같은 문제를 해결할 수 있다. 
    - flush() : 현재 캐시의 내용을 즉시 DB에 반영한다.
