
**todo.**
- service에 dto 클래스 보면 response에 AllArg* 있는데 굳이 필요하지 않아 보인다. 
	- 오히려 pojo 방식으로 선언해서 할당하는게 가독성이 좋아 보임
	- article, comment 모듈 수정하기 


## 게시글 

<img src="./image/article_erd.png"/>




## 댓글 

> **대댓글 (2depth) 방식**과 **무한 depth 방식** 둘 다 구현

<img src="./image/comment_erd.png"/>
- **대댓글** : comment_parent_id 기반으로 계층형 구조
- **무한 depth**: varchar(25) path, 25개 문자열 기반으로 5개씩 나눠 계층 표현
	- 총 5depth 존재 00000 00000 00000 00000 ~ zzzzz zzzzz zzzzz zzzzz zzzzz
	- 각 자리는 **0-9A-Za-z**, 총 **62개** 문자열 중 하나
		- 텍스트 선언 후 인덱스 기반으로 다음 문자열 가져와 처리
	- depth 별로 댓글은 62^5 = 약 9억개 정도 할당 가능 
	- depth 별로 댓글이 9억개가 넘어가면 문자열 길이(path)를 늘리거나 하면 됨
- **삭제**의 경우 
	- 자식이 있으면 논리적 삭제(`deleted = true`업데이트)
	- 자식이 없으면 물리적 삭제(DB)

**두 방식 비교.**



**TODO.**
- 게시판을 두개다 만들건지 아니면 하나만 할건지 선택해서 남겨두자 


## 좋아요

> **비관적 락(pessmistic lock)**, **낙관적 락(optimistick lock)** 방식을 둘 다 사용해 봄

<img src="./image/article_like_erd.png"/>

| 기능     | REST 표현                | HTTP 메서드 | 설명                   |
| ------ | ---------------------- | -------- | -------------------- |
| 좋아요    | `/articles/{id}/likes` | `POST`   | 좋아요 추가 (Like 리소스 생성) |
| 좋아요 해제 | `/articles/{id}/likes` | `DELETE` | 좋아요 취소 (Like 리소스 제거) |
- 좋아요 응답은 201 created
- 좋아요 해제 응답은 204 no content

|서비스 유형|추천 방식|
|---|---|
|**정합성 최우선 (재고/주문)**|비관적 락 (Pessimistic Lock)|
|**많은 트래픽, 정확성 중요 (좋아요)**|낙관적 락 + 재시도, 또는 Redis 누적 후 DB 반영|
|**캐시 허용 가능 (조회수, 좋아요)**|Redis 기반 처리 (성능 우선)|
|**낮은 충돌 가능성 (로그 등)**|락 없이 처리|
- 비관적 락을 사용하는 경우 
	- update 방식과 select .. for update 방식이 있음
	- 정합성은 높으나 데드락 위험 있음
- 낙관적 락을 사용하는 경우 
	- 재시도 로직이 필요함 (spring-retry)
	- 재시도 로직이 없는 경우 데이터 유실 높음


> 통합 테스트만 작성하는 걸로 함, 왜냐하면 service, repository 둘다 의미 없어 보이고 
> 동시성 이슈에 대한 확인만 하면 되기 때문에 controller 단에서만 테스트하는게 맞다고 생각
> - 처음에는 강의와 같이 비관적 락 (2개, `update`, `select .. for update`), 낙관적 락 테스트(재시도 x)
> - `spring-retry` 추가해서 낙관적 락만 사용하는 걸로 최종 결정 후 리팩터링 


낙관적락 테스트시 beforeEach에서 데이터 초기화하면 바로 OptimisticLockingFailureException 발생  > beforeEach를 빼면 정상 실행은 되나 재시도 로직이 없어 전체 1000개 실패 





1000개 넣는데 
- 비관적 락1(update) 300ms
- 비관적 락2(select for update) 1156ms
- 낙관적 락 (재시도x) 테스트 실패 (7/3)
	- (7/4) 문제 원인은 h2는 read committed인데 flush(), clear() 한다고 commit 되는게 아니었음 > 하나의 트랜잭션이 종료되어야 commit 실행함
	- ArticleLikeCount에 `of(..)` 정적 팩토리 메서드에서 version = 0L 초기화하는 것과, @BeforeEach에서 ArticleLikeCount 초기화하는거 지우니 정상 동작
	- success : 212, fail:788, time: 533ms, count: 212
		- retry가 없어서 OptimisticLockingFailureException 터져서 실패가 많음
- 낙관적 락 (재시도 o, 7/4) 

| 구분                          | success | failure | time    |
| --------------------------- | ------- | ------- | ------- |
| 비관적 락(update)               | 1,000   | 0       | 300ms   |
| 비관적 락(select .. for update) | 1,000   | 0       | 1,156ms |
| 낙관적 락(재시도 x)                | 212     | 788     | 533ms   |
| 낙관적 락(재시도 o)                | 978     | 22      | 1313ms  |
- 재시도 3회, 100ms 간격
	- scale-out하더라도 단일 DB의 경우 row write로 인해 병목 현상 발생 가능
	- Redis 저장해서 일정 주기마다 db 백업 (flush)
		- **실시간 반응성 + 정확도**가 필요한 영역 사용 가능
		- 단, 인프라 비용 및 복잡도 증가
- ✅ 낙관적 락은 재시도와 함께 사용하면 정합성 유지에 매우 효과적
	- 단, 재시도가 없으면 실패율이 압도적이며, 재시도를 추가하면 성공률은 높지만 성능 비용이 증가
		- 따라서 **낙관적 락은 재시도 로직 없이는 실전에서 쓰기 어렵다는 교훈**을 주는 테스트였다.******


> [!note] @Version은 되도록이면 직접 초기화하지 않기 
> - 초기값이 null 이고, 다음이 0이되야 하는데 null이 아닌 경우 JPA는 해당 엔티티를 이미 DB에 존재하는 엔티티라고 잘못 판단하게 됨 
> - 결과적으로 Insert가 아닌 update를 시도하게 되는데, DB에 데이터가 없으니`StaleObjectStateException` 또는 `OptimisticLockException` 발생
> - 버전이 null인 경우 새 엔티티로 인식


