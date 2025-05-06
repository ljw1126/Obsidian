
## Redis
> [!note] 
> 변경이 적고, 접근 빈도가 높은 데이터의 경우 캐싱

**cache aside pattern**
- 대량의 읽기 요청을 고려한 패턴
- 1차적으로 분산 캐시(Redis)에 데이터를 저장
- 그리고 각 서버는 local cache를 사용해 성능 향상하는 전략
- 캐시 성능과 높은 응답성 보장 
- 데이터 수정이 발생하면 1차 분산 캐시(Redis)의 수정이 먼저 일어나야 함

> 하나의 서비스/데이터 특성에 대해 여러 캐시를 나눠 사용하기도 한다

**Write back pattern**
- 대량의 쓰기 요청을 고려한 패턴
- 데이터를 저장할 때 DB에 바로 쿼리하지 않고, 캐시에 모아서 일정 주기 배치 작업을 통해 DB에 반영
- 캐시에 모아놨다가 DB에 쓰기 때문에 쓰기 쿼리 횟수 비용과 부하를 줄일 수 있다
- 단, 캐시에서 오류가 발생하면 데이터를 영구 소실 할 수 있다 -> 디스크에 저장하는 방식 있는 걸로 암
	- 그리고 자주 사용되지 않은 불필요한 리소스를 Redis 저장

