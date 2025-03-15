
**시스템 디자인** 
- 규모 추정
- 시스템 성능/비용/확장성 고려 가능

**주요 지표** 
- `MAU`
- `DAU`
- `QPS`: 초당 처리 되는 쿼리 수 (시스템의 처리 능력, 읽기 작업)
- `TPS`: 초당 처리 되는 트랜잭션 수(데이터 베이스의 처리 능력, 쓰기 작업)
- `Peak Users` : 특정 시간대 동시 접속하는 최대 유저 수
- `Data Storage` : 시스템이 처리할 데이터 총량
- `Bandwidth` : 시스템이 초당 사용하는 네트워크 대역폭 
- `Latency` : 지연시간, 요청 처리 평균 시간
- `Throughput` : 단위시간당 처리되는 처리량

**MAU 1천만 가정할 때**
- DAU : 10% 가정 => 100만명의 활성 사용자 예상
- QPS, TPS 
	- QPD = DAU * (세션 당 쿼리수) = 100만 * 세션당 2번 호출 = 200만
	- QPS, TPS가 1:1 비율이라면 100만씩 됨 (실제는 읽기와 쓰기가 8:2 비율이 보통)
	- 하루를 초로 계산하면 86,400(`60*60*24`)초 
	- 초당 QPS, TPS = 100만 / 86,400초 = 11.57로 추정 가능
	- 피크 시간에 얼마나 몰리는지도 예상해야 한다
		- 보통 1.2배, 많으면 2배 이상 예상 가능
- Volume
	- 사용자 검색 요청시 쌓이는 데이터의 크기 예상치
		- ID(8byte), 검색어(50byte), 검색 시간(8byte), meta(64byte) 추정 = 대략 130byte
		- 100만(TPS) * 130byte = 130,000,000 (byte) = 약 130Mb가 하루에 쌓인다


>[!info] 한글, 영문 바이트 수
>- UTF-8 기준 : 영문, 숫자 1byte, 한글 3byte
>- UTF-16 기준 : 영문, 숫자, 한글 전부 2byte
>- EUC-KR 기준 : 영문, 숫자 1byte, 한글 2byte


### 프로젝트 생성
- 프로젝트 명 : libaray-search
- gradle
- Java 17
- Spring web만 우선 의존성 추가


