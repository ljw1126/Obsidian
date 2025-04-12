> Spring 환경에서 바로 적용하는 테스트의 모든 것 초격차 패키지 Online 


## Ch1. 결제 시스템 설계하기 
### 02. 결제 흐름 알아보기
- 시스템 내 잔액 결제 : 별도의 결제 수단을 통해 시스템에 잔액을 충전
  - 충전 이후에는 시스템 내에서 관리되는 잔액으로 결제
- 결제 수단을 통한 결제
  - 일반적으로 PG(Payment Gateway)사를 이용함
  - ex. 네이버페이, 토스 페이먼츠, KG 이니시스 등
  
  
참고. [화이트보드 툴](excalidraw.com)

**우리 시스템에서 지원할 것**
- 시스템 내 잔액 결제
	- PG에 연동하는 부분을 Mock으로 우선 개발
	- 데이터 무결성이 보장되어야 함
		- 🤔사용자가 구매 버튼 세번 클릭했을 때 결제가 세번이 된다면 .. 
- 결제수단을 통한 결제(PG)
	- PG 시스템의 샌드박스에 연결해 개발 및 테스트를 진행
	- PG와 결제 시스템은 네트워크를 통해 통신한다
		- 네트워크는 항상 신뢰하기는 어렵다
			- 결제는 안 되었는데 강의를 볼 수 있는 경우
			- 결제는 됐는데 강의를 볼 수 없는 경우

> 카드 결제는 `인증 - 승인 - 매입` 순으로 이뤄진다.

**1. 인증**
- `카드 소유주`를 확인한다
- 카드 정보를 입력하거나, PG 시스템에서 카드 앱으로 결제하는 창을 넘기고 간편 비밀번호를 입력하는 것을 생각하면 된다

**2. 승인**
- 거래가 발생하고 사용자의 한도를 차감한다
- 승인이 됐다면 금융 시스템 상에서 거래가 가능하다는 것을 뜻함
- 매입 전에는 실제 금액이 차감되지 않는다
	- ex. 숙박시설 보증금 결제

**3.매입**
- 승인된 결제를 확정함
- 매입까지 되어야 판매자가 카드사로부터 **정산** 받을 수 있다
- 승인, 정산까지의 과정을 저희는 PG사를 통해서 진행하게 된다


### 03. 우선순위 결정하기


**상황 설정**
- 결제 시스템을 만든다
	- 계정/주문/수업/스트리밍 등 마이크로 서비스로 구성되어 있다
	- 여기에 **결제/정산**이 빠져있다
	- 이제 시스템이 충분히 켰고, 서비스 운영을 위해 **결제/정산** 시스템을 만들어본다

**결제**
- 기본적으로 주문 시스템은 있었다고 가정
- 주문을 만들 때 결제 시스템과의 연동을 만들어주면 될 것 같음
- 어떤 결제들을 지원할 것인지, 보안에 문제는 없을지, 처리 속도, 중복 결제 등의 문제는 없을지 등을 검토해야 한다

**정산**
- 결제가 있으면 필연적으로 따라오는 시스템
	- 매출이 실제로 얼마나 일어났는지
	- 우리 플랫폼이 인지하고 있는 결제액과 사용자가 실제 결제한 금액에 차이가 있는지
	- 수익을 분배하는 대상이 있다면 수익은 어떤 정책을 통해 분배할 것인지 등을 다룸
- **다만 이 강의에서는 결제를 만드는데에 집중합니다.**
- <u>정산은 고민해보고 스스로 만드는 연습을 해도 좋고, 다른 강의를 보셔도 좋을것 같다</u>


### 04. 결제 시스템의 책임을 결정하기

**기능적 요구사항**
- 사용자에게 제공할 기능
	- 결제 수단 지원 및 처리 (PG 또는 잔액 결제)
	- 복합 결제 지원 여부 (ex. 잔액 + 포인트 조합으로 결제)
	- 더블 클릭으로 인한 중복 결제 방지
	- 사용자 **인증**: 
		- 외부 시스템으로 나갈 때 우리 시스템에서 온 요청을 명확히 밝히기 위한 인증
		- 클라이언트가 주문 요청하는 경우 
			- 보통 `체크섬 > HMAC`을 사용한다함
	- 그외 보안, 성능 
