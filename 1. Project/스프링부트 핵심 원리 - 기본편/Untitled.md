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
- AppConfig만 수정하면 된다 

> 사용 영역과 구성 영역(AppConfig)을 구분하는게 눈에 띄네

> 리팩터링을 통해 DIP, OCP 준수하게 됨 


### 전체 흐름 정리 
(생략)


### 좋은 객체 지향 설계의 5가지 원칙의 적용 

**SRP, 단일 책임 원칙**
- ..

**DIP, 의존관계 역전 원칙**
- ..

**OCP, 개방폐쇄 원칙**
- ..

