#### @MVC

- 애노테이션을 중심으로 한 새로운 MVC의 확장 기능

1. @RequestMapping 핸들러 매핑
- DefaultAnnotationHandlerMapping 빈을 등록해야한다.
- 매핑정보로 @RequestMapping 애노테이션을 활용한다.
- 타입 레벨에 @RequestMapping을 이용해서 타입 내의 매핑 메소드의 공통 조건(url)을 지정할 수 있다.
- @Controller를 사용하면 클래스 레벨에서는 @RequestMapping을 생략할 수 있다.
- @RequestMapping 정보는 상속되지만 서브 클래스에서 재정의 하면 슈퍼 클래스의 정보는 무시된다.
- 속성
    1. value()
        - URL 패턴 지정, 와일드 카드를 사용할 수 있다.
        - {}로 PATH VARIABLE 지정할 수 있다.
    2. method() 
        - 요청 메소드에 따라 다른 메소드에 매핑해 줄 수 있다.
    3. params()
        - 매핑을 위한 요청 파라미터 지정
        - @RequestMapping(value="/user/edit", params="type=admin")
    4. headers()
        - 해더 정보로 매핑할 요청 지정

2. @Controller 메소드 파라미터

    - WebRequest : HttpServletRequest의 요청정보 대부분을 가지고 있는 서블릿에 종속적이지 않는 오브젝트 타입
    - Locale : 지역정보 리졸버가 결정한 Locale 오브젝트를 받을 수 있다.
    - @PathVariable : 패스 변수 ({})를 받는다. 파라미터를 URL 패스에 넣을 때 사용
    - @RequestParam : 단일 HTTP 요청 파라미터를 타입을 변환해서 메소드 파라미터에 넣어준다. required 속성으로 필수인지 설정할 수 있다. 
    - @CookieValue : Http 요청과 전달된 쿠키 값을 메소드 파라미터에 넣어준다. (@CookieValue("쿠키이름") String value)
    - @RequestHeader : 요청 헤더정보를 메소드 파라미터에 넣어준다.
    - @ModelAttribute : 모델 맵에 담겨서 뷰에 전달되는 모델 오브젝트, 클라이언트로부터 받는 오브젝트 형태의 정보. 컨트롤러가 return하는 모델에 파라미터로 전달한 오브젝트를 자동으로 추가해준다.
    - 애노테이션이 없는 파라미터 : 단순 타입은 @RequestParam이 생략된 것으로 생각하고 오브젝트는 @ModelAttribute가 생략된 것으로 생각한다.
    - BindingResult/Error : 오브젝트에 파라미터를 바인딩하다가 발생한 오류와 검증 오류가 저장된다. 반드시 @ModelAttribute 파라미터 뒤에 나와야 한다.
    - @RequestBody : 파라미터에 HTTP 요청의 BODY가 그대로 전달된다.
    - @Value : 빈의 값 주입 방식으로 파라미터에 사용
    - @Valid : 빈 검증기로 모델 오브젝트를 검증하도록 지시하는 지시자 역할을 한다.
```java
    // 모든 요청 파라미터를 받을 수 있다.
    // key : 파라미터 이름, value : 값
    public String add(@RequestParam Map<String,String> params)
```

3. 자동 추가 모델 오브젝트와 자동생성 뷰 이름

- @ModelAttribute 모델 오브젝트 또는 커맨드 오브젝트
- Map, Model ModelMap 파라미터
- @ModelAttribute 메소드 : 뷰에서 참고 정보로 사용되는 모델 오브젝트를 생성하는 메소드 지정. 모델 오브젝트 생성 전담(select option들)
- BindingResult : 바인딩 오류 메시지를 생성할 때 사용된다.

4. 메소드 리턴 타입

- ModelAndView 
- String : 뷰 이름으로 사용된다. 모델정보는 모델 맵 파라미터에 추가해주는 방법을 사용해야한다.
- void : RequestToViewNameResolver 전략으로 자동 생성되는 뷰 이름이 사용된다.(URL)
- 모델 오브젝트 : return한 객체를 모델에 자동으로 추가해준다. 모델 이름은 return 값의 타입
- Map/Model/ModelMap : 모델로 사용된다. 맵을 return하면 그 안의 엔트리 하나하나를 모델로 등록하기 때문에 조심히 사용해야한다.
- View : 뷰 오브젝트 직접 리턴
- @ResponseBody : reutrn하는 오브젝트를 HTTP 응답 메세지 본문으로 전환.
