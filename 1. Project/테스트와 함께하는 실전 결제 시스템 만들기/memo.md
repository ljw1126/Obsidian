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


### 02. 프로젝트 셋업
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