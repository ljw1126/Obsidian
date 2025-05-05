**Reference**
- [올리브영 WebClient](https://oliveyoung.tech/2022-11-10/oliveyoung-discovery-premium-webclient/)



**트러블슈팅 (미해결)**
- Mock Server를 만들어서 `/posts?id={{id}}`에 대한 예제 응답 생성
	- [공식 문서](https://learning.postman.com/docs/design-apis/mock-apis/mock-with-examples/)
- WebClient로 https 요청 -> Mock Server 정상 응답 로그 확인 -> Webflux Application에서 **404 Not Found** 응답 받음 -> PostService에 `onErrorResume()` 처리되고 종료 되버림
- WebClient에 https 에 대한 설정이나 타임아웃 설정이 필요한가 싶었지만, 결과는 동일

```java
@Configuration  
public class WebClientConfig {  
  
    @Bean  
    public WebClient webClient() throws SSLException {  
        SslContext context = SslContextBuilder.forClient().trustManager(InsecureTrustManagerFactory.INSTANCE).build();  
        HttpClient httpClient = HttpClient.create().secure(provider -> provider.sslContext(context)).responseTimeout(Duration.ofSeconds(10)).option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000);  
  
        return WebClient.builder()  
                .clientConnector(new ReactorClientHttpConnector(httpClient))  
                .build();  
    }}
```

**실패로그**
```text
2025-05-04T15:56:38.048+09:00 DEBUG 17561 --- [helloWebflux] [or-http-epoll-2] r.netty.http.client.HttpClientConnect    : [efcea33a-1, L:/39.122.15.136:43986 - R:0e55d1aa-c6e7-4d5e-af1f-37ca0d1152b8.mock.pstmn.io/52.207.14.143:443] Handler is being applied: {uri=https://0e55d1aa-c6e7-4d5e-af1f-37ca0d1152b8.mock.pstmn.io/posts?id=5, method=GET}
2025-05-04T15:56:38.050+09:00 DEBUG 17561 --- [helloWebflux] [or-http-epoll-2] r.n.http.client.HttpClientOperations     : [efcea33a-1, L:/39.122.15.136:43986 - R:0e55d1aa-c6e7-4d5e-af1f-37ca0d1152b8.mock.pstmn.io/52.207.14.143:443] No sendHeaders() called before complete, sending zero-length header
2025-05-04T15:56:38.370+09:00 DEBUG 17561 --- [helloWebflux] [or-http-epoll-2] r.n.http.client.HttpClientOperations     : [efcea33a-1, L:/39.122.15.136:43986 - R:0e55d1aa-c6e7-4d5e-af1f-37ca0d1152b8.mock.pstmn.io/52.207.14.143:443] Received response (auto-read:false) : RESPONSE(decodeResult: success, version: HTTP/1.1)
HTTP/1.1 404 Not Found
```


---

## R2DBC

- `io.asyncer:r2dbc-mysql` 버전 정보 확인 ([링크](https://github.com/asyncer-io/r2dbc-mysql))
- [R2DBC Getting Started](https://docs.spring.io/spring-data/relational/reference/r2dbc/getting-started.html)

의존성 추가
```text
implementation 'org.springframework.boot:spring-boot-starter-data-r2dbc'  
implementation 'io.asyncer:r2dbc-mysql:1.4.0'
```

application.yml 설정 
```text
spring:  
  application:  
    name: helloWebflux  
  r2dbc:  
    url: r2dbc:mysql://localhost:3306/testdb  
    username: tester  
    password: tester1234  
logging:  
  level:  
    root: INFO  
  
server:  
  port: 8080
```

config 생성해서 애플리케이션 실행시 연결되는지 확인 
```java
@Slf4j  
@Configuration  
@RequiredArgsConstructor  
@EnableR2dbcRepositories // 추가
@EnableR2dbcAuditing // 엔티티 날짜 관련
public class R2dbcConfig implements ApplicationListener<ApplicationReadyEvent> {  
    private final DatabaseClient databaseClient;  
  
    @Override  
    public void onApplicationEvent(ApplicationReadyEvent event) {  
        databaseClient.sql("SELECT 1").fetch().one()  
                .subscribe(  
                        success -> {  
                            log.info("Initilize r2dbc database connection.");  
                        },                        error -> {  
                            log.error("Failed to initialize r2dbc database connection.", error);  
                        }                
				);    
		}
	}
}
```

```text
2025-05-04T16:40:18.809+09:00  INFO 26368 --- [helloWebflux] [tor-tcp-epoll-2] c.e.hellowebflux.config.R2dbcConfig      : Initilize r2dbc database connection.
```


엔티티에 어노테이션 추가
```java
@Getter  
@Setter  
@AllArgsConstructor  
@Builder  
@Table("users")  
public class User {  
    @Id  
    private Long id;  
    private String name;  
    private String email;  
    @CreatedDate  
    private LocalDateTime createdAt;  
    @LastModifiedDate  
    private LocalDateTime updatedAt;  
}



@Builder  
@Table("posts")  
@Getter  
@AllArgsConstructor  
@NoArgsConstructor  
public class Post {  
  
    @Id  
    private Long id;  
  
    @Column("user_id")  
    private Long userId;  
  
    private String title;  
  
    private String content;  
  
    @Column("created_at")  
    @CreatedDate  
    private LocalDateTime createdAt;  
  
    @Column("updated_at")  
    @LastModifiedDate  
    private LocalDateTime updatedAt;  
}

```


UserR2dbcRepository 인터페이스 생성해서 UserService에서 사용하도록 리팩터링 


PostR2dbcController
```java
@RestController  
@RequiredArgsConstructor  
@RequestMapping("/users")  
public class UserController {  
  
    private final UserService userService;  
  
    @PostMapping  
    public Mono<UserResponse> create(@RequestBody UserCreateRequest request) {  
        return userService.create(request.getName(), request.getEmail())  
                .map(UserResponse::of);  
    }  
    @GetMapping  
    public Flux<UserResponse> findAll() {  
        return userService.findAll()  
                .map(UserResponse::of);  
    }  
    @GetMapping("/{id}")  
    public Mono<ResponseEntity<UserResponse>> findById(@PathVariable Long id) {  
        return userService.findById(id)  
                .map(u -> ResponseEntity.ok(UserResponse.of(u)))  
                .switchIfEmpty(Mono.just(ResponseEntity.notFound().build()));  
    }  
    @DeleteMapping("/{id}")  
    public Mono<ResponseEntity<?>>  delete(@PathVariable Long id) {  
        return userService.deleteById(id)  
                .then(Mono.just(ResponseEntity.noContent().build()));  
    }  
    @PutMapping("/{id}")  
    public Mono<ResponseEntity<UserResponse>> update(@PathVariable Long id, @RequestBody UserUpdateRequest request) {  
        return userService.update(id, request.getName(), request.getEmail())  
                .map(u -> ResponseEntity.ok(UserResponse.of(u)))  
                .switchIfEmpty(Mono.just(ResponseEntity.notFound().build()));  
    }}
```


유저가 작성한 모든 Posts를 조회하는 기능을 추가
- 이때 인터페이스의 **다중 상속**을 활용
	- 자바에서 클래스 다중 상속은 지원❌

```java
public interface PostR2dbcRepository extends ReactiveCrudRepository<Post, Long>, PostCustomR2dbcRepository {  
}

public interface PostCustomR2dbcRepository {  
    Flux<Post> findAllByUserId(Long userId);  
}
```

```java
// 구현체
@Repository  
@RequiredArgsConstructor  
public class PostCustomR2dbcRepositoryImpl implements PostCustomR2dbcRepository{  
    private final DatabaseClient databaseClient;  
  
    @Override  
	public Flux<Post> findAllByUserId(Long userId) {  
	String sql = """  
	        SELECT  p.id as pid,   
					p.user_id as userId,   
					p.title,   
					p.content,   
					p.created_at as createdAt,   
					p.updated_at as updatedAt,  
					u.id as uid,   
					u.name as name,   
					u.email as email,   
					u.created_at as uCreatedAt,   
					u.updated_at as uUpdatedAt  
	        FROM posts p LEFT JOIN users u ON p.user_id = u.id  
	        WHERE p.user_id = :userId  
	    """;  
	  
	return databaseClient  
	    .sql(sql)  
	    .bind("userId", userId)  
	    .fetch()  
	    .all()  
	    .map(  
	        row ->  
	            Post.builder()  
	                .id(((Number)row.get("pid")).longValue())  
	                .userId(((Number)row.get("userId")).longValue())  
	                .title((String) row.get("title"))  
	                .content((String) row.get("content"))  
	                .user(User.builder()  
	                        .id(((Number)row.get("uid")).longValue())  
	                        .name((String) row.get("name"))  
	                        .email((String) row.get("email"))  
	                        .createdAt(toLocalDateTime(row.get("uCreatedAt")))  
	                        .updatedAt(toLocalDateTime(row.get("uUpdatedAt")))  
	                        .build())  
	                .createdAt(toLocalDateTime(row.get("createdAt")))  
	                .updatedAt(toLocalDateTime(row.get("updatedAt")))  
	                .build())  
	        .log();  
	}  
  
	private LocalDateTime toLocalDateTime(Object time) {  
	    if (time instanceof ZonedDateTime zdt) {  
	        return zdt.toLocalDateTime();  
	    }    
	    
	    return time instanceof LocalDateTime ldt ? ldt : null;  
	}
}
```


```java
@Service  
@RequiredArgsConstructor  
public class PostR2dbcService {  
    private final PostR2dbcRepository postR2dbcRepository;

	//..
}
```

`PostCustomR2dbcRepositoryImpl` 구현체가 있고 `ReactiveCrudRepository` 구현체가 있는데 무엇이 우선 순위를 가질까? (Chat-GPT 답변 🤖)
- Spring은 `PostR2dbcRepository`의 구현체로 내부에서 `ReactiveCrudRepository`와 `PostCustomR2dbcRepositoryImpl`을 자동 조합합니다.
- **자동 조합된 프록시가 전체를 구현한 하나의 Bean으로 등록**되는 방식입니다.
- 따라서, **사용자 구현체의 이름이 `PostCustomR2dbcRepositoryImpl`인지가 핵심**입니다. (?)
	- `PostR2dbcRepository` 타입으로 주입 받는거지 이름은 `@Qualifier` 지정하던가 해야함
	- 그리고 타입이 같은 구현체가 있다면 `@Primary`로 우선 순위를 정하거나 `@Qualifier` 로 지정하거나 

**트러블슈팅** 
- `PostCustomR2dbcRepositoryImpl` 의 이름이 `PostCustom%2dbcRepositoryImpl`와 같이 이상하게 되어 있어서 의존성 주입이 런타임에 되지 않았음
	- 규칙이 `{인터페이스}Impl` 이어야 하는 듯함?
- `PostCustomR2dbcRepositoryImpl`에서 Long 캐스팅이 안되었음 
	- `(Long) row.get("id)`하면 Integer를 Long으로 ClassCaseException 발생
	- ((Number)row.get("pid")).longValue()

**TODO**
- left join이기 때문에 uid가 Null 인 경우 Response 맵핑시 NPE 발생가능
	- Null Object Pattern을 사용하거나 하는게 좋을듯함 
- Testcontainers 사용해서 Repository 통합 테스트 추가하기

```java
map(row -> {
	Long uid = (Long) row.get("uid");
    User user = null; // Null Object 객체 주입 해주면 됨
    if (uid != null) {
        user = User.builder()
                .id(uid)
                .name((String) row.get("name"))
                .email((String) row.get("email"))
                .createdAt(toLocalDateTime(row.get("uCreatedAt")))
                .updatedAt(toLocalDateTime(row.get("uUpdatedAt")))
                .build();
    }
    
	//..
})
```

---

## Reactive Redis

**reference**
- [Spring Data Redis](https://docs.spring.io/spring-data/redis/reference/redis/redis-streams.html)
- [Lettuce](https://github.com/redis/lettuce)
	- async 지원 redis client
- [reactive redis template](https://docs.spring.io/spring-data/redis/reference/redis/template.html)

`compose.yml` 에 redis 설정 추가
```text
services:  
  mysql:  
    container_name: r2dbc-mysql  
    image: mysql:8  
    ports:  
      - 3306:3306  
    environment:  
      MYSQL_ROOT_PASSWORD: root  
      MYSQL_DATABASE: testdb  
      MYSQL_USER: tester  
      MYSQL_PASSWORD: tester1234  
    volumes:  
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  
  redis:  
    container_name: r2dbc-redis  
    image: redis:7.2-alpine  
    command: redis-server --port 6380  
    labels:  
      - "name=redis"  
      - "mode=standalone"  
    ports:  
      - 6380:6380        // 기본 포트는 6379
```

```shell
$ docker exec -it -uroot r2dbc-redis /bin/sh
$ redis-cli -h localhost -p 6380 
```


build.gradle 의존성 추가
```text
implementation 'org.springframework.boot:spring-boot-starter-data-redis-reactive'
```


RedisConfig 생성해서 ApplicationReadyEvent 통해서 주입이 되었는지 확인해보기 !