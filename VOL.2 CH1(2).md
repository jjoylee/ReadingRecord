#### 메타정보 값 설정 방법

1. \<property\> 와 전용태그 
    - ref 애트리뷰트로 빈 아이디 지정
    - value 애트리뷰트로 런타임 시에 주입할 값 지정

2. @Value
    - 값이나 오브젝트를 외부에서 주입해야 하는 용도 : 환경에 따라 매번 달라질 수 있는 값, 테스트나 특별한 이벤트 때 초기값 대신 다른 값을 지정하고 싶을 때
    - @Value 애노테이션을 이용해서 프로퍼티 값을 지정할 수 있다. (외부로부터 주입을 받아 초기화가 필요하다)
    - 테스트 코드처럼 컨테이너 밖에서 사용되면 @Value 애노테이션은 무시된다.
    - 주로 외부 리소스나 환경정보에 담긴 값을 사용하도록 지정할 때 사용한다.
    
``` java
    public class Hello {
        private String name;

        @Value("${database.username}")
        private String name;

        @Value("Everyone")
        public void setName(String name){
            this.name = name;
        }
    }
```

3. 타입 변환기
    - 스프링 컨테이너는 PropertyEditor를 구현한 타입 변환기로 문자열로 된 값을 프로퍼티 타입으로 변환해준다.(XML, @Value에서 사용)
    - 스프링이 제공하는 ConversionService를 사용해서 타입 변환기를 작성할 수 있다.
    - util 스키마 전용 태그를 활용해서 컬렉션을 별도의 빈으로 만들 수 있다.
    - null 값은 \<null/\> 태그를 사용한다. 

``` xml
    <!-- 배열 -->
    <property name="array" value="1,2,3"/>

    <!-- List, Set : <list> -> <set>  -->
    <property name="list">
        <list>
            <value>Spring</value>
            <value>IoC</value>
        </list>
    </property>

    <!-- Map -->
    <property name="map">
        <map>
            <entry key ="Kim" value="30"/>
            <entry key ="Lee" value="20"/>
        </map>
    </property>

    <!-- Properties -->
    <property name="properties">
        <props>
            <prop key="username">Spring</prop>
        </props>
    </property>

    <property name="null"><null/></property>
```

4. 프로퍼티 파일을 이용해서 값 설정하기
    - 환경에 따라 자주 변경될 수 있는 내용은 프로퍼티 파일로 분리한다.
    - 소스코드의 수정 없이 @Value를 통해 프로퍼티에 주입되는 값을 변경할 수 있다.
    - 프로퍼티 치환자 이용하기 : 치환자의 값이 변경되지 않아도 예외가 발생하지 않는다.
    - SpEL 이용하기 : 다른 빈 오브젝트에 직접 접근할 수 있는 표현식을 이용해 프로퍼티 값을 능동적으로 가져오는 방법

    ```xml
       <!--프로퍼티 치환자-->
        <bean id="dataSource" class="...SimpleDriverDataSource">
            <property name="driverClass" value="${db.driverclass}"/>
        </bean>

        <!-- 등록되는 PropertyPlaceHolderConfigurer 빈이 ${} 값을 프로퍼티 파일의 내용으로 바꿔준다. -->
        <context:property-placeholder location="classpath:database.properties"/>

        <!--SpEL-->
        <bean id="hello" ...>
            <property name="name" value="Spring"/>
        </bean>
        <bean id="names">
            <property name="helloname" value="#{hello.name}"/>
        </bean>
    ```
5. 컨테이너가 자동으로 등록하는 빈
- ApplicationContext, BeanFactory : 스프링에서는 컨테이너 자신을 빈으로 등록해두고 필요하면 일반 빈에서 DI 받아서 사용할 수 있다.
- ResourceLoader : 코드를 통해 서블리 컨텍스트의 리소스를 읽어오고 싶을떄 컨테이너를 ResourceLoader 타입으로 DI 받아서 활용할 수 있다.
- ApplicationEventPublisher : 빈 사이에 이벤트 발생
- systemProperties : System.getProperties()가 돌려주는 오브젝트를 읽기전용으로 접근할 수 있게 만든 빈 오브젝트
- systemEnvironment : System.getenv()에서 제공하는 환경변수가 담긴 Map 오브젝트
