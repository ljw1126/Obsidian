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