- **결제 수단, 데이터 처리, 성능 위주로 시스템을 설계 해보고자 한다**

**비기능적 요구사항**
- 품질, 제약사항 등


>[!tip] API 설계할 때 외부에 공유하는 API들을 찾아보면 좋은 공부가 된다

**성능**
- 도메인 특성에 대해 먼저 고민해볼 필요가 있다
	- 우리가 만들 강의 시스템이 사용자는 얼마나 될까
	- 얼마나 큰 데이터를 저장할거고, 저장하고 관리해야 하는 데이터는 정합성을 일부 포기해도 되는 형태인지 아닌지
	- 평소의 트래픽과 사용자가 몰리는 지점의 트래픽을 얼마나 차이날지
	- 트래픽이 얼마나 늘어나고 있고 최대 어느 수준까지 갈 수 있는지 등
- 도메인을 먼저 이해해야 달성해야 할 성능 수준을 결정할 수 있을 것 같다
- 이런 다양한 변수를 고려해 적정한 기술과 자원을 통해 개발하고 운영할 숭 시는 시스템을 만드는 것이 좋다.


### 05. 시스템 목표 설정하기

> 누적 수강생 70만명이 선택한 xxx 


**시스템 성능이란**
- 보통 처리량을 생각할 수 있을 것이다
- 도메인 특성, 환경에 따라 달성해야 할 성능 목표가 다르다
- 정확한 기준을 잡기는 어렵고, 사용하지 편한 서비스 환경을 제공해주는 것을 목표로 해야 한다
- 지표
	- 처리량: TPS 같은 것으로 얼마나 많은 트랜잭션을 처리할 수 있는가 뜻함
	- 응답시간: 요청 ~ 응답까지의 시간
		- <u>네트워크 구간에 대한 이해, 톰캣의 요청 모델, DB 등 다양한 구간에 어떤 변수들이 있을지 생각해보면 좋다</u>

**시스템 목표 성능 추청**
- `70만명이 선택한 .. ` 
- 서비스를 제공한 시간은 1년으로 가정
	- 1년간 누적 결제 수 70만
	- 타이트하게 하루 24시간 중 6시간 정도의 범위 내에서 결제가 집중적으로 이뤄진다고 가정함
	- 6시간을 초로 환산 > 21,600 초
	- 결제 트래픽을 가정하면 TPS가 1건이 되지 않음
		- TPS = 70만 / 365 / 21,600 = 0.088 
- 피크타임, 조회, 결제 등 다양한 트래픽 조건이 있을 수 있지만 러프하게 생각
- 생각보다 훨씬 작은 목표치를 만들어 놓고 시스템 설계를 하게 되더라도 문제가 없을 것 같다는 생각이 든다
	- 성능을 배제한다기보다는 성능이 상대적으로 덜 중요한 시스템이라 생각
	- 결제 시스템의 특성 상 상대적으로 더 중요한 가치인 데이터 무결성에 집중하고, 성능에는 좀 더 캐주얼하게 접근해보독 하겠다.
- **마지막에는 성능 테스트를 하면서 지표 기반으로 확인해본다**


### 06. 데이터 모델링 하기

**잔액 결제 데이터베이스 모델링**
- 잔액 충전이 있기 때문에 간단한 테이블 설계를 해본다
- **Wallet** 테이블 
- **Transaction** 테이블
	- `tansaction_type` : 충전, 잔액 결제
	- <u>PG사를 이용하면 별도로 관리할 필요 없을 수도 있다</u>
- 우선 PG는 고려하지 않음
	- 핵심 아이디어에 대해 최대한 빠르고, 간단하게 만들고 디테일을 채워나간다 (**공부에 적합할 수 있다**)

**인덱스**
- 데이터베이스 설계에 매우 중요하다
	- **📚 Real MySQL (추천✨)**
	- **📚 MySQL 퍼포먼스 최적화 (추천✨)**
