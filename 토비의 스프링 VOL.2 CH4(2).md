#### @SessionAttributes와 SessionStatus

- 서블릿은 기본적으로 상태를 유지하지 않는다. 

도메인 오브젝트로 사용자 정보를 수정하기

- 사용자 정보는 사용자가 수정할 수 없는 데이터와 수정할 수 있는 데이터가 있다.
- 수정하면 안되는 정보는 form에 보여주지 않는 것이 좋다.
- form submit으로 수정할 때 고객이 수정하지 못하는 데이터는 null, 0으로 전달될 가능성이 있다. 

1. 히든 필드 사용해서 해결
    - 사용자가 수정하면 안되는 정보는 hidden 필드에 넣는다.
    - 데이터 보안에 문제를 일으킨다. (html에서 조작할 수 있다.)
    - 새로운 필드가 추가될 때 히든 필드도 추가해야한다. 실수할 가능성이 높아진다.

2. DB 재조회하기
    - 수정된 정보를 받아 도메인 객체에 넣어줄 때 DB에서 데이터를 다시 가져온 뒤 수정할 데이터만 set 해준다.
    - form submit 할 때 데이터를 다시 읽어와야한다는 단점이 있다.
    - 복사할 필드를 잘 알아야한다.

3. 계층 사이에 강한 결합을 주기
    - 수정할 데이터만을 update하는 DAO 함수, Service 메서드를 만든다.
    - 코드 중복이 많아진다는 단점이 있다.
    - 테스트하기 힘들다.
    - Map<String, String> 타입 파라미터를 사용하는 방법도 있다.

4. @SessionAttributes 사용하기
    - 컨트롤러메소드가 생성하는 모델정보 중에서 @SessionAttributes에 지정한 이름과 동일하면 세션에 저장.
    - @ModelAttribute가 @SessionAttributes에 지정한 이름과 같으면 세션에 오브젝트가 있으면 그 오브젝트를 넣어준다.
    - form submit할 떄 DB에서 처음 가져왔던 오브젝트에 변경된 프로퍼티만 바인딩
    - 이름 충돌이 발생하지 않도록 조심해야한다.
    - 더 이상 필요 없는 세션 애프리뷰트를 제거해줘야한다.
    - 등록 폼에서는 빈 오브젝트 모델을 사용하면 된다.

```java
    @Controller
    @SessionAttributes("user")
    public class UserController{
        ...
        public String form(@RequestParam int id, Model model){
            model.addAttribute("user", userService.getUser(id));
            return "user/edit";
        }

        @RequestMapping(value="/user/edit", method=RequestMethod.POST)
        public String submit(@ModelAttribute User user, SessionStatus sessionStatus){
            this.userService.updateUser(user);
            sessionStatus.setComplete();
            return "user/editSuccess"; // 세션 오브젝트 제거
        }
    }
```

#### 모델 바인딩

- 파라미터 타입의 오브젝트를 만든다. 디폴트 생성자가 꼭 필요하다!
- 모델 오브젝트의 프로퍼티에 웹 파라미터를 바인딩해준다. 바인딩 문제가 있으면  BindingResult에 오류를 저장한다.
- 모델의 값을 검증한다.

1. PropertyEditor

- 스프링이 기본적으로 제공하는 바인딩용 타입 변환 API
- 변환할 파라미터 또는 모델 프로퍼티의 타입에 맞는 프로퍼티 에디터를 자동으로 선정한다.
- 커스텀 프로터티 에디터
    - PropertyEditorSupport 클래스를 상속해서 커스텀 프로퍼티 에디터를 만들 수 있다.
    - WebDataBinder에 문자열을 파라미터 타입의 오브젝트를 변경하는 기능이 있다.
    - 커스텀 프로퍼티 에디터를 등록하려면 WebDataBinder 초기화 메소드를 사용해야한다.
    - 특정 프로퍼티 이름에만 적용할 수도 있다.
    - 모든 컨트롤러에 적용해야할 때는 WebBindingInitializer를 이용한다.
    - 상태를 가지고 있기 때문에 싱글톤 빈으로 등록하면 안된다.

    ``` java
    @InitBinder // 메소드 파라미터를 바인딩 하기 전에 자동을로 호출된다.

    public void initBinder(WebDataBinder dataBinder){
        dataBinder.registerCustomEditor(Level.class, new LevelPropertyEditor());
    }
    ```
