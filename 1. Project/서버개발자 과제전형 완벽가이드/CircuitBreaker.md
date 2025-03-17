
> 특정 서버나 기능에 장애 발생시 전체 시스템에 장애 전파 되지 않도록 하는 장치
> - **Close**: 회로가 닫혀서 정상적으로 요청이 전달되고 처리되는 상태
> - **Open**: (실패가 임계값 이상일 때) 회로가 열려서 차단된 상태 
> - **Half-Open**: 일부 요청을 전달해서 정상 처리 가능한지 확인하는 상태 (실패시 다시 `Open`, 정상이면 `Close` 상태로 변경)

- [https://spring.io/projects/spring-cloud-circuitbreaker](https://spring.io/projects/spring-cloud-circuitbreaker)
- [https://github.com/resilience4j/resilience4j](https://github.com/resilience4j/resilience4j)
- [https://resilience4j.readme.io/docs/circuitbreaker](https://resilience4j.readme.io/docs/circuitbreaker)


**대표적인 설정**  ([공식문서](https://resilience4j.readme.io/docs/circuitbreaker))
- slidingWindowType
- slidingWindowSize
- minimumNumberOfCalls
- waitDurationInOpenState
- permittedNumberOfCallsInHalfOpenState
- maxWaitDurationInHalfOpenState
- ignoreExceptions

**\*.yml 설정**
```text
resilience4j:  
  circuitbreaker:  
    configs:  
      default:  
        sliding-window-size: 100  
        minimum-number-of-calls: 10  
        wait-duration-in-open-state:  
          seconds: 20  
        failure-rate-threshold: 50  
        permitted-number-of-calls-in-half-open-state: 10
```
- 인스턴스별로 설정을 다르게 할 수도 있다
- 지금은 default 설정만

>[!warning] Lombok does not copy the annotation 'org. springframework. beans. factory. annotation. Qualifier' into the constructor 

- 롬북 사용시 Qualifier 애노테이션이 생성자에 복사되지 않는 문제가 있다
```java
@Service  
@RequiredArgsConstructor  
public class BookQueryService {  
    @Qualifier("naverBookRepository")  
    private final BookRepository naverBookRepository;  
  
    @Qualifier("kakaoBookRepository")  
    private final BookRepository kakaoBookRepository;

    //..
}
```
- 루트 디렉터리에 `lombok.config `파일을 생성 후 아래 내용 추가
	- [stackoverflow](https://stackoverflow.com/questions/38549657/is-it-possible-to-add-qualifiers-in-requiredargsconstructoronconstructor)
	- gradle 새로고침시 warning이 사라짐

```text
lombok.copyableAnnotations += org.springframework.beans.factory.annotation.Qualifier
```


테스트를 수정한다. 정상적인 경우 kakao\*쪽을 호출하지 않는다
```java
class BookQueryServiceTest extends Specification {  
    BookRepository naverBookRepository = Mock(BookRepository)  
    BookRepository kakaoBookRepository = Mock(BookRepository)  
  
    BookQueryService bookQueryService;  
  
    void setup() {  
        bookQueryService = new BookQueryService(naverBookRepository, kakaoBookRepository)  
    }  
    def "search시 인자가 그대로 넘어가고 naver 쪽으로 호출한다"() {  
        given:  
        def givenQuery = "HTTP 완벽가이드"  
        def givenPage = 1  
        def givenSize = 10  
  
        when:  
        bookQueryService.search(givenQuery, givenPage, givenSize)  
  
        then:  
        1 * naverBookRepository.search(*_) >> {  
            String query, int page, int size ->  
                assert query == givenQuery  
                assert page == givenPage  
                assert size == givenSize  
        }  
  
        and:  
        0 * kakaoBookRepository.search(*_)  
    }}
```


```java
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_EACH_TEST_METHOD)  
@ActiveProfiles("test")  
@SpringBootTest  
class BookQueryServiceIntegrationTest extends Specification {  
  
    @Autowired  
    BookQueryService bookQueryService  
  
    @Autowired  
    CircuitBreakerRegistry circuitBreakerRegistry;  
  
    @SpringBean  
    KakaoBookRepository kakaoBookRepository = Mock()  
  
    @SpringBean  
    NaverBookRepository naverBookRepository = Mock()  
  
    def "정상 상태에서는 circuit의 상태가 closed 상태가 되고, naver 쪽으로 호출이 들어간다"() {  
        given:  
        def keyword = "HTTP"  
        def page = 1  
        def size = 10  
  
        when:  
        bookQueryService.search(keyword, page, size)  
  
        then:  
        1 * naverBookRepository.search(keyword, page, size) >> new PageResult<>(1, 10, 0, [])  
  
        and:  
        def circuitBreaker = circuitBreakerRegistry.getAllCircuitBreakers().stream().findFirst().get()  
        circuitBreaker.state == CircuitBreaker.State.CLOSED  
  
        and:  
        0 * kakaoBookRepository.search(*_)  
    }  
    def "circuit이 오픈되면 kakao 쪽으로 요청한다"() {  
        given:  
        def keyword = "HTTP"  
        def page = 1  
        def size = 10  
        def kakaoResponse = new PageResult(1, 10, 0, [])  
  
        def config = CircuitBreakerConfig.custom()  
                .slidingWindowSize(1)  
                .minimumNumberOfCalls(1)  
                .failureRateThreshold(50)  
                .build()  
        circuitBreakerRegistry.circuitBreaker("naverSearch", config)  
  
        and: "naver 쪽은 항상 예외가 발생한다"  
        naverBookRepository.search(keyword, page, size) >> { throw new RuntimeException("error!") }  
  
        when:  
        def result = bookQueryService.search(keyword, page, size)  
  
        then: "kakao 쪽으로 Fallback 된다"  
        1 * kakaoBookRepository.search(keyword, page, size) >> kakaoResponse  
  
        and: "circuit가 OPEN 상태가 된다"  
        def circuitBreaker = circuitBreakerRegistry.getAllCircuitBreakers().stream().findFirst().get()  
        circuitBreaker.state == CircuitBreaker.State.OPEN  
  
        and:  
        result == kakaoResponse  
    }  
}
```
- `@DirtiesContext` : 각 테스트마다 스프링 컨텍스트를 재실행하여 메서드 영향 안 받도록 함
	- 해당 설정 없이 할 경우 두번째 테스트에서 실패 확인됨 (OPEN 예상했는데 CLOSE)
	- 두번째 테스트에서 서킷 브레이커 설정을 커스텀 했지만, 반영되기 까지 시간이 충분하지 않았을 수 있다

---

**참고. 개발자 유미 채널**
- [resilience4j 서킷 브레이커](https://www.youtube.com/watch?v=UpwJOImeYcA&list=PLJkjrxxiBSFCAvgvqYaIFlSWYCfa1x4TQ)
