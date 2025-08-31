> [!info]  PPT 강의 자료는 있는데 실습 코드 전체가 주어지진 않음. 스스로 타이핑 해야 함


## 섹션3. 예제 만들기 


**비즈니스 요구사항과 설계**
- **회원** 
	- 회원을 가입하고 조회할 수 있다 
	- 회원은 일반과 VIP 두 가지 등급이 있다 
	- 회원 데이터는 자체 DB를 구축할 수 있고, 외부  시스템과 연동할 수 있다. (미확정)
- **주문**과 **할인 정책**
	- 회원은 상품을 주문할 수 있다
	- 회원 등급에 따라 할인 정책을 적용할 수 있다
	- 할인 정책에서 모든 VIP는 1,000원을 할인해주는 고정 금액 할인을 적용해달라 (변경 가능)
	- 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수도 있다. (미확정)

**회원 도메인 설계**
- 협력 관계 
	- ..
- 회원 클래스 다이어그램 
	- ..
- 객체 다이어그램
	- ..


> 13.회원 도메인 실행과 테스트에서
> - MemberServiceImpl은 추상화에도 의존하고 구현체에도 의존하고 있다.
> - 이로인해 데이터베이스 변경시 `OCP`, `DIP` 원칙 위반하게 됨


**주문과 할인 도메인 설계**
- (역할과 정책 확인)

> ✅ **역할**: 인터페이스 
> ✅ **정책**: 구현체


> OrderServiceImpl에서 DiscountPolicy 인터페이스에 구현체를 주입받아 사용하는데, 할인 정책에 대한 책임은 구현체에 있다는 점에서 SRP 원칙을 잘 준수했다는 것을 설명하는게 좋음 





## 섹션4. 객체 지향 원리 적용

- RateDiscountPolicy 생성
- AppConfig 생성
- MemberServiceImpl, OrderServiceImpl 리팩터링 

### 새로운 할인 정책 개발
- 악덕 기획자 , 순진 개발자의 대화가 웃기네 

**참고. 애자일 소프트웨어 개발 선언**
- https://agilemanifesto.org/iso/ko/manifesto.html

**RateDiscountPolicy(구현체) 정책 추가**
- DiscountPolicy(인터페이스)는 역할 


### 새로운 할인 정책 적용과 문제점 

- RateDiscountPolicy를 적용하기 앞서 
	- 역할과 구현을 충실하게 분리함 
	- 다형성도 활용하고, 인터페이스와 구현 객체를 분리함 
	- 다만, OCP, DIP 같은 객체지향 설계 원칙을 충실히 준수 x
		- OrderServiceImpl에서는 DiscountPolicy(추상) , RateDiscountPolicy(구현체) 의존함


### 관심사의 분리 
- 애플리케이션을 하나의 공연이라 생각해보자 
	- 각각의 인터페이스를 배역(배우 역할)이라 생각
	- 그런데 실제 배역에 맞는 배우를 선택하는 것은 누가 하는가 ?
		- 공연 기획자가 배우를 섭외해야 한다 
- `관심사를 분리하자`
	- 배우는 본인의 역할인 배열을 수행하는 것만 집중해야 한다 
	- 공연을 구성하고, 담당 배우를 섭외하고, 역할에 맞는 배우를 지정하는 책임을 담당하는 별도의 `공연 기획자`가 필요한 시점이다
	- 공연 기획자를 만들고, 배우와 공연 기획자의 책임을 확실히 분리하자 

> 공연 기획자 => **AppConfig** 설정 클래스를 생성하여 의존성을 주입하는 책임을 맡긴다 


### AppConfig 리팩터링 
- 중복이 있고, 역할에 따른 구현이 잘 안보임
- 메서드 추출을 통해 파라미터 객체를 뽑는다 (=중복 감소)
	- ✅ 메서드 명에 **역할** 들어가고, 메서드 내부에 **구현**이 들어감 🤔


### 새로운 구조와 할인 정책 적용 
- Fix* -> Rate* 할인 정책으로 변경해본다 
- `AppConfig`만 수정하면 된다 

> 사용 영역과 구성 영역(AppConfig)을 구분하는게 눈에 띄네

> 리팩터링을 통해 DIP, OCP 준수하게 됨 


### 전체 흐름 정리 
(생략)


### 좋은 객체 지향 설계의 5가지 원칙의 적용 

**SRP, 단일 책임 원칙**
- 한 클래스는 하나의 책임만 가져야 한다
	- 클라이언트 객체는 직접 구현 객체를 생성하고, 연결하고, 실행하는 다양한 책임을 가지고 있음 
	- SRP 단일 책임 원칙을 따르면서 관심사를 분리함
	- 구현 객체를 생성하고 연결하는 책임은 AppConfig가 담당
	- 클라이언트 객체는 실행하는 책임만 담당

**DIP, 의존관계 역전 원칙**
- 저수준 모듈(구현체)에 의존하는게 아닌, 추상화된 고수준 모듈(인터페이스)에 의존해야 한다

**OCP, 개방폐쇄 원칙**
- 소프트웨어 요소는 확장에는 열려 있고, 변경에는 닫혀있어야 한다
	- 생성자 주입을 통해 DiscountPolicy 주입을 받아서 사용하게 된다
	- 현재 스프링 컨테이너 없이 AppConfig에서 의존 관계 설정을 담당하는데 여기서 FixDiscountPolicy -> RateDiscountPolicy 구현체로 변경할 경우 클라이언트 코드는 변경없이 기능을 확장 가능하다


**제어의 역전 IoC(Inversion of Control)**
- 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다. 
- 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다. 이는 개발자 입장에서는 자연스러운 흐름이다. 
- 반면에 AppConfig가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당한다 
- (..) 이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전이라 한다 


