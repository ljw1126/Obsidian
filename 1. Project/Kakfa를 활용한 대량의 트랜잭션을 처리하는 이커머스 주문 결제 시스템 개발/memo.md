
### 프로젝트 구조 
- **Presentation** (표현 영역)
	- 사용자의 요청을 처리하고 사용자에게 정보를 보여주는 역할을 한다.
		- 이때 사용자는 웹 브라우저를 이용해서 Request를 보내는 Client 뿐만 아니라 외부 시스템일 수도 있다.
		- 또한, 외부 시스템은 Front Server가 될 수도 있고, OpenAPI 형태로 Request를 요청하는 시스템이 될 수도 있다
- **Domain** (도메인 영역)
	- 시스템(비즈니스)이/가 제공해야 하는 도메인 규칙을 구현한다
	- 비즈니스의 핵심 규칙을 정의하고 구현하는 곳이다
- **Application** (응용 영역)
	- 사용자가 요청한 기능을 실행한다
	- 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다
	- <u>도메인 계층에 구현된 비즈니스의 핵심 규칙들을 조합해서 온전한 기능을 제공한다</u>
	- 기능 구현을 책임지고 있는 것이 아니라, 만들어져 있는 기능을 조합해서 온전한 기능을 구현하는 계층이라고 생각하는 것이 좋다 
	- ex. Application Service, CommandQueryHandler
- **Infrastructure** (인프라 영역)
	- Application의 Context 경계를 벗어나는 외부 시스템과의 연동으르 구현한다
	- 인프라 스트럭처 및 도메인 계층에서는 `저수준 모듈`을 구현하여 실제 기능을 제공한다
		- 나머지 계층에서는 `고수준 모듈`에 의존해서 코드를 작성한다

**DIP 원칙(Dependency Inversion Principle)**
- A 객체에서 B 객체를 참조해서 사용해야 하는 경우
- 즉, 다른 클래스의 메서드를 사용해야 하는 경우, 그 Class(구현체)를 직접 참조하는 것이 아니라 그 대상의 상위 요소 (추상 클래스 또는 인터페이스)를 참조하라는 원칙
- 클라이언트와 구현체가 인터페이스를 의존하고 있는 경우
	- 구현체의 변경이 클라이언트에게 전파되지 않는다
	- 클라이언트의 구현체의 내용을 알지 않더라도 인터페이스에 따라 결과를 기대함
	- 구현체가 변경되더라도 클라이언트에게는 영향을 주지 않고, 기능 확장이 가능하다
	- 결국 응집도가 높고, 결합도가 낮은 설계가 가능해진다


**고수준 모듈**
- 도메인(비즈니스)에서 제공해야 하는 기능을 온전하게(?) 제공하는 **단일 기능 or 메서드**를 말함
- 쉽게 말하면 Application Layer(응용 계층)의 Methods를 고수준 모듈이라고 한다
- 예를 들어 `CalculateDiscountService#calculateDiscountAmount`는 "할인 계산"이라는 온전한 기능을 구현 및 제공한다.


**저수준 모듈**
- 고수준 모듈은 작은 메서드로 구성되어 있다
- 이 메서드를 저수준 모듈이라고 한다. 독립적으로 온전한 기능을 제공할 수 없는 메서드를 말한다
- 예를 들어 `CalculateDiscountService#calculateDiscountAmount`의 로직을 실질적으로 수행하는 작은 메서드(`selectDiscountPolicy`, `isEnableDiscount`)들을 저수준 모듈이라고 한다



**테이블 설계**
- `purchase_order` : 구매 정보
- `order_items` : 주문 상세 정보 테이블
- `payment_transaction` : 거래 원장 테이블
- `card_payment` : 거래 수단 정보 테이블

**참고.** [ERD 다이어그램](https://dbdiagram.io/d/6667deb36bc9d447b15c0b8c)


### 개발 환경 구성


**참고. JDK 설치** 
- **방법1**. 설치 파일을 직접 다운로드 받아 설치하는 방법
- **방법2**. 바이너리 파일을 다운로드 받아서 압축을 풀고 `Home Path`를 잡아주는 방법 (선호✨)
	- 설치 경로를 특정 경로로 지정 가능
	- 다른 버전 설치시 환경변수에서 `Home Path`만 변경하면 된다
	- OS마다 설정 방법에 차이가 있지만, 설치 방법 대다수가 비슷하다
		- `JAVA_HOME=경로`
		- `PATH=$JAVA_HOME/bin`
	- [sdkman 설치](https://sdkman.io/)

**참고. 유용한 IntelliJ 플러그인** 
- `GitToolBox` : Commit History를 확인할 수 있다. 마지막 Commit의 범인을 색출 가능
- `Rainbow Brackets` : 괄호의 색상을 다르게 설정해 주는 기능
- `JPA Buddy` : 영속성과 관련된 다양한 자동화 기능을 제공
- `Kafka` : Kafka 클러스터와 연동하여 메세지를 produce한다던가 consum을 UI 통해 확인 가능
- `IdeaVim` : IntelliJ의 Edit 및 단축키를 Vim과 동일하게 해줌


### 헥사고날 아키텍처 (Port and Adapter 아키텍처)

- **목표**: 
	- 인터페이스나 기반 요소(Infrastructure)의 변경에 영향을 받지 않는 핵심 코드를 만들고 이를 견고하게 관리하는 것
- **정의**
	- `Port`는 인터페이스(usecase)이다 
	- `Adapter`는 클라이언트에게 제공해야 할 인터페이스(기능 명세)를 따른다
		- **내부 구현은 인터페이스(Port)의 구현체로 위임하는 것**을 말한다
		- 포트를 호출해주는 역할을 하는 것이 Adapter이다
			- Controller(Adapter) -> Interface (port, usecase) -> 구현체 
	- `Service` 
		- Domain Layer에서 구현된 비즈니스 로직과 Repository에서 제공하는 ORM 기능을 조합해서 Port가 제공해야 하는 온전한 기능 제공하는 역할을 한다
	- `Domain`
		- 도메인 핵심 로직, 비즈니스 로직을 구현
		- 이처럼 비즈니스 로직을 도메인 중심으로 개발함으로써 응집도는 높고, 결합도를 낮출 수 있다. 그리고 이를 통해 지속 가능한 소프트웨어를 만들 수 있다
	- 러닝 커브가 높아서 잘못 사용하기 쉽다

어답터가 `컨트롤러, 서비스, 인터페이스 구현체`이고 Port(인터페이스)를 통해 요청한다

**참고.** [Line 엔지니어링 - Port And Adapter](https://engineering.linecorp.com/ko/blog/port-and-adapter-architecture)


> [!note]
> OOP 관점에서 보면 **Entity(상태)와 비즈니스 로직(행동)을 하나의 객체에 정의**하는 것이 맞다. 이렇게 함으로써 **비즈니스 로직이 파편화 되지 않게 되고 응집도가 높은 코드**가 된다


> [!note] Aggregate
> DDD에서 하나의 Aggregate(Entities +  Value Object)는 개념상 **완전한 한 개의 도메인 영역(모델)을 표현**한다. 그리고 **도메인 모델**(ex. 주문/결제/고객정보)은 영속성을 가져야 하는 한다.
> 
> 그렇기 때문에 정의한 Aggregate에 JPA와 관련된 어노테이션을 선언하고 ORM의 주체가 되어야 한다. 그리고 영속성 관리하는 **Repository(Port, Interface)** 는 Aggregate 단위별 생성된다. 이유는 데이터의 일관성을 유지하기 위함이다.

---

### Spring Boot의 요청 처리 프로세스
// ch4-1