### 멀티 모듈
- [https://docs.gradle.org/current/userguide/java_library_plugin.html](https://docs.gradle.org/current/userguide/java_library_plugin.html)
- [https://docs.gradle.org/current/userguide/intro_multi_project_builds.html](https://docs.gradle.org/current/userguide/intro_multi_project_builds.html)

**Gradle Plugin**
의존성 키워드 관련해서
- `api` : 다른 모듈도 의존성 접근 가능
	- 재사용성 높아짐 
	- 컴파일패스가 커져 빌드 성능 저하 가능
	- 모듈간의 결합도가 높아짐
- `implementation` : 다른 모듈은 의존성에 접근 못함
	- 재사용성은 떨어짐
	- 컴파일 패스가 작아져 빌드 성능 향상
	- 모듈의 캡슐화가 잘됨

**멀티 모듈 구조**
```text
my-multi-module-project/
- build.gradle
- settings.gradle
- module1
	- build.gradle
- module2
	- build.gradle
```


```text
// settings.gradle
rootProject.name = 'my-multi-module-project'
include 'module1', 'module2'
```


```text
// module1/build.gradle
plugins {
	id 'java-library'
}

dependencies {
	// ..
}


// module2/build.gradle
plugins {
	id 'java-library'
}

dependencies {
	implementation project(':module1') // 모듈2는 모듈1을 의존함
	// ..
}

```


**멀티 모듈 생성시 아래와 같이 gradle 구조가 나타남**
- library-search
	- common
	- external
	- search-api 

## 섹션4. 프로젝트 구현

### 외부 API 연동 - OpenAPI & RestClient
- 네이버 개발자 센터 : https://developers.naver.com/main/
	- [도서 검색 API](https://developers.naver.com/docs/serviceapi/search/book/book.md#%EC%B1%85)


> [!note]
> RestTemplate - RestClient - FeignClient

API 호출에 앞서 Run/Debug Configuration에서 Environment variable에 client id와 secret key 를 추가하면 환경 변수가 인식된다

최대한 빠르게 작성가능한 `FeignClient` 사용
- SpringCloud: [https://spring.io/projects/spring-cloud](https://spring.io/projects/spring-cloud)
- FeignClient: [https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html](https://spring.io/projects/spring-cloud)
- [우아한 feign 적용기](https://techblog.woowahan.com/2630/)

FeignClient는 HTTP API를 호출할 수 있는 선언적 웹 서비스 클라이언트다. API 호출시 필요한 복잡한 코드를 최소화하고 마치 로컬 메서드 호출하든 간편함을 제공한다.

```java

@FeignClient("stores")
public interface StoreClient {
	@RequestMapping(method = RequestMethod.GET, value = "/stores")
	List<Store> getStores();
}

// 커스텀 configuration 클래스 설정 가능 (없으면 default 설정 적용됨)
// 인터셉터(헤더에 특정 값 추가), 인코더, 디코더 등을 추가 가능
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
	// ..
}
```

**사용방법**
```java
@EnableFeignClients

// 해당 패키지 기준 스캔
@EnableFeignClients(basePackages = "com.example.clients")

// 정의한 클라이언트 명시적으로 작성
@EnableFeignClients(clients = InventoryServiceFeignClient.class) 
```

의존성 추가 

```text
// external/naver-client/build.gradle
jar.enabled = false  
  
dependencies {  
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'  
}


// 루트/build.gradle
subprojects {  
    //..
    
    ext {  
        set('springCloudVersion', "2024.0.0")  
    }  
    dependencyManagement {  
        imports {  
            mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"  
        }  
    }
    
	//..
```



```java
@FeignClient(name = "naverClient", url = "${external.naver.url}", configuration = NaverClientConfiguration.class)  
public interface NaverClient {  
    @GetMapping("/v1/search/book.json")  
    String search(@RequestParam("query") String query,  
                  @RequestParam("start") int start,  
                  @RequestParam("display") int display);  
}


public class NaverClientConfiguration {  
  
    @Bean  
    public RequestInterceptor requestInterceptor(  
            @Value("${external.naver.headers.client-id}") String clientId,  
            @Value("${external.naver.headers.client-secret}") String clientSecret  
    ) {  
        return requestTemplate -> requestTemplate.header("X-Naver-Client-Id", clientId)  
                .header("X-Naver-Client-Secret", clientSecret);  
    }}
}


@SpringBootTest(classes = NaverClientTest.TestConfig.class)  
@ActiveProfiles("test")  
class NaverClientTest {  
  
    @EnableAutoConfiguration  
    @EnableFeignClients(clients = NaverClient.class)  
    static class TestConfig {}  
  
    @Autowired  
    NaverClient naverClient;  
  
    @Test  
    void callNaver() {  
        String http = naverClient.search("HTTP", 1, 10);  
        System.out.println(http);  
  
        assertThat(http.isEmpty()).isFalse();  
    }}
}
```


JUnit5가 아닌 groovy 언어 기반 테스트 **spock**를 사용해본다
- 강력한 mocking 기능도 지원한다
- 일반적으로 `Specification` 상속받아 작성
- Spock: [https://spockframework.org](https://spockframework.org/)
- Spock Repository: [https://mvnrepository.com/artifact/org.spockframework](https://spockframework.org/)

build.gradle 
```text
plugins {  
    id 'org.springframework.boot' version '3.4.3'  
    id 'io.spring.dependency-management' version '1.1.7'  
}  
  
subprojects {  
    apply plugin: 'java'  
    apply plugin: 'groovy'   // 여기
    apply plugin: 'org.springframework.boot'  
    apply plugin: 'io.spring.dependency-management'  
  
    group = 'com.library'  
    version = '0.0.1-SNAPSHOT'  
  
    dependencies {  
        //.. 두 개 추가
        testImplementation 'org.spockframework:spock-core:2.4-M4-groovy-4.0'  
        testImplementation 'org.spockframework:spock-spring:2.4-M4-groovy-4.0'  
    }  
  
}
```


**NaverErrorDecoder 생성**
- 테스트 작성시 관심사가 아닌건 Mocking 처리한다
- reponse builder 호출시 request가 필요한데 관심사가 아니니 적당히 stub처리

```java
package com.library.feign  
  
import com.fasterxml.jackson.databind.ObjectMapper  
import feign.Request  
import feign.Response  
import spock.lang.Specification  
  
class NaverErrorDecoderTest extends Specification {  
    ObjectMapper objectMapper = Mock()  
    NaverErrorDecoder errorDecoder = new NaverErrorDecoder(objectMapper);  
  
    def "에러 디코더에서 에러 발생시 RuntimeException 예외가 throw 된다" () {  
        given:  
        def responseBody = Mock(Response.Body)  
        def inputStream = new ByteArrayInputStream()  
        def response = Response.builder()  
                                .status(400)  
                                .request(Request.create(Request.HttpMethod.GET, "testurl", [:], null, null, null))  
                                .body(responseBody)  
                                .build()  
  
        1 * responseBody.asInputStream() >> inputStream  
        1 * objectMapper.readValue(*_) >> new NaverErrorResponse("error!", "E01");  
  
        when:  
        errorDecoder.decode(_ as String, response);  
  
        then:  
        RuntimeException e = thrown()  
        e.message == "error!"  
    }  
}
```


### 외부 API 연동 - 응답 객체 정의
- `NaverBookResponse`, `Item` 객체 정의
- API 네이밍이 이상한 경우 `@JsonProperty("pubdate")  private String pubDate;` 와 같이 맵핑처리 후 백엔드에서는 카멜 케이스로 다루도록 한다

>[!info] 매번 environmenet variables 에 API 키값 넣기 번거로운 경우 
>- Run/Debug Configuration 에서 좌측 하단 Edit configuration templates.. 선택
>- gradle에서 환경 변수 추가 후 Modify options에서 run as test 추가
>- configuration에 전체 삭제 후 다시 실행시 탬플릿 통해 환경 변수 추가됨

>[!warning] NaverErrorResponse에 기본 생성자가 누락되어 있어서 추가
>API 호출 에러 발생시 decoder() 실행하여 mapping 처리하는데, jackson은 기본 생성자가 필요


### 공통 모듈 예외 처리 
- `common` 모듈 정의 (공통 사용하기 위해)
- `external/naver-client` 모듈에서 common 모듈 추가 후 `NaverErrorDecoder`에서 에러 정의

### API 구현 
- [spock](https://github.com/spockframework/spock)
- [spock-example](https://github.com/spockframework/spock-example)
- Controller, Service, Dto 에 대한 테스트 진행
	- Controller, Service의 경우 Mocking 통해 verify 검증
	- Dto는 생성 테스트


>[!warning] 이슈 기록
>search-api 실행시 external > naver-client 클래스를 찾지 못하는 이슈가 발생
- 설정에 Build, Execution, Deployment > Gradle에서 gradle 설정되어 있는지 확인
	- Gradle user home이 잘 설정되어 있는지도 확인
- external > naver-client 모듈 패키지 경로에서 
	- `com.library` > `com.library.naverclient`로 변경
- 루트에 위치한 `gradle` 폴더에서 wrapper 빼고 다 삭제 (**캐시 디렉터리가 문제였을수도 있음**)
- 인텔리제이 캐시 초기화

SearchApiApplication을 아래와 같이 변경 

```java
@EnableFeignClients(clients = {NaverClient.class})   // naver-client의 의존성이 api를 사용하고 있어야 함
@SpringBootApplication  
public class SearchApiApplication {  
  
    public static void main(String[] args) {  
       System.setProperty("spring.config.name", "application-naver-client"); // naver-client 모듈의 설정 사용 
       SpringApplication.run(SearchApiApplication.class, args);  
    }  
}
```
- 추가로 naver-client에서 `api()` 메소드 찾지 못했었는데 root에 위치한 build.gradle에 `java-library` 플러그인 추가하니 해결
- run configuration에서 profile = local 설정, NAVER 키 값 2개 환경변수 설정하고 실행


