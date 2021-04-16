### 예외 처리할 때 주의할 것

1. Exception을 catch하면 꼭 처리를 해야한다. 넘어가면 안된다.
2. System.println으로 콘솔에 출력하면 메세지를 놓치기 쉽다. 
3. 모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 분명하게 통보돼야한다.
4. 모든 예외를throws Exception으로 처리하면 의미있는 정보를 얻을 수 없다.

### 예외의 종류와 특징

1. Error(java.lang.Error) 
    * 시스템에 비정상적인 상황이 발생했을 때
    * 주로 자바 VM에서 발생. 애플리케이션 코드에서 잡으려고 해도 대응 방법이 없다.
    * ex) OutOfMemoryError, ThreadDeath

2. Exception과 체크예외(java.lang.Exception)
    * 개발자가 만든 애플리케이션 코드의 작업 중에 예외상황이 발생했을 경우에 사용
    * Check 예외 : Exception 클래스의 서브클래스이면서 RuntimeException 클래스를 상속하지 않은 것    
      * 예외처리하지 않으면 컴파일 에러 발생 (일반적인 예외)
      * ex) IOException, SQLException
    * UnCheck 에외: Exception 클래스의 서브클래스이면서 RuntimeException 클래스를 상속한 것
      * 명시적인 예외처리를 강제하지 않는다.
      * 프로그램의 오류가 있을 떄 발생 ( 복구할 가능성이 낮다.)
      * ex) NullPointerException, IllegalArgumentException

### 예외 처리 방법

1. 예외 복구
    * 예외상황 파악 -> 문제 해결 -> 정상상태
    * 체크 예외는 예외를 복구할 가능성이 있는 경우에 사용

2. 예외처리 회피
    * 예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버린다.

3. 예외 전환
    * 예외를 적당한 예외로 전환해서 메소드 밖으로 던진다.
    * 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해 ex) DuplicateUserIdException
    * initCause로 처음에 발생한 예외를 넣을 수 있다.
    * 복구하지 못할 예외는 런타임 예외로 포장해서 던진다. ( 불필요한 catch/throws 제거 )

### 예외 처리 전략

복구할 수 있는 예외가 아니라면 일단 언체크 예외로 만든다.

``` java
public void add() throws DuplicateUserIdException {
    try{
        // jdbc 코드
    } catch (SQLExcpetion e){
        if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY){
            // 예외전환
            // add 사용하는 오브젝트에서 중복 아이디 오류 처리 가능
            throw new DuplicateUserIdException(e);
        } 
        else throw new RuntimeException(e); // 예외 포장
    }
}
```

### 애플리케이션 예외 (애플리케이션 자체의 로직에 의해 의도적으로 발생시키는 예외) 처리 방법
- 리턴 값으로 결과 상태를 나타낸다. (ex return -1 = 잔고부족) : 리턴 값 관리 필요
- 에외 상황에서는 비즈니스적인 의미를 띤 체크예외를 던지도록 만드는 것 

### JdbcTemplate의 예외 처리 
- 대부분의 SQLException은 복구 불가능하다.
- 언체크/런타임 예외로 전환한다.(DataAccessException)
- throws SQLException 제거

### JDBC를 사용해도 DB를 자유롭게 바꾸기 어려운 이유
1. 비표준 SQL : 표준을 따르지 않는 비표준 문법과 기능이 있다.
    * 해결 : DAO를 DB별로 만들어 사용하거나 SQL을 외부에서 독립시켜 바꿔 쓸 수 있게 한다.
2. DB마다 에러의 종류와 원인이 다르다. (에러코드 다름)
    * 해결 : DB별 에러 코드를 분류해서 예외 클래스와 매핑해놓은 에러 코드 매핑정보 테이블을 만들고 이를 사용한다.


```java
// JPA
public void add(User user) throws PersistentExcpetion  
// Hibernate
public void add(User user) throws HibernateExcpetion
// JDO
public void add(User user) throws JdoException 
```

DAO로 메소드의 구현을 추상화시켜도 구현 기술마다 던지는 예외가 다르기 때문에 메소드의 선언이 달라진다.   
스프링은 데이터 엑세스 기술을 사용할 떄 발생하는 예외들을 추상화해서 DataAccessException 계층구조 안에 정리했다.