다시 낙관적 락(재시도x)에 setup을 추가하더라도 에러가 발생해서 chat-gpt 질문함
```java
@BeforeEach 
void setUp() { 
	articleLikeCountRepository.save(ArticleLikeCount.of(1L, 0L)); //
	articleLikeCountRepository.flush(); 
	entityManager.clear(); 
}
```

```text
// 무한 루프 돌면서 insert 시도
article version : null, count : 0
article version : null, count : 0
article version : null, count : 0
article version : null, count : 0
article version : null, count : 0
article version : null, count : 0
article version : null, count : 0
article version : null, count : 0
article version : null, count : 0
..
```

**🔍 문제의 본질**
**테스트 초기화는 정상적으로 되어 있는데도, 동시성 테스트 시 `findById`가 null을 반환하는 이유는 무엇일까?**
> ▶ 이는 SpringBootTest에서의 멀티 스레드 환경에서 **트랜잭션 전파/격리 레벨**과 **H2의 MVCC 특성**, 그리고 **영속성 컨텍스트/쓰기 지연 플러시** 등의 복합적인 요인이 작용한 결과입니다.

|원인|설명|
|---|---|
|**스레드마다 트랜잭션이 분리**|SpringBootTest 환경에서 `@Transactional`이 클래스 레벨에 없다면, 각 테스트 메서드는 **스레드마다 트랜잭션을 독립적으로 실행**|
|**flush 이후에도 `clear()` 하지 않으면 `1차 캐시` 영향**|`entityManager.clear()`를 호출했더라도, 스레드가 생성한 새로운 트랜잭션 내에서는 다시 DB를 읽어야 함|
|**H2의 트랜잭션 격리 수준**|H2는 기본적으로 `READ COMMITTED`, 즉 다른 트랜잭션에서 커밋하지 않으면 `SELECT` 시 조회되지 않음|
|**flush가 커밋이 아니다**|`flush()`는 DB로 쿼리를 날리지만, `commit`을 하기 전까지는 다른 트랜잭션에서 조회 불가 (MVCC)|
**✅ 정리하자면**
- `@BeforeEach`에서 save → flush → clear 해도 **commit이 되지 않으면 다른 스레드가 SELECT로 읽을 수 없음**
- → 그래서 각 스레드가 `findById()`를 했을 때, 아직 커밋되지 않은 데이터라 `Optional.empty()` 발생
- → insert 시도 → 중복 key 예외 또는 JPA 내부 충돌 발생

**🔍 왜 멀티 스레드에서 하나도 `commit`되지 않았나?**
테스트 흐름은 다음과 같았을 가능성이 높습니다:
1. `@BeforeEach`에서 `save + flush + clear`를 호출했으나,  
    → **JUnit 테스트 클래스에 `@Transactional`이 붙어 있어서 해당 트랜잭션이 끝날 때까지 커밋이 안 됨**  
    → 즉, DB에는 쿼리는 날아가되 **물리적 커밋이 안 된 상태**입니다.
2. 테스트 본문(`@Test`)에서 1000개의 스레드를 만들어 `likeOptimisticLock()`을 호출  
    → 이 중 일부는 `findById(...)`가 실패하여 `ArticleLikeCount.of(...)`로 새 엔티티 생성  
    → 동시에 같은 PK로 insert 시도 → **JPA 내부 충돌 or DB Constraint 위반 발생**💩
3. 그 와중에 초기화 데이터의 트랜잭션도 끝나지 않음 (즉, 여전히 `commit` 안 된 상태)  
    → **다른 트랜잭션에서는 해당 데이터를 보지 못함**


✅ `@BeforeEach` setup 메서드 없이도 통과한 이유는 ?
- `@BeforeEach`에서 `save()` 후 `flush()`를 하더라도 **트랜잭션이 커밋되지 않으면**, 다른 스레드나 트랜잭션에서는 **해당 데이터가 보이지 않음**  → 왜냐면 H2의 기본 격리 수준은 `READ COMMITTED`이기 때문입니다.
- `@BeforeEach` 없이 진행하면, **멀티 스레드 중 하나가 가장 먼저 `ArticleLikeCount.of(...)`로 생성 및 저장**  
	- → 그 스레드의 트랜잭션이 **우연히 가장 먼저 커밋되면**  
	- → 이후에 오는 다른 스레드들이 `findById(...)`로 정상 조회 가능  
	- → 따라서 일부 요청만 성공하게 되는 현상이 생깁니다
- commit은 트랜잭션이 종료되었을때 물리 트랜잭션 통해 발생하는건데, 이전에는 save(), flush(), clear()만 하고, 커밋되지 않아 h2에는 데이터가 없고 영속성 컨텍스트 내부에서 계속 insert 시도 발생하여 중복 발생


성공하는 테스트 (낙관적 락)
```java
@Transactional  
@ActiveProfiles("test")  
@SpringBootTest  
class ArticleLikeServiceTest {  
  
    @Autowired  
    private ArticleLikeService articleLikeService;  
  
    @Autowired  
    private ArticleLikeCountRepository articleLikeCountRepository;  
  
    @Autowired  
    private ArticleLikeRetryService articleLikeRetryService;  
  
    @Autowired  
    private PlatformTransactionManager transactionManager;  

    // 트랜잭션 탬플릿 통해 바로 커밋(h2)하도록 초기화
    @BeforeEach  
    void setUp() {  
        TransactionTemplate tx = new TransactionTemplate(transactionManager);  
        tx.executeWithoutResult(status -> {  
            articleLikeCountRepository.save(ArticleLikeCount.of(1L, 0L));  
        });    
	}  
	
    @Test  
    void optimisticLockPerformanceWithRetry() throws InterruptedException {  
        int threadCount = 1000;  
        ExecutorService executorService = Executors.newFixedThreadPool(10);  
  
        List<Callable<Void>> tasks = new ArrayList<>();  
        for (int i = 2; i <= threadCount + 1; i++) {  
            long userId = i;  
            tasks.add(() -> {  
                articleLikeService.likeOptimisticLock(1L, userId);  
                return null;  
            });        
		}  
        long start = System.currentTimeMillis();  
  
        List<Future<Void>> futures = executorService.invokeAll(tasks);  
  
        AtomicInteger success = new AtomicInteger();  
        AtomicInteger failure = new AtomicInteger();  
  
        for (Future<Void> f : futures) {  
            try {  
                f.get();  
                success.incrementAndGet();  
            } catch (ExecutionException ex) {  
                failure.incrementAndGet();  
            }        
		}  
		
        long end = System.currentTimeMillis();  
  
        System.out.println("success : " + success.get() + ", failure : " + failure.get());  
        System.out.println((end - start) + "ms");  
  
        ArticleLikeCount articleLikeCount = articleLikeCountRepository.findById(1L).get();  
        System.out.println(articleLikeCount.getLikeCount());  
    }}
```