>[!info] 프레임워크 vs 라이브러리
>제어의 흐름을 직접 담당한다면 `라이브러리`, 대신 담당한다면 `프레임워크`이다


**의존 관계 주입, DI(Dependency Injection)**
- 정적인 클래스 의존 관계와 실행 시점에 결정되는 동적인 객체(인스턴스) 의존 관계 둘을 분리해서 생각해야 한다 
- **정적인 클래스 의존관계**
	- 클래스가 사용하는 import 코드만 보고 의존관계를 쉽게 판단할 수 있다
	- 정적인 의존 관계는 애플리케이션을 실행하지 않아도 분석할 수 있다
	- ex. 인터페이스 
		- 이때 어떤 구현체가 주입되는지는 알 수 없다
- **동적인 객체 인스턴스 의존관계**
	- 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존관계다
		- 외부에서 실제 구현 객체를 생성하고, 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관게가 연결되는 것을 **의존관계 주입**이라 한다
		- 객체 인스턴스를 생성하고, 그 참조값을 전달해서 연결된다
	- DI를 사용하면 
		- 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다. 
		- 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다 (OCP, DIP 원칙 준수)

---

## 섹션5. 스프링 컨테이너와 스프링 빈 
- `ApplicationContext`를 스프링 컨테이너라 한다
- 이때 `ApplicationContext`는 인터페이스이다
- 스프링 컨테이너는 XML 이나 애노테이션 기반의 자바 설정 클래스로 만들 수 있다
- 스프링 빈 저장소에 빈 이름과 빈 객체가 저장된다 
	- 빈 이름은 **메서드명**을 따른다
	- `@Bean(name="memberService2")`과 같이 직접 커스텀 부여 가능 


// 빈을 컨테이너에 등록하고 조회하는 예제를 살펴봄

**정리**
- ApplicationContext는 BeanFactory의 기능을 상속받는다 
- ApplicationContext는 빈 관리 기능 + 편리한 부가 기능을 제공한다
- BeanFactory를 직접 사용할 일은 거의 없다. 
	- 부가기능이 포함된 ApplicationContext를 사용한다
- BeanFactory나 ApplicationContext를 스프링 컨테이너라 한다 


ApplicationContext 구현체 중에
- AnnotationConfigApplicationContext는 클래스 기반으로 빈 등록 가능 
- GenericXmlApplicationContext는 xml 기반으로 빈 등록 가능
	- `appConfig.xml` 파일 참고
		- AppConfig.java 설정 정보와 거의 동일


**스프링 빈 설정 메타 정보 - BeanDefinition**
- 빈 설정 메타 정보를 가진다
	- 스프링 컨테이너는 메타정보를 기반으로 스프링 빈을 생성한다

**BeanDefinition 살펴보기**
- (정보 생략)

---

## 섹션6. 싱글톤 컨테이너 

**싱글톤 패턴의 문제점**
- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다 
- 의존관계상 클라이언트가 구체 클래스에 의존한다 ➡️ DIP를 위반한다 
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위한발 가능성이 높다 
	- `getInstance()` 를 직접 호출하니..
- 테스트하기 어렵다 
	- 리플렉션 취약
- 내부 속성을 변경하거나 초기화하기 어렵다 
- private 생성자로 인해 자식 클래스를 만들기 어렵다 
- 결론적으로 유연성이 떨어진다
- 안티패턴으로 불리기도 한다

> ✅ 스프링 컨테이너를 싱글톤 컨테이너라고도 한다. ➡️ 싱글톤 패턴의 문제점을 해결


> [!info] 참고
> 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것은 아니다. 요청할때 객체를 생성해서 반환하는 기능도 제공한다. (자세한 내용은 `빈 스코프`에서 다룸)

**싱글톤 방식의 주의점**
- 하나의 객체 인스턴스를 여러 클라이언트가 공유하기 때문에 `무상태(stateless)`로 설계해야 한다
	- 특정 클라이언트에 의존적인 필드가 있으면 안된다
	- 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다
	- 가급적 읽기만 가능해야 한다
	- 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다
- 💣 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애 발생 가능 !! 


// @Configuration 클래스에서 팩토리 메서드 호출시 싱글톤 유지되는지 테스트하는 예제 살펴봄 
// 스프링 컨테이너 사용시 **CGLIB** 통해 상속받아서 호출시 생성해서 반환하기 때문에 싱글톤 보장 
// 만약 스프링 컨테이너를 사용하지 않으면 매번 객체 인스턴스를 생성하게 됨 (참조 주소값이 틀림)

> 스프링 설정 정보를 다룰 때는 @Configuration을 사용하자

---
## 섹션 7. 컴포넌트 스캔

```java
package hello.core;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import static org.springframework.context.annotation.ComponentScan.
*;

@Configuration
@ComponentScan(excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class))
public class AutoAppConfig {
	
}
```
- MemberRepository 구현체 
	- `MemoryMemberRepository`
- DiscountPolicy 구현체 
	- `RateDiscountPolicy`
- MemberService 구현체 
	- `MemberServiceImpl`
- OrderService 구현체 
	- `OrderServiceImpl`
- 전부다 @Component와 @Autowired 애노테이션 사용해서 선언