- `Wallet 테이블` : user_id
- `Transaction 테이블` : wallet_id, user_id, order_id
- 검증이 필요하다면 **쿼리 플랜**을 확인하고 개선해 볼 수 있다


## Ch02. 핵심 기능 만들어보기

- 우선 외부 시스템 없이 잔액 충전과 잔액 결제 API를 먼저 만들어본다
- 외부 시스템 연결 없이 내부망에서 데이터의 정합성을 잘 지키도록 결제하는 파트가 **잔액 결제**이다
- 그 다음으로 외부 시스템과 연결하고, 외부-내부 시스템의 데이터 정합성까지 잘 지켜줘야 하는 PG 결제 파트를 진행한다

// 웹 flow 다이어그램 추가하기 (충전, 잔액 결제 두 가지)


**충전 API**

테스트용 잔액 충전 API
- **Endpoint**: `/api/balance`
- **Method**: POST
- **Request Body**

```json
{
	"userId" : {userId},
	"amount" : 10000
}
```


**결제 API**
잔액 결제에는 인증이 필요하다 (사용자가 누구인지 확인)
- Http Basic 인증이나 토큰 기반 인증을 사용할 수 도 있다
- 이런 인증 프로세스는 한번 학습해보길 권장

강의 결제 API
- **Endpoint**: `/api/balance`
- **Method**: POST
- **Request Body**
```json
{
	"courseId" : "course_id"
}
```


### 02. 프로젝트 셋업 ~ 04
> java 17, spring boot 3.x, spock, jpa, lombok, Thymeleaf, mysql

프로젝트명: simple-payment-system
강의자료: https://github.com/gracefulife/fastcampus-test-payment-system

