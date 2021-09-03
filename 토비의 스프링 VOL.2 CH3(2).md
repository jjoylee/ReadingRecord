#### 컨트롤러 역할

- 서비스 계층의 메소드 선정,  메소드 파라미터 타입에 맞게 요청 정보 변환
- 뷰에 출력할 내용을 모델에 넣어준다.
- 세션을 관리한다.

#### 컨트롤러 종류

1. Servlet과 SimpleServletHandlerAdapter
    - 표준 서블릿
    - 서블릿 클래스 코드를 유지하면서 스프링 빈으로 등록할 수 있다.
    - 서블릿이 컨트롤러 빈으로 등록된 경우 생명주기 메소드는 자동으로 호출되지 않는다. 
2. HttpRequestHandler와 HttpRequestHandlerAdapter 
    - HttpRequestHandler 인터페이스를 구현해서 컨트롤러를 만든다.
    - 로우레벨 서비스를 개발할 때 이용할 수 있다.
3. Controller와 SimpleControllerHandlerAdapter
    - Controller 인터페이스를 구현해서 만든다.
    - 유연하게 컨트롤러 클래스를 설계할 수 있지만 주로 AbstractController를 상속해서 만든다.
    - synchronizeOnSession : HTTP 세션에 대한 동기화 여부 결정 (동시 접근 x)
    - supportedMethod : 컨트롤러가 허용하는 HTTP 메소드 지정
    - Controller 인터페이스 컨트롤러를 확장해서 기반 컨트롤러를 만들어 효과적으로 컨트롤러를 만들 수 있다.
4. AnnotationMethodHandlerAdapter
    - 지원하는 컨트롤러 타입이 정해져 있지 않다.
    - 컨트롤러가 하나 이상의 URL에 매핑될 수 있다.(메소드 단위)
    - DefaultAnnotationHandlerMapping

#### 핸들러 매핑

- 요청을 처리할 컨트롤러를 찾아준다.

1. BeanNameUrlHandlerMapping
    - 빈의 이름에 들어 있는 URL을 HTTP 요청 URL과 비교해서 일치하는 빈을 찾아준다.
    - 사용하기에는 간편하지만 컨트롤러 개수가 많아지면 매핑구조를 파악하기 어렵기에 복잡한 애플리케이션에서는 잘 사용하지 않는다.
    - 디폴트 핸들러 매핑
2. ControllerBeanNameHandlerMapping 
    - 빈 아이디나 빈 이름을 이용해 매핑해준다.
    - 자동으로 빈 아이디에 /를 붙여준다. (Ex- @Component("hello") > /hello)
    - prefix와 suffix를 지정할 수 있다.
3. ControllerClassNameHandlerMapping
    - 빈 이름 대신 클래스 이름을 URL에 매핑해준다.
    - Controller로 끝날 때는 Controller를 뺀 나머지 이름을 매핑 (ex- HelloController > /hello)
4. SimpleUrlHandlerMapping
    - URL과 컨트롤러 매핑 정보를 한 곳에 모아놓을 수 있다는 장점이 있다.
    - 관리하기 쉽다
    - 오타가 발생할 가능성이 높다.

``` xml
    <!--SimpleUrlHandlerMapping-->
    <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <property name="mappings">
            <props>
                <prop key="/hello">helloController</prop>
                <prop key="/sub/*">myController</prop>
            </props>
        </property>
    </bean>
```

5. DefaultAnnotationHandlerMapping
    - @RequestMapping 애노테이션을 부여하고 이를 이용해 매핑하는 전략.
    - 메소드 단위로 URL을 매핑할 수 있기에 컨트롤러 개수를 줄일 수 있다.

6. 주요 프로퍼티
    - order : 두개 이상의 핸들러 매핑을 사용할 때 우선순위 지정
    - defaultHandler : 매핑할 대상을 찾지 못할 경우 디폴트 핸들러 선택
    - alwaysUseFullPath : url 전체를 사용해서 컨트롤러를 매핑해야할 때 true로 설정하면 된다.
    - detectHandlersInAncestorContext : 부모 컨텍스트에서도 컨트롤러 찾도록 설정할 수 있다. 하지만 절대 사용 x


#### 핸들러 인터셉터

- 핸들러 매핑은 핸들러 인터셉터를 적용해준다.
- preHandle() : 컨트롤러가 호출되기 전에 실행된다.
- postHandle() : 컨트롤러를 실행하고 난 후에 실행된다. preHandle()이 false 리턴하면 실행되지 않는다.
- afterCompletion() : 뷰 작업까지 포함한 모든 작업이 완료된 후 실행된다.
- 핸들러 매핑 클래스를 빈으로 등록한 뒤 interceptors 프로퍼티를 사용해 적용할 수 있다.
- 컨트롤러에 공통적으로 적용해야하는 부가기능이 있을 떄 사용하면 좋다.

#### 커스텀 핸들러 어댑터
- HandlerAdapter 인터페이스 구현
- support()로 지원하는 컨트롤러 타입이 맞는지 확인
- handle()로 컨트롤러 호출

#### 뷰

- 뷰 오브젝트는 View 인터페이스를 구현해야한다.
- View 타입의 오브젝트나 이름을 return한다. 이름을 return할 때는 ViewResolver가 필요하다.
- InternalResouceView : 뷰 이름을 포워딩할 JSP 이름으로 사용. 모델 정보를 애트리뷰트에 넣는 작업을 InternalResouceView와 DispatcherServlet이 대신 해준다.
- JstlView : 지역화된 메시지를 JSP 뷰에 사용할 수 있게 해준다.
- RedirectView : HttpServletResponse의 sendRedirect()를 호출해주는 기능을 가진 뷰 (redirect:로 시작하는 뷰 이름 사용)
- VelocityView : 자바 템플릿 엔진을 뷰로 사용하게 해준다.
- MarshallingView : OXM 추상화 기능을 사용해 XML 콘텐트를 작성하게 해주는 뷰

#### 뷰 리졸버
- 뷰 이름으로부터 사용할 뷰 오브젝트를 찾아준다.
- InternalResourceViewResolver : 디폴트 뷰 리졸버, 주로 JSP를 뷰로 사용할 때 사용한다.
- VelocityViewResolver, FreeMakrerViewResolver : 템플릿 엔진 기반 뷰를 사용하게 해주는 뷰 리졸버.
- ResourceBundleViewResolver, XmlViewResolver : 외부 리소스 파일에 뷰 이름에 해당하는 뷰 클래스와 설정을 담아둔다.
- ContentNegotiationViewResolver : 미디어 타입 정보를 활용해서 적절한 뷰를 찾아준다.(위임)
