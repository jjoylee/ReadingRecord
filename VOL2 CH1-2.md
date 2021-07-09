#### 빈 메타정보 작성 

- 빈 설정 메타정보 > BeanDefinition  
- 주로 외부 리소스로 빈 메타정보를 작성하고 이를 리더나 변환기를 통해 애플리케이션 컨텍스트가 사용할 수 있는 정보로 변환해주는 방법을 사용한다.

#### 빈 등록 방법

1. XML <bean> 사용하기
2. XML 네임스페이스와 전용태그 사용하기
    - 스프링 빈 = 애플리케이션 핵심 코드 + 서비스 또는 컨테이너 설정을 위한 빈
    - 기술적인 설정과 기반 서비스를 빈으로 등록할 떄 네임스페이스와 태그를 가진 설정 방법을 사용 <aop:pointcut>
    - 전용태그 하나로 동시에 여러개의 빈을 만들 수 있다. <context:annotation-config/>
    - 개발자가 커스텀 태그를 만들어서 적용할 수 있다.
3. 자동 인식(빈 스캐닝)을 통한 빈 등록
    - 빈으로 사용될 클래스에 특별한 애노테이션을 부여해주면 자동으로 빈으로 등록
    - @Component(스테레오타입 애노테이션) 또는 @Component를 매타 애노테이션으로 가진 애노테이션이 부여된 클래스 스캔
    - 기본적으로 클래스 이름을 빈 아이디로 사용한다. 
    - 이름지정 : @Component(빈 이름)
    
#### 빈 스캐너 등록 방법

1. XML 사용 <context:component-scan>
2. 빈 스캐너를 내장한 애플리케이션 컨텍스트 사용 (AnnotationConfigApplicationContext)
3. 자바 코드에 의한 빈 등록 
    - @Configuration 클래스의 @Bean 메소드를 통해 빈 정의
    - @Configuration 클래스도 빈으로 등록된다.
    - @Configuration 클래스가 아닌데 @Bean 메서드를 이용해 빈 설정정보를 넣으면 @Bean 메소드를 직접 호출하면 매번 다른 오브젝트를 받게 된다. 
    - 장점
        - 컴파일러나 IDE를 이용해서 타입을 검증할 수 있다.
        - IDE 기능을 사용할 수 있다.
        - XML 보다 이해하기 쉽다
        - 복잡한 빈 설정이나 초기화 작업을 쉽게 할 수 있다.

#### 빈 등록 메타정보 구성 전략

1. XML 단독사용 : 모든 빈을 명시적으로 XML에 등록. 모든 빈을 XML에서 확인할 수 있다는 장점이 있지만 빈 개수가 많아지면 파일을 관리하기 어려워진다.
2. XML과 빈 스캐닝의 혼용 : 핵심 로직을 담고 있는 빈 클래스는 자동인식 방식으로 등록한다. 기술 서비스, 기반서비스, 컨테이너 빈 등은 XML을 사용한다.빈 중복등록을 주의해야한다.
3. XML 없이 빈 스캐닝 단독 사용 : 빈의 설정정보를 타입에 안정한 방식으로 작성할 수 있다. 하지만 XML 전용 태그를 사용할 수 없다.

#### 빈 의존관계 설정 방법

DI할 빈의 아이디를 직접 지정
타입 비교를 통해 타입의 빈을 DI 후보로 삼는다.(자동 와이어링)

1. XML 수정자 주입 (property) 
- ref : 빈이름을 이용해 주입할 빈을 찾는다.
- value : 단순 값 또는 빈이 아닌 오브젝트 주입

2. XML 생성자 주입 (constructor-arg)
- 순서나 타입을 명시히하는 방법 필요

``` xml
    <bean id="hello" class="springbook.learningtest.spring.ioc.bean.Hello">
        <constructor-arg index="0" value="Spring"/>
        <constructor-arg index="1" ref="printer"/>
    </bean>   
```

