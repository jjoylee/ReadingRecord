### ApplicationContext 구현 클래스

- Bean : 스프링 컨테이너가 관리하는 오브젝트  
- 스프링의 설정 메타정보는 BeanDefinition 인터페이스로 표현되는 정보다.  
- 애플리케이션 컨텍스트는 BeanDefinition으로 만들어진 메타정보를 담은 오브젝트를 사용해 IoC와 DI 작업을 수행한다.

```java
    StaticApplicationContext ac = new StaticApplicationContext();
    BeanDefinition helloDef = new RootBeanDefinition(Hello.class);
    helloDef.getPropertyValues().addPropertyValue("name","Spring");
    ac.registerBeanDefinition("hello1",helloDef);
```

1. StaticApplicationContext 
    - 코드를 통해 메타정보를 등록할 때 사용한다.

2. GenericApplcationContext 
    - XML 파일과 같은 외부의 리소스에 있는 빈 설정 메타정보를 리더(BeanDefinitionReader)를 통해 읽어서 메타정보로 전환해서 사용한다.

3. GenericXmlApplicationContext
    - XmlBeanDefinitionReader을 내장
    - 애플리케이션 컨텍스트 생성과 동시에 xml 파일을 읽고 초기화까지 수행한다.
    - XML을 사용하는루트 컨텍스트를 만들때만 사용.

4. WebApplicationContext
    - 웹 환경에서 사용할때 필요한 기능이 추가된 애플리케이션 컨텍스트
    - XmlWebApplicationContext(XML), AnnotationConfigWebApplicationContext(애노테이션)
    - 서블릿을 만들고 애플리케이션 컨택스트를 생성해둔 다음, 요청이 서블릿으로 들어올 때마다 필요한 빈을 가져와 메소드 실행

- 계층구조
    - 애플리케이션 컨텍스트에서 검색 순서는 자신 > 직계부모 순서다. 
    - 자신이 이미 가지고 있다면 부모 컨텍스트에 정의되어 있는 것은 무시된다.

### 웹 애플리케이션 IoC 컨테이너 구성

프론트 컨트롤러 패턴 : 웹 요청을 받는 대표 서블릿을 등록해두고 공통적인 선행 작업을 한 뒤 핸들러 클래스를 호출하는 방식으로 개발    

스프링 애플리케이션 요청을 처리하는 서블릿 안에서 만들어짐 + 웹 애플리케이션 레벨에서 만들어짐    

애플리케이션 컨텍스트를 가지면서 프론트 컨트롤러 역할을 하는 서블릿 + 루트 애플리케이션 컨텍스트    

- 웹 애플리케이션 컨텍스트 구성 방법
    1. 서블릿 컨텍스트 + 루트 애플리케이션 컨텍스트 계층구조 
        - 웹 관련 빈들은 서블릿의 컨텍스트에 두고 나머지는 루트 애플리케이션 컨텍스트에 등록
    2. 루트 애플리케이션 컨텍스트 단일구조
        - 스프링 웹 기술을 사용하지 않고 프레젠테이션 계층을 만들때 사용
    3. 서블릿 컨텍스트 단일구조


- 루트 애플리케이션 컨텍스트 등록
    - 서블릿의 이벤트 리스너(ContextLoaderListener) 이용 : 웹 애플리케이션이 시작될 때 루트 애플리케이션 컨텍스트를 만들어 초기화
    - Default는 XmlWebApplicatoinContext. 다른 구현 클래스로 변경하고 싶을 때는 contextClass 파라미터 사용

``` xml
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- 설정 파일 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            /WEB-INF/applicationContext.xml
        </param-value>
    </context-param>
```

- 서블릿 애플리케이션 컨텍스트 등록
    - DispatcherServlet : 서블릿이 초기화 될 때 자신만의 컨텍스트를 생성하고 초기화한다.
    - 루트 애플리케이션의 컨텍스트를 찾아서 부모 컨텍스트로 사용한다.
    - load-on-startup : 서블릿을 언제 만들고 초기화할지, 순서를 지정하는 값 

 ``` xml
    <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 0 이상 : 웹 애플리케이션이 시작되는 시점에 서블릿을 로딩하고 초기화한다. -->
        <load-on-startup>1</load-on-startup>
    </servlet>
```   
