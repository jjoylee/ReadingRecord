### 트랜잭션 

-   더 이상 나눌 수 없는 작업단위
-   중간에 예외가 발생해서 작업을 완료할 수 없다면 아예 작업이 시작되지 않은 것처럼 초기 상태로 돌려놔야한다
-   트랜잭션 롤백 : 한 트랜잭션에서 모든 sql문이 성공적으로 수행되기 전에 문제가 발생할 경우 앞에서 처리한 sql 작업도 취소해야한다. 
-   트랜잭션 커밋 : 한 트랜잭션에서 모든 sql 작업이 다 성공적으로 마무리외었을 때 db에 알려줘서 작업 확정

``` java

// 트랜잭션 경계설정
Connection c = dataSource.getConnection();
c.setAutoCommit(false); // 트랜잭션 시작
try{
    // 쿼리
    c.commit(); // 트랜잭션 시작
} catch(Exception e){
    c.rollback(); // 트랜잭션 롤백
}

```

데이터 엑세스 코드를 DAO로 분리해놓았을 경우 DAO 메소드를 호출할 때마다 하나의 새로운 트랜잭션이 만들어지는 구조가 된다.  
위의 예제처럼 서비스에서 Connection을 파라미터로 넘겨서 트랜젝션을 적용하려면 다음과 같은 문제가 발생한다.

1. JdbcTemplate을 더 이상 사용하지 못한다.
2. Dao 메소드와 Service 메소드에 Connection 파라미터를 추가해야한다.
3. Dao는 데이터 엑세스 기술에 독립적일 수 없다. Service와 의존하는 관계가 된다.  

### 트랜잭션 동기화 

트랜잭션을 시작하기 위해 만든 Connection 오브젝트를 특별한 저장소에 보관해두고 이후에 Dao 메소드에서는 저장된 Connection을 가져다 사용하게 하는 것.  
트랜잭션 동기화 저장소에 등록된 DB 커넥션이나 트랜잭션이 없는 경우에는 JdbcTemplate이 직접 DB 커넥션을 만들고 트랜잭션을 시작해서 JDBC 작업을 진행한다.  
트랜잭션 동기화를 시작해놓았다면 그때부터 실행되는 JdbcTemplate의 메소드에서는 트랜잭션 동기화 저장소에 들어 있는 DB 커넥션을 가져와서 사용한다.  

장점 : JdbcTemplate는 작업에서 동기화시킨 DB 커넥션을 사용하게 된다. (Connection을 파라미터로 넘길 필요 없음)  
문제점 : 트랜잭션 코드가 Service에 들어가면 Dao에 간접적으로 의존하게 된다. 기술환경에 따라 트랜잭션 관리 코드를 수정해야한다.  
예 : 한개 이상의 DB로의 작업을 하나의 트랜잭션으로 만들기 위해서는 글로벌 트랜잭션 방식을 사용해야한다.(별도의 트랜잭션 관리자를 통해 트랜잭션 관리) > JTA 사용  

``` java 

TransactionSynchronizationManager.initSynchronization(); // 트랜잭션 동기화 작업 초기화
Connection c = DataSourceUtils.getConnection(dataSource); // DB Connection
c.setAutoCommit(false); // start transaction
try {
    // 코드
    c.commit(); // transaction commit
}
catch (Exception e){
    c.rollback(); // transaction rollback
    throw e;
} finally {
    DataSourceUtils.releaseConnection(c, dataSource);
    TransactionSynchronizationManager.unbindResource(this.dataSource);
    TransactionSynchronizationManager.clearSynchronization();
}
```

# 트랜잭션 처리 코드에 추상화를 도입하기

-   트랜잭션 추상화 기술 
-   PlatformTransactionManager : 스프링이 제공하는 트랜잭션 경게설정을 위한 추상 인터페이스
-   PlatformTransactionManager를 빈으로 DI 받아서 사용하면 트랜잭션 기술에서 완전히 독립적인 코드를 만들 수 있다. 

``` java
PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
try{
    // 코드
    transactionManager.commit(status);
} catch (Exception e){
    transactionManager.rollback(status);
    throw e;
} 
```

### 정리

1. UserService : 비즈니스 로직, UserDao 인터페이스와 DI로 연결됨, 낮은 결합도 (DAO의 인터페이스를 사용하고 DI를 적용해야한다.)
2. UserDao : 데이터 엑세스 로직, DB 연결을 생성하는 방법에 독립적(DataSource)
3. UserService는 PlatformTransactionManager 인터페이스를 사용해서 트랜잭션 기술에 독립되었다.

스프링DI를 이용해서 코드간 결합도를 낮춘다.  
단일 책임 원칙 : 하나의 모듈은 한 가지 책임을 가져야 한다.  
장점 : 변경이 필요할 때 수정 대상이 명확해진다. 수정할 대상이 적어지기 때문에 수정하면서 실수가 발생할 가능성도 낮아진다.   
예 : UserService는 UserDao의 구현 클래스가 바뀌어도 영향을 받지 않는다.  


* * *
책 : 토비의 스프링 3.1 Vol.1 스프링의 이해와 원리 (저 이일민)
