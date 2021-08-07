#### 데이터 엑세스 계층

- 데이터 엑세스 계층은 DAO 패턴으로 분리한다.
- DAO 패턴 : DTO 또는 도메인 오브젝트만을 사용하는 인터페이스를 통해 데이터 엑세스 기술을 외부에 노출하지 않도록 만드는 것
- DAO는 인터페이스르 이용해 접근하고 DI 되어야한다.
- DAO 내부에서 발생하는 예외는 모두 런타임 예외로 전환해야한다.

#### DataSource 종류

- 미리 정해진 개수만큼의 DB 커넥션을 풀에 준비해두고, 애플리케이션이 요청할 때마다 풀에서 꺼내 하나씩 할당해준다.(풀링기법)
- SimpleDriverDataSource : 매번 DB 커넥션을 새로 만든다. 풀 관리 X. 테스트용으로 사용
- SingleConnectionDataSource : 하나의 물리적인 db 커넥션만 계쏙 사용한다. 
- 서버가 제공하는 DB 풀을 사용해야하는 겅우 JNDI를 통해 서버의 DataSource에 접근해야한다.

#### JDBC

1. 사용
    - SimpleJdbcTemplate : JDBC의 모든 기능을 최대한 활용할 수 있는 유연성을 갖고 있다.
    - SimpleJdbcInsert, SimpleJdbcCall: 메타정보에서 컬럼 정보와 파라미터 정보를 가져와 삽입용 SQL과 프로시저 호출 작업에 사용
2. JDBC가 하는 작업
    - Connection 열기와 닫기 : Connection과 관련된 모든 작업은 스프링 JDBC가 필요한 시점에 진행
    - Statement 준비와 닫기 : Statement 또는 PreparedStatement를 생성하고 필요한 준비작업을 한다.
    - Statement 실행
    - ResultSet 루프 : ResultSet에 담긴 쿼리 결과가 한 건 이상이면, ResultSet의 루프를 만들어 반복
    - 예외처리와 반환 : SQLException을 DataAccessException 타입으로 변환
    - 트랜잭션 처리 : 트랜잭션을 시작한 후에 스프링 JDBC 작업을 요청하면 트랜잭션에 참여한다.

#### SimpleJdbcTemplate

- 멀티스레드 환경에서도 안전하게 공유해서 쓸 수 있다.
- 작업을 요청할 때는 문자열로 된 SQL을 제공해줘야한다.(위치 치환자 ?, 이름 치환자 사용)

``` java
    public class MemberDao{
        SimpleJdbcTemplate simpleJdbcTemplate;

        public void setDataSource(DataSource dataSource){
            simpleJdbcTemplate = new SimpleJdbcTemplate(dataSource);
        }
    }
```

- 파라미터
    - Insert Into Member(id) VALUES(:id) : Map 또는 MapSqlParameterSource 사용 > map.put("id",1)
    - BeanPropertySqlParameterSource : 오브젝트의 프로퍼티 이름과 SQL의 이름 치환자를 매핑해서 파라미터 값을 넣는다.

``` java 
    Member member = new Member(1,"Spring",3.5);
    BeanPropertySqlParameterSource params = BeanPropertySqlParameterSource(member);
    simpleJdbcTemplate.update("INSERT INTO MEMBER(ID,NAME,POINT,args) VALUES(:id,:name,:point)", params);
```
- SQL 실행 메소드
    - INSERT, UPDATE, DELETE : update() 메소드 사용
    - queryForInt(String sql, SQL 파라미터) : 하나의 int 타입 값을 조회할 때 사용한다. 하나의 로우 하나의 컬럼 필요.
    - queryForObject(String sql, requiredType, SQL 파라미터) : 하나의 값을 가져온다. 결과 타입을 지정할 수 있다.
    - queryForObject(String sql, RowMapper, SQL 파라미터)
    - query(String sql, RowMapper, SQL 파라미터) : 여러 개의 컬럼을 가진 로우를 RowMapper 콜백을 이용해 오브젝트에 매핑

- SQL 배치 메소드
    - update()로 실행하는 SQL들을 배치모드로 실행
    - 많은 SQL을 실행하는 경우 배치 방식 사용
    - 동일한 SQL을 파라미터만 바꿔가면서 실행
    - batchUpdate(sql, batchArgs) : 배열로 파라미터를 제공

``` java
    dao.simpleJdbcTemplate.batchUpdate("update member set name=:name where id = :id", new SqlParamterSource[]{
        new MapSqlParameterSource().addValue("id",1).addValue("name","Spring3"),
        new BeanPropertySqlParameterSource(new Member(2,"Book3"));
    })
```

#### SimpleJdbcInsert

- 초기화
    - withTableName(테이블이름) : 지정된 테이블 이름을 가지고 테이블의 메타정보를 읽는다.
    - usingColums(컬럼 이름들) : 일부 컬럼만 사용해서 insert문 작성
    - usingGeneratedKeyColums(컬럼 이름들) : DB에 의해 자동으로 생성되는 키 컬럼을 지정
- 실행
    - execute()
    - executeAndReturnKey(sql 파라미터) : 쿼리 실행하고 자동 생성된 키 값을 돌려준다.

``` java
    SimpleJdbcInsert registerInsert = new SimpleJdbcInsert(dataSource)
                                    .withTableName("register")
                                    .usingGeneratedKeyColumns("id");
    int key = registerInsert.executeAndReturnKey(new MapSqlParameterSource("name","Spring")).intValue();
```

#### SimpleJdbcCall

- 저장 프로시저 또는 저장 function을 호출할 때 사용
- executeFunction(리턴 타입, SQL 파라미터) : 저장 함수 실행
- executeObject(리턴 타입, SQL 파라미터) : 저장 프로시저 실행

``` java
    SimpleJdbcCall call = new SimpleJdbcCall(dataSource).withFunctionName("find_name");
    String ret = call.executeFunction(String.class, id);
```

#### iBatis

- 자바 오브젝트와 SQL 문 사이의 자동매핑 기능을 지원하는 ORM 프레임워크.
- SQL을 자바 코드에서 분리해서 별도의 XML 파일 안에 작성하고 관리할 수 있다.
- iBatis를 이용하려면 SqlMapClientFactoryBean을 이용해서 SqlMapClient를 빈으로 등록해줘야한다.
- SqlMapClientTemplate 사용

1. 설정파일과 매핑파일 필요

- 설정파일
```xml
    <!--매핑정보를 담은 파일의 클래스 패스 지정-->
    <sqlMapConfig>
        <sqlMap resource="springbook/learningtest/spring/ibatis/Member.xml"/>
    </sqlMapConfig>
```

- 매핑파일 
```xml
    <sqlMap namespace="Member">
        <!-- 파라미터와 결과 매핑에 사용할 클래스 별칭 등록-->
        <typeAlias alias="Member" type="springbook.learningtest.spring.jdbc.Member"/>
        <select id="findMemberById" parameterClass="int" resultClass="Member">
            select * from member where id =#id#
        </select>
    </sqlMap>
```

2. DAO
- insert(), update(), delete() 메서드 사용
- 단일 로우 조회 : queryForObject()
- 다중 로우 조회 : queryForList(), queryForMap(), queryWithRowHandler()

``` java
    public void insert(Member member){
        sqlMapClientTemplate.insert("insertMember",member);        
    }

    public void deleteAll(){
        sqlMapClientTemplate.delete("deleteMemberAll");
    }
```

#### Reference
* * *
책 : 토비의 스프링 3.1 Vol.2 스프링의 기술과 선택 (저 이일민)
