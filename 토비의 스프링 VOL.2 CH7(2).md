#### 캐시 추상화

- 스프링은 빈 메소드에 캐시 서비스를 적용할 수 있는 기능을 제공해준다.

1. 캐시 
    - 임시 저장소라는 뜻으로 복잡한 계산이나 DB 작업 등의 처리 결과를 캐시에 저장했다가 동일한 요청이 들어오면 캐시에 저장한 결과를 돌려주는 방식
    - 반복적으로 동일한 결과를 주는 작업에만 이용할 수 있다.
    - 캐시에 저장해둔 내용이 바뀌는 상황에 캐시에서 해당 내용을 지워주는 작업이 필요하다.
    - cache:annotation-driven 태그 또는 @EnableCaching을 사용하면 캐시 기능을 사용할 수 있다.

2. 애노테이션으로 캐시 속성 부여하기
    - AOP를 사용한다.
    - 캐시를 적용할 메서드에 @Cacheable 애노테이션을 붙여 캐시 속성을 부여할 수 있다.
    - 메서드의 파라미터나 파라미터의 hashCode로 캐시의 key가 된다. 파라미터가 없으면 0. 
    - @Cacheable의 key로 캐시 키를 설정할 수 있다.

3. 캐시 제거하기
    - 일정한 주기로 캐시를 제거한다. 
    - 캐시에 저장한 값이 변경되는 상황에 제거
    - 케시 제거에 사용될 메소드에 @CacheEvict 애노테이션을 붙여주면, 키 값에 해당하는 캐시만 제거한다.

4. 캐시 매니저
    - ConcurrentMapCacheManager : ConcurrentMapCache를캐시로 사용한다. 캐시 정보를 Map 타입으로 저장해서 속도가 빠르다.
    - SimpleCacheManager : 제공하는 캐시가 없다. 프로퍼티를 이용해서 사용할 캐시를 설정해야한다.
    - CompositeCacheManager : 하나 이상의 캐시 매니저를 사용하도록 지원해준다.

#### 빈 설정 모듈화

- 반복적으로 사용되는 빈 설정을 분리해 재사용하기 위함

1. @Import를 이용한 단순 재사용
    - 재사용하고 싶은 빈 설정을 @Configuration 클래스로 만들어두고 이를 @Import로 가져온다.
    - 설정 정보를 바꿀 수 없다는 단점이 있다.

2. @Configuration 클래스 상속과 오버라이딩
    - AppConfig extends CommonConfig
    - 오버라이딩으로 @Bean 메서드르 재정의할 수 있다. 일부 설정정보를 바꿔 재사용할 수 있다.
    - 다중 상속이 불가능해서 여러 Config 클래스의 빈을 재사용해야 할 때는 사용할 수 없는 방법이다.

3. @Enable 애노테이션과 ImportAware 인터페이스로 옵션지정
    - @Enable : 다른 @Configuration 클래스의 설정정보를 가져오지만 @Import를 직접 노출하지 않는다.
    - @Target : 애노테이션을 적용할 수 있는 대상을 지정한다.
    - @Retention : 애노테이션 정보의 유지 시점을 지정한다.
    - @ImportSelector로 Import에 적용할 Configuration 클래스를 다르게 결정하게 할 수 있다.

``` java
public class CommonConfig implements ImportAware {
    @Override
    public void setImportMetadata(AnnotationMetadata importMetadata){
        Map<String,Object> elements = importMetadata.getAnnotationAttributes(EnableCommon.class.getName());
        String name = (String)elements.get("name");
        common().setName(name); // name값 다시 설정
    }
}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(CommonConfig.class)
public @interface EnableCommon {
    String name();
}

```

4. 빈 설정자
    - 코드를 이용해 복잡한 설정을 추가해야할 때 사용한다.
    - 빈 설정자는 인터페이스로 만든다.
    - 빈 설정자를 적용하려면 구현한 클래스가 빈으로 등록돼야 한다.

``` java
@Configuration
@EnableCommon
public class CommonConfig implements CommonConfigure {
    @Override
    public void configName(Common common){
        common.setName("test");
    }
}

```

#### Reference
* * *
책 : 토비의 스프링 3.1 Vol.2 스프링의 기술과 선택 (저 이일민)