```java
package hello.core.scan;
import hello.core.AutoAppConfig;
import hello.core.member.MemberService;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import
org.springframework.context.annotation.AnnotationConfigApplicationContext;
import static org.assertj.core.api.Assertions.
*;
public class AutoAppConfigTest {
	@Test
	void basicScan() {
		ApplicationContext ac = new
		AnnotationConfigApplicationContext(AutoAppConfig.class);
		MemberService memberService = ac.getBean(MemberService.class);
		
		assertThat(memberService).isInstanceOf(MemberService.class);
	}
}
```
- 스프링 컨테이너 생성 후 AutoAppConfig를 등록하게 되면 컴포넌트 스캔을 수행하여 빈 등록 
- `@ComponentScan` 은 `@Component` 가 붙은 모든 클래스를 스프링 빈으로 등록한다.
- 이때 스프링 빈의 기본 이름은 클래스명을 사용하되 맨 앞글자만 소문자를 사용한다.
	- **빈 이름 기본 전략** : MemberServiceImpl 클래스 ➡️ memberServiceImpl
	- **빈 이름 직접 지정** : 만약 스프링 빈의 이름을 직접 지정하고 싶으면`@Component("memberService2")` 이런식으로 이름을 부여하면 된다.
- `@Autowired` 의존관계 자동 주입
	- 기본 조회 전략은 타입이 같은 빈을 찾아서 주입하게 된다

```text
ClassPathBeanDefinitionScanner - Identified candidate component class:
.. RateDiscountPolicy.class
.. MemberServiceImpl.class
.. MemoryMemberRepository.class
.. OrderServiceImpl.class
```


### 탐색 위치와 기본 스캔 대상

```java
@ComponentScan(basePackages = "hello.core", .. }
```
- 

---
## 섹션8. 의존관계 자동 주입
### 다양한 의존관계 주입 방법
- 생성자 주입 
- 수정자 주입 (setter)
- 필드 주입 
- 일반 메서드 주입

`생성자 주입`
- 생성자 호출 시점에 딱 1번만 호출되는 것이 보장
- **불변, 필수** 의존관계에서 사용

>[!tip] 생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입된다. 물론 스프링 빈에만 해당

`수정자 주입(setter)`
- setter 메서드를 통해 주입 
- 선택, 변경 가능성이 있는 의존관계에 사용
- 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법

```java
@Component
public class OrderServiceImpl implements OrderService {
	private MemberRepository memberRepository;
	private DiscountPolicy discountPolicy;
	
	@Autowired
	public void setMemberRepository(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
	}
	
	@Autowired
	public void setDiscountPolicy(DiscountPolicy discountPolicy) {
		this.discountPolicy = discountPolicy;
	}
}
```
- 과거 자바빈 프로퍼티 규약에서는 setter, getter 사용하여 읽거나 수정 
	- 요즘에는 생성자 주입 방식이 선호됨
	- ORM 사용시 `setter` 통해서 연관관계 갱신을 하기도 함

> [!note] @Autowired 의 기본 동작은 주입할 대상이 없으면 오류가 발생한다. 
> 주입할 대상이 없어도 동작하게 하려면 @Autowired(required = false) 로 지정하면 됨


`필드 주입 방식`
- 코드가 간결해서 많은 개발자들을 유횩하지만 외부에서 변경이 불가능해서 테스트하기 힘든 단점이 있다
- DI 프레임워크가 없으면 아무것도 할 수 없다 (✅ 사용하지 말자)
	- 만약 사용한다면 
		- 애플리케이션의 실제 코드와 관계 없는 테스트 코드 
		- 스프링 설정을 목적으로 하는 @Configuration 같은 곳에만 특별한 용도로 사용하자

> 생각해보니 Property 주입 받을때나 사용했던거 같기도 하고


```java
@Component
public class OrderServiceImpl implements OrderService {
	@Autowired
	private MemberRepository memberRepository;
	
	@Autowired
	private DiscountPolicy discountPolicy;
}
```
- 순수 자바 테스트 코드에서는 당연히 @Autowired 동작하지 x
	- @SpringBootTest처럼 스프링 컨테이너를 테스트에 통합한 경우에만 가능하다

아래와 같이 @Bean에서 파라미터에 의존관계는 자동 주입된다. 수동 등록시 자동 등록된 빈의 의존 관계가 필요할때 문제를 해결할 수 있다 .

```java
@Bean
OrderService orderService(MemberRepository memberRepoisitory, DiscountPolicy
discountPolicy) {
	return new OrderServiceImpl(memberRepository, discountPolicy);
}
```

`일반메서드 주입`
- 한번에 여러 필드를 주입 받을 수 있다 
- 일반적으로 잘 사용하지 x
```java
@Component
public class OrderServiceImpl implements OrderService {
	private MemberRepository memberRepository;
	private DiscountPolicy discountPolicy;
	
	@Autowired
	public void init(MemberRepository memberRepository, DiscountPolicy
	discountPolicy) {
		this.memberRepository = memberRepository;
		this.discountPolicy = discountPolicy;
	}
}
```

> [!note] 당연한 얘기지만 의존관계 자동 주입은 스프링 컨테이너가 관리하는 스프링 빈만 가능하다.
> 스프링 Bean이 아닌 POJO 클래스의 경우 @Autowired 코드를 적용해도 아무 기능도 동작하지 않는다.


### 옵션 처리
주입할 빈이 없더라도 동작해야 할 때 아래와 같은 3가지 방법으로 처리 가능하다

- `@Autowired(required=false)` : 자동 주입 대상이 없으면 수정자 메서드 자체가 호출되지 x
- `org.springframework.lang.@Nullable` : 자동 주입 대상이 업스면 null이 입력된다
- `Optional<클래스명>` : 자동 주입 대상이 없으면 Optional.empty가 입력된다

```java

//호출 안됨
@Autowired(required = false)
public void setNoBean1(Member member) {
	System.out.println("setNoBean1 = " + member);
}

//null 호출
@Autowired
public void setNoBean2(@Nullable Member member) {
	System.out.println("setNoBean2 = " + member); // null
}

//Optional.empty 호출
@Autowired(required = false)
public void setNoBean3(Optional<Member> member) {
	System.out.println("setNoBean3 = " + member); // Optional.empty
}
```
- **Member**는 스프링 빈이 아니다
	- 고로 setNoBean1은 호출 자체가 되지 않는다


