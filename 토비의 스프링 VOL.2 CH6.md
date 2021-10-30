#### 테스트 프레임워크

- 스프링은 테스트가 사용하는 컨텍스트를 캐싱해서 여러 테스트에서 하나의 컨텍스트를 공유할 수 있는 방법을 제공한다.

1. 테스트에 테스트 컨텍스트 프레임워크 적용하기

```java
@RunWith(SpringJUnit4ClassRunner.class) // 스프링이 제공하는 JunitRunner
@ContextConfiguration("/applicationContext") // 테스트시 사용할 컨텍스트 지정
```
- 다른 테스트 클래스에서도 동일한 컨텍스트를 사용하면 컨텍스트를 공유한다. 
- 하나의 테스트에서 여러 컨텍스트를 사용하면 구성이 동일한 테스트끼리 컨텍스트를 공유한다.
- 컨텍스트 파일 정보는 상속된다. 그래서 서브 클래스의 컨텍스트는 슈퍼클래스에서 정의한 것까지 포함된다. 이를 무시하려면 @ContextConfiguration의 inheritLocations를 false로 설정해야한다.
- 컨텍스트를 공유할 경우 변경할 때 조심해서 다뤄야한다.
- 컨텍스트의 빈 오브젝트를 조정하고 수정해야하는 테스트의 경우 @DirtiesContext를 붙여서 테스트 후 컨텍스트를 제거하고 새로운 컨텍스트를 사용하도록 하는 것이 좋다.

2. 트랜잭션 지원 테스트

- 트랜잭션 매니저 
    - 스프링은 트랜젝션 매니저로 트랜젝션을 제어하기 때문에 트랜잭션 매니저 빈을 주입받아 테스트에서 사용하면 된다.
    - TransactionTemplate과 TransactionCallback을 이용해 트랜잭션 경계를 설정하고 테스트를 진행한다.
- @Transactional 어노테이션을 붙여준다. 
    - 강제롤백 옵션이 설정된다.
    - @Before, @After도 트랜잭션 안에서 실행된다.
    - 트랜잭션 전 후에 실행될 부분은 @BeforeTransaction, @AfterTransaction이 선언된 메서드에서 작업한다.
    - ORM에서 테스트를 할 경우 INSERT, UPDATE 후 SessionFactory.getCurrentSession().flush()로 바로 DB에 반영하게 해야한다.

```java
    
    new TransactionTemplate(transactionManager).execute(
        new TransactionCallback<Object>(){
            public Object doTransaction(TransactionStatus status){
                status.setRollbackOnly(); // 테스트 끝나면 롤백
                //dao 테스트
            }
        }
    )

    @Test
    @Transactional
    @Rollback(false) // 강제 롤백 x
    public void test(){

```

3. 컨텍스트 테스트 프레임 워크

- @ContextConfiguration(classes=컨피그클래스.class) 형식으로 @Configuration 클래스를 컨텍스트 정보로 사용할 수 있다.
- @ActiveProfiles 어노테이션을 클래스 레벨에 추가해 테스트용 활성 프로파일을 지정할 수 있다.

#### Reference
* * *
책 : 토비의 스프링 3.1 Vol.2 스프링의 기술과 선택 (저 이일민)
