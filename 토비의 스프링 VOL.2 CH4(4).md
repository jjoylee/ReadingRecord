#### JSP와 form

1. 뷰에서 모델 오브젝트 사용하기

- JSP EL : 스프링은 JSP 뷰에서 모델에 담긴 오브젝트를 JSP EL을 통해 접근할 수 있게 해준다. (${name})
- 스프링 SpEL : 표현식을 지원해주고 메소드 호출이 가능하다. 그리고 컨버전 서비스를 이용해서 포맷터를 자동으로 적용할 수 있다.
- 지역화 메세지 출력하기 : <spring:message code="code"/> > 코드 값을 키로 하는 메시지를 찾는다.

2. 폼 사용하기

- 순서
    1. 폼에 출력할 모델 결정
    2. 폼을 그릴 뷰를 만든다.
    3. 폼을 처음 출력할 때는 GET 요청을 사용한다.
    4. 사용자가 폼 작성 또는 수정한다.
    5. submit 버튼을 누르면 POST 메소드로 요청
    6. 폼에 입력된 데이터가 @ModelAttribute User에 바인딩 된다.
    7. 검증작업을 해서 오류가 발생하면 같은 폼 뷰를 띄우고 에러 메시지를 보여준다.
    8. 오류가 없으면 다른 뷰를 출력 또는 리다이렉트 

- 폼에서 EL을 사용할 때의 한계 
    - 바인딩 오류가 있을 떄 에러 메시지를 출력할 수 없다.
    - 타입 변환에서 오류가 나는 경우 값을 출력할 수 없다.

- spring:bind 태그 사용으로 해결
    - <spring:bind> 태그는 태그 내부에서 status 변수를 사용해서 BindStatus 오브젝트에 접근할 수 있다.
    - status.value 프로퍼티로 오류가 있어도 이전에 입력했던 값을 얻을 수 있다.
    - * form 태그 라이브러리를 이용하면 코드를 더 간략하게 만들 수 있다!

- form 태그 라이브러리 애트리뷰트
    - commandName, modelAttribute : 폼에 적용할 모델의 이름을 지정한다. default는 command
    - path : 태그의 id, name에 할당된다. 또한 모델의 프로퍼티 이름으로 사용된다. value에 출력될 때 컨버젼 서비스가 등록되어 있으면 변환해서 표시된다.
    - <form:errors> : 바인딩 에러 메시지 출력할 떄 사용한다. delimiter 속성으로 에러 메세지가 여러개일 때 구분자를 지정할 수 있다.
    - <form:checkbox> : checkox 태그를 만들어준다. 필드마커가 붙은 히든필드를 등록해준다.
    - 일정한 사용패턴이 있을 떄 커스텀 태그를 만들어 코드를 더 간결하게 만들 수 있다. 

``` jsp
    <!-- cssErrorClass : 필드에 에러가  있을 떄 적용되는 css 클래스 지정-->
    <form:label path="name" cssErrorClass="errorMessage">Name</form:label> :
    <form:input path="name" size="30">
    <form:errors path="name" cssClass="errorMessage">
```

#### 메시지 컨버터와 AJAX

메시지 컨버터
- HTTP 요청 메시지 본문과 응답 메시지 본문을 메시지로 다루는 방식
- AnnotationMethodHandlerAdapter를 통해 등록한다.
- 종류
    - ByteArrayHttpMessageConverter : Http 요청 메시지 본문을 byte배열로 가져오거나 response로는 application/octet-stream으로 타입이 설정된다.
    - StringHttpMessageConverter : HTTP 요청 본문을 스트링으로 가져온다.
    - FormHttpMessageConverter :  폼 데이터를 주고 받을 떄 사용한다. 그러나 주로 ModelAttribute로 바인딩한다.
    - SourceHttpMessageConverter : XML 문서를 Source 타입의 오브젝트로 전환할 때 사용할 수 있다.
    - MappingJacksonHttpMessageConverter : 자바 오브젝트와 JSON 문서를 자동 변환해준다.

#### MVC 네임스페이스

1. mvc:annotation-driven 태그 
- 애노테이션 방식의 컨트롤러를 사용할 때 필요한 DispatcherServlet 빈과 MVC 지원 기능 빈들을 자동으로 등록해준다.
- DefaultAnnotationHandlerMapping : @RequestMapping을 이용한 핸들러 매핑 전략.
- AnnotationMethodHandlerAdapter : DispatcherServlet이 자동으로 등록해주는 디폴트 핸들러 어댑터.
- ConfigurableWebBindingInitializer : WebDataBinder 초기화용 빈등 등록하고 AnnotationHandlerAdapter의 프로퍼티로 연결.

2. mvc:interceptors 태그
- 핸들러 매핑에 일괄 적용되는 인터셉터를 한 번에 설정할 수 있다.
- url 패턴을 지정할 수 있다.

```xml
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/common/*"/>
            <bean class="...CommonInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```

3. mvc:view-controller 태그
- URL 패턴과 뷰 이름을 넣으면 해당 URL을 매핑하고 뷰 이름을 리턴하는 컨트롤러를 등록한다.

4. mvc:default=servlet-handler 태그
- 디폴트 서블릿으로 포워딩하는 기능을 가진 핸들러
- @RequestMapping 조건에 맞지 않는 정적 리소스는 디폴터 서블릿으로 매핑

5. url:resource 태그
- 웹 리소스를 모듈화 할 수 있게 해준다.
- /META-INF/webresources 폴더 아래에 리소스 파일을 넣고 이를 ui.jar로 패키징 한 뒤 프로젝트 라이브러리 폴더에 넣는다.
- 정적 파일을 최적화된 방식으로 다룰 수 있다.

``` xml
    <!-- /ui로 시작되는 요청이 들어오면 /META-INF/webresources의 파일로 매핑해주는 핸들러가 등록된다-->
    <mvc:resources mapping="/ui/**" location="classpath:/META-INF/webresources/"/>
```
#### Reference
* * *
책 : 토비의 스프링 3.1 Vol.2 스프링의 기술과 선택 (저 이일민)