setter로 주입 받을 경우 테스트에서 초기화하지 않아 뒤늦게 NPE 발생 가능하다 !

```java
@Component
public class OrderServiceImpl implements OrderService {
	private final MemberRepository memberRepository;
	private final DiscountPolicy discountPolicy;
	
	@Autowired
	public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy
	discountPolicy) {
		this.memberRepository = memberRepository;
	}
	//...
}
```
- `final` 키워드 통해서 생성자 초기화시 에러 확인 가능 

```java
@Test
void createOrder() {
	OrderServiceImpl orderService = new OrderServiceImpl();
	orderService.createOrder(1L, "itemA", 10000);
}

```

>[!tip] 기억하자! 컴파일 오류는 세상에서 가장 빠르고, 좋은 오류다
>수정자 주입을 포함한 나머지 주입 방식은 모두 생성자 이후에 호출되므로, 필드에 `final` 키워드를 사용
>할 수 없다. 오직 생성자 주입 방식만 `final` 키워드를 사용할 수 있다


**📌정리**
- 생성자 주입 방식을 선택하는 이유는 여러가지가 있지만, 프레임워크에 의존하지 않고, 순수한 자바 언어의 특징을 잘 살리는 방법이기도 하다.
- 기본으로 생성자 주입을 사용하고, 필수 값이 아닌 경우에는 수정자 주입 방식을 옵션으로 부여하면 된다. 생성자 주입과 수정자 주입을 동시에 사용할 수 있다.
- 항상 생성자 주입을 선택해라! 그리고 가끔 옵션이 필요하면 수정자 주입을 선택해라. 필드 주입은 사용하지 않는 게 좋다.

### 최신 트렌드, lombok
```gradle
plugins {
	id 'org.springframework.boot' version '2.3.2.RELEASE'
	id 'io.spring.dependency-management' version '1.0.9.RELEASE'
	id 'java'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

//lombok 설정 추가 시작
configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

//lombok 설정 추가 끝
repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	//lombok 라이브러리 추가 시작
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testCompileOnly 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'
	//lombok 라이브러리 추가 끝
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
	exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
useJUnitPlatform()
}
```
1. Preferences(윈도우 File Settings) plugin lombok 검색 설치 실행 (재시작)
2. Preferences Annotation Processors 검색 Enable annotation processing 체크 (재시작)
3. 임의의 테스트 클래스를 만들고 @Getter, @Setter 확인


### 같은 타입의 컴포넌트 빈이 여러 가지 일 때
- 조회 대상 빈이 2개 이상일 때 해결 방법
	- 1. `@Autowired` 필드 명 매칭
	- 2. `@Qualifier` @Qualifier끼리 매칭 빈 이름 매칭
	- 3. `@Primary` 사용


**@Autowired** 매칭 정리
1. 타입 매칭
2. 타입 매칭의 결과가 2개 이상일 때 필드 명, 파라미터 명으로 빈 이름 매칭

`@Qualifier` 사용
- `@Qualifier` 는 추가 구분자를 붙여주는 방법이다. 주입시 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것은 아니다.

빈 등록시 @Qualifiter를 붙여 준다
```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {}

@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {}

```

주입 예시
```java
// 생성자 주입
@Autowired
public OrderServiceImpl(MemberRepository memberRepository,
@Qualifier("mainDiscountPolicy") DiscountPolicy
discountPolicy) {
	this.memberRepository = memberRepository;
	this.discountPolicy = discountPolicy;
}

// 수정자 주입
@Autowired
public DiscountPolicy setDiscountPolicy(@Qualifier("mainDiscountPolicy")
DiscountPolicy discountPolicy) {
	this.discountPolicy = discountPolicy;
}

```
`@Qualifier`로 주입할 때 `@Qualifier("mainDiscountPolicy")` 를 못찾으면 어떻게 될까? 그러면
mainDiscountPolicy라는 이름의 스프링 빈을 추가로 찾는다. 하지만 경험상 `@Qualifier` 는 `@Qualifier` 를 찾는 용도로만 사용하는게 명확하고 좋다.

아래와 같이 직접 빈 등록시에도 동일하게 사용할 수 있다.
```java
@Bean
@Qualifier("mainDiscountPolicy")
public DiscountPolicy discountPolicy() {
	return new ...
}
```

**@Qualifier 정리**
1. @Qualifier끼리 매칭
2. 빈 이름 매칭
3. `NoSuchBeanDefinitionException` 예외 발생


**@Primary 사용**
- 빈의 우선 순위를 정하는 방법이다.
- @Autowired시에 여러 빈이 매칭되면 @Primary가 우선권을 가진다

```java
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {}

@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```

```java

//생성자 주입의 경우
@Autowired
public OrderServiceImpl(MemberRepository memberRepository,
DiscountPolicy discountPolicy) {
	this.memberRepository = memberRepository;
	this.discountPolicy = discountPolicy;
}


//수정자의 경우
@Autowired
public DiscountPolicy setDiscountPolicy(DiscountPolicy discountPolicy) {
	this.discountPolicy = discountPolicy;
}

```


@Primary와  @Qualifier 중에 어떤 것을 사용하면 좋을지 고민이 될 것이다. 
- @Qualifier의 단점은 주입 받을 때 다음과 같이 모든 코드에 @Qualifiter를 붙여주어야 한다는 점이다. 
- 반면 @Primary의 경우 애노테이션을 빈쪽에 붙일 필요는 없다