3. XML 자동와이어링
- default-autowire로 디폴트 자동 와이어링 옵션을 설정할 수 있다. <beans default-autowire="byName">
- byName : 프로퍼티 이름과 동일한 빈을 찾아서 자동으로 등록
- byType : 프로퍼티의 타입과 빈의 타입을 비교해서 자동으로 연결
- constructor : 생성자에 자동와이어링 적용
- 빈 사이의 의존관계를 알기 힘들고, 하나의 빈에 대해 한가지 방식밖에 지정할 수 없다는 단점이 있다.

``` xml
    <!-- printer는 자동으로 추가 -->
    <bean id="hello" class="springbook.learningtest.spring.ioc.bean.Hello" autowire="byName">
        <property name="name" value="Spring"/>
    </bean>   
```

4. XML 네임스페이스와 전용 태그
- 전용 태그에 의해 만들어지는 빈을 다른 빈이 참조할 경우에는 id 애트리뷰트를 사용해 아이디를 지정한다.
- -ref로 끝나는 애트리뷰트를 이용해 DI 할 빈을 지정하는 관례가 있다.

5. @Resource 애노테이션
- 주입할 빈을 아이디로 지정하는 방법
- 수정자 메소드, 필드 주입
- 메소드 이름으로 부터 프로퍼티 이름을 끌어낼 수 있어서 프로퍼티 이름을 지정하지 않아도 괜찮다.
- 참조할 빈을 찾을 수 없다면 예외가 발생한다.
- 애노테이션으로 DI하기 위한 방법
    - XML <context:anntation-config/> 사용 : 애노테이션 의존관계를 읽어서 메타정보를 추가해주는 빈 후처리기 등록
    - XML <context:component-scan/> 사용 : 빈 스캐닝을 통한 빈 등록 방법 지정, <context:anntation-config/>로 만들어지는 빈을 함께 등록해준다.
    - AnnotationConfigApplicationContext, AnnotationConfigWebApplicationContext 사용
``` java 
    @Resource
    public void setPrinter(Printer printer){
        this.printer = printer;
    }
```

6. @Autowired/@Inject 애노테이션
- 기본적으로 타입에 의한 자동와이어링 방식
- @Autowired는 스프링 전용 애노테이션이다.
- @Inject는 @Autowired와 비슷하나, required를 설정할 수 없다.(DI를 선택적으로 가능하게 설정 불간)
- 타입의 의한 자동와이어링 방식을 생성자, 필드, 수정자 매소드, 일반 메소드에 적용 가능
- 생성자 주입을 사용할 때, 단 하나의 생성자에만 사용할 수 있다는 제한이 있다.
- 일반 메소드를 사용하는 DI 방법 : 파라미터를 가진 메소드에 @Autowired 적용. XML을 통해서는 의존관계를 설정할 수 없다.
- 이름을 이용해 빈을 지정할 때는 @Resource, 타입과 한정자를 활용하고 싶을때는 @Autowired를 사용한다.
- 동일한 타입의 빈이 하나 이상 존재할 때 처리하기
    - @Autowired 대상이 되는 필드나 프로퍼티, 메소드의 파라미터를 컬랙션이나 배열로 선언하면 모든 빈을 DI 받을 수 있다.( 빈 자체가 컬렉션일 경우 @Resource 사용)
    - @Qualifier로 빈 이름과는 별도로 추가적인 메타정보 지정해서 의미부여

``` java
    @Autowired 
    @Qualifier("mainDB")
    DataSource dataSource;

    @Component
    @Qualifier("mainDB")
    public class OracleDataSource{
        
    }
```

7. 자바 코드에 의한 의존관계 설정
- @Autowired, @Resource 사용
- @Bean 메소드 호출
- @Bean과 메소드 자동와이어링 : 의존 정보를 생성할 때 @Bean이 붙은 메소드를 직접 호출하는 대신 빈의 레퍼런스를 파라미터로 주입받는 방식을 사용하는 것

``` java
    //메소드 자동와이어링
    @Configuration
    public class Config{
        // printer 파라미터에는 Printer 타입의 빈이 자동으로 주입
        @Bean public Hello hello(Printer printer){
            Hello hello = new Hello();
            hello.setPrinter(printer);
            return hello;
        }

        @Bean public Printer printer(){
            return new Printer();
        }
    }
```
