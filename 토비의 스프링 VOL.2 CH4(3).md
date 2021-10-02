#### Converter와 Formatter

1. Converter
- Converter는 파라미터 변환 과정에서 메소드가 한 번만 호출된다.
- 상태가 없어서 싱글톤 빈으로 등록해둘 수 있다.
- 소스 타입에서 타깃 타입으로의 단방향 변환만 지원한다.
- 컨트롤러에 컨버터를 적용하려면 ConversionService 타입의 오브젝트를 사용해서 WebDataBinder에 설정해야한다.

``` java
public class LevelToStringConverter implements Converter<Level,String> {
    public String convert(Level level){
        return String.valueOf(level.intValue()); // Level to String
    }
}
```
- @InitBinder로 Converter 수동 등록하기

```java
@Controller
public static class SearchController {
    @Autowired
    ConversionService conversionService // ConversionFactoryBean을 등록해두어야 한다.

    @InitBinder
    public void initBinder(WebDataBinder dataBinder){
        dataBinder.setConversionService(this.conversionService);
    }
}

```

- ConfigurableWebBindingInitializer를 이용해 일괄로 등록하기 
    - 빈 설정 만으로 WebBindingInitializer빈을 등록할 수 있다.
    - Controller 코드가 단순해진다.

2. Formatter
- 스트링 타입의 폼 필드와 컨트롤러 메소드 파라미터 사이에 양방향으로 적용할 수 있도록 두 개의 변환 메소드를 가지고 있다.
- FormattingConversionService를 통해서 적용할 수 있다.
- object > string : print() (view에서 오브젝트 내용을 HTML로 만들어 줄 때), string > object : parse()
- Locale 타입의 현재 지역정보도 제공된다.
- @NumberFormat
    - 다양한 타입의 숫자 변환(java.lang.Number)을 지원하는 포맷터
    - style 로 숫자, 통화, 퍼센트 표시를 지정, pattern으로 숫자 패턴을 지정할 수 있다. (@NumberFormat("$###,##0.00"))
    - FormattingConversionFactoryBean 등록이 필요하다.
- @DateTimeFormat
    - 날짜 시간 정보 라이브러리 Joda Time을 이용하는 포맷터

### 바인딩 기술 적용 우선순위

1. 사용자 정의 타입이고 모델에서 자주 활용되는 타입
    - 컨버터로 만들고 ConversionService로 일괄로 적용한다.
    - 싱글톤으로 사용할 수 있어서 유리하다.
2. 조건부 변환 기능
    - ConditionalGenericConverter를 이용해야 한다.
    - GenericConversionSerivce에 등록해서 바인딩에 일괄적용
3. 애노테이션 정보를 활용한 HTTP 요청과 모델 필드 바인딩.
    - AnnotationFormatterFactory를 이용해 애노테이션에 따른 포맷터를 생성해주는 팩토리를 구현해야한다.
    - FormattingConversionService로 팩토리를 등록할 수 있다.
4. 특정 필드만 변환
    - 커스텀 프로퍼티 에디터를 만들어 사용한다.

#### WebDataBinder

- allowedFieldss : 바인딩이 허용된 필드 목록 지정 (dataBinder.setAllowed("필드"))
- disallowedFields : 바인딩 금지 필드 목록 지정. 와일드 카드도 사용할 수 있다.
- requiredFields : 필수 HTTP 파라미터 지정
- fieldMarkerPrefix : filedMaker(submit 데이터가 없어도 필드가 존재함을 알려줌, checkbox에서 사용)로 사용되는 히든필드의 이름 앞에 붙이는 접두어를 지정한다.
- fieldDefaultPrefix : fieldDefault( 체크박스 체크 x 했을 떄의 디폴트 값 지정 )의 접두어를 지정한다. 기본값은 !다.

#### Validator와 BindingResult

1. Validator
    - 결과는 BindingResult에서 확인할 수 있다.
    - Validator의 supports() : 해당 검증기가 검증할 수 있는 오브젝트 타입인지 확인
    - Validator의 validate() : supports()가 true면 validate() 호출
    - 필드 값에 문제가 있으면 ErrorS의 rejectValue 메서드 또는 ValidatorUtils의 메서드로 어느 필드에 문제가 있는지 등록해야한다.
    - Validator는 빈으로 등록이 가능하기 떄문에 Controller에서 DI 받아 메소드에서 필요할 때 호출할 수 있다.

2. @Valid 이용해서 검증하기 
    - 바인딩 과정에서 자동으로 검증이 진행된다.
    - WebDataBinder의 setValidator 메서드로 검증용 오브젝트를 설정한다.
    - 컨트롤러 메소드의 @ModelAttribute 파라미터에 @Valid를 추가하면 validator가 모델을 검증하고 결과를 BindingResult에 넣는다.

3. JSR-303 빈 검증
    - 모델 오브젝트의 필드에 제약조건 애노테이션을 추가해서 검증을 진행한다. (@Min, @NotNull 등)
    - LocaleValidatorFactoryBean을 등록해줘야한다.
    - 등록한 빈을 DI받아서 사용하면된다.

4. BindingResult
    - DefaultMessageCodeResolver가 reject 함수에서 사용한 에러 코드로 메세지 키를 찾아 message.properties에서 에러 메세지를 찾는다.
    - 특정 필드가 아닌 모델레벨의 글로벌 에러는 reject()로 사용해서 등록할 수 있다.
