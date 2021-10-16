#### RequestMapping

- 컨트롤러의 메소드가 독립적인 요청을 처리할 수 있도록 메서드 레벨에 매핑정보를 넣는다.
- 스프링 3.1은 전략 클래스를 새롭게 설계해서 메소드를 핸들러 오브젝터로 매핑할 수 없었던 한계를 극복했다.
- 기존의 Controller 대신 HandlerMethod 오브젝트를 핸들러(매핑 결과)로 넘겨준다.
- HandlerMethod : @RequestMapping이 붙은 메소드의 정보를 추상화한 오브젝트 타입
- RequestMappingHandlerMapping은 컨트롤러 빈의 메소드를 확인해서 매핑 후보 메소드를 추출한 뒤 HandlerMethod 오브젝트로 저장해두고 요청이 들어오면 조건이 맞는 오브젝트를 찾아 돌려준다.

1. 요청 조건 : 매핑의 기준이 되는 정보
- value : URL 패턴
- method : HTTP 요청 메소드
- params : 파라미터
- headers : HTTP 헤더
- consumes : Content-Type 헤더
- produces : Accept 헤더

2. 요청 조건 결합 방식
- URL 패턴 
    - 하나 이상이면 OR 조건으로 연결, 클래스와 메소드에 패턴이 존재하는 경우에는 모든 가능한 조합을 만든다.
    - 한쪽에만 패턴이 있으면 그 패턴만 사용한다.
    - useSuffixPatternMatch 프로퍼티(true/false)로 .* 를 포함시킬지 결정한다.
- HTTP 요청 방법
    - HTTP 요청 방법에 대한 요청 조건은 methods 엘리먼트로 지정한다.
    - 요청 방법들을 OR로 연결한다.
- 파라미터 요청 조건 : 모든 조건을 AND로 연결
- 헤더 요청 조건 : 모든 조건을 AND로 연결
- Content-Type 헤더 : consumes 엘리먼트로 지정하며, OR로 연결된다. 타입과 메소드의 조건을 조합해서 사용하지 않고, 메소드 조건이 우선이다.
- Accept 헤더 : produces 엘리먼트로 지정한다. 다른 것은 consumes와 동일

3. @RequestMapping 핸들러 어댑터
- @Validated/@Valid : 빈 검증기 애노테이션을 이용해서 모델의 값 검증. 모델을 검증할 때 특정 그룹의 제약 조건을 적용할 때 @Validated를 사용한다.
- UriComponentsBuilder : 여러 요소가 조합된 URI를 안전하고 편리하게 다룰 수 있는 기능을 제공한다. 코드에서 URI를 생성하거나 가공할 때 사용하면 좋다.

4. RedirectAttributes

- 브라우저에게 새로운 URL로 요청을 다시 보내라고 지시하는 응답방식.
- RedirectView나 redirect: 접두어가 붙은 뷰 이름을 리턴.
- 리다이렉트 뷰를 사용할 떄 model에 넣은 데이터는 리다이렉트 url에 자동으로 추가된다. 
- 모델을 이용해 URL 파라미터에 값을 넣을 수 있다.
- RediretAttributes를 사용하면 모델 대신 RedirectAttributes 정보로 URL을 생성한다.

5. HandlerMethodArgumentResolver
- 위의 인터페이스를 구현해서 핸들러 메소드에서 사용할 수 있는 새로운 종류의 파라미터 타입을 추가할 수 있따.

``` java
    model.addAttribute("status", status);
    return "redirect:/result/{status}";
```

#### @EnableWebMvc와 WebMvcConfigurationSupport

- @Configuration 클래스에 @EnableWebMvc를 붙여주면 <mvc:annotation-config/>를 등록했을 떄와 동일하게 동작한다.
- @Enable 전용 애노테이션의 설정을 위해 사용되는 빈을 Configurer라 한다.

1. WebConfigurer 메소드
- addFormatters() : FormatterRegistry를 이용해서 간단히 포매터를 등록할 수 있다.
- configureMessageConverters() : 기본 메시지 컨버터 구성을 사용하지 않고 직접 구성할 때 사용한다.
- getValidator() : 기본 LocalValidatorFactoryBean을 대신하는 검증기를 등록할 때 사용한다.
- addArguentResolvers() : 새로운 파라미터 종류를 지원하기 위한 HandlerMethodArgumentResolver를 추가할 수 있는 메서드
- addReturnValueHandlers() : 새로운 리턴 값 처리 방식을 추가하기 위해 HandlerMethodReturnValueHandler를 추가할 떄 사용한다.
- configureHandlerExceptionResolvers() : 핸들러 예외 전략을 새로 구성할 떄 사용한다.
- addInterceptors() : 인터셉터를 새로 등록해준다.
- addViewControllers() : URL 패턴을 뷰 이름으로 돌려주는 컨트롤러를 등록하는 메소드. 정적 페이지를 출력할 때 주로 사용.
- addResoucerHandlers() : <mvc:resources> 기능을 담당하는 메소드 

2. @MVC 설정자 빈 등록하기

1. @Bean 이용해서 등록하기

``` java
@Configuration
@EnableWebMvc
public class AppConfig {
    @Bean
    public WebMvcConfiguer webMvcConfigurer(){
        return new MyWebMvcConfigurer(); // WebMvcConfiguer 구현한 클래스11
    }
}
```

2. WebConfig가 직접 WebMvcConfiguer를 구현하거나 WebMvcConfiguerAdapter를 상속

``` java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfiguer {
```

3. WebMvcConfigurationSupport 클래스를 상속하는 빈 등록하기
- @EnableWebMvc에 의해 등록되는 전략 빈의 내용을 @Bean 메소드로 갖고 있다. 오버리이딩으로 구성을 바꿀 수 있다.
