#### Spring AOP

- 관점 지향 프로그래밍
- 어떤 로직을 기준으로 핵심적인 관점(핵심 비즈니스 로직), 부가적인 관점(로그, 트랜잭션, DB연결) 을 나누어셔 보고 그 관점을 기준으로 각각 모듈화하는 것.
- 프록시 패턴 사용
- 어떤 클래스가 AOP 대상이면, 기존 클래스 빈이 만들어질 때 프록시를 자동으로 만들고 원본 클래스 대신 프록시를 빈으로 등록한다. 그리고 원본 클래스가 사용되는 지점에서 프록시를 대신 사용한다.
- 공통 기능이 한 장소에서 관리된다.
- 다른 서비스 모듈이 본인의 목적에만 충실하고 그 외 사항들은 신경쓰지 않아도 된다.

#### 개념

- Aspect : 관심사를 모듈화한 것(부가기능)
- Target : Aspect를 적용하는 곳, 부가기능을 부여할 대상
- Advice : 실질적인 부가기능을 담은 구현체
- JointPoint : Advice가 Target 에 적용되는 위치 (메서드 진입 시점, 생성자 호출 시점 등 다양한 시점에 적용 가능)
- PointCut : JointPoint의 상세한 스펙을 정의한 것. 더 구체적으로 Advice가 실행될 시점을 정할 수 있다. 부가기능이 적용된 대상을 선정하는 방법
- Weaving : 지정된 객체에 애스팩트를 적용해서 새로운 프록시 객체를 생성하는 과정

``` java
@Aspect
public class TimeTraceAop {

    // @Around : Advice (타겟 메소드 호출전과 후에 어드바이스 기능 수행)
    // execution(* hello.hellospring..*(..)) : PointCut 표현식
    @Around("execution(* hello.hellospring..*(..))") 
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed(); // 타겟 메소드 실행
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}


```
* * *
참고

https://engkimbs.tistory.com/746
https://atoz-develop.tistory.com/entry/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-AOP-%EA%B0%9C%EB%85%90-%EC%9D%B4%ED%95%B4-%EB%B0%8F-%EC%A0%81%EC%9A%A9-%EB%B0%A9%EB%B2%95
https://jojoldu.tistory.com/71
