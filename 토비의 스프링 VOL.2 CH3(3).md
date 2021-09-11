#### 기타 리졸버

1. 핸들러 예외 리졸버
- HandlerExceptionResolver : 컨트롤러 작업 중 발생한 예외를 어떻게 처리할지를 결정하는 전략.
- 예외 발생 > 핸들러 예외 리졸버 확인 > 있으면 핸들러 예외 리졸버가 처리, 아니면 서블릿 컨테이너가 처리
- AnnotationMethodHandlerExcpetionResolver : @ExceptionHandler 애노테이션이 붙은 메소드가 예외처리한다. 특정 컨트롤러 작업 중 발생하는 예외를 처리하는 핸들러를 만들 때 유용하다.
- ResponseStatusExceptionResolver : 예외를 특정 HTTP 응답 상태로 전환해준다.
- DefaultHandlerExceptionResolver : 스프링에서 내부적으로 발생하는 주요 예외를 처리.
- SimpleMappingExceptionResolver : 예외를 처리할 뷰를 지정할 수 있게 해준다.

2. 지역정보 리졸버
- 애플리케이션에서 사용하는 지역정보를 결정하는 전략
- 보통 브라우저의 기본 설정 사용
- 세션이나 쿠키 값을 사용하게도 설정할 수 있다.

3. 멀티파트 리졸버
- 파일 업로드와 같은 요청정보를 처리하는 전략을 설정할 수 있다.
- CommonsMultipartResolver 빈을 등록해야한다.
- RequestToViewNameTranslator : 컨트롤러에서 뷰 이름이나 뷰 오브젝트를 주지 않았을 경우 HTTP 요청정보로 뷰 이름을 생성하는 로직을 담고 있다.

### 스프링 MVC

1. 플래시 맵 
- 플래시 애트리뷰트(하나의 요청에서 생성되어 다음 요청으로 전달되는 정보)를 저장하는 맵
- POST 단계의 결과 메세지를 리다이렉트된 페이지로 전달할 때 주로 사용한다.
- 플래시 맵 매니저 : 플래시 맵을 저장, 조회, 제거하는 등의 작업을 담당한다. FlashMapManager 인터페이스를 구현해서 만든다.

2. WebApplicationInitializer
- 구현한 클래스를 찾아 컨텍스트의 초기화 작업을 위임한다
- 이 클래스를 이용해 루트 컨텍스트나 서블릿 컨텍스트를 자바 코드에서 직접 등록하고 생성할 수 있다.
- 리소스를 잘 반환하기 위해 루트 컨텍스트는 리스너를 이용해 관리하는 것이 좋다.

``` java
// 루트 컨텍스트 등록하기
AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
servletContext.scan("com.mycompany.myproject.config");
ServletContextListener listener = new ContextLoaderListener(ac);
servletContext.addListener(listener);

// 서블릿 컨텍스트 
ServletRegistration.Dynamic dispatcher = servletContext.addServlet("spring", new DispatcherServlet());
dispatcher.setLoadOnStartup(1);
dispatcher.addMapping("/app/*");
``` 
 

#### Reference
* * *
책 : 토비의 스프링 3.1 Vol.2 스프링의 기술과 선택 (저 이일민)