[📚 Hibernate @Version](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#locking-optimistic)

**TODO.**
- `userId`를 컨트롤러에서 path로 받는데 스프링 시큐리티나 그런걸로 주입받아 사용하는게 좋을듯함 
	- 정책 : 로그인한 사용자에 한해서 `좋아요/좋아요 해제` 가능

**비관적 락 (경량형, 업데이트 선 시도 전략)**
- update
- 초기 데이터를 넣지 않고 테스트를 할 경우 아래와 같은 에러가 발생하고 테스트는 실패💩
	- update할 articleLikeCount가 없어서 insert에 대한 멀티 스레드의 경합이 발생한게 원인으로 추측

```text
2025-07-04T12:01:48.993+09:00 ERROR 24181 --- [article-like-service] [pool-2-thread-2] o.h.engine.jdbc.spi.SqlExceptionHelper   : Unique index or primary key violation: "PRIMARY KEY ON PUBLIC.ARTICLE_LIKE_COUNT(ARTICLE_ID) ( /* key:1 */ CAST(1 AS BIGINT), CAST(1 AS BIGINT))"; SQL statement:
insert into article_like_count (like_count,article_id) values (?,?) [23505-232]
```

**🤖 주의사항** 
- **멀티 쓰레드에서 동시에 `increase()`가 실패할 수 있음**
    - 여러 쓰레드가 동시에 `increase()` 시도 → 모두 0 return → 여러 insert 시도 → **PK 충돌**
    - 이때는 **`insert or ignore` 혹은 `insert on duplicate key update`** 같은 **DB 종속적인 방법**이 필요하거나, `try-catch`로 방어해야 함
- **정상적인 케이스에서도 insert 분기 발생**
    - 최초 1회는 insert, 그 외는 update지만, 경쟁이 있을 땐 **insert 시도 여러 번 발생 가능**
- **실제로는 완전한 비관적 락은 아님**
    - `select for update`처럼 명시적 락은 없음 → 트랜잭션 컨트롤이 더 어려울 수 있음

```java
 // @Transactional을 지워야 정상 동작함.. commit이 안되서 save에서 경합 발생
@ActiveProfiles("test")  
@SpringBootTest 
class ArticleLikeServiceTest {  
  
    @Autowired  
    private ArticleLikeService articleLikeService;  
  
    @Autowired  
    private ArticleLikeCountRepository articleLikeCountRepository;  
  
    @Autowired  
    private PlatformTransactionManager transactionManager;  
  
    @BeforeEach  
    void setUp() {  
        TransactionTemplate tx = new TransactionTemplate(transactionManager);  
        tx.executeWithoutResult(status -> {  
            articleLikeCountRepository.save(ArticleLikeCount.of(1L, 0L));  
        });    
	}  
	
    @Test  
    void like() throws InterruptedException {  
        int threadCount = 1000;  
        ExecutorService executorService = Executors.newFixedThreadPool(10);  
        CountDownLatch latch = new CountDownLatch(threadCount);  
  
        long start = System.currentTimeMillis();  
        for (int i = 2; i <= threadCount + 1; i++) {  
            long userId = i;  
            executorService.submit(() -> {  
                articleLikeService.like(1L, userId);  
                latch.countDown();  
            });        }  
        latch.await();  
  
        long end = System.currentTimeMillis();  
        System.out.println((end - start) + "ms");  
  
        ArticleLikeCount articleLikeCount = articleLikeCountRepository.findById(1L).get();  
        assertThat(articleLikeCount.getLikeCount()).isEqualTo(threadCount);  
    }
}
```


**낙관적 락(retry x)**
- @Version을 사용하니 controller 통합 테스트에서 @Transactional 없으면 테스트 실패함 
- ArticleLikeCount에 **@Version private Long version** 만 선언하고 팩토리에서 초기화 하지 않기 !!
	- 💩 default null로 version 필드 초기화되고, null이 아니면 update 발생해 테스트시 충돌로 인해 비정상 종료됨

```java
@ActiveProfiles("test")  
@AutoConfigureMockMvc  
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)  
@Transactional  // 이거 추가 안해주면 Primary key 충돌 나서 unlike 제외하고 모두 실패
class ArticleLikeControllerTest {  
    @Autowired  
    private MockMvc mockMvc;  
  
    @Autowired  
    private ObjectMapper objectMapper;  
  
    @Autowired  
    private PlatformTransactionManager transactionManager;  
  
    @Autowired  
    private ArticleLikeRepository articleLikeRepository;  
  
    @Autowired  
    private ArticleLikeCountRepository articleLikeCountRepository;  
  
    @BeforeEach  
    void setUp() {  
        TransactionTemplate tx = new TransactionTemplate(transactionManager);  
        tx.executeWithoutResult(status -> {  
            articleLikeRepository.save(ArticleLike.of(1L, 1L, 1L));  
            articleLikeCountRepository.save(ArticleLikeCount.of(1L, 1L));  
        });    }  
    @Test  
    void read() throws Exception {  
        mockMvc.perform(get("/v1/article-like/article/{articleId}/user/{userId}", 1L, 1L))  
                .andExpectAll(status().isOk(),  
                        jsonPath("$.articleId").value(1L),  
                        jsonPath("$.userId").value(1L)  
                );    }  
    @Test  
    void like() throws Exception {  
        mockMvc.perform(post("/v1/article-like/article/{articleId}/user/{userId}", 1L, 2L))  
                .andExpect(status().isCreated());  
  
        ArticleLikeCount articleLikeCount = articleLikeCountRepository.findById(1L).get();  
        assertThat(articleLikeCount.getLikeCount()).isEqualTo(2L);  
    }  
    
    @Test  
    void unlike() throws Exception {  
        mockMvc.perform(delete("/v1/article-like/article/{articleId}/user/{userId}", 1L, 1L))  
                .andExpect(status().isNoContent());  
  
        ArticleLikeCount articleLikeCount = articleLikeCountRepository.findById(1L).get();  
        assertThat(articleLikeCount.getLikeCount()).isEqualTo(0L);  
    }  
    
    @Test  
    void count() throws Exception {  
        MvcResult mvcResult = mockMvc.perform(get("/v1/article-like/article/{articleId}/count", 1L))  
                .andExpect(status().isOk())  
                .andReturn();  
  
        MockHttpServletResponse response = mvcResult.getResponse();  
        String contentAsString = response.getContentAsString();  
        Long result = objectMapper.readValue(contentAsString, Long.class);  
  
        assertThat(result).isEqualTo(1L);  
    }}
```

**서비스 테스트**
- 재시도가 없으므로 낙관적 락이 실패하는 경우가 생겨 매번 결과 예측 불가 
- success : 201, failure : 799, 실행 시간 : 447ms, 카운트 : 201
```java
@ActiveProfiles("test")  
@SpringBootTest  
class ArticleLikeServiceTest {  
  
    @Autowired  
    private ArticleLikeService articleLikeService;  
  
    @Autowired  
    private ArticleLikeCountRepository articleLikeCountRepository;  
  
    @Autowired  
    private PlatformTransactionManager transactionManager;  
  
    @BeforeEach  
    void setUp() {  
        TransactionTemplate tx = new TransactionTemplate(transactionManager);  
        tx.executeWithoutResult(status -> {  
            articleLikeCountRepository.save(ArticleLikeCount.of(1L, 0L));  
        });    
	}  
	
    @Test  
    void likeWithoutRetry() throws InterruptedException {  
        int threadCount = 1000;  
        ExecutorService executorService = Executors.newFixedThreadPool(10);  
  
        List<Callable<Void>> tasks = new ArrayList<>();  
        for (int i = 2; i <= threadCount + 1; i++) {  
            long userId = i;  
            tasks.add(() -> {  
                articleLikeService.like(1L, userId);  
                return null;  
            });        }  
        long start = System.currentTimeMillis();  
  
        List<Future<Void>> futures = executorService.invokeAll(tasks);  
  
        AtomicInteger success = new AtomicInteger();  
        AtomicInteger failure = new AtomicInteger();  
  
        for (Future<Void> f : futures) {  
            try {  
                f.get();  
                success.incrementAndGet();  
            } catch (ExecutionException e) {  
                failure.incrementAndGet();  
            }        }  
        long end = System.currentTimeMillis();  
  
        System.out.println("success : " + success.get() + ", failure : " + failure.get());  
        System.out.println((end - start) + "ms");  
  
        ArticleLikeCount articleLikeCount = articleLikeCountRepository.findById(1L).get();  
        System.out.println(articleLikeCount.getLikeCount());  
    }
}
```

**낙관적 락2(재시도 o)**
- build.gradle에 추가
```text
implementation 'org.springframework.retry:spring-retry'
implementation 'org.springframework.boot:spring-boot-starter-aop'
```

- `@EnableRetry`를 LikeApplication 클래스에 추가
- 어노테이션 기반으로 서비스 로직에 선언 

```java
@Service  
@RequiredArgsConstructor  
@Transactional  
public class ArticleLikeService {  
    private final Snowflake snowflake = new Snowflake();  
  
    private final ArticleLikeRepository articleLikeRepository;  
    private final ArticleLikeCountRepository articleLikeCountRepository;  
  
    @Transactional(readOnly = true)  
    public ArticleLikeResponse read(Long articleId, Long userId) {  
        return articleLikeRepository.findByArticleIdAndUserId(articleId, userId)  
                .map(ArticleLikeResponse::from)  
                .orElseThrow();  
    }  
    @Retryable(  
            retryFor = {ObjectOptimisticLockingFailureException.class, StaleObjectStateException.class},  
            maxAttempts = 3,  
            backoff = @Backoff(delay = 100) // 100ms 간격 재시도  
    )  
    public void like(Long articleId, Long userId) {  
        articleLikeRepository.save(ArticleLike.of(snowflake.nextId(), articleId, userId));  
  
        ArticleLikeCount articleLikeCount = articleLikeCountRepository.findById(articleId)  
                .orElseGet(() -> ArticleLikeCount.of(articleId, 0L));  
  
        articleLikeCount.increase();  
        articleLikeCountRepository.save(articleLikeCount);  
    }  
    @Retryable(  
            retryFor = {ObjectOptimisticLockingFailureException.class},  
            maxAttempts = 3,  
            backoff = @Backoff(delay = 100) // 100ms 간격 재시도  
    )  
    public void unlike(Long articleId, Long userId) {  
        articleLikeRepository.findByArticleIdAndUserId(articleId, userId)  
                .ifPresent(articleLike -> {  
                    articleLikeRepository.delete(articleLike);  
                    ArticleLikeCount articleLikeCount = articleLikeCountRepository.findById(articleId).orElseThrow();  
                    articleLikeCount.decrease();  
                });    }  
    @Transactional(readOnly = true)  
    public Long count(Long articleId) {  
        return articleLikeCountRepository.findById(articleId)  
                .map(ArticleLikeCount::getLikeCount)  
                .orElse(0L);  
    }}
```

- 테스트 코드는 그대로, 마찬가지로 낙관적 락이 실패하는 경우가 있음
	- success : 963, failure : 37, 실행시간: 1222ms, 카운터 : 963
- 대규모 서비스가 아니라면 **비관적 락(select for update..)** 만으로 충분하다고 판단됨

✅ 커밋 이력 정리
히스토리를 아래와 같이 기록, revert로 이력을 남기고 최종적으로 비관적 락(select for update)르 사용
- 낙관적 락 + retry
- 낙관적 락 + retry x
- 비관적 락 (select for update)
- 비관적 락 (update)

```shell
$ git log --oneline

23ddfb2 (HEAD -> article-like) refactor: optimistic lock + retry
7852c11 add spring-aop, retry dependency
4733bc7 refactor: @Version 낙관적 락 적용 (재시도 x)
21cbb87 refactor: select..for update 비관적 락 사용
e1666f6 create application-test.yml
5c24c87 create ArticleLikeController

$ git revert 23ddfb2 7852c11 4733bc7

$ git log --oneline
ec80657 (HEAD -> article-like) Revert "refactor: @Version 낙관적 락 적용 (재시도 x)"
aa6c31e Revert "add spring-aop, retry dependency"
811758c Revert "refactor: optimistic lock + retry"
23ddfb2 refactor: optimistic lock + retry
7852c11 add spring-aop, retry dependency
4733bc7 refactor: @Version 낙관적 락 적용 (재시도 x)
21cbb87 refactor: select..for update 비관적 락 사용
e1666f6 create application-test.yml
5c24c87 create ArticleLikeController
```

---
## 5. 조회수

<img src="./image/article_view_count_erd.png"/>

**절차** 
- redis에 게시글 조회수를 카운팅한다 
- 게시글 조회수가 `1000`(=BATCH_SIZE) 단위로 올라가면 DB에 백업한다 
- 이때 조회수 어뷰징을 위해 redis 분산락을 사용한다 (ttl = 10분)


<img src="./image/article_view_flow.png"/>


**고려할 부분. 서버 재시작시**
- 조회수 데이터가 Redis 캐시에 없어서 데이터 부정합 발생 
	- 지연 로딩이나 서버 시작 시 일괄 초기화


**추가**
- 임베디드 레디스 사용 
	- `testImplementation 'com.github.codemonstur:embedded-redis:1.0.0'`
	- 2024.03.26까지 release 발행
	- https://github.com/codemonstur/embedded-redis
- redis connection factory로 lettuce 사용
	- 기억보단 기록을 기술 블로그 참고
- redis container가 종료되어 있는데 실행되는 이유
- redis 서버 재시작시 조회 캐시가 없는데 이 경우는 어떻게? (by Chat-GPT)
	- 지연(Lazy) 초기화 전략
	- 서버 시작시 일괄 초기화 (`@EventListener(ApplicationReadyEvent.class)`)
		- top N 게시글에 대해서만 
	- Redis AOF/RDB 설정으로 복원 
		- AOF : Append Only File
		- RDB : Snapshotting

참고 
- [조회수를 rdb에만 저장하고 있는 서비스에 redis 도입 질문](https://www.inflearn.com/community/questions/1550681/%EC%A1%B0%ED%9A%8C%EC%88%98%EB%A5%BC-rdb%EC%97%90%EB%A7%8C-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B3%A0-%EC%9E%88%EB%8A%94-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%97%90%EC%84%9C-redis-%EB%8F%84%EC%9E%85-%EA%B4%80%EB%A0%A8%ED%95%B4%EC%84%9C-%EC%A7%88%EB%AC%B8%EC%9E%85%EB%8B%88%EB%8B%A4)
- [조회수 redis 장애시 fallback 관련해서 질문](https://www.inflearn.com/community/questions/1620809/%EC%A1%B0%ED%9A%8C%EC%88%98-redis-%EC%9E%A5%EC%95%A0%EC%8B%9C-fallback-%EA%B4%80%EB%A0%A8%ED%95%B4%EC%84%9C-%EC%A7%88%EB%AC%B8)


---

## 6. 인기글 

> kafka 활용
- consumer는 `hot-article`에 위치
	- producer는 각 서비스 모듈마다 위치해야 하는건가? 
		- `:common:outbox-message-relay` 에서 kafka 메시지 발송 책임
		- article, view, like, comment 모듈에서는 해당 모듈 주입받아 메시지 전달하여 발송

### kafka 셋팅 (by docker)

```shell
> docker run -d --name board-kafka -p 9092:9092 apache/kafka:3.8.0
> docker exec --workdir /opt/kafka/bin/ -it board-kafka sh 

# 토픽 생성 (article, comment, like, view)

$ ./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic board-article --replication-factor 1 --partitions 3
Created topic board-article.
$ ./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic board-comment --replication-factor 1 --partitions 3
$ ./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic board-like --replication-factor 1 --partitions 3
$ ./kafka-topics.sh --bootstrap-server localhost:9092 --create --topic board-view --replication-factor 1 --partitions 3

## 기타
# 토픽 확인 
$ ./kafka-topics.sh --bootstrap-server localhost:9092 --list

# 특정 토픽 상세 
$ ./kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic board-article

# 소비자 그룹 확인
$ ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# 특정 소비자 그룹의 오프셋 확인 
$ ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-consumer-group


```
**250725**
- docker-compose.yml에 `bitnami/kafka:3.8` 설정 추가
- volumn을 연결했는데 `create-topics.sh`이 실행은 되지 않음 ❌
	- 직접 컨테이너 접속해 실행해야 함 
- docker-compose에 health check 속성이있었는데, 그걸 하니 kafka-init 컨테이너 하나 더 올라가고 원하는데로 토픽 4개 생성 되지도 않음 
```shell
$ docker exec -uroot -it board-kafka /bin/bash

$ cd /bitnami/kafka-init-scripts/
$ ./create-topics.sh

$ cd /opt/bitnami/kafka/bin/
$ ./kafka-topics.sh --bootstrap-server localhost:9092 --list
// 토픽 4개 출력
```

### outbox 테이블 설계 
- article, view, like, comment 데이터베이스에 각각 outbox 테이블 생성해야 함
	- 자기 메시지만 관리함
- 각 서버가 scale-out하는 경우 shard 분배를 통해 메시지 중복 처리 및 충돌 방지 
	- redis 통해 운영 서버 관리 및 AssignedShard 클래스 통해 샤드 분배 (**아래 flow 이미지 참고**)

```sql
-- 인기글  
-- article, article_view, article_like, comment DB에 각각 테이블과 인덱스 생성해준다  
create table outbox (  
    outbox_id bigint not null primary key,  
    shard_key bigint not null,  
    event_type varchar(100) not null,  
    payload varchar(5000) not null,  
    created_at datetime not null  
);  
  
-- 생성 10초 이후 조건 조회를 위한 인덱스  
create index idx_shard_key_created_at on outbox(shard_key asc, created_at asc);

```

<img src="./image/outbox-erd.png"/>


### 인기글 consumer 설계 
- kafka 메시지를 polling해서 처리
- redis sorted set 에 데이터 저장 
	- 게시글 생성/삭제
	- 좋아요, 댓글수, 조회수에 대한 가중치 계산 및 인기글 목록


<img src="./image/hot-article-event.png"/>

> 책임 연쇄 패턴은 또 다른 내용이네, 체이닝 방식으로 핸들러를 연속해서 처리 
> https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-Chain-Of-Responsibility-%ED%8C%A8%ED%84%B4-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0#%EC%B1%85%EC%9E%84_%EC%97%B0%EC%87%84_%ED%8C%A8%ED%84%B4_%EA%B5%AC%EC%A1%B0


### 전체 flow

**Producer**
- article, comment, article-like, article-view 모듈에서 :common:outbox-message-relay 의존
- request 요청시 이벤트발송에 대한 책임을 OutboxPublisher에서 위임
- 정상적인 경우 
	- outbox 테이블에 메시지 저장 
	- kafka 메시지 발송 
	- (1초후) outbox 메시지 삭제
- kafka 서버 다운 된 경우 
	- outbox 테이블에 메시지가 저장된 상태
	- 스케쥴러 통해 운영 중인 프로세스 확인 
		- 이때 redis에 각 모듈 그룹 단위로 해서 프로세스 정보를 관리함 
		- article 1, 2 서버가 올라가 있으면 각 서버에 스케쥴러가 실행됨 
		- redis 조회해서 각 서버가 담당할 shard 계산 
		- outbox 테이블에서 미처리된 메시지(shard) 가져와 재발송 후 삭제

**Consumer**
- `hot-article` 서버에서는
	- 이벤트를 구독하여 인기글 관련 데이터를 Redis에 저장한다
	- 금일 생성된 게시글의 데이터를 Redis에 저장/삭제/조회한다 (`article`)
	- 금일 처리된 인기글 정보는 다음날 인기글 목록으로 제공된다
		- 20250715 게시글을 가공한 후 20250716에는 하루전 날 인기글 정보 제공
	- 인기글 점수를 계산하기 위해 Redis에 데이터를 저장한다
		- comment, article-like, article-view 모듈에서 이벤트 발송
		- 좋아요 수, 댓글 수, 조회 수에 가중치를 두어 계산
		- Sorted Set에 인기글을 limit 개수만큼 유지한다 
	- 인기글 조회시 redis에서 인기글의 articleId 목록을 조회하고 ArticleClient 통해서 조회 후 응답
- hot-article은 comment, article-like, article-view, article 모듈을 모른다.
	- 단지 Kafka에 의존하고 있고, 메시지를 구독하여 인기글 비즈니스를 처리하는 역할, 책임을 가진다
	- 반대편 모듈도 마찬가지로 hot-article을 모르고, 인기글 비즈니스에도 관심없고 메시지를 발송할 뿐이다. 
	- 이를 통해 서버(모듈)간 결합도를 낮추고, 확장성 가지게 됨

<img src="./image/hot-article-flow.png"/>

- 참고로 인기글에서는 `ArticleUpdatedEventPayload` 사용하지 않음 
	- `HotArticleService`에서도 핸들러 조회시 ArticleUpdatedEventHandler 빈이 없기 때문에 null 반환되어 안전하게 return 처리됨 ➡️ `article-read` 모듈에서 ARTICLE_UPDATED 이벤트 활용함

**트러블슈팅 기록(7/14)**
- 1. 전체 서버 실행 후 인기 게시글 e2e 테스트 실행 
	- 테스트 파일은 `hot-article`모듈의 api 테스트 패키지에 있음 
	- article / comment / view / like 서버에 직접 데이터 생성 후 인기글 확인 
	- 이때 RestClient 사용해 각 서버에 생성 요청을 했는데 comment / view / like 서버에 전송되지 않음 
		- RestClient 사용시 `retrieve()` 에서 끝내면 안되고 뒤에 하나더 붙여야 함  (**해결✨**)
		- RestClient가 WebClient를 wrapper하고 있어 비동기로 동작한다고 함
		- 자세한건 나중에 더 알아보기
- 2. article / comment / view / like 모듈에 outbox-message-relay 모듈 추가 후 테스트 전체 실패
	- redis, kafka에 대한 접속 정보가 application-test.yml에 없었음 (추가하여 해결)
	- 그리고 OutboxPublisher나 kafka producer는 article / comment / view / like 모듈의 관심사가 아니라서 @MocktioBean 처리나 config exclude 설정하여 **해결✨**

설정이 먹히는 이유가 outbox-message-relay 모듈에 META-INF/spring 경로에 자동구성 설정 정보를 해두었기 때문에 exclude 정상 동작함
```text
spring:
	autoconfigure:
		exclude: com.example.outboxmessagerelay.MessageRelayConfig
```

- 3. HotArticleService에서 인기글 비즈니스 로직이 동작하지 않는 이슈 
	- redis에 인기글 조회해도 데이터가 없음 
	- 제네릭 타입이 맞지 않아 컨테이너 초기화시 빈 주입이 되지 않은 것으로 확인
	- before : `private final List<EventHandler<EventPayload>> eventHandlers;`
	- after : `private final List<EventHandler> eventHandlers` (**해결**✨)


---

## 7. 조회 최적화 (CQRS, Request Collapsing)

**요약**
- CQRS 분리
	- `article-read` 모듈에서는 kafka 의존 
		- 다른 모듈에서 이벤트 전송하면 이를 구독하여 Redis에 데이터를 저장 
		- 최대 1000개 최신글을 Redis 캐싱하고 없는 경우 직접 DB 조회하여 반환
	- 게시글 단건 조회, 게시글 목록 조회(페이징, 무한 스크롤) 기능 구현
- 게시글 조회수 캐싱 
	- 변경이 빨리 일어나고 , ttl이 짧다보니 멀티 스레드 환경에서 RestClient 요청이 여러번 발생하게 됨 
	- 한번만 요청하고 갱신할 수 있도록 논리/물리 ttl과 분산락 활용  (`논리 ttl < 물리 ttl`)
	- Request Collapsing 기법 적용

<img src="./image/article-read-flow.png"/>

**페이징 목록 조회**
- `readAllArticleIds(boardId, page, pageSize)` 
	- `articleIdListRepository` 통해 먼저 Redis에 조회하고, 없으면 `articleClient` 통해 목록 요청 
- `readAll(List<Long> articleIds)` 
	- `articleQueryModelRepository` 통해 Redis 조회하고, articleId에 해당하는 ariticleQueryModel이 없으면 fetch(..) 요청 
	- `fetch(..)` 의 경우 articleClient 통해 요청 후 ArticleQueryModel을 Redis 저장 후 반환
- `articleCount(boardId)`
	- 게시판에 게시글 수를 Redis에서 조회하고, 없으면 실서버 요청 후 Redis 캐싱

```java
public class ArticleReadService {

	public ArticleReadPageResponse readAll(Long boardId, Long page, Long pageSize) {  
    return ArticleReadPageResponse.of(  
            readAll(readAllArticleIds(boardId, page, pageSize)),  
            articleCount(boardId)  
    );}  
  
	private List<ArticleReadResponse> readAll(List<Long> articleIds) {  
	    Map<Long, ArticleQueryModel> articleQueryModelMap = articleQueryModelRepository.readAll(articleIds);  
	    return articleIds.stream()  
	            .map(articleId -> articleQueryModelMap.containsKey(articleId) ? articleQueryModelMap.get(articleId) : fetch(articleId).orElse(null))  
	            .filter(Objects::nonNull)  
	            .map(articleQueryModel -> ArticleReadResponse.of(articleQueryModel, viewClient.count(articleQueryModel.getArticleId())))  
	            .toList();  
	}  
	  
	private List<Long> readAllArticleIds(Long boardId, Long page, Long pageSize) {  
	    List<Long> articleIds = articleIdListRepository.readAll(boardId, (page - 1) * pageSize, pageSize);  
	    if(pageSize == articleIds.size()) {  
	        log.info("[ArticleReadService.readAllArticleIds] return redis data");  
	        return articleIds;  
	    }  
	    log.info("[ArticleReadService.readAllArticleIds] return origin data");  
	    return articleClient.readAll(boardId, page, pageSize)  
	            .getArticles()  
	            .stream()  
	            .map(ArticleClient.ArticleResponse::getArticleId)  
	            .toList();  
	}  
	  
	private Long articleCount(Long boardId) {  
	    Long result = boardArticleCountRepository.read(boardId);  
	    if(result != null) {  
	        log.info("[ArticleReadService.articleCount] return redis data");  
	        return result;  
	    }  
	    log.info("[ArticleReadService.articleCount] return origin data");  
	    Long articleCount = articleClient.count(boardId);  
	    boardArticleCountRepository.save(boardId, articleCount);  
	    return articleCount;  
	}

}
```
- 현실적으로 서비스를 mock 테스트하기 힘들어 **통합 테스트**로 `*Client` 만 @MockitoBean 처리하여 테스트함 (250719)


### ViewCount 캐싱 관련 
- article-read에서 게시글 조회시 ViewCount를 view 서버에 조회하는 형태로 구현
	- view count는 실시간 정확도가 중요하지 않으므로 빡시게 할 필요는 없음
- `ViewClient.count(..)` 호출시 Redis에 캐싱을 하도록 했으나, 멀티 스레드 요청시 view 서버와 redis의 부하가 증가하게 됨 (여러 번 RestClient 요청이 가니)
- `Request Collapsing` 기법을 적용하여 멀티스레드 환경에서 RestClient 요청이 여러번 가지 않도록 함 
	- 논리 TTL < 물리 TTL 
	- 논리 TTL이 만료되었으면 RestClient 요청하고, 그 사이 다른 스레드는 물리 TTL이 살아 있으니 캐싱 데이터를 반환한다.

> 강의에서는 Redis Config를 정의했지만, 나는 하지 않음..
> RedisCacheManager Config설정 후 @Cacheable 달아서 Redis에 짧게 캐싱 되도록 하는 건데 
> Requet Collapsing 전 단계라서 안함


**spring-aop 의존성 추가**
```text
implementation 'org.springframework.boot:spring-boot-starter-aop'
```

`@OptimizedCacheable` 애노테이션 선언
```java
@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.METHOD)  
public @interface OptimizedCacheable {  
    String type();  
    long ttlSeconds();  
}
```


ViewClient에 애노테이션 추가
```java
@Slf4j  
@Component  
public class ViewClient {  
    private final RestClient restClient;  
  
    public ViewClient(RestClient.Builder builder, @Value("${endpoints.board-view-service.url}") String baseUrl) {  
        log.info("[ViewClient] baseUrl = {}", baseUrl);  
        this.restClient = builder.baseUrl(baseUrl).build();  
    }  

	// 부착
    @OptimizedCacheable(type = "articleViewCount", ttlSeconds = 1)  
    public Long count(Long articleId) {  
        try {  
            return restClient.get()  
                    .uri("/v1/article_view/article/{articleId}/count", articleId)  
                    .retrieve()  
                    .body(Long.class);  
        } catch (Exception e) {  
            log.error("[ViewClient.count] articleId = {}", articleId, e);  
            return 0L;  
        }    
	}
}
```


**OptimizedCacheAspect 모듈 선언**
```java
@Aspect  
@Component  
@RequiredArgsConstructor  
public class OptimizedCacheAspect {  
    private final OptimizedCacheManager optimizedCacheManager;  
  
    @Around("@annotation(OptimizedCacheable)")  
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {  
        OptimizedCacheable cacheable = findAnnotation(joinPoint);  
        return optimizedCacheManager.process(  
            cacheable.type(),  // 애노테이션 정보
            cacheable.ttlSeconds(),  // 애노테이션 정보
            joinPoint.getArgs(),  // ViewClint count(..) 파라미터
            findReturnType(joinPoint),  // ViewClint count(..)의 리턴 타입
            joinPoint::proceed   // 원본 메소드 호출 ViewClint count(..)
        );  
    }  
    private OptimizedCacheable findAnnotation(ProceedingJoinPoint joinPoint) {  
        Signature signature = joinPoint.getSignature();  
        MethodSignature methodSignature = (MethodSignature) signature;  
        return methodSignature.getMethod().getAnnotation(OptimizedCacheable.class);  
    }  
    
    private Class<?> findReturnType(ProceedingJoinPoint joinPoint) {  
        Signature signature = joinPoint.getSignature();  
        MethodSignature methodSignature = (MethodSignature) signature;  
        return methodSignature.getReturnType();  
    }}
```
- `joinPoint::proceed` 에 파라미터가 전달되어 원본 메서드 실행된다. 
	- ViewClient.count(1L)
	- () -> joinPoint.proceed(joinPoint.getArgs())


**🔍 왜 args를 명시적으로 넘기지 않아도 되는가?**
`ProceedingJoinPoint.proceed()`는 다음 두 가지 오버로드를 가집니다:

```java
Object proceed() throws Throwable              // 현재 args 그대로 호출 
Object proceed(Object[] args) throws Throwable // 새로운 args로 호출
```
- `proceed()`는 원래 메서드의 파라미터를 그대로 사용합니다.
- `proceed(Object[] args)`는 새로운 파라미터를 사용해 메서드를 호출합니다.
	- 임의로 aop에서 가공해서 넘겨줄 수 있다는거다

따라서 `joinPoint::proceed`는 단순히 "현재의 파라미터 그대로 메서드를 실행하겠다"는 의미고, 프록시된 실제 메서드(`ViewClient.count(Long articleId)`)에 인자들이 알아서 전달됩니다.



```java
@Slf4j  
@Component  
@RequiredArgsConstructor  
public class OptimizedCacheManager {  
    private static final String DELIMITER = "::";  
  
    private final StringRedisTemplate redisTemplate;  
    private final OptimizedCacheLockProvider optimizedCacheLockProvider;  
  
    public Object process(String type, long ttlSeconds, Object[] args, Class<?> returnType,  
                          OptimizedCacheOriginDataSupplier<?> supplier) throws Throwable {  
        String key = generateKey(type, args);  
  
        String cachedData = redisTemplate.opsForValue().get(key);  
        if(cachedData == null) { // 만료되거나 최초이거나  
            log.info("no cached data");  
            return refresh(supplier, key, ttlSeconds); // RestClient 호출  
        }  
  
        OptimizedCache optimizedCache = DataSerializer.deserialize(cachedData, OptimizedCache.class);  
        if(optimizedCache == null) {  
            log.info("no optimizedCache then refresh");  
            return refresh(supplier, key, ttlSeconds);  
        }  
        // logical ttl not expired  
        if(!optimizedCache.isExpiredData()) {  
            log.info("logical ttl not expired then cached data");  
            return optimizedCache.parseData(returnType); // then return cached data  
        }  
  
        // logical ttl expired & do not have distributed lock  
        if(!optimizedCacheLockProvider.lock(key)) {  
            log.info("logical ttl expired & do not have distributed lock then cached data");  
            return optimizedCache.parseData(returnType); // then return cached data  
        }  
  
        log.info("lock acquired fail");  
        try {  
            return refresh(supplier, key, ttlSeconds); // refresh cache data  
        } finally {  
            optimizedCacheLockProvider.unlock(key);  
        }    
	}  
        
    private Object refresh(OptimizedCacheOriginDataSupplier<?> supplier, String key, long ttlSeconds) throws Throwable {  
        Object originData = supplier.get();  
  
        OptimizedCacheTTL optimizedCacheTTL = OptimizedCacheTTL.of(ttlSeconds);  
        OptimizedCache optimizedCache = OptimizedCache.of(originData, optimizedCacheTTL.getLogicalTTl());  
  
        redisTemplate.opsForValue()  
                .set(key, DataSerializer.serialize(optimizedCache), optimizedCacheTTL.getPhysicalTTl());  
  
        return originData;  
    }  
    
    private String generateKey(String prefix, Object[] args) {  
        return prefix + DELIMITER +  
                Arrays.stream(args)  
                        .map(String::valueOf)  
                        .collect(joining(DELIMITER));  
    }
}
```
- **절차** 
	- 1. 캐시 데이터가 없으면 
		- `refresh` 메서드 실행 
		- `ViewClient.count(..)` 호출 + 캐싱 후 반환
	- 2. OptimizedCache == null 인 경우
		- 마찬가지로 `refresh` 메서드 실행 
		- `ViewClient.count(..)` 호출 + 캐싱 후 반환
			- 혹여나 비즈니스 로직 실수로 인해 누락할 수 있으니 null 체크 하는듯
	- 3. `!optimizedCache.isExpiredData()`
		- **논리적 ttl이 만료되지 않은 경우** 캐싱 데이터에서 조회수를 반환
	- 4. `!optimizedCacheLockProvider.lock(key)`
		- 논리적 ttl이 만료되었고, 락을 획득하지 못한 경우
		- 아직 물리적 ttl이 살아있다 판단하여 캐싱 데이터에서 조회수를 반환 
	- 5. 분산락을 획득한 경우 
		- `refresh` 메서드 실행하여 캐시 갱신
		- lock을 반환
- 이때 **논리적 TTL < 물리적 TTL** 보장
- `OptimizedCacheLockProvider`에서 락 획득/해제 관리

> 이를 통해 멀티스레드 환경에서 ViewClient 여러 번 요청하는 것을 방지하고 캐싱을 반환하는 동안 락을 획득한 스레드만 원본 데이터 요청해 재갱신


<img src="./image/request-collapsing-sequence.png"/>
- 멀티 스레드 환경에서 logical TTL이 만료되었을 때 
	- 첫번째 스레드가 락을 획득해 refresh 한다 
	- 두번째 스레드는 락을 획득 실패하고, physical TTL은 살아있는 상태라서 캐시 데이터를 반환한다
- 이를 통해 멀티스레드 환경에서 RestClient 요청을 줄일 수 있다

---