🛜 \[REDIS\] 캐시(Cache) 설계 전략 지침 100 % 총 정리 - 인파 기술블로그 ([바로가기](https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-%EC%BA%90%EC%8B%9CCache-%EC%84%A4%EA%B3%84-%EC%A0%84%EB%9E%B5-%EC%A7%80%EC%B9%A8-%EC%B4%9D%EC%A0%95%EB%A6%AC#write_back_%ED%8C%A8%ED%84%B4))
## Spring Webflux
- Reactive Stream API
- non-block
- aysnchronous

### CPU Bound vs I/O Bound
> Spring Webflux는 I/O Bound에 가깝다

**CPU Bound**
- 컨텍스트 스위칭 발생
- 최소화하려면 코어 수 증가 필요 (1 Core가 번갈아 가며 두 앱을 실행)

**I/O Bound**
- Web Application 관점에서 I/O Bound가 발생하는 것은
	- 클라이언트 요청/응답
	- DB
	- 외부 API Server
- I/O 요청 처리량을 늘릴려면
	- 전통적인 방법으로는 **스레드** 늘림 (= 멀티 스레드 방식)
		- 하지만 한계있어서 OOM이 발생가능
		- 스레드를 무리하게 만들거나, 생성/소멸에 대한 리소스 낭비
		- 그래서 `스레드 풀 전략`을 사용함 (물론 컨텍스트 스위칭 비용은 있음)
	- 그리고 서버를 scale-out하고 로드밸런싱 하는 등의 전략을 추가로 생각가능

### Sync / Async

**Async**
- 요청 작업을 기다리지 않고 다음 작업을 실행 가능하다
- Async 작업은 완료 순서를 보장하지 않는다
- ComputableFuture 사용하면 메인 스레드 대신 다른 스레드가 생성되서 처리된다

**I/O Multiplexing**
- i/o 요청에 대한 이벤트를 보내고, 대기 중에 있다가 응답 이벤트를 받으면 다음 i/o 요청 발생

**Thread per request**
- 스레드 풀 전략 중에 캐시 전략이 있었다
- 요청만큼 스레드 바로 생성해서 처리하는 것 (결국 비즈니스 상황에 맞는 전략을 선택하는게 중요)
### Spring MVC vs Webflux

Spring MVC의 경우 
- Servlet 컨테이너와 Spring 컨테이너를 거쳐 하나의 요청 당 Thread 하나씩 할당되어 처리하기 때문에 문제 파악이나 흐름을 이해하는데 직관적
- blocking i/o로 이뤄지기 때문에 대량의 동시성 처리에 한계가 있다

### Reactive Stream
참고
- https://www.reactive-streams.org/
- https://projectreactor.io/

**1. Reatice Stream**
- 비동기 스트림 처리 
- 논블로킹 백프레셔 제공
- Publisher, Subscriber, Subscription, Processor 로 구성
- 이를 구현한 프로젝트 Reactor를 Spring Webflux 에서 활용


### R2DBC
> Reactive Relational DataBase Connectivity

참고. 
- [spring blog - Reactive Programming and Relational Databases](https://spring.io/blog/2018/12/07/reactive-programming-and-relational-databases)
- [Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc#overview)
- [Spring Data R2DBC Reference](https://docs.spring.io/spring-data/r2dbc/docs/current-SNAPSHOT/reference/html/#preface)
- https://github.com/asyncer-io/r2dbc-mysql


앞에 처리 로직이 비동기이더라도 후반 데이터베이스에 저장하는 로직이 동기이면 
결국 전체가 동기로 동작하는거랑 같음

R2DBC는 JDBC와 달리 
- Asynchoronous Database Access
- Reactive Stream
- Nonblocking I/O
- Open specification
	- H2, MariaDB, MySQL, Oracle, PostgreSQL, ...

r2dbc-api 기준 스펙이 있는듯함

Spring MVC
- RestTemplate (그 외 RestClient, FeignClient을 사용 가능, `library 프로젝트 참고`)
- JDBC, JPA

Spring Webflux
- WebClient
- R2DBC

[R2DBC 공식 문서를 읽어보자](https://docs.spring.io/spring-data/r2dbc/docs/current-SNAPSHOT/reference/html/#preface)


### Reactive Redis
- 일반 Redis 서버와의 통신은 동기적으로 실행됨
- Reactive Redis를 사용해서 비동기 통신 가능하다 함
	- Reactive Stream
	- Nonblocking I/O
	- Spring Data Reactive Redis
		- lettuce - Advanced Java Redis client (깃 저장소)
- spring webflux가 redis 통신할 때 reactive redis 사용
	- spring data reactive redis
	- [스프링 공식 예제](https://spring.io/guides/gs/spring-data-reactive-redis)


### Spring MVC vs Spring Webflux 성능 비교✨
- Spring MVC
	- JPA, Spring Data Redis 사용 
- Spring Webflux
	- R2DBC, Spring Data Reactive Redis 사용 
- 도커 컨테이너로 Redis, MySQL 올리고 성능 비교
- Apache JMeter 사용해 성능 테스트 **3가지** 수행
	- `/health` 체크에 대한 성능 테스트 
	- MySQL 요청에 대한 성능 테스트 
	- Redis 요청에 대한 성능 테스트

### Blockhound
- 비동기 어플리케이션에서 블로킹을 검출하는 기능 제공
- 의존성 추가 필요 

---

**TODO**
- queue 문자열을 파라미터로 받지 않고, restful 하게 분리하기 
- 로그인한 사용자만 대기열 등록하도록 수정 
	- Spring Security + JWT
- 모놀리식과 Webflux 방식으로 Rest API 성능 비교 해보기 
	- 간단하게 JSON 응답 요청
	- RedisTemplate, ReactiveRedisTemplate
- 대기열 페이지 이동하기 전 사용자 유효성 검사하는 로직이 여러 개로 보인다. 
	- 허용된 유저인가?
	- 대기열에 유저가 있는가? 
		- 있다면 대기열 랭크 반환
		- 없다면 대기열 추가 후 랭크 반환
	- 부하 증가시 네트워크 통신 비용이 늘지 않나 싶다..
		- 개인적인 생각으로 서버를 scale-out한다면 각각 부하 분산도 잘 되서 성능이 향상되지 않을가 싶다. (**PASS**)


### 대기열 등록 API 개발

Spring webflux 서버에서 파라미터를 받는 이유에 대해
```java
@RestController  
@RequiredArgsConstructor  
@RequestMapping("/api/v1/queue")  
public class WaitingQueueController {  
    private final WaitingQueueService waitingQueueService;  
  
    @PostMapping  
    public Mono<WaitingQueueResponse> enqueueUser(
    @RequestParam(name = "queue", defaultValue = "default") String queue,  
	@RequestParam(name = "userId") Long userId) {  
        return waitingQueueService.enqueue(queue, userId)  
                .map(WaitingQueueResponse::new);  
    }  
}
```

**queue 문자열**
✅ 가능성 있는 이유
- 여러 종류의 큐를 운영하기 위한 유연성 때문
	- 하지만 지금 큐는 `대기열 큐`와 `처리 큐` 두 가지만 있으면 되서 의미가 없어 보인다
⚠️ 단점
- 외부에서 queue 이름을 임의로 입력 가능하여 **보안 이슈** 또는 **무의미한 큐 생성** 가능
✅ 개선 방안
- 큐의 이름이 고정이라면, RESTFUL하게 주소(리소스)를 고정한다거나 queue 파라미터를 enum으로 제한
	- 서비스에서 repository에 enum 을 파라미터로 주입하는 방식을 사용하는 것도 유연성에 좋을듯함.

**userId를 직접 받음**
✅ 가능성 있는 이유
- 테스트의 편의성을 위해서 `userId`를 사용하는 것으로 추측
⚠️ 문제점
- userId를 파라미터로 직접 받기 보다, 인증된 사용자 정보를 가져오는 것이 보안상 안전
	- spring security 사용해서 jwt 토큰을 발급받아 spring webflux에서 사용하는 형태(?)
✅ 권장 방식
- 로그인 후 인증된 유저를 기반으로 처리 


```java
@Service  
@RequiredArgsConstructor  
public class WaitingQueueService {  
    private final RedisRepository redisRepository;  
  
    public Mono<Long> enqueueWaitingQueue(final Long userId) {  
        long unixTimestamp = Instant.now().getEpochSecond();  
        return redisRepository.addZSet(userId, unixTimestamp)  
                .filter(i -> i)  // Mono<Boolean> 반환
                .switchIfEmpty(Mono.error(ALREADY_RESISTER_USER.build()))  
                .flatMap(i -> redisRepository.zRank(userId))  
                .map(i -> i >= 0 ? i + 1 : i);  
    }}
```
- filter를 하는 이유는 Sorted Set에 데이터 추가가 되면 boolean을 반환하는데 이미 등록된 유저의 경우 false를 반환하게 된다.
	- false 인 경우(이미 대기열 등록된 유저) : switchIfEmpty(..) 실행하여 예외 던짐 
	- true 인 경우 (처음 대기열 등록 유저) : rank 조회하여 반환


### 진입 요청 API 개발

1. **대기열 큐**에 있는 유저 중 일정 수(`count`)만큼 뽑아서 **진입허용 큐**에 추가 
2. 진입이 허용된 상태인지 확인하는 API 필요

서비스 renaming
```java
@Service  
@RequiredArgsConstructor  
public class QueueService {  
    private final RedisRepository redisRepository;  
  
    public Mono<Long> enqueueWaitingQueue(final Long userId) {  
        long unixTimestamp = Instant.now().getEpochSecond();  
        String queue = WAITING_QUEUE.getKey();  
        return redisRepository.addZSet(queue, userId, unixTimestamp)  
                .filter(i -> i)  
                .switchIfEmpty(Mono.error(ALREADY_RESISTER_USER.build()))  
                .flatMap(i -> redisRepository.zRank(queue, userId))  
                .map(i -> i >= 0 ? i + 1 : i);  
    }  
    public Mono<Long> allow(Long count) {  
        return redisRepository.popMin(WAITING_QUEUE.getKey(), count)  
                .flatMap(member -> redisRepository.addZSet(PROCEED_QUEUE.getKey(), Long.parseLong(Objects.requireNonNull(member.getValue())), Instant.now().getEpochSecond()))  
                .count();  
    }  
    public Mono<Boolean> isAllowed(Long userId) {  
        return redisRepository.zRank(PROCEED_QUEUE.getKey(), userId)  
                .defaultIfEmpty(-1L)  
                .map(rank -> rank >= 0);  
    }
}
```
- Queue 이름을 관리하기 어려워 **enum**으로 뽑음



```java
@Repository  
@RequiredArgsConstructor  
public class RedisRepositoryImpl implements RedisRepository {  
    private final ReactiveRedisTemplate<String, String> reactiveRedisTemplate;  
  
    @Override  
    public Mono<Boolean> addZSet(String queue, Long userId, Long timestamp) {  
        return reactiveRedisTemplate.opsForZSet()  
                .add(queue, userId.toString(), timestamp);  
    }  
    @Override  
    public Mono<Long> zRank(String queue, Long userId) {  
        return reactiveRedisTemplate.opsForZSet()  
                .rank(queue, userId.toString());  
    }  
    @Override  
    public Flux<ZSetOperations.TypedTuple<String>> popMin(String queue, Long count) {  
        return reactiveRedisTemplate.opsForZSet().popMin(queue, count);  
    }}
```

ReactiveRedisTemplate로 popMin 테스트 
```java
@SpringBootTest  
@Import(EmbeddedRedis.class)  
@ActiveProfiles("test")  
public class RedisRepositoryTest {  
  
    @Autowired  
    private ReactiveRedisTemplate<String, String> reactiveRedisTemplate;  
  
    private RedisRepositoryImpl redisRepository;  
  
    @BeforeEach  
    void setUp() {  
        redisRepository = new RedisRepositoryImpl(reactiveRedisTemplate);  
  
        ReactiveRedisConnection reactiveConnection = reactiveRedisTemplate.getConnectionFactory().getReactiveConnection();  
        reactiveConnection.serverCommands().flushAll().subscribe();  
    }  
    
    @Test  
    void addZSet() {  
        Long userId = 1L;  
        long timestamp = Instant.now().getEpochSecond();  
        String queue = QueueManager.WAITING_QUEUE.getKey();  
  
        StepVerifier.create(redisRepository.addZSet(queue, userId, timestamp))  
                .expectNext(true)  
                .verifyComplete();  
    }  
    @Test  
    void addZSetWhenDuplicated() {  
        Long userId = 1L;  
        long timestamp = Instant.now().getEpochSecond();  
        String queue = QueueManager.WAITING_QUEUE.getKey();  
  
        StepVerifier.create(redisRepository.addZSet(queue, userId, timestamp))  
                .expectNext(true)  
                .verifyComplete();  
  
        StepVerifier.create(redisRepository.addZSet(queue, userId, timestamp))  
                .expectNext(false)  
                .verifyComplete();  
    }  
  
    @Test  
    void zRank() {  
        String queue = QueueManager.WAITING_QUEUE.getKey();  
  
        StepVerifier.create(redisRepository.addZSet(queue,1L, 100L)  
                .then(redisRepository.zRank(queue,1L)))  
                .expectNext(0L)  
                .verifyComplete();  
    }  
    @Test  
    void zRankByMultiUser() {  
        String queue = QueueManager.WAITING_QUEUE.getKey();  
  
        StepVerifier.create(redisRepository.addZSet( queue,1L, 100L)  
                .then(redisRepository.addZSet(queue,2L, 99L))  
                .then(redisRepository.zRank( queue,2L)))  
                .expectNext(0L)  
                .verifyComplete();  
    }  
    @Test  
    void zRankByNoneUserId() {  
        String queue = QueueManager.WAITING_QUEUE.getKey();  
  
        StepVerifier.create(redisRepository.zRank(queue,99L))  
                .expectComplete()  
                .verify();  
    }  
    @Test  
    void popMin() {  
        String queue = QueueManager.WAITING_QUEUE.getKey();  
  
        Mono<Boolean> setup = redisRepository.addZSet(queue, 1L, 100L)  
                .then(redisRepository.addZSet(queue, 2L, 101L))  
                .then(redisRepository.addZSet(queue, 3L, 103L));  
  
        Flux<ZSetOperations.TypedTuple<String>> result = setup.thenMany(redisRepository.popMin(queue, 4L)); // Mono -> Flux  
  
        StepVerifier.create(result)  
                .expectNextMatches(tuple -> tuple.getValue().equalsIgnoreCase("1"))  
                .expectNextMatches(tuple -> tuple.getValue().equalsIgnoreCase("2"))  
                .expectNextMatches(tuple -> tuple.getValue().equalsIgnoreCase("3"))  
                .verifyComplete();  
    }
}
```


```java
@SpringBootTest  
@Import(EmbeddedRedis.class)  
@ActiveProfiles("test")  
class QueueServiceTest {  
    @Autowired  
    private QueueService queueService;  
  
    @Autowired  
    private ReactiveRedisTemplate<String, String> reactiveRedisTemplate;  
  
    @BeforeEach  
    void setUp() {  
        ReactiveRedisConnection reactiveConnection = reactiveRedisTemplate.getConnectionFactory().getReactiveConnection();  
        reactiveConnection.serverCommands().flushAll().subscribe();  
    }  
    @Test  
    void enqueueWaitingQueue() {  
        StepVerifier.create(queueService.enqueueWaitingQueue(1L))  
                .expectNext(1L)  
                .verifyComplete();  
  
        StepVerifier.create(queueService.enqueueWaitingQueue(2L))  
                .expectNext(2L)  
                .verifyComplete();  
    }  
    @DisplayName("이미 등록된 유저가 재시도하는 경우 예외를 던진다")  
    @Test  
    void alreadyEnqueueWaitingQueue() {  
        StepVerifier.create(queueService.enqueueWaitingQueue(1L))  
                .expectNext(1L)  
                .verifyComplete();  
  
        StepVerifier.create(queueService.enqueueWaitingQueue(1L))  
                .expectError(WaitingQueueException.class)  
                .verify();  
    }  
    @DisplayName("대기열 큐에 유저가 없는 경우 0을 반환한다")  
    @Test  
    void emptyAllowUser() {  
        StepVerifier.create(queueService.allow(100L))  
                .expectNext(0L)  
                .verifyComplete();  
    }  
    @DisplayName("대기열 큐에 허용한 유저 수 만큼 카운팅을 반환한다")  
    @Test  
    void allowUser() {  
        Mono<Long> setup = queueService.enqueueWaitingQueue(1L)  
                .then(queueService.enqueueWaitingQueue(2L))  
                .then(queueService.enqueueWaitingQueue(3L))  
                .then(queueService.enqueueWaitingQueue(4L))  
                .then(queueService.enqueueWaitingQueue(5L));  
  
        StepVerifier.create(setup.then(queueService.allow(3L)))  
                .expectNext(3L)  
                .verifyComplete();  
    }  
    @DisplayName("허용된 유저가 아니면 false를 반환한다")  
    @Test  
    void isNotAllowed() {  
        StepVerifier.create(queueService.isAllowed(99L))  
                .expectNext(false)  
                .verifyComplete();  
    }  
    @DisplayName("대기열 큐에서 허용된 후 다른 아이디로 허용 여부 확인하면 false 반환한다")  
    @Test  
    void isAllowedOtherUserId() {  
        StepVerifier.create(queueService.enqueueWaitingQueue(100L)  
                        .then(queueService.allow(3L))  
                        .then(queueService.isAllowed(101L)))  
                .expectNext(false)  
                .verifyComplete();  
    }  
    @DisplayName("대기열 큐에서 허용된 아이디로 여부 확인하면 true 반환한다")  
    @Test  
    void isAllowed() {  
        Long userId = 100L;  
  
        StepVerifier.create(queueService.enqueueWaitingQueue(userId)  
                        .then(queueService.allow(3L))  
                        .then(queueService.isAllowed(userId)))  
                .expectNext(true)  
                .verifyComplete();  
    }}
```


### 접속 대기 웹 페이지 개발 
- `/index` 접속시 대기열 여부를 확인하고 페이지로 이동하도록 한다
	- web mvc 서버 ➡️ webflux 서버로 통신 
		- 허용된 사용자인지 확인한다 (PROCEED QUEUE)
		- 대기열에 있는 사용자인가 ? (WAITING QUEUE)
			- 대기열에 등록된 사용자라면 순위를 반환한다
			- 대기열에 등록된 사용자가 아니라면 대기열에 추가하고 순위를 반환한다


**website 모듈**
```java
@Controller  
@RequiredArgsConstructor  
public class HomeController {  
    private final WaitingQueueService waitingQueueService;  
  
    @GetMapping("/index")  
    public String home(@ModelAttribute(name = "userId") Long userId, Model model) {  
        QueueStatusResponse response = waitingQueueService.accessibleCheck(userId);  
        if(response.accessible()) {  
            return "index";  
        }  
        model.addAttribute("rank", response.rank());  
        return "waiting-room";  
    }
}
```

이때 WebClient 빈을 생성해서 주입받아 사용
```java
@Service  
@RequiredArgsConstructor  
public class WaitingQueueService {  
    private final WebClient webClient;  
  
    public QueueStatusResponse accessibleCheck(Long userId) {  
        return webClient.get()  
                .uri(uriBuilder -> uriBuilder.path("/api/v1/queue/checked").queryParam("userId", userId).build())  
                .retrieve()  
                .bodyToMono(QueueStatusResponse.class)  
                .block();  
    }  
}
```


**webflux 모듈**
```java
@RestController  
@RequiredArgsConstructor  
@RequestMapping("/api/v1")  
public class QueueController {  
    private final QueueService queueService;  

	//..
		
    @GetMapping("/queue/checked")  
    public Mono<QueueStatusResponse> checked(@RequestParam("userId") Long userId) {  
        return queueService.checked(userId)  
                .map(rank -> new QueueStatusResponse(rank == 0, rank));  
    }
}
```


```java
@Service  
@RequiredArgsConstructor  
public class QueueService {  
    private final RedisRepository redisRepository;  
  
    public Mono<Long> enqueueWaitingQueue(Long userId) {  
        long unixTimestamp = Instant.now().getEpochSecond();  
        String queue = WAITING_QUEUE.getKey();  
        return redisRepository.addZSet(queue, userId, unixTimestamp)  
                .filter(i -> i)  
                .switchIfEmpty(Mono.error(ALREADY_RESISTER_USER.build()))  
                .flatMap(i -> redisRepository.zRank(queue, userId))  
                .map(i -> i >= 0 ? i + 1 : i);  
    }  
    public Mono<Long> allow(Long count) {  
        return redisRepository.popMin(WAITING_QUEUE.getKey(), count)  
                .flatMap(member -> redisRepository.addZSet(PROCEED_QUEUE.getKey(), Long.parseLong(Objects.requireNonNull(member.getValue())), Instant.now().getEpochSecond()))  
                .count();  
    }  
    public Mono<Boolean> isAllowed(Long userId) {  
        return redisRepository.zRank(PROCEED_QUEUE.getKey(), userId)  
                .defaultIfEmpty(-1L)  
                .map(rank -> rank >= 0);  
    }  
    public Mono<Long> checked(Long userId) {  
        return isAllowed(userId)  
                .filter(Boolean::booleanValue) // PROCEED QUEUE에 추가된 유저의 경우  
                .flatMap(allowed -> Mono.just(0L))  
                .switchIfEmpty(enqueueWaitingQueue(userId)  
                                .onErrorResume(ex -> redisRepository.zRank(WAITING_QUEUE.getKey(), userId).map(i -> i >= 0 ? i + 1 : i))  
                ).log();  
    }
}
```
- PROCEED QUEUE에 추가된 유저인 경우 true를 반환한다. - isAllowed
	- `flatMap(..)` 통해 **0L**을 반환 
- PROCEED QUEUE에 없는 경우 
	- WAITING QUEUE에 추가한다. 
		- 대기열에 없는 경우 WAITING QUEUE에 순위를 반환하게 된다
		- 만약 추가되어 있는 경우 예외 발생하여 onErrorResume(..) 실행해서 순위를 조회하여 반환


webflux 서버에 요청시 cors 에러 발생하여 아래와 같이 설정했지만 의미 없었음 ..
```java
@Configuration  
@EnableWebFlux  
public class CorsGlobalConfig implements WebFluxConfigurer {  
  
    @Override  
    public void addCorsMappings(CorsRegistry registry) {  
        registry.addMapping("/**")  
                .allowedOrigins("http://localhost:8080")  
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")  
                .allowedHeaders("*")  
                .allowCredentials(true)  
                .maxAge(3600);  
    }
}
```

- [Inpa 기술블로그 - CORS ] (https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-CORS-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-%F0%9F%91%8F)
- [Baeldung- spring webflux cors](https://www.baeldung.com/spring-webflux-cors)


**트러블슈팅**
- checked 요청시 userId의 score가 갱신되는 이슈 발생
- 아래 테스트가 통과되는 이유는 timestamp가 동일했기 때문에 false를 반환 
	- 만약 timestamp가 다르면 addZSet 호출 시 score가 갱신되어 버린다💩

```java
@Test  
void addZSetWhenDuplicated() {  
    Long userId = 1L;  
    long timestamp = Instant.now().getEpochSecond();  
    String queue = QueueManager.WAITING_QUEUE.getKey();  
  
    StepVerifier.create(redisRepository.addZSet(queue, userId, timestamp))  
            .expectNext(true)  
            .verifyComplete();  
  
    StepVerifier.create(redisRepository.addZSet(queue, userId, timestamp))  
            .expectNext(false)  
            .verifyComplete();  
}
```


그래서 timestamp를 매번 생성해서 넣었는데도 통과해버림 ㄷㄷ 
- 임베디드 레디스라서 그런게 아닌가 싶다..
```java
@Test  
void addZSetWhenDuplicated() {  
    Long userId = 1L;  
    String queue = QueueManager.WAITING_QUEUE.getKey();  
  
    StepVerifier.create(redisRepository.addZSet(queue, userId, Instant.now().getEpochSecond()))  
            .expectNext(true)  
            .verifyComplete();  
  
    StepVerifier.create(redisRepository.addZSet(queue, userId, Instant.now().getEpochSecond()))  
            .expectNext(false)  
            .verifyComplete();  
}

```