@Primary, @Qualifier 활용
- 코드에서 자주 사용하는 메인 데이터베이스의 커넥션을 획득하는 스프링 빈이 있고, 코드에서 특별한 기능으로 가끔 사용하는 서브 데이터베이스의 커넥션을 획득하는 스프링 빈이 있다고 생각해보자. 메인 데이터베이스의 커넥션을 획득하는 스프링 빈은 `@Primary` 를 적용해서 조회하는 곳에서 `@Qualifier` 지정 없이 편리하게 조회하고, 서브 데이터베이스 커넥션 빈을 획득할 때는 `@Qualifier` 를 지정해서 명시적으로 획득 하는 방식으로 사용하면 코드를 깔끔하게유지할 수 있다. 물론 이때 메인 데이터베이스의 스프링 빈을 등록할 때 `@Qualifier` 를 지정해주는 것은 상관없다.

**우선순위**
- `@Primary` 는 기본값 처럼 동작하는 것이고, `@Qualifier` 는 매우 상세하게 동작한다. 이런 경우 어떤 것이 우선권을가져갈까? <u>스프링은 자동보다는 수동이, 넒은 범위의 선택권 보다는 좁은 범위의 선택권이 우선 순위가 높다. 따라서 여기서도 `@Qualifier` 가 우선권이 높다</u>
	- 구체적인게 우선 순위가 높다!


### 애노테이션 직접 만들기 
`@Qualifier("mainDiscountPolicy")` 이렇게 문자를 적으면 컴파일시 타입 체크가 안된다. 다음과 같은 애노테이션을 만들어서 문제를 해결할 수 있다.

```java
import org.springframework.beans.factory.annotation.Qualifier;
import java.lang.annotation.*;
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER,
ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}


// 애노테이션 추가
@Component
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {}


//생성자 자동 주입
@Autowired
public OrderServiceImpl(MemberRepository memberRepository,
@MainDiscountPolicy DiscountPolicy discountPolicy) {
	this.memberRepository = memberRepository;
	this.discountPolicy = discountPolicy;
}

//수정자 자동 주입
@Autowired
public DiscountPolicy setDiscountPolicy(@MainDiscountPolicy DiscountPolicy
discountPolicy) {
	this.discountPolicy = discountPolicy;
}
```

애노테이션에는 상속이라는 개념이 없다. 이렇게 여러 애노테이션을 모아서 사용하는 기능은 스프링이 지원해주는 기능이다. `@Qualifier` 뿐만 아니라 다른 애노테이션들도 함께 조합해서 사용할 수 있다. 단적으로 `@Autowired`도 재정의 할 수 있다. 물론 스프링이 제공하는 기능을 뚜렷한 목적 없이 무분별하게 재정의 하는 것은 유지보수에 더 혼란만 가중할 수 있다.

### 조회한 빈이 모두 필요할 때 (List, Map)

```java
package hello.core.autowired;
import hello.core.AutoAppConfig;
import hello.core.discount.DiscountPolicy;
import hello.core.member.Grade;
import hello.core.member.Member;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import
org.springframework.context.annotation.AnnotationConfigApplicationContext;
import java.util.List;
import java.util.Map;
import static org.assertj.core.api.Assertions.assertThat;
public class AllBeanTest {
	
	@Test
	void findAllBean() {
		ApplicationContext ac = new
		AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
		DiscountService discountService = ac.getBean(DiscountService.class);
		Member member = new Member(1L, "userA", Grade.VIP);
		int discountPrice = discountService.discount(member, 10000,
		"fixDiscountPolicy");
		
		assertThat(discountService).isInstanceOf(DiscountService.class);
		assertThat(discountPrice).isEqualTo(1000);
	}

static class DiscountService {
	private final Map<String, DiscountPolicy> policyMap;
	private final List<DiscountPolicy> policies;
	
	public DiscountService(Map<String, DiscountPolicy> policyMap,
	List<DiscountPolicy> policies) {
		this.policyMap = policyMap;
		this.policies = policies;
		System.out.println("policyMap = " + policyMap);
		System.out.println("policies = " + policies);
	}
	
	public int discount(Member member, int price, String discountCode) {
		DiscountPolicy discountPolicy = policyMap.get(discountCode);
		System.out.println("discountCode = " + discountCode);
		System.out.println("discountPolicy = " + discountPolicy);
		return discountPolicy.discount(member, price);
	}
}
}
```
**📌 주입 분석**
- `Map<String, DiscountPolicy>` : map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로 `DiscountPolicy` 타입으로 조회한 모든 스프링 빈을 담아준다.
- `List<DiscountPolicy>` : `DiscountPolicy` 타입으로 조회한 모든 스프링 빈을 담아준다.
- 만약 해당하는 타입의 스프링 빈이 없으면, 빈 컬렉션이나 Map을 주입한다.

**참고. 스프링 컨테이너를 생성하면서 스프링 빈 등록하기**

스프링 컨테이너는 생성자에 클래스 정보를 받는다. 여기에 클래스 정보를 넘기면 해당 클래스가 스프링 빈으로 자동 등록된다.`new AnnotationConfigApplicationContext(AutoAppConfig.class,DiscountService.class);`
- AutoAppConfig와 DiscountService가 스프링 빈 생성/등록된다


### 자동, 수동의 올바른 실무 운영 기준 

**편리한 자동 기능을 기본으로 사용하자**
그러면 어떤 경우에 컴포넌트 스캔과 자동 주입을 사용하고, 어떤 경우에 설정 정보를 통해서 수동으로 빈을 등록하고,의존관계도 수동으로 주입해야 할까?