참고. 
- [Spock 소개 및 튜토리얼 - 기억보단 기록을](https://jojoldu.tistory.com/228)
- [Spock으로 테스트하기 - Naver D2](https://d2.naver.com/helloworld/568425)

`spock 관련`
```text
plugins {  
    id 'java'  
    id 'org.springframework.boot' version '3.4.4'  
    id 'io.spring.dependency-management' version '1.1.7'  
    id 'groovy'   // 추가
}
dependencies {
	// ..

    // 추가
	testImplementation 'org.spockframework:spock-core:2.4-M4-groovy-4.0'  
	testImplementation 'org.spockframework:spock-spring:2.4-M4-groovy-4.0'
}
```


**WalletService**
- `addBalance`
	- 정책1. 잔액이 마이너스가 되면 오류가 발생해야 한다
	- 정책2. 최대 충전한도는 10만원이다

```java
@RequiredArgsConstructor  
@Service  
public class WalletService {  
    private final static BigDecimal BALANCE_LIMIT = new BigDecimal(100_000);  
  
    private final WalletRepository walletRepository;  
    
    // 트랜잭션 스크립트 스타일
    @Transactional  
    public AddBalanceWalletResponse addBalance(AddBalanceWalletRequest request) {  
        final Wallet wallet = walletRepository.findById(request.walletId())  
                .orElseThrow(() -> new IllegalStateException("지갑이 없습니다"));  
  
        BigDecimal balance = wallet.getBalance();  
        balance = balance.add(request.amount());  
  
        if(balance.compareTo(BigDecimal.ZERO) < 0) {  
            throw new IllegalStateException("잔액이 충분하지 않습니다");  
        }  
        if(BALANCE_LIMIT.compareTo(balance) < 0) {  
            throw new IllegalStateException("한도를 초과했습니다");  
        }  
        wallet.setBalance(balance);  
        wallet.setUpdatedAt(LocalDateTime.now());  
        walletRepository.save(wallet);  
  
        return new AddBalanceWalletResponse(  
                wallet.getId(),  
                wallet.getUserId(),  
                wallet.getBalance(),  
                wallet.getCreatedAt(),  
                wallet.getUpdatedAt()  
        );    }}



```


Wallet 도메인으로 비즈니스 로직을 위임

```java
@NoArgsConstructor  
@AllArgsConstructor  
@Data  
@Entity  
public class Wallet {  
    private final static BigDecimal BALANCE_LIMIT = new BigDecimal(100_000);  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    private Long userId;  
  
    private BigDecimal balance;  
  
    private LocalDateTime createdAt;  
  
    private LocalDateTime updatedAt;  
  
    public Wallet(Long userId) {  
        this.userId = userId;  
        this.balance = new BigDecimal(0);  
        this.createdAt = LocalDateTime.now();  
        this.updatedAt = LocalDateTime.now();  
    }  
    public void addBalance(BigDecimal amount, LocalDateTime timestamp) {  
        BigDecimal newBalance = this.balance.add(amount);  
  
        if(newBalance.compareTo(BigDecimal.ZERO) < 0) {  
            throw new IllegalStateException("잔액이 충분하지 않습니다");  
        }  
        if(BALANCE_LIMIT.compareTo(newBalance) < 0) {  
            throw new IllegalStateException("한도를 초과했습니다");  
        }  
        this.balance = newBalance;  
        this.updatedAt = timestamp;  
    }}
```


```java
@Transactional  
public AddBalanceWalletResponse addBalance(AddBalanceWalletRequest request) {  
    final Wallet wallet = walletRepository.findById(request.walletId())  
            .orElseThrow(() -> new IllegalStateException("지갑이 없습니다"));  
  
    wallet.addBalance(request.amount(), LocalDateTime.now());  
    walletRepository.save(wallet);  
  
    return new AddBalanceWalletResponse(  
            wallet.getId(),  
            wallet.getUserId(),  
            wallet.getBalance(),  
            wallet.getCreatedAt(),  
            wallet.getUpdatedAt()  
    );}
```


spock  언더바 
- https://spockframework.org/spock/docs/2.3/interaction_based_testing.html

spock with WebMvcTest
- https://blog.allegro.tech/2018/04/Spring-WebMvcTest-with-Spock.html
- https://blog.naver.com/dodi258/222190863323


WalletController @WebMvcTest 중에 날짜 포맷이 맞지 않는 에러가 발생함 
```java
def "지갑 조회 요청을 하면 정보를 반환한다"() {  
    given:  
    def userId = 1L  
    def yesterday = LocalDateTime.now().minusDays(1L)  
    walletService.findWalletByUserId(_) >> new SearchWalletResponse(1L, userId, BigDecimal.ZERO, yesterday, yesterday)  
  
    when:  
    def response = mockMvc.perform(MockMvcRequestBuilders.get("/api/${userId}/wallet"))  
  
    then:  
    response.andExpect(status().isOk())  
            .andExpect(content().contentType(MediaType.APPLICATION_JSON))  
            .andExpect(jsonPath('$.id').value(1L))  
            .andExpect(jsonPath('$.userId').value(userId))  
            .andExpect(jsonPath('$.balance').value(BigDecimal.ZERO))  
            .andExpect(jsonPath('$.createdAt').value(yesterday)) //toString()변경시 통과
            .andExpect(jsonPath('$.updatedAt').value(yesterday))  
}
```


**🔧 JSON 직렬화에 영향을 주는 건? (Chat-GPT)**
- `ObjectMapper` 전역 설정 (예: `WRITE_DATES_AS_TIMESTAMPS`)
- `@JsonFormat` 어노테이션
- 커스텀 `Module` (예: `JavaTimeModule`)
- `@WebMvcTest`에서 자동 설정 미적용 이슈

Jackson 라이브러리에 설정된 내용에 따라 직렬화/역직렬화가 결정되는데, 설정 포맷에 따라 날짜가 다르게 출력할 수 있구나 


---

### 05. 결제 만들기


Transaction 도메인을 생성
- 이때 Wallet 에서 잔액 충전을 하지 않도록 정책 변경 
	- `/api/balance`
	- 잔액 충전에 대한 책임을 TransactionController에서 담당
- TransactionService는 WalletService를 의존한다
	- `charge(..)` : 충전
		- orderId : UUID 사용하는데.. snowflake 사용하면 좋지 않을까?
	- `payment(..)` : 결제
- ChargeTransactionResponse, ChargeTransactionRequest record 생성
- 참고. 10분 시퀀스 다이어그램

**중복 충전 방지**
- 요청시 orderId를 같이 보내서 서비스에서 확인
- 데이터베이스에 orderId에 해당하는 Transaction 정보가 있으면 예외 반환




**회고**
- Transaction, Wallet 서비스에 구현 문제가 아직 남아있다

---

### 06. 핵심 기능에 발생할 수 있는 흔한 문제 상황

**1. 충전 요청시**
- orderId로 충전된 거래인지 확인하는 부분
	- charge가 동시에 여러 번 충전 요청오는 경우 가정해보자
		- 동시에 들어왔을때 정상동작하지 않는데 (데이터 부정합)
	- update 할 때도 atomic update 불가능한 케이스

**2.결제/충전 등 삽입시 중복 요청**
- `orderId (멱등키)` 사용하는 케이스, 클라이언트가 요청시 같이 보냄 

**3.네트워크 분단**
- `클라이언트 - 서버`, `서버 - 결제서버` 요청 간에 **connect timeout**, **read timeout** 발생 가능 
	- 찾아보기 ! 
	- 클라이언트 - 주문 서버 간의 요청이 끊긴 후 주문 요청은 정상 처리되는 경우 
		- 새로고침하면 되니 큰 이슈는 아닌 것으로 보입
	- 단, 주문 서버 - 결제 서버 간의 요청이 끊긴 경우 결제 서버 문제가 발생해서 주문 서버가 응답을 받지 못하는 경우 (잔고 데이터를 갱신되고 주문 데이터는 삽입 못한 경우)
		- 대표적인 대응 방법 몇가지 확인 예정


### 07. 시뮬레이션 통해 문제 확인하기
- 3번은 PG 다룰 때 확인 


**1.지갑 생성시**
- WalletService에서 멀티 스레드로 동시 요청해서 1개가 생성되는지 확인한다 👍
	- 지갑이 생성되지 않은 유저로 확인한다
	- findBy\*로 조회하니 한건만 조회해야 하는데 여러개 확인되면 원치 않는 에러가 발생
**2.충전/결제시**
- TransacationService에서 충전 요청을 멀티스레드로 동시 실행해본다


- 패키지 구분 :  `wallet / transaction / common`
- Snowflake 추가
- createdAt, updateAt 공통 처리


>[!error] Error creating bean with name 'jpaAuditingHandler': Cannot resolve reference to bean 'jpaMappingContext' while setting constructor argument

@WebMvcTess시 jpaMappingContext 빈이 주입되지 않아 발생
```java
@EnableJpaAuditing  // Entity 날짜 처리 관련
@SpringBootApplication  
public class SimplePaymentApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(SimplePaymentApplication.class, args);  
    }  
}
```

아래와 같이 분리 처리하여 해결 ([참고](https://computerlove.tistory.com/entry/%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1-%EC%A4%91-%EC%98%88%EC%99%B8-%EB%B0%9C%EC%83%9D-1))
```java
@EnableJpaAuditing  
@Configuration  
public class JpaAuditingConfig {  
}
```

---
## Ch3. 테스트 기반으로 문제 해결하기 

> [!info] 결론적으로 Redis 활용한 락만을 학습하면 된다

### 01. 멱등성 문제 해결하기

**예시**
- 클릭을 여러 번해서 요청이 여러 번 되는 경우
- 네트워크 장애로 인한 문제 

> [!info] 멱등키(rid)로 재시도를 실행하는 것으로 문제 해결할 수 있다. 그러나 모든 문제를 해결할 수 없다.

패캠서버 -> 패캠payment 서버 요청시
- 패캠서버에서 timeout을 1초로 짧게 설정 
- 패캠payment 서버에서는 Thread.sleep(5000) 설정
- 테스트
	- 패캠서버에서 요청시 `read time out` 발생
	- 그런데 패캠 payment 서버에서는 요청 처리가 완료되어 있음
	- **중복처리하지 않고 사용자 요청에 멱등하게 처리하는게 중요하다**
		- 이때는 `orderId`(**멱등키**) 를 요청시 보내다 보니 재요청시 이미 처리된 요청이므로 예외를 반환한다 

### 02. 데이터 무결성 보장하기 with Java (Unit test)
- 잔액 충전시 orderId를 사용하더라도 문제가 발생할 수 있다
- `synchronized` 키워드 사용해서 Java에서 작업을 해본다
	- 이걸한다고 데이터베이스에 동시성 이슈가 발생하기 때문에 
- walletMap을 서비스에 만들고 여기에다가 통합 테스트를 한다네 
- `synchronized` 키워드가 붙은 `addBalanceJava(..)` 호출하는 형태로 한다
	- 순차적으로 실행되기 때문에 속도가 느려질 수 있지만 안정적으로 처리 가능(자바 프로세스 내에서)
	- <u>굳이 Service에 추가할 필요없이 Fake 클래스 만들어서 주입해가지고 사용하면 되지 않는가??</u>
- 하지만 `synchronized` 키워드를 사용하더라도 **여러 개의 서버를 띄우는 경우 의미가 없어짐(한계)** -> **데이터베이스의 동시성 이슈 제어가 필요함(MySQL or Redis)**


### 03. 데이터 무결성 보장하기 with MySQL (통합 테스트)
- `비관적 락(레코드 락 점유)`과 `낙관적 락(버전 정보 비교)`이 있다
- 충전 테스트시 
	- WalletRepository에서 findByUserId()에 `@Lock(LockModeType.PESSMISTIC_WRITE)` , 비관적 락을 건다
	- 멀티 스레드로 TransactionService 통해 충전 처리시 비관적 락을 걸었기 때문에 나머지 스레드는 대기하게 되고, 락을 반환하면 그때 부터 순차적으로 처리하여 정상적으로 충전 요청 처리된다
- Wallet 조회시 비관적 락을 걸었는데, Transaction에 걸 이유는 없지 않나??
	- Wallet의 락을 반환하기 전까지 다른 스레드는 대기하기 때문이고, orderId 멱등키를 기준으로 또 조회하기 때문에 중복 처리는 되지 않는다
	- 그러므로 Transaction 테이블에 락을 따로 걸지 않아도 괜찮다
- 그러나 `isolation level` 문제나 `skew` 문제라는게 있을 수 있다함
	- 우선 비관적 락을 테이블에 걸었기 때문에 데이터베이스의 부하는 늘어날 뿐 동시성 제어는 되지 않을까..?

> [!tip] 통합 테스트보다는 DBMS 툴로 트랜잭션을 확인해보는게 좋다


### 04. 데이터 무결성 보장하기 with Redis (통합 테스트)
- 레디스를 통해 락을 획득하고, 반환하는 형태로 처리
- Docker 활용해 레디스 컨테이너 추가 (docker-compose로 활용하자)
- RedisConfiguration 생성
- WalletLockService 생성 (이걸 왜하지??)
	- `walletId` 기준으로 락을 잡는다
- 트랜잭션 범위보다 레디스 락의 범위가 더 커야한다.
	- LockTransactionService 를 만든다
- **강의에서는 직접 RedisTemplate 호출하여 점유하는 방식을 사용했는데 Redission이 좀더 안정적인듯 하다 (비교 해보기 !)**

> [!note] keyword: 락 분할 기법


### 05. 락 다시 공부하기 

- 멱등키 사용
- 동시 요청시 쓰기 되기 전에 읽어 버려서 데이터 무결성 깨짐 (데이터 부정합)
	- synchronize 사용(method level) - 자바 서버가 여러 개면 의미 없음
	- 데이터베이스 비관적 락, 낙관적 락 사용 - 데이터베이스 부하 증가
	- Redis 락 사용
		- 그러나 레디스에서도 조회와 쓰기 타이밍이 다르다면 레드스 내에서 동시성 이슈가 발생할 수 있다 (?)
		- Redis의 원자적 연산 기능을 활용해 락을 취득하는 구현을 만듦 (언제 그랬짘ㅋ?)
- 다른 해결 방법은?
	- 트랜잭션 격리 수준을 조정하는 방법도 있다는데 > 당연히 서버 레벨에서 의미없지 않나? MSA는 의미 없는 걸로 암
	- Redis 같은 외부 시스템과 혼용할 때는 **락 선점 → 트랜잭션 처리**의 흐름을 명확히 유지해야 안전 (Chat-GPT)
- **실제 서비스에서는 DB, Redis, Zookeeper 등의 저장소를 이용한 락 기법이 널리 사용된다**
- 서비스에 동시성 제어를 위해서는 DB의 동작을 이해하는 것이 중요
	- 트랜잭션
	- DB 격리 수준
	- 락
	- 인덱스
	- 위와 같은 주제들을 책과 테스트 통해 공부하는 것을 권장


---
## Ch4. 외부 인터페이스 연동하기

### 01. PG(Payment Gateway) 란

**사용하는 이유**
- 결제 수단 연동이 쉽다
- 정상이 쉬움
- 보안 
	- PCI DSS 등 카드 보안 표준 준수의 어려움 등 
	- PG를 통한다면 직접 할 필요가 없다
	- 이상 거래 탐지 등의 기능
- 관리를 위한 어드민도 제공함
	- 직접 계약한다면 어드민을 만들어 제공하던, 개발자 지원을 통해 운영하던 방법들이 필요함

**요청-승인 패턴**
- PG 사 연동에 널리 쓰이는 패턴
- 인증은 결제 임시 처리, 승인은 확정으로 생각하면 됨
- 승인시 자동 매입이 되는지 아닌지는 PG 사마다, 설정마다 다름 
		- 전 강의에서 카드 결제 프로세스 살펴봄 (?)
- PG사와 결제 서버의 네트워크 분단 이슈가 발생하면 **데이터 정합성 문제 발생**가능
	- 그래서 정산을 진행한다
	- PG 정산 결과를 통해 내 데이터가 일치하는지 비교하는 대사 과정을 진행 가능
	- 이때 불일치가 발생한다면 시스템 때문인지 PG 사의 문제인지 확인하고 문제, 해결하는 과정을 거친다
- **토스 페이먼츠**를 연동한다
	- 문서도 잘되어있고, 쉬움

### 03. PG 연동하기 
- 결제 화면 띄우기, 결제 프로세스 정산 동작하는지 확인을 목표
	- [결제 연동하기](https://docs.tosspayments.com/guides/payment-widget/integration?backend=java)
	- API 키별 연동키 사용하기 // 사업자 등록번호가 없으니 
- checkout 패키지를 만들었는데 
	- external 모듈로 따로 뽑는게 낮지 않나 싶다.
- 화면은 `checkout.html`을 복사 붙여 넣기 한다.
	- 페이지 랜더링시 orderId, amount를 따로 주입하는 형태로 처리할 수 있지 않을까 싶다 (?)
- `success.html`을 생성하고 복사 붙여넣기 한다
	- success로 이동하는 듯하다 (?)
	- /confirm 까지 해야지 완료인듯
		- 결제 승인하면 결제 수단에서 금액이 차감된다
		- 디버그를 걸고 evaluate 활성화하면 메서드 호출가능하네 (Debug 콘솔창에 세로로 점 3개 찍힌거에 있다)
		- 상태 코드 별로 분기 가능한데 **생략**한다 
- `fail.html`을 생성하고 복사 붙여넣기 한다


**todo**
1. checkout 페이지 렌더링시 orderId를 만들어 주어야 한다
	1. 주문 서비스로부터 오는 걸 가정
2. PG와 주고받은 데이터를 저장할 필요가 있다
	1. 요청 > 승인 
	2. 요청, 승인 데이터 둘 다 저장
3. confirm 데이터 기반으로 상품 구매 API와 연결

// 주문 서비스와 결제 서비스 구분되네 

### 03. 요청 과정 구현하기 (요청 - 승인 절차)
- 🤔시퀀스 이미지 **설명 확인하기** (R&R, Role and Responsibilities, 역할과 책임)
	- 정리가 필요할거 같다 (4/11)
- (13분) order 테이블 생성
	- request_id (== orderId)를 인덱스 건다
- Controller에 Repository 쌩으로 주입해버리네 ㅋㅋㅋㅋㅋㅋㅋㅋㅋㅋ
- `/order` 
	- 디테일한 구현 내용은 생략하고 주문을 그냥 생성해서 랜더링하는 방향으로 설명
	- 결제 요청하고, 카드 결제(테스트)하면 success 페이지로 넘어간다
		- 이때 토스에 데이터가 쌓이고, `/sucess`로 리턴됨
		- 그리고 success 페이지에서 **승인 API**를 호출해야 한다


```text
강의 사이트에서 주문 요청 할때 팝업으로 토스 API를 랜더링 해주는 걸까?
성공을 하면 팝업 창을 닫고 주문 모듈에서 보여주는건가?
```

### 04. 승인 과정 구현하기 (요청 - 승인 절차)
- 토스에 결제 요청 후 리다이렉트 돌아온 다음 승인이 필요 (`/confirm`)
- 절차
	- 주문 서비스 - 주문 상태 변경됨(**REQUESTED**)
	- 주문 서비스 > 결제 서비스 승인 요청 (`/confirm`)
	- 결제 서비스 > PG 승인 요청 
	- 결제 서비스 > 결제 기록 저장
		- 결제 수단으로 바로 결제하는 메서드 구현 
	- 주문서비스 응답
	- 주문을 **APPROVED** 상태로 변경
- `external`패키지: 결제 승인 요청 (서버 -> 토스)
- `processing` 패키지: PaymentProcessingService 에서 PG 승인요청, 결제 기록 저장, 주문 상태 업데이트 등을 처리하는데.. 진짜 엉망으로 한다 ㅋㅋㅋㅋ
	- 유닛 테스트 실행
	- 진짜 엉망으로한다 TransactionService에서 pgPayment() 내역을 저장하는 걸로 보인다
		- walletId가 null인 이유는 PG사 모듈 통해 카드 결제 하므로 그렇게 처리하는듯
		- wallet은 충전 결제 방식
### 05. 충전 과정 구현하기 (요청 - 승인 절차)
- PG 결제를 앞에서 다룸
- 충전
	- PG 연동해서 결제하고, 결제 금액을 서버에 저장한다
	- <u>주문에서 강의 결제 주문이 있고, 충전 주문이 있을 수 있구나(상태 구분)</u>
	- **문제 상황**. PG사 결제는 완료했는데, 결제 서버에서 처리가 되지 않는 경우 

### 06. 
- 문제1. PG에 승인을 했는데, 결제 서버에서 처리가 되지 않는 경우 (사용자 돈만 차감)
- 문제2. PG에 승인 요청시 timeout이 났는데, 결제 서버에서 처리가 되는 경우(무한 충전)
	- 좋지 않은 사용자 경험 제공, 신뢰도 하락
- postman의 **mock 서버**를 만들어서 활용할 수 있다!
	- 딜레이를 줄 수 도 있고, 어떤 요청이 왔는지 확인가능하다
	- test profile을 만들어서 api 서버 url을 mock 서버로 변경해준다..
	- 테스트 컨테이너로 만들 수는 없을까??
- RestClient 생성시 timeout 설정을 하고 통합테스트 통해 `read time out`  발생 확인
	- Postman 은 귀찮다...
---
멱등키 (자주 쓰인다함, orderId)
Fixture Monkey 
멀티 스레드 환경 테스트 방법이.. board-msa 였나?
JPA에서 비관적 락과 낙관적 락 방법 찾아보자

도서 추천 
- Real MySQL
- 데이터베이스 중심 어플리케이션 설계 
- 데이터베이스 인터널스 

흐름 이해가 안되네.. (요청-승인)
모듈을 어떻게 분리해야 하는거야  (주문 서비스, 결제 서비스, PG 서비스)
- 주문 + PG 서비스가 되야 하지 않을까?

결제/충전 기능 구현 
동시성 제어 (Redission)
멀티 모듈 구성 
jacoco, sonarcloud 연동