결론부터 이야기하면, 스프링이 나오고 시간이 갈 수록 점점 자동을 선호하는 추세다. 스프링은 `@Component` 뿐만 아니라 `@Controller` , `@Service` , `@Repository` 처럼 계층에 맞추어 일반적인 애플리케이션 로직을 자동으로 스캔할 수 있도록 지원한다. 거기에 더해서 최근 스프링 부트는 컴포넌트 스캔을 기본으로 사용하고, 스프링 부트의 다양한 스프링 빈들도 조건이 맞으면 자동으로 등록하도록 설계했다.

설정 정보를 기반으로 애플리케이션을 구성하는 부분과 실제 동작하는 부분을 명확하게 나누는 것이 이상적이지만, 개발자 입장에서 스프링 빈을 하나 등록할 때 `@Component` 만 넣어주면 끝나는 일을 `@Configuration` 설정 정보에 가서 `@Bean` 을 적고, 객체를 생성하고, 주입할 대상을 일일이 적어주는 과정은 상당히 번거롭다.
- 또 관리할 빈이 많아서 설정 정보가 커지면 설정 정보를 관리하는 것 자체가 부담이 된다.
- 그리고 결정적으로 자동 빈 등록을 사용해도 OCP, DIP를 지킬 수 있다.

**그러면 수동 빈 등록은 언제 사용하면 좋을까?**

애플리케이션은 크게 업무 로직과 기술 지원 로직으로 나눌 수 있다.
- ****업무** **로직** **빈**:** 웹을 지원하는 컨트롤러, 핵심 비즈니스 로직이 있는 서비스, 데이터 계층의 로직을 처리하는 리포지토리등이 모두 업무 로직이다. 보통 비즈니스 요구사항을 개발할 때 추가되거나 변경된다.
- ****기술** **지원** **빈**:** 기술적인 문제나 공통 관심사(AOP)를 처리할 때 주로 사용된다. 데이터베이스 연결이나, 공통 로그처리 처럼 업무 로직을 지원하기 위한 하부 기술이나 공통 기술들이다.
- 업무 로직은 숫자도 매우 많고, 한번 개발해야 하면 컨트롤러, 서비스, 리포지토리 처럼 어느정도 유사한 패턴이 있다. 이런 경우 자동 기능을 적극 사용하는 것이 좋다. 보통 문제가 발생해도 어떤 곳에서 문제가 발생했는지 명확하게 파악하기 쉽다.
- 기술 지원 로직은 업무 로직과 비교해서 그 수가 매우 적고, 보통 애플리케이션 전반에 걸쳐서 광범위하게 영향을 미친다. 그리고 업무 로직은 문제가 발생했을 때 어디가 문제인지 명확하게 잘 드러나지만, 기술 지원 로직은 적용이 잘 되고 있는지 아닌지 조차 파악하기 어려운 경우가 많다. 그래서 이런 기술 지원 로직들은 가급적 수동 빈 등록을 사용해서 명확하게 드러내는 것이 좋다.

****애플리케이션에** **광범위하게** **영향을** **미치는** **기술** **지원** **객체는** **수동** **빈으로** **등록해서** **딱**! **설정** **정보에** **바로** **나타나게** **하는****것이** **유지보수** **하기** **좋다**.**


**비즈니스 로직 중에서 다형성을 적극 활용할 때**

의존관계 자동 주입 - 조회한 빈이 모두 필요할 때, List, Map을 다시 보자.

`DiscountService` 가 의존관계 자동 주입으로 `Map<String, DiscountPolicy>` 에 주입을 받는 상황을 생각해보자. 여기에 어떤 빈들이 주입될 지, 각 빈들의 이름은 무엇일지 코드만 보고 한번에 쉽게 파악할 수 있을까? 내가 개발했으니 크게 관계가 없지만, 만약 이 코드를 다른 개발자가 개발해서 나에게 준 것이라면 어떨까?

자동 등록을 사용하고 있기 때문에 파악하려면 여러 코드를 찾아봐야 한다.이런 경우 수동 빈으로 등록하거나 또는 자동으로하면 ****특정** **패키지에** **같이** **묶어****두는게 좋다! 핵심은 딱 보고 이해가 되어야 한다!

이 부분을 별도의 설정 정보로 만들고 수동으로 등록하면 다음과 같다.
```java
@Configuration
public class DiscountPolicyConfig {
	@Bean
	public DiscountPolicy rateDiscountPolicy() {
		return new RateDiscountPolicy();
	}
	
	@Bean
	public DiscountPolicy fixDiscountPolicy() {
		return new FixDiscountPolicy();
	}
}
```
이 설정 정보만 봐도 한눈에 빈의 이름은 물론이고, 어떤 빈들이 주입될지 파악할 수 있다. 그래도 빈 자동 등록을 사용하고 싶으면 파악하기 좋게 `DiscountPolicy` 의 구현 빈들만 따로 모아서 특정 패키지에 모아두자.

참고로 ****스프링과** **스프링** **부트가** **자동으로** **등록하는** **수** **많은** **빈들은** **예외****다. 이런 부분들은 스프링 자체를 잘 이해하고 스프링의 의도대로 잘 사용하는게 중요하다. 스프링 부트의 경우 `DataSource` 같은 데이터베이스 연결에 사용하는 기술 지원 로직까지 내부에서 자동으로 등록하는데, 이런 부분은 메뉴얼을 잘 참고해서 스프링 부트가 의도한 대로 편리하게 사용하면 된다. 반면에 ****스프링** **부트가** **아니라** **내가** **직접** **기술** **지원** **객체를** **스프링** **빈으로** **등록한다면** **수동으로** **등록해****서** **명확하게** **드러내는** **것이** **좋다**

**✅ 정리**
- 편리한 자동 기능을 기본으로 사용하자
- 직접 등록하는 기술 지원 객체는 수동 등록
- 다형성을 적극 활용하는 비즈니스 로직은 수동 등록을 고민해보자

---
## 섹션 9. 빈 생명주기 콜백
```java
package hello.core.lifecycle;

public class NetworkClient {
	private String url;
	
	public NetworkClient() {
		System.out.println("생성자 호출, url = " + url);
		connect();
		call("초기화 연결 메시지");
	}
	
	public void setUrl(String url) {
		this.url = url;
	}
	
	//서비스 시작시 호출
	public void connect() {
		System.out.println("connect: " + url);
	}
	
	public void call(String message) {
		System.out.println("call: " + url + " message = " + message);
	}
	
	//서비스 종료시 호출
	public void disconnect() {
		System.out.println("close: " + url);
	}
}
```


```java
package hello.core.lifecycle;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import
org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

public class BeanLifeCycleTest {
	
	@Test
	public void lifeCycleTest() {
		ConfigurableApplicationContext ac = new
		AnnotationConfigApplicationContext(LifeCycleConfig.class);
		NetworkClient client = ac.getBean(NetworkClient.class);
		ac.close(); //스프링 컨테이너를 종료, ConfigurableApplicationContext 필요
	}

	@Configuration
	static class LifeCycleConfig {
		@Bean
		public NetworkClient networkClient() {
			NetworkClient networkClient = new NetworkClient();
			networkClient.setUrl("http://hello-spring.dev");
			return networkClient;
		}
	}
}
```

```text
생성자 호출, url = null
connect: null
call: null message = 초기화 연결 메시지
```
- 생성자 초기화시 url이 주입되지 않은 상태
	- Bean 초기화시 생성자 초기화후 setter 호출하여 위와 같은 메시지 출력


스프링 빈은 간단하게 다음과 같은 라이프사이클을 가진다.

> 객체 생성 ➡️ 의존관계 주입

스프링 빈은 객체를 생성하고, 의존관계 주입이 다 끝난 다음에야 필요한 데이터를 사용할 수 있는 준비가 완료된다. 따라서 초기화 작업은 의존관계 주입이 모두 완료되고 난 다음에 호출해야 한다. 그런데 개발자가 의존관계 주입이 모두 완료된 시점을 어떻게 알 수 있을까? (초기화와 종료 시점에 콜백 제공📌)

****스프링은** **의존관계** **주입이** **완료되면** **스프링** **빈에게** **콜백** **메서드를** **통해서** **초기화** **시점을** **알려주는** **다양한** **기능을** **제공****한다. 또한 ****스프링은** **스프링** **컨테이너가** **종료되기** **직전에** **소멸** **콜백****을 준다. 따라서 안전하게 종료 작업을 진행할 수 있다.

**스프링 빈의 이벤트 라이프 사이클**
> 스프링 컨테이너 생성 ➡️ 스프링 빈 생성 ➡️ 의존관계 주입 ➡️ 초기화 콜백 ➡️ 사용 ➡️ 소멸전 콜백 ➡️ 스프링 종료
- 초기화 콜백 : 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출 
- 소멸전 콜백: 빈이 소멸되기 직전에 호출

> [!note] 객체의 생성과 초기화를 분리하자 
> 생성자는 필수 정보(파라미터)를 받고, 메모리를 할당해서 객체를 생성하는 책임을 가진다. 반면에 초기화는 이렇게 생성된 값들을 활용해서 외부 커넥션을 연결하는등 무거운 동작을 수행한다. 따라서 생성자 안에서 무거운 초기화 작업을 함께 하는 것 보다는 객체를 생성하는 부분과 초기화 하는 부분을 명확하게 나누는 것이 유지보수 관점에서 좋다. 물론 초기화 작업이 내부 값들만 약간 변경하는 정도로 단순한 경우에는 생성자에서 한번에 다 처리하는게 더 나을 수 있다

>[!note] 
> 싱글톤 빈들은 스프링 컨테이너가 종료될 때 싱글톤 빈들도 함께 종료되기 때문에 스프링 컨테이너가 종료
되기 직전에 소멸전 콜백이 일어난다. 뒤에서 설명하겠지만 싱글톤 처럼 컨테이너의 시작과 종료까지 생존하는 빈도 있지만, 생명주기가 짧은 빈들도 있는데 이 빈들은 컨테이너와 무관하게 해당 빈이 종료되기 직전에 소멸전 콜백이 일어난다. 자세한 내용은 스코프에서 알아보겠다.

**스프링은 크게 3가지 방법으로 빈 생명주기 콜백을 지원한다**
- 인터페이스(InitializingBean, DisposableBean)
- 설정 정보에 초기화 메서드, 종료 메서드 지정
- @PostConstruct, @PreDestroy 애노테이션 지원

### InitializingBean, DisposableBean 방식 

```java
package hello.core.lifecycle;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
public class NetworkClient implements InitializingBean, DisposableBean {
	private String url;
	
	public NetworkClient() {
		System.out.println("생성자 호출, url = " + url);
	}
	
	public void setUrl(String url) {
		this.url = url;
	}
	
	//서비스 시작시 호출
	public void connect() {
		System.out.println("connect: " + url);
	}
	
	public void call(String message) {
		System.out.println("call: " + url + " message = " + message);
	}
	
	//서비스 종료시 호출
	public void disConnect() {
		System.out.println("close + " + url);
	}
	
	@Override
	public void afterPropertiesSet() throws Exception {
		connect();
		call("초기화 연결 메시지");
	}
	
	@Override
	public void destroy() throws Exception {
		disConnect();
	}
}
```

```text
생성자 호출, url = null
NetworkClient.afterPropertiesSet
connect: http://hello-spring.dev
call: http://hello-spring.dev message = 초기화 연결 메시지
13:24:49.043 [main] DEBUG
org.springframework.context.annotation.AnnotationConfigApplicationContext -
Closing NetworkClient.destroy
close + http://hello-spring.dev
```
- url이 생성자 초기화에서는 null이지만 setter 통해 주입되서 확인 가능하다
- 그리고 스프링 컨테이너의 종료가 호출되자 소멸 메서드가 호출 된 것도 확인할 수 있다.

**인터페이스 방식의 단점**
- 이 인터페이스는 스프링 전용 인터페이스다. 해당 코드가 스프링 전용 인터페이스에 의존한다.
- 초기화, 소멸 메서드의 이름을 변경할 수 없다
- 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다 

> 이 방식은 **✅ 스프링 초창기에 나온 방법**이고, 다음의 더나은 방법들이 있어 거의 사용하지 않는다


### 빈 등록 초기화, 소멸 메서드 지정

@Configuration 클래스 정의시 설정 정보에 `@Bean(initMethod = "init", destroyMethod = "close")` 처럼 초기화, 소멸 메서드를 지정할 수 있다.

```java
package hello.core.lifecycle;
public class NetworkClient {
private String url;
public NetworkClient() {
System.out.println("생성자 호출, url = " + url);
}
public void setUrl(String url) {
this.url = url;
}
//서비스 시작시 호출
public void connect() {
System.out.println("connect: " + url);
}
public void call(String message) {
System.out.println("call: " + url + " message = " + message);
}
//서비스 종료시 호출
public void disConnect() {
System.out.println("close + " + url);
}
public void init() {
System.out.println("NetworkClient.init");
connect();
call("초기화 연결 메시지");
}
public void close() {
System.out.println("NetworkClient.close");
disConnect();
}
}
```

```java
@Configuration
static class LifeCycleConfig {
	
	@Bean(initMethod = "init", destroyMethod = "close")
	public NetworkClient networkClient() {
		NetworkClient networkClient = new NetworkClient();
		networkClient.setUrl("http://hello-spring.dev");
		return networkClient;
	}
}
```

**동일 결과**
```text
생성자 호출, url = null
NetworkClient.init
connect: http://hello-spring.dev
call: http://hello-spring.dev message = 초기화 연결 메시지
13:33:10.029 [main] DEBUG
org.springframework.context.annotation.AnnotationConfigApplicationContext -
Closing NetworkClient.close
close + http://hello-spring.dev
```

**설정 정보 방식의 특징** 
- 메서드 이름을 자유롭게 줄 수 있다.
- 스프링 빈이 스프링 코드에 의존하지 않는다.
- 코드가 아니라 설정 정보를 사용하기 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다

**종료 메서드 추론 지원**
- @Bean의 destroyMethod` 속성에는 아주 특별한 기능이 있다.
- 라이브러리는 대부분 `close` , `shutdown` 이라는 이름의 종료 메서드를 사용한다.
- @Bean의 `destroyMethod` 는 기본값이 `(inferred)` (추론)으로 등록되어 있다.
- 이 추론 기능은 `close` , `shutdown` 라는 이름의 메서드를 자동으로 호출해준다. 이름 그대로 종료 메서드를 추론해서 호출해준다.
- 따라서 직접 스프링 빈으로 등록하면 종료 메서드는 따로 적어주지 않아도 잘 동작한다.
- 추론 기능을 사용하기 싫으면 `destroyMethod=""` 처럼 빈 공백을 지정하면 된다


### 애노테이션 방식, @PostConstruct, @PreDestory

```java
package hello.core.lifecycle;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
public class NetworkClient {
private String url;
public NetworkClient() {
System.out.println("생성자 호출, url = " + url);
}
public void setUrl(String url) {
this.url = url;
}
//서비스 시작시 호출
public void connect() {
System.out.println("connect: " + url);
}
public void call(String message) {
System.out.println("call: " + url + " message = " + message);
}
//서비스 종료시 호출
public void disConnect() {
System.out.println("close + " + url);
}
@PostConstruct
public void init() {
System.out.println("NetworkClient.init");
connect();
call("초기화 연결 메시지");
}
@PreDestroy
public void close() {
System.out.println("NetworkClient.close");
disConnect();
}
}
```

```java
@Configuration
static class LifeCycleConfig {
	@Bean
	public NetworkClient networkClient() {
		NetworkClient networkClient = new NetworkClient();
		networkClient.setUrl("http://hello-spring.dev");
		return networkClient;
	}
}

```

**동일 실행 결과**
```text
생성자 호출, url = null
NetworkClient.init
connect: http://hello-spring.dev
call: http://hello-spring.dev message = 초기화 연결 메시지
19:40:50.269 [main] DEBUG
org.springframework.context.annotation.AnnotationConfigApplicationContext -
Closing NetworkClient.close
close + http://hello-spring.dev
```

애노테이션(`@PostConstruct`, `@PreDestory`) 방식의 특징
- **최신 스프링에서 가장 권장하는 방식**
- 애노테이션 하나만 붙이면 되므로 매우 편리하다. 패키지를 잘 보면 `javax.annotation.PostConstruct` 이다. 스프링에 종속적인 기술이 아니라 JSR-250라는 자바 표준이다. 따라서 스프링이 아닌 다른 컨테이너에서도 동작한다.
- 컴포넌트 스캔과 잘 어울린다.
- 유일한 단점은 외부 라이브러리에는 적용하지 못한다는 것이다. 
	- 외부 라이브러리를 초기화, 종료 해야 하면 @Bean의 기능을 사용하자.

**최종 정리**
- `@PostConstruct`, `@PreDestroy` 애노테이션을 사용하자
- 코드를 고칠 수 없는 외부 라이브러리를 초기화, 종료해야 하면 `@Bean` 의 `initMethod` , `destroyMethod` 를 사용하자

---
## 섹션 10. 빈 스코프



