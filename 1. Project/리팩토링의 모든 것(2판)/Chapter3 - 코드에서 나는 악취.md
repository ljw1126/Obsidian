> 냄새 나면 당장 갈아라. - 켄트 백 할머니의 육아 원칙

'적용 방법'을 아는 것과 '제때적용'할 줄 아는 것은 다르다
리팩터링을 언제 시작하고 언제 그만할지를 판단하는 일은 리팩터링의 작동 원리를 아는 것 못지 않게 중요하다.

리택터링할 '시점'을 설명하는데 '냄새(smell)'란 표현을 사용

하지만 리팩터링을 언제 멈춰야 하는지를 판단하는 정확한 기준을 제시하지는 않을 것이다. 
우리 경험에 따르면 숙련된 사람의 직관만큼 정확한 기준은 없다. 
-> 종료 기준보다 리팩터링하면 해결할 수 있는 문제의 징후를 제시

### 3.1 기이한 이름(Mysterious Name)

함수, 모듈, 변수, 클래스 등은 그 이름만 보고도 각각이 무슨 일을 하고 어떻게 사용해야 하는지 명확히 알 수 있도록 엄청나게 신경써서 이름을 지어야 한다.

**리팩터링 종류**
- 함수 선언 바꾸기  (책 6.5)
- 변수 이름 바꾸기  (책 6.7)
- 필드 이름 바꾸기 

이름 바꾸기는 단순히 이름을 다르게 표현하는 연습이 아니다. 마땅한 이름이 떠오르지 않는다면 설계에 더 근본적인 문제가 숨어 있을 가능성이 높다. 그래서 혼란스러운 이름을 잘 정리하다 보면 코드가 훨씬 간결해질 때가 많다.

👨🏻‍💻 추가 조사

|       | 변수 이름 바꾸기                 | 필드 이름 바꾸기                  |
| ----- | ------------------------- | -------------------------- |
| 적용 대상 | 메서드 내부의 로컬 변수(임시변수), 매개변수 | 클래스의 멤버 변수(필드)             |
| 영향 범위 | 메서드 내부에서만 적용됨             | 클래스 전체에서 사용됨               |
| 목적    | 가독성 향상, 변수의 역할을 명확히 표현    | 객체가 가지는 고유한 속성 명칭을 명확하게 표현 |

**변수(Variable)**
- 값을 저장하는 공간
- 자료를 저장할 수 있는 이름이 주어진 기억장소
- 저장된 값을 잘 나타낼 수 있는 이름을 지어줘야 한다
	- 무엇을 담고 있는지 잘 나타낼 수 있어야 한다
	- **명사**로 짓는다
	- 카멜 케이스 사용 (db는 스네이크 케이스 사용해서 분리)
- 이때 <u>구체적일 수록, 의미 있는 이름</u>일수록 좋다
- 네이밍 예시 
	- student
	- students
	- apple
	- readApple
	- activatedStudent
	- 이때 관사(a/an)는 사용하지 않는다

리팩터링 전
```java
int a = height * witdh;

String cpyNm = "애플";

String tpHd = "제목없음";
```

리팩터링 후
```java
int area = height * witdh;

String companyName = "애플";

String title = "제목없음";
```

> 함수 안에서 반환되는 결과값을 result로 짓기도 한다




**함수(Function)**
- 특정한 일을 수행하는 코드의 집합
- 프로그램을 이루는 가장 기본적인 단위
- 프로그램을 작은 단위로 나누는 주된 수단
- <u>의미있는 이름을 부여하여 함수 이름만으로 코드 목적이 파악되도록 한다</u>
	- 재사용성 향상, 가독성 향상
	- 함수는 무엇을 하는지 잘 나타낼 수 있어야 한다 (목적)
	- **동사**로 짓는다
- 네이밍 예시 
	- calculate()
	- displayBanner()
	- getTotalPrice
	- totalPrice : 클래스 안에서 getter를 가지는 속성(필드) 경우 get 생략 가능
- 그리고 함수에 **매개변수, 인자**는 어떤 데이터가 필요한지 잘 나타낼 수 있어야 한다

Customer 클래스 (리팩터링 1판)
- getinvcdtlmt(..) -> getInvoiceableCreditLimit(..) 
- 메소드의 이름에 목적이 들어나도록 한다

p314 리팩터링 1판
`getTelephoneNumber()` 를 `getOfficeTelephoneNumber()` 로 바꿀 경우
```java
public String getTelephoneNumber() {
	return ("(" + officeAreaCode + ")" + officeNumber);
}

```

**절차**
1. 기존 메서드와 변경하는 메서드를 공존시킨다
	1. 이때 원래 메소드 몸체를 새로운 메소드로 복사한다
2. 원래 메소드는 새로운 메소드를 호출하도록 한다
3. 테스트가 통과하면 원래 메소드 호출하는 부분을 찾아, 새로운 메소드를 호출하도록 변경한다
4. 모든 부분을 바꾸고 난 후, 원래 메소드를 없앤다 (인텔리제이 기능 이용)

> 파라미터를 추가/삭제해야하는 경우도 절차가 동일하다


### 3.2 중복 코드 (Duplicated Code)

- 함수 추출하기
- 문장 슬라이드하기
	- 비슷한 부분을 한 곳에 모아 함수 추출하기를 더 쉽게 적용할 수 있는지 살펴본다
- 메서드 올리기
	- 같은 부모로부터 파생된 서브 클래스들에 코드 중복이 있다면, 각자 따로 호출되지 않도록 부모로 옮긴다

// 1장 예제 참고 가능
// 탬플릿 메소드 패턴이 생각난다 메서드 올리기는..

**도서 참고.**
- 7장 DB를 활용해 데이터를 영구적으로 저장하기
- JdbcTemplate를 직접 만들어가는 과정
- https://dev-ljw1126.tistory.com/404

### 3.3 긴 함수 (Long Function)

간접 호출(indirection) 효과, 즉 코드를 이해하고, 공유하고, 선택하기 쉬워진다는 장점은 함수를 짧게 구성할 때 나오는 것이다.

함수가 길수록 이해하기 어렵다
- 요즘 언어 프로세스 안에서의 함수 호출 비용을 거의 없애 버렸다
- 하지만 읽는 사람 입장에서 함수가 하는 일을 파악하기 위해 스크롤 피로감 증가

짧은 함수로 구성된 코드를 이해하기 쉽게 만드는 가장 확실한 방법은 좋은 이름이다. 
함수 이름을 잘 지어두면 본문 코드를 볼 이유가 사라진다.
그러기 위해서는 훨씬 적극적으로 함수를 쪼개야 한다. 

예. 주석을 달아야 할 만한 부분은 무조건 함수로 만든다
- 함수 본문에는 원래 주석으로 설명하려던 코드가 담기고, 함수 이름은 동작 방식이 아닌 의도(intention)가 드러나게 짓는다
- 이렇게 함수로 묶는 코드는 여러 줄일 수 있고 단 한 줄일 수도 있다.
- 심지어 원래 코드보다 길어지더라도 함수로 뽑는다.
- 단, **함수 이름에 코드의 목적을 드러낸다**
- 즉, <u>'무엇을 하는지'를 코드가 잘 설명해주지 못할수록 함수로 만드는게 유리하다</u>

<u>기본적으로 함수 추출하기가 99% 차지</u>

매개변수와 임시 변수가 너무 많은 경우 
- **임시 변수를 질의 함수로 바꾸기**로 임시 변수의 수를 줄임
- **매개변수 객체 만들기**, **객체 통째로 넘기기**로 매개변수의 수를 줄임
- 여전히 많다면 **함수를 명령으로 바꾸기**(11.9)를 고려       //?

추출할 코드 덩어리를 찾는 방법
- 좋은 방법은 주석을 참고하는 것
	- 주석은 코드만으로는 목적을 이해하기 어려운 부분에 달려있는 경우가 많다
	- 함수로 빼내고, 함수 이름을 주석 내용을 토대로 지음
- 조건문이나 반복문도 추출 대상의 실마리 제공
	- **조건문 분해하기**(10.1)         // ?
	- **함수 추출하기** : switch문의 각 case 본문을 함수 호출문 하나로 바꾼다
	- **조건부 로직을 다형성으로 바꾸기** : 같은 조건을 기준으로 나뉘는 switch문이 여러 개인 경우
- 반복문
	- 반복문 자체를 추출해서 독립된 함수로 만든다
	- **반복문 쪼개기** : 추출할 반복문 코드에 적합한 이름이 떠오르지 않는다면 성격이 다른 두 가지 작업이 섞여 있을 수도 있다. // 1장 예제 참고

**함수를 명령으로 바꾸기(11.9)**
```javascript 
// 리팩터링 전
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  // do something
}


// 리팩터링 후
class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
     this._candidate = candidate;
     this._medicalExam = medicalExam;
     this._scoringGuide = scoringGuide;
  }

  execute() {
     this._result = 0;
     this._healthLevel = 0;
     // do somethign
  }
}

```


**조건문 분해하기 (10.1)**
```javascript 
// 리팩터링 전
if(!aDate.isBefore(plan.summerStar) && !aDate.isAfter(plan.summerEnd)) 
  charge = quantity * plan.summerRate;
else 
  charge = quantity * plan.regularRate + plan.regularServiceCharge;


// 리팩터링 후
if(summer()) 
  charge = summerCharge();
else 
  charget = regularCharge();

```



### 3.4 긴 매개변수 목록 (Long Parameter List)
종종 다른 매개변수에서 값을 얻어올 수 있는 매개변수가 있을 때
- **매개변수를 질의 함수로 바꾸기로 제거**(11.5)할 수 있다 // ?

데이터 구조에서 값들을 뽑아 각각 별개의 매개변수로 전달하는 경우
- **객체 통째로 넘기기**로 원본 데이터 구조 그대로 전달

항상 함께 전달되는 매개변수들은 
- **매개변수 객체로 만들기**로 하나로 묶는다

함수의 동작 방식을 정하는 플래그 역할의 매개변수는 
- **플래그 인수 제거하기**(11.3)로 없애준다  // ??

클래스는 매개변수 목록을 줄이는 데 효과적인 수단이다. 특히 여러 개의 함수가 특정 매개변수들의 값을 공통 사용할 때 유용하다
- 이 경우 **여러 함수를 클래스로 묶기**(6.9) 이용하여 공통 값들을 클래스의 필드로 정의한다  // ?


**매개변수를 질의 함수로 바꾸기로 제거**(11.5) // 매개변수 줄이는 게 목적
```javascript
// 리팩터링 전
availableVacation(anEmployee, anEmployee.grade);

function availabelVacation(anEmployee, grade) {
    // 연휴 계산
}


// 리팩터링 후
availableVacation(anEmployee);

function availabelVacation(anEmployee) {
    const grade = anEmployee.grade;
    // 연휴 계산
}
```


**플래그 인수 제거하기**(11.3)
- 조건이 복잡한 경우는 더 찾아보기
```javascript
// 리팩터링 전
function setDimension(name, value) {
  if(name === "height") {
    this._height = value;
	return;
  }

  if(name === "width") {
    this._width = value;
	return;
  }
}


// 리팩터링 후
function setHeight(value) {this._height = value;}
function setWidth(value) {this._width = value;}
```


**여러 함수를 클래스로 묶기**(6.9)
- 연관성 있는 함수들을 모은다
- ex. 일급 컬렉션
```javascript
// 리팩터링 전
function base(aReading) {...}
function taxableCharge(aReading) {...}
function calculateBaseCharge(aReading) {...}


// 리팩터링 후, aReading은 생성자 주입하여 사용하는 듯
class Reading {
 base() {...}
 taxableCharge() {...}
 calculateBaseCharge() {...}
}

```


### 3.5 전역 데이터 (Global Data)

> "전역 데이터는 이를 함부로 사용한 프로그래머들에게 벌을 주는 지옥 4층에 사는 악마들이 만들었다"

**내 생각**
- 멀티 스레드 환경에서 동시성 이슈, 데이터 정합성 문제 등을 일으킬 수 있다
- 디버깅이 어려워서 문제 원인을 파악하는데 시간 많이 소요
- 좋은 예시로는 정규 표현식 객체, 아토믹한 원자 객체를 사용하는 거 생각나네 , 싱글톤이면 enum
- 불변 객체를 만들거나 

p117
전역 데이터는 코드 베이스 어디에서든 건드릴 수 있고 값을 누가 바꿨는지 찾아낼 매커니즘이 없다는게 문제다

p118
전역 데이터의 대표적인 형태는 전역 변수지만 클래스 변수와 싱글톤에서 같은 문제가 발생한다.  // ?

이를 방지하기 위해 우리가 사용하는 대표적인 리팩터링은 **변수 캡슐화하기**(6.6)이다 // ?, 불변 객체?
다른 코드에서 오염시킬 가능성이 있는 데이터를 발견할 때마다 이 기법을 가장 먼저 적용한다.
이런 데이터를 함수로 감싸는 것만으로도 데이터를 수정하는 부분을 쉽게 찾을 수 있고 접근을 통제할 수 있게 된다. 더 나아가 접근자 함수들을 클래스나 모듈에 집어 넣고 그 안에서만 사용할 수 있도록 접근 범위를 최소로 줄이는 것도 좋다

// 자동차 게임에서 전역 변수로 둘 경우, 멀티스레드 접근시 문제 가능하지 않을까? 

전역 데이터가 아주 조금만 있더라도 캡슐화하는 편이다. 그래야 s/w가 진화하는 데 따른 변화에 대처할 수 있다.

> 변수 캡슐화하기(6.6) 다시 읽어보기, 불변 객체라는건지.. 자바스크립트라서 덜 이해되네
### 3.6 가변 데이터 (Mutable Data)
데이터를 변경했더니 예상치 못한 결과나 골치 아픈 버그로 이어지는 경우가 종종있다. 
특히 이 문제가 아주 드문 조건에서만 발생한다면 원인을 알아내기가 매우 어렵다.
이런 이유로 함수형 프로그래밍에서느느 데이터는 절대 변하지 않고, 데이터를 변경하려면 반드시(원래 데이터는 그대로 둔채) 변경하려는 값에 해당하는 복사본을 만들어서 반환한다는 개념을 기본으로 삼고 있다

불변성이 주는 장점을 포기할 필요는 없다. <u>무분별한 데이터 수정에 따른 위험을 줄이는 방법은 얼마든지 있다</u>
- **변수 캡슐화하기** : 정해 놓은 함수를 거쳐야만 값을 수정할 수 있도록하면 관리하기 쉬워짐
- **변수 쪼개기** : 하나의 변수에 용도가 다른 값들을 저장하느라 값을 갱신하는 경우, 용도별로 독립 변수에 저장하게 하여 값 갱신이 문제를 일으킬 여지를 없앤다. 갱신 로직은 다른 코드와 떨어뜨려 놓는 것이 좋다
- **문장 슬라이드하기**, **함수 추출하기** : 무언가를 갱신하는 코드로부터 부작용이 없는 코드를 분리한다
- **질의 함수와 변경 함수 분리하기** 
- **세터 제거하기(기본)
- **파생 변수를 질의함수로 바꾸기**
- **여러 함수를 클래스로 묶기**, **여러 함수를 변환 함수로 묶기** : 변수를 갱신하는 코드들의 유효범위를 제한
- **참조를 값으로 바꾸기** : 구조체처럼 내부 필드에 데이터를 담고 있는 변수라면, 내부 필드를 직접 수정하지 말고 구조체를 통째로 교체하는 편이 낫다


**질의 함수와 변경 함수 분리하기**(11.1)
```javascript 
// 리팩터링 전, 메서드가 두 가지 책임을 가지고 있네
function getTotalOutstandingAndSendBill() {
	const result = customer.invoices.reduce((total, each) => each.amount + total, 0); // 질의 함수
	sendBill(); // 변경 함수
	return result;
}


// 리팩터링 후, 아마 클래스 필드 사용하는 듯
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);
}

function sendBill() {
  emailGateway.send(formatBill(customer));
}
```

**파생 변수를 질의함수로 바꾸기**(9.3)
- 가변 데이터를 다루는 방법을 나타내내
```javascript
// 리팩터링 전
get discountedTotal() { return this._discountedTotal;}

set discount(aNumber) {
  const old = this._distcount;
  this._discount = aNumber;
  this._discountedTotal += old - aNumber;
}


// 리팩터링 후
get discountedTotal() {return this._baseTotal - this._discount;}
set discount(aNumber) {this._discount = aNumber;}
```

**여러 함수를 변환 함수로 묶기**(6.10)
- 깊은 복사해서 연산한 결과를 할당하고 반환하네
- 여러개가 계산되고 할당되어야 하면 한군데서 모으는게 나은듯
```javascript
// 리팩터링 전
function base(aReading) {...}
function taxableCharge(aReading) {...}


// 리팩터링 후
function enrichReading(argReading) {
  const aReading = _.cloneDeep(argReading);
  aReading.baseCharge = base(aReading);
  aReading.taxableCharge = taxableCharge(aReading);
  return aReading;
}
```

**참조를 값으로 바꾸기**(9.4)
- 참조로 다루는 경우에는 내부 객체는 그대로 둔 채 그 객체의 속성만 갱신하며, 값으로 다루는 경우에는 새로운 속성을 담은 객체로 기존 내부 객체를 통째로 대체한다(불변)
- 반대 리팩터링: 값을 참조로 바꾸기(9.5)
```javascript
// 리팩터링 전
class Product {
  applyDiscount(arg) {this._price.amount -= arg;}
}


// 리팩터링 후
class Product {
  applyDiscount(arg) {
    this._price = new Money(this._price.amount - arg, this._price.currency);
  }
}
```


### 3.7 뒤엉킨 변경(Divergent Change)
코드를 수정할 때는 시스템에서 고쳐야 할 딱 한 군데를 찾아서 그 부분만 수정할 수 있기를 바란다. 
이렇게 할 수 없다면 (서로 밀접한 악취인) **뒤엉킨 변경**과 **산탄총 수술** 중 하나가 풍긴다 // 상속?

뒤엉킨 변경은 SRP가 제대로 지켜지지 않을 때 나타난다. 
<u>즉, 하나의 모듈이 서로 다른 이유들로 인해 여러 가지 방식으로 변경되는 일이 많을때 발생한다</u>

ex. 데이터베이스가 추가될 때마다 함수 세개를 바꿔야 하고, 금융 상품이 추가될 때마다 또 다른 함수 네 개를 바꿔야 하는 모듈이 있다면 **뒤엉킨 변경이 발생**했다는 뜻이다

- 데이터 베이스 연동과 금융 상품 처리는 <u>서로 다른 맥락</u>에서 이뤄지므로 독립된 모듈로 분리해야 프로그래밍이 편하다.
	- 개발 초기에는 맥락 사이의 경계를 명확히 나누기가 어렵고 s/w 시스템의 기능이 변경되면서 이 경계도 끊임없이 움직이기 때문이다.
- 권장 리팩터링 기법
	- **단계 쪼개기** : 다음 맥락에 필요한 데이터를 특정한 데이터 구조에 담아 전달하게 하는 식으로 단계를 분리
	- **함수 옮기기** : 전체 처리 과정 곳곳에서 각기 다른 맥락의 함수를 호출하는 빈도가 높다면, 각 맥락에 해당하는 적당한 모듈들을 만들어서 관련 함수들을 모은다 <u>(캡슐화?)</u>.  처리 과정이 매락별로 구분됨
	- **함수 추출하기** : 여러 맥란의 일에 관여하는 함수가 있다면 옮기기 전에 추출한다
	- **클래스 추출하기** : 모듈이 클래스라면 클래스 추출하여 맥락별 분리 방법을 잘 안내해 줄 것이다


### 3.8 산탄총 수술(Shotgun Surgery)
산탄총 수술은 뒤엉킨 변경과 비슷하면서도 정반대다.
이 냄새는 코드를 변경할 때마다 자잘하게 수정해야 하는 클래스가 많을 때 풍긴다.

p120 하단 표

|           | 뒤엉킨 변경        | 산탄총 수술      |
| --------- | ------------- | ----------- |
| 원인        | 맥락을 잘 구분하지 못함 |             |
| 해법(원리)    | 맥락을 명확히 구분    |             |
| 발생 과정(현상) | 한 코드에 섞여 들어감  | 여러 코드에 흩뿌려짐 |
| 해법(실제 행동) | 맥락별로 분리       | 맥락별로 모음     |

// 뒤엉킨 변경은 나누기고 산탄총 수술은 합치기네
// 응집도와 결합도와 연관지어 설명할 수 있을까?

- **함수 옮기기**, **필드 옮기기** : 함께 변경되는 대상들을 모두 한 모듈에 묶기
- **여러 함수를 클래스로 묶기** : 비슷한 데이터를 다루는 함수가 많은 경우
- **여러 함수를 변환 함수로 묶기** : 데이터 구조를 변환하거나 보강하는 함수에 적용
- **단계 쪼개기** : 이렇게 묶은 함수들의 출력 결과를 묶어서 다음 단계의 로직으로 전달할 수 있다면 적용
- 그 외
	- **함수 인라인하기**, **클래스 인라인하기** : 인라인 리팩터링으로 어설프게 분리된 로직을 하나로 합치는 것도 산탄총 수술에 대처하는 좋은 방법

메서드나 클래스가 비대해지지만, 나중에 추출하기 리팩터링으로 더 좋은 형태로 분리할 수도 있다. <u>사실 우리는 작은 함수와 클래스에 지나칠 정도로 집착하지만, 코드를 재구성하는 중간 과정에서는 큰 덩어리로 뭉치는데 개의치 않는다</u>


### 3.9 기능 편애 (Feature Envy)
프로그램을 모듈화할 때는 코드를 여러 영역으로 나눈 뒤 영역 안에서 이뤄지는 상호작용은 최대한 늘리고 영역 사이에서 이뤄지는 상호작용은 최소로 줄이는 데 주력한다.

기능 편애는 흔히 어떤 함수가 자기가 속한 모듈의 함수나 데이터보다 다른 모듈의 함수나 데이터와 상호작용 할 일이 더 많을 때 풍기는 냄새다

- 함수 옮기기(8.1)
- 함수 추출하기(6.1)

**참고**. 디자인 패턴 
- 전략 패턴(Strategy Pattern)
- 방문자 패턴(Visitor Pattern)
	- https://ko.wikipedia.org/wiki/%EB%B9%84%EC%A7%80%ED%84%B0_%ED%8C%A8%ED%84%B4

가장 기본이 되는 원칙은 '함께 변경할 대상을 한데 모으는 것'이다.


## 3.10 데이터 뭉치(Data Clumps)

- 클래스 추출하기(7.5)
- 매개변수 객체 만들기(6.8) 또는 객체 통째로 넘기기(11.4)

데이터 뭉치인지 판별하려면 값 하나를 삭제해보자. 그랬을 때 나머지 데이터만으로는 의미가 없다며 객체로 환생하길 갈망하는 데이터 뭉치라는 뜻이다. -> 이어서 그 클래스로 옮기면 좋을 동작은 없는지 살펴보자

데이터 뭉치가 생산성에 기여하는 정식 멤버로 등극하는 순간이다

안티패턴 예제 💩
```java
class Customer {
    private String firstName;
    private String lastName;
    private String phoneNumber;
    private String email;

    public Customer(String firstName, String lastName, String phoneNumber, String email) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.phoneNumber = phoneNumber;
        this.email = email;
    }

    public void printCustomerInfo() {
        System.out.println(firstName + " " + lastName + " | " + phoneNumber + " | " + email);
    }
}

class Order {
    private String firstName;
    private String lastName;
    private String phoneNumber;
    private String email;

    public Order(String firstName, String lastName, String phoneNumber, String email) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.phoneNumber = phoneNumber;
        this.email = email;
    }

    public void printOrderInfo() {
        System.out.println(firstName + " " + lastName + " | " + phoneNumber + " | " + email);
    }
}

```
- 필드가 두 클래스 중복
- 관련된 데이터가 분산되어 있어 유지보수가 어렵고(여러번 수정) 일관성이 깨질 가능성이 큼


리팩터링1. 클래스 추출하기 
```java
class ContactInfo {
    private String firstName;
    private String lastName;
    private String phoneNumber;
    private String email;

    public ContactInfo(String firstName, String lastName, String phoneNumber, String email) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.phoneNumber = phoneNumber;
        this.email = email;
    }

    public void printInfo() {
        System.out.println(firstName + " " + lastName + " | " + phoneNumber + " | " + email);
    }
}

class Customer {
    private ContactInfo contactInfo;

    public Customer(ContactInfo contactInfo) {
        this.contactInfo = contactInfo;
    }

    public void printCustomerInfo() {
        contactInfo.printInfo();
    }
}

class Order {
    private ContactInfo contactInfo;

    public Order(ContactInfo contactInfo) {
        this.contactInfo = contactInfo;
    }

    public void printOrderInfo() {
        contactInfo.printInfo();
    }
}

```
- 중복제거, 유지보수성 개선, 객체간 결합도가 낮아짐


리팩터링2. 매개변수 객체로 만들기 
```java
class ShippingService {
    public void shipPackage(String street, String city, String zipCode) {
        System.out.println("Shipping to: " + street + ", " + city + " " + zipCode);
    }
}

```


```java
class Address {
    private String street;
    private String city;
    private String zipCode;

    public Address(String street, String city, String zipCode) {
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
    }

    public String format() {
        return street + ", " + city + " " + zipCode;
    }
}

class ShippingService {
    public void shipPackage(Address address) {
        System.out.println("Shipping to: " + address.format());
    }
}
```
- 코드 가독성 향상
- 매개변수 순서를 잘못 입력하는 실수 방지 
- Address 객체에 새로운 기능이나 필드 추가시 유지보수 용이


리팩터링3. 객체 통째로 넘기기 
이미 객체가 있다면, 개별 필드 대신 객체 자체를 넘기기
```java
class Customer {
    private ContactInfo contactInfo;

    public Customer(ContactInfo contactInfo) {
        this.contactInfo = contactInfo;
    }

    public ContactInfo getContactInfo() {
        return contactInfo;
    }
}

class OrderProcessor {
    public void processOrder(Customer customer) {
        ContactInfo contactInfo = customer.getContactInfo();
        System.out.println("Processing order for: " + contactInfo.format());
    }
}

```
- 데이터 구조가 바뀌어도 호출부 코드 변경 최소화
- 불필요한 개별 필드 전달 대신 **객체 하나로 깔끔하게 전달**


**정리**

|             |                       |                       |
| ----------- | --------------------- | --------------------- |
| 클래스 추출하기    | 관련 필드가 반복됨            | 중복 제거, 응집도 향상         |
| 매개변수 객체 만들기 | 매개변수 개수가 많음           | 코드 가독성 개선, 실수 방지      |
| 객체 통째로 넘기기  | 이미 존재하는 객체에서 데이터만 꺼내씀 | 불필요한 필드 전달 제거, 코드 단순화 |
// 생성자로 치면 빌더 패턴이 생각날 수 있겠네



### 3.11 기본형 집착(Primitive Obsession)
**절차**
- 기본형을 객체로 바꾸기(7.3)     // 원시값을 포장하기를 뜻하는 듯 
- 타입 코드를 서브 클래스로 바꾸기(12.6)와 조건부 로직을 다형성으로 바꾸기(10.4)

자주 함께 몰려다니는 기본형 그룹도 데이터 뭉치다. 따라서 클래스 추출하기(7.5)와 매개변수 객체 만들기(6.8)를 이용하여 반드시 문명사회로 이끌어줘야 한다


자료형들을 문자열로만 표현하는 악취는 아주 흔해서, 소위 '문자열화된(stringly typed) 변수'라는 이름까지 붙었다
```java
// 💩
class Order {
    private String status; // "NEW", "SHIPPED", "DELIVERED", "CANCELLED"

    public Order(String status) {
        this.status = status;
    }

    public boolean isDelivered() {
        return "DELIVERED".equals(this.status);
    }

    public void setStatus(String status) {
        this.status = status;
    }
}

```
- `"DELIVERED".equals(this.status)` 가 중복되어서 사용될 수 있음
- `status` 값이 잘못된 문자열로 설정될 가능성이 있음 
-  IDE 자동 완성 기능을 활용하기 어려움


```java
class Order {
    private final OrderStatus status;

    public Order(OrderStatus status) {
        this.status = status;
    }

    public boolean isDelivered() {
        return status.isDelivered();
    }
}

enum OrderStatus {
    NEW, SHIPPED, DELIVERED, CANCELLED;

    public boolean isDelivered() {
	   return this == DELIVERED;
	}
}
```
- `enum` 사용해서 한 군데에서 관리하고 재사용
- 생성자 초기화 이후 변경 못하도록 막음 (불변)

### 3.12 반복되는 switch 문(Repeated Switches)
- switch문 모조리 **조건부 로직을 다형성으로 바꾸기**(10.4)

중복된 switch문이 문제가 되는 이유는 조건절을 하나 추가할 때마다 다른 switc문들도 모두 찾아서 함께 수정해야 하기 때문이다. 이럴 때 다형성은 반복된 switch문이 내뿜는 사악한 기운을 제압하여 코드베이스를 최신 스타일로 바꿔주는 세련된 무기인 셈이다.


### 3.13 반복(Loops)
- **반복문을 파이프라인으로 바꾸기**(8.8) 적용
	- 컬렉션 파이프라인을 이용 (primitive type도 stream 지원)


### 3.14 성의 없는 요소(Lazy Element)
나중에 본문을 더 채우거나 다른 메서드를 추가할 생각이었지만, 어떠한 사정으로 인해 그렇게 하지 못한 결과일 수 있다. 혹은 풍성했던 클래스가 리팩터링을 거치면서 역할이 줄어들었을 수 있다. 사정이 어떠하든 이런 프로그램 요소는 고이 보내드리는 게 좋다
- 함수 인라인하기(6.2)
- 클래스 인라인(7.6) // ?
- 계층합치기(12.9) // 상속을 사용한 경우 ?

// 이것도 야그니 아닌가?
### 3.15 추측성 일반화(Speculative Generality)
// 이것도 야그니 인거 같은데?

이 냄새는 '나중에 필요할 거야'라는 생각으로 당장은 필요 없는 모든 종류의 후킹(hooking) 포인트와 특이 케이스 처리 로직을 작성해둔 코드에서 풍긴다. 그 결과는 물론 이해하거나 관리하기 어려워진 코드다. 미래를 대비해 작성한 부분을 실제로 사용하게 되면 다행이지만, 그렇지 않는다면 쓸데없는 낭비일 뿐이다. <u>당장 걸리적거리는 코드는 눈앞에서 치워버리자.</u>

- **계층 합치기**(12.9) : 추상 클래스
- **함수 인라인하기**(6.2) 나 **클래스 인라인하기**(7.6) : 쓸데없이 위임하는 코드 경우
- **함수 선언 바꾸기**(6.5) : 본문에서 사용되지 않는 매개변수 경우

추측성 일반화는 테스트 코드 말고는 사용하는 곳이 없는 함수나 클래스에서 흔히 볼 수 있다.
- 테스트 케이스 삭제한 뒤에 **죽은 코드 제거하기**(8.9)로 제거하자


### 3.16 임시 필드 (Temporary Field)
- 특정 상황에서만 값이 설정되는 필드를 가진 클래스가 있다
- 객체를 가져올 때 당연히 모든 필드가 채워져 있을거라 기대하는게 보통이지만, 임시 필드를 사용하면 <u>코드를 이해하기 어려워진다</u>

적용해 볼 리팩토링 기법
- **클래스 추출하기**(7.5)
- **함수 옮기기**(8.1)
- **특이 케이스 추가하기**(10.5) // ?

안티패턴 예제 💩
```java
class ReportGenerator {
    private String reportType;
    private String pdfTemplate;
    private String excelTemplate;

    public ReportGenerator(String reportType) {
        this.reportType = reportType;
        if (reportType.equals("PDF")) {
            this.pdfTemplate = "default-pdf-template";
        } else if (reportType.equals("EXCEL")) {
            this.excelTemplate = "default-excel-template";
        }
    }

    public void generate() {
        if (reportType.equals("PDF")) {
            System.out.println("Generating PDF using template: " + pdfTemplate);
        } else if (reportType.equals("EXCEL")) {
            System.out.println("Generating EXCEL using template: " + excelTemplate);
        } else {
            throw new IllegalArgumentException("Unsupported report type");
        }
    }
}
```
- 문자열화 된(stringly typed) 변수사용
- `reportType` 추가시 조건문 로직 변경 전파 -> **OCP 원칙 위반**
- 생성자 초기화시 없는 타입의 경우 특정 필드만 초기화됨 -> 객체의 일관성 부족 (이해 어려워짐)

리팩터링1. 클래스 추출하기
```java
abstract class Report {
    abstract void generate();
}

class PdfReport extends Report {
    private String template = "default-pdf-template";

    @Override
    public void generate() {
        System.out.println("Generating PDF using template: " + template);
    }
}

class ExcelReport extends Report {
    private String template = "default-excel-template";

    @Override
    public void generate() {
        System.out.println("Generating EXCEL using template: " + template);
    }
}

```

리팩터링2. 함수 옮기기
```java
class ReportGenerator {
    private Report report;

    public ReportGenerator(Report report) {
        this.report = report;
    }

    public void generateReport() {
        report.generate();
    }
}
```
- 보고서 타입별 로직이 사라지고 Report 클래스에 책임을 위임
- 새로운 보고서 타입으 추가될 때 기존 클래스를 수정할 필요가 없어짐

리팩터링3. 특이 케이스 추가하기
```java
class NullReport extends Report {
    @Override
    public void generate() {
        System.out.println("No report to generate.");
    }
}
```
- <u>null 체크 없이도 안전한 기본 동작을 제공할 수 있다</u>
- 클라이언트 코드에서도 null 체크 없이 report.generate() 호출 가능


|             | 적용 이유               | 개선점                    |
| ----------- | ------------------- | ---------------------- |
| 클래스 추출하기    | 서로 다른 속성을 가진 필드를 분리 | 객체의 책임을 분리하여 응집도를 높임   |
| 함수 옮기기      | 특정 로직이 한 클래스에 집중됨   | 클래스 간 책임을 분배하여 가독성을 향상 |
| 특이 케이스 추가하기 | null 체크 반복적으로 수행    | 안전한 기본 동작을 제공          |
결과적으로 임시 필드를 제거하면 코드 가독성이 좋아지고 유지보수가 쉬워진다


### 3.17 메시지 체인 (Message Chains)
- **위임 숨기기**(7.7)
- **함수 추출하기**(6.1)로 결과 객체를 사용하는 코드 일부를 따로 빼낸 다음 **함수 옮기기**(8.1)로 체인을 숨길 수 있는지 살펴보자

체인을 구성하는 객체 중 특정 하나를 사용하는 클라이언트 중 그 이후의 객체들도 사용하길 원하는 클라이언트가 제법 된다면, 이 요구를 처리해줄 메서드를 추가한다.

// chat gpt 예제 
```java hl:44
class Person {
    private Department department;

    public Person(Department department) {
        this.department = department;
    }

    public Department getDepartment() {
        return department;
    }
}

class Department {
    private Manager manager;

    public Department(Manager manager) {
        this.manager = manager;
    }

    public Manager getManager() {
        return manager;
    }
}

class Manager {
    private String name;

    public Manager(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

public class Main {
    public static void main(String[] args) {
        Manager manager = new Manager("Alice");
        Department department = new Department(manager);
        Person person = new Person(department);

        // 메시지 체인 발생 (체인 길어짐) 💩
        String managerName = person.getDepartment().getManager().getName();
        System.out.println(managerName); // Alice
    }
}

```
- 메서드 체인이 깊어질수록 캡슐화가 깨지고 결합도 증가 -> 변경시 사이드 이펙트 전파 발생
- `Department`나 `Manager`의 내부 구현이 바뀌면 `Person`을 사용하는 모든 코드가 영향을 받음

**리팩터링 후**
```java
class Person {
    private Department department;

    public Person(Department department) {
        this.department = department;
    }

    // ✅ 위임 숨기기 (Department를 직접 호출하지 않도록 메서드 추가)
    public Manager getManager() {
        return department.getManager();
    }

    // ✅ 함수 추출하기 (Manager의 이름을 바로 가져올 수 있도록)
    public String getManagerName() {
        return getManager().getName();
    }
}

class Department {
    private Manager manager;

    public Department(Manager manager) {
        this.manager = manager;
    }

    public Manager getManager() {
        return manager;
    }
}

class Manager {
    private String name;

    public Manager(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

public class Main {
    public static void main(String[] args) {
        Manager manager = new Manager("Alice");
        Department department = new Department(manager);
        Person person = new Person(department);

        // ✅ 메시지 체인 제거
        String managerName = person.getManagerName();
        System.out.println(managerName); // Alice
    }
}

```
- **위임 숨기기 (Hide Delegate)**
    - `Person` 클래스가 `Department`를 직접 참조하지 않고 `getManager()`를 통해 접근
- **함수 추출하기 (Extract Function)**
    - `getManagerName()` 메서드를 `Person`에 추가하여 `Main`에서 `person.getManager().getName()`을 직접 호출하지 않도록 수정
- **함수 옮기기 (Move Function) - 필요 시**
    - `getManagerName()`은 `Person`이 자주 호출하므로 `Person`에 추가하는 것이 적절
    - 만약 `Department`를 통해 접근하는 것이 더 자연스럽다면 `Department`에 `getManagerName()`을 추가할 수도 있음

**리팩터링 후 장점**
✅ **결합도 감소**: `Person`이 `Department`와 `Manager`의 내부 구조를 몰라도 됨  
✅ **코드 가독성 향상**: 클라이언트 코드가 간결해지고 의미가 분명해짐  
✅ **유지보수성 증가**: `Department`나 `Manager`의 구현이 바뀌어도 `Person`만 변경하면 됨



// 헤드 퍼스트 디자인 패턴 책 내용 중
**참고. 최소 지식 원칙 (Principle of Least Knowledge) = 데미터/데메테르 법칙**

> 객체 지향 구현시 각 모듈간의 결합도를 최소화하여 설계한다는 원칙

**장점**
- 객체 사이의 의존성/결합도를 줄일 수 있으며, S/W 관리가 더 편해짐

**단점**
- 메소드 호출을 처리하는 Wrapper (래퍼) 클래스를 더 만들어야 할 수 있음
→시스템이 복잡해지고, 개발시간도 늘어나고, 성능도 떨어질 수 있다

1. 원칙을 따르지 않는 경우

```java
public float getTemp() {
  return station.getThermometer().getTemperature();
}
```

메소드 체인이 깊어질수록 아래와 같은 단점을 가진다
- 서로 관련없는 객체의 내부 구조에 접근하게 되어 의존성/결합도가 높아짐
- 코드 가독성 저하된다
- 연쇄 호출된 메소드가 수정되면 다른 메소드에 영향을 줄 수 있다

2. 원칙을 따르는 경우

```java
public float getTemp(Station station) {
	return station.getTemperature();
}
```

**최소 지식 원칙 - 가이드 라인**
- 객체 자기 자신의 메소드
- 메소드 인자로 전달된 객체의 메소드
- 맴버 변수 또는 지역 스코프 내에서 생성한 객체의 메소드
- 객체에 속하는 구성요소 (ex. Wrapper 클래스)

### 3.18 중재자(Middle Man)
객체의 대표적인 기능 하나로, 외부로부터 세부사항을 숨겨주는 **캡슐화**가 있다
캡슐화하는 과정에서 위임(delegation)이 자주 활용된다.

지나치면 문제가 된다. 클래스가 제공하는 메서드 중 절반이 다른 클래스에 구현을 위임하고 있다면 어떤가? 이럴때는
- **중개자 제거하기**(7.8) : 실제로 일을 하는 객체와 직접 소통하게 하자
- **함수 인라인하기**(6.2) : 위임 메서드를 제거한 후 남는 일이 거의 없다면 호출하는 쪽으로 인라인하자

// 메서드 체인에서 다룬 안티 패턴으로 복구시키네..
 **언제 중개자를 제거해야 할까?**(chat-gpt)
- 클래스의 **메서드 절반 이상이 단순히 다른 객체의 메서드를 호출할 때**
- **중개자를 제거해도 캡슐화가 크게 깨지지 않는 경우**
- 유지보수를 위해 불필요한 계층을 줄이는 것이 더 유리한 경우

// chat-gpt 
나 : 메서드 체인이랑 중재자랑 비슷한데 에제가 너무 간단해서 판단이 어려워요

| 냄새 유형  | 특징                      | 문제점                                   | 해결 방법                                        |
| ------ | ----------------------- | ------------------------------------- | -------------------------------------------- |
| 메서드 체인 | 연속적인 메서드 호출             | 객체의 내부 구조가 너무 많이 노출됨 (캡슐화 깨지고 결합도 상승) | **위임 숨기기(7.7)**, **함수 추출하기(6.1) → 옮기기(8.1)** |
| 중재자    | 클래스가 하는 일 없이 대부분 객체에 위임 | 클래스가 실제 로직 없이 중계 역할만 함 (불필요한 추상화)     | **중개자 제거하기(7.8)**, **함수 인라인하기(6.2)**         |

 ✅ **핵심 기준 (언제 리팩터링해야 할까?)**
1. **객체 내부 구조를 너무 많이 노출하는가?**  
    → YES ✅ → **메서드 체인 문제** 🚨 (위임 숨기기 적용)
2. **이 클래스가 실질적인 역할 없이 중계만 하는가?**  
    → YES ✅ → **중재자 냄새 문제** 🚨 (중개자 제거하기 적용)
3. **객체가 직접 처리해야 하는 책임이 있는가?**  
    → YES ✅ → 메서드를 해당 객체로 옮기기

🎯 **중재자 냄새 개선이 중요한 경우 (Side Effect 전파 문제)**
예를 들어, **어떤 객체가 다른 객체를 수정하거나 영향을 끼치는 경우**,  너무 깊은 체인은 **사이드 이펙트**(부작용)를 일으킬 수 있음
```java
// ❌ 중재자 냄새 + 사이드 이펙트 전파 (문제 발생 가능)
person.getDepartment().getManager().setName("Bob");  // 🚨 깊숙한 객체까지 변경
```
**👉 값 조회하는 단순한 체인은 괜찮지만, 변경이 일어나는 경우 최소화 필요!**  

**✅ 리팩터링 방향**
- 객체의 변경은 직접 담당하는 객체가 하도록 한다
- 너무 깊은 체인은 숨기고, 적절한 메서드로 감싸서 제공

📌 **정리**
✔ **메서드 체인은 캡슐화 문제 → 적절히 숨기기**  
✔ **중재자는 불필요한 위임 문제 → 불필요하면 제거**  
✔ **조회는 문제없지만, 변경이 있는 경우 최소화 필요**

### 3.19 내부자 거래(Insider Trading) 

커피 자판기 옆에서 은밀히 데이터를 주고 받는 모듈들이 있다면
- **함수 옮기기**(8.1)와 **필드 옮기기**(8.2)
- **위임 숨기기**(7.7): 여러 모듈이 같은 관심사를 공유한다면 공통 부분을 정식으로 처리하는 제 3의 모듈을 새로 만들거나 다른 모듈이 중간자 역할을 하게 만든다


상속 구조에서 자식 클래스는 항상 부모 클래스가 공개하고 싶은 것 이상으로 부모에 대해 알려고 한다. 그러다가 부모 품을 떠나야 할 때가 온다면 **서브 클래스를 위임으로 바꾸기**(12.10)나 **슈퍼 클래스를 위임으로 바꾸기**(12.11)를 활용하자

// 내 생각 
관련성있는 필드나 함수를 한 곳에 모아 응집도와 유지보수성을 높이고 해당 객체를 메시지로 주고 받으며 두 모듈이 통신하도록 한다는 의미인듯 


//chat-gpt 
**"내부자 거래 (Insider Trading)"** 냄새는 **서로 너무 밀접하게 결합된 모듈 간의 데이터 공유**를 뜻해요.

 **내부자 거래(Insider Trading) 냄새**는
- **모듈 간 결합도가 너무 높아 내부 구현을 서로 깊이 알게 되는 것**이 문제
- **관련 있는 함수와 필드를 적절한 객체로 이동하여 책임을 위임**
- **각 객체가 직접 데이터를 주고받는 대신, 적절한 인터페이스를 통해 소통**하도록 리팩토링하는 게 핵심입니다.

 **✅ 내부자 거래의 핵심 문제점**
- 특정 클래스가 다른 클래스의 내부 구현에 **직접 의존**하면, 변경 시 연쇄적으로 영향을 받음 → **결합도가 높아짐**
- 모듈이 독립적으로 동작하지 못하고, **데이터를 은밀하게 공유**하는 관계가 형성됨

잘못된 예: 서비스가 컨트롤러의 내부 구조에 의존
```java
class OrderController {
    private final OrderService orderService;
    
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    public void placeOrder(HttpServletRequest request) {  // ❌ 요청 객체를 그대로 넘김
        orderService.processOrder(request);
    }
}

class OrderService {
    public void processOrder(HttpServletRequest request) { // ❌ 컨트롤러의 내부 구조를 사용
        String productId = request.getParameter("productId");
        int quantity = Integer.parseInt(request.getParameter("quantity"));
        System.out.println("Processing order: " + productId + ", Quantity: " + quantity);
    }
}
```
**🚨 문제점**
- `OrderService`가 **컨트롤러의 내부 구현(HttpServletRequest)** 에 의존 → **내부자 거래 발생**
- 서비스는 `request`를 알 필요가 없음 → **DTO를 사용해 데이터를 전달해야 함**

개선: DTO를 사용해 명확한 경계 구분 
```java
class OrderRequest {
    private final String productId;
    private final int quantity;
    
    public OrderRequest(String productId, int quantity) {
        this.productId = productId;
        this.quantity = quantity;
    }
    
    public String getProductId() { return productId; }
    public int getQuantity() { return quantity; }
}

class OrderController {
    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    public void placeOrder(HttpServletRequest request) {  
        OrderRequest orderRequest = new OrderRequest(
            request.getParameter("productId"),
            Integer.parseInt(request.getParameter("quantity"))
        );
        orderService.processOrder(orderRequest);
    }
}

class OrderService {
    public void processOrder(OrderRequest orderRequest) { 
        System.out.println("Processing order: " + orderRequest.getProductId() 
                           + ", Quantity: " + orderRequest.getQuantity());
    }
}

```
✅ **서비스는 컨트롤러 내부 구조를 몰라도 됨**  
✅ **의존성이 줄어들어 결합도가 낮아지고, 유지보수성 증가**


잘못된 예: 서비스 계층이 도메인 직접 접근하여 사용 
```java
class Order {
    String productId;
    int quantity;
}

class OrderService {
    public void processOrder(Order order) {
        // ❌ Order의 내부 필드를 직접 사용 → 내부자 거래 냄새 발생
        System.out.println("Processing order: " + order.productId + ", Quantity: " + order.quantity);
    }
}
```

개선: 서비스에 있는 도메인 로직 이동 
```java
class Order {
    private final String productId;
    private final int quantity;

    public Order(String productId, int quantity) {
        this.productId = productId;
        this.quantity = quantity;
    }

    // ✅ 내부 필드 대신 메서드를 통해 데이터 제공 (캡슐화)
    public String getProductId() { return productId; }
    public int getQuantity() { return quantity; }

    // ✅ 도메인 로직을 Order 내부로 이동
    public void process() {
        System.out.println("Processing order: " + productId + ", Quantity: " + quantity);
    }
}

class OrderService {
    public void processOrder(Order order) {
        // ✅ Order 객체에게 메시지를 보내서 작업하도록 위임
        order.process();
    }
}
```


✔ **내부자 거래 냄새란?**
- **모듈 간 결합도가 너무 높아 내부 구현을 서로 깊이 알게 되는 문제**
- **객체의 속성을 직접 접근하거나, 불필요한 데이터 공유가 많을 때 발생**

✔ **해결 방법**
- **연관된 필드와 메서드를 같은 객체에 위치** → 응집도를 높이고 결합도를 낮춤
- **DTO를 활용하여 명확한 데이터 전달 방식 유지**
- **객체 간 직접 속성을 공유하지 말고, 메시지를 주고받는 방식으로 변경**

🎯 결국 **책임을 올바른 곳으로 이동하여 유지보수성을 높이는 게 핵심!**

💡 **DTO, 캡슐화, 메서드 이동을 활용해 내부자 거래를 줄이고 결합도를 낮추는 것이 핵심!**

### 3.20 거대한 클래스 (Large Class)
한 클래스가 너무 많은 일을 하려다 보면 필드 수가 상당히 늘어난다. 그리고 클래스에 필드가 너무 많으면 중복 코드가 생기기 쉽다. 

- **클래스 추출하기**(7.5) : 필드 일부를 따로 묶는다
- **슈퍼 클래스 추출하기**(12.8)나 **타입 코드를 서브 클래스로 바꾸기**(12.6)
	- 위에서 분리한 컴포넌트를 원래 클래스와 상속 관계로 만드는 게 좋다면 적용

// 넥스트 스텝 - 블랙잭 게임이 생각나네 https://dev-ljw1126.tistory.com/420

클래스가 항시 모든 필드를 사용하지는 않을 수도 있다. 이럴 때는 앞에서 언급한 추출 기법들을 여러 차례 수행해야 할지도 모른다. 

<u>필드가 너무 많은 클래스와 마찬가지로 코드량이 너무 많은 클래스</u>도 중복 코드와 혼동을 일으킬 여지가 크다. 가장 간단한 해법은 그 클래스 안에서 자체적으로 중복을 제거하는 것이다 (..)

<u>클라이언트들이 거대 클래스를 이용하는지 패턴을 파악하여 그 클래스를 어떻게 쪼갤지 단서를 얻을 수도 있다. 먼저 클라이언트들이 거대 클래스의 특정 기능 그룹만 주로 사용하는지 살핀다. 이때 각각의 기능 그룹이 개별 클래스로 추출될 후보다.</u>
- **클래스 추출하기**(7.5)
- **슈퍼 클래스 추출하기**(12.8)
- **타입 코드를 서브 클래스로 바꾸기**(12.6)


### 3.21 서로 다른 인터페이스의 대안 클래스들 (Alternative Classes with Different Interfaces)
클래스를 사용할 때의 큰 장점은 필요에 따라 언제든 다른 클래스로 교체할 수 있다는 것이다.
(단, 교체하려면 인터페이스가 같아야 한다)

- **함수 선언 바꾸기**(6.5) : 메서드 시그니처를 일치시킨다
- **함수 옮기기**(8.1) : 인터페이스가 같아질 때까지 필요한 동작들을 클래스 안으로 밀어 넣는다
- **슈퍼 클래스 추출하기**(12.8) : 그러다 대안 클래스들 사이에 중복 코드가 생기면 슈퍼 클래스 추출하기를 적용할지 고려해본다

 **정리**(chat gpt)
- **슈퍼 클래스(Superclass)** = 여러 서브클래스의 공통 속성을 가진 부모 클래스 (추상 클래스일 필요는 없음)
- **슈퍼 클래스 추출하기 (12.8)** = **중복된 코드가 많을 때, 부모 클래스로 정리**
- **추상 클래스(Abstract Class)** 는 반드시 오버라이드해야 하는 **추상 메서드** 포함 가능

즉, **공통된 기능이 있다면 슈퍼 클래스로 정리하는 게 좋지만, 필요 없는 상속을 남발하지 않도록 주의**해야 해요


### 3.22 데이터 클래스 (Data Class)
>[!note] 데이터 클래스
>데이터 필드와 게터/세터 메서드로만 구성된 클래스를 말한다.
>

그저 데이터 저장 용도로만 쓰이다 보니 다른 클래스가 너무 깊이까지 함부로 다룰 때가 많다.

- **레코드 캡슐화하기**(7.1) : public 필드는 private로 변경
- **세터 제거하기**(11.7) : 변경하면 안되는 필드는 접근을 원천 봉쇄
- **함수 옮기기**(8.1): 다른 클래스에서 데이터 클래스의 게터나 세터를 사용하는 메서드를 찾아서 그 메서드를 데이터 클래스로 옮길 수 있는지 살펴보자
- **함수 추출하기**(6.1) : 메서드를 통째로 옮기기 어렵다면 옮길 수 있는 부분만 별도 메서드로 뽑아낸다

한편, 데이터 클래스는 필요한 동작이 엉뚱한 곳에 정의돼 있다는 신호일 수 있다. 
이런 경우라면 클라이언트 코드를 데이터 클래스로 옮기기만 해도 대폭 개선된다.
물론 예외도 잇다. 
특히 다른 함수를 호출해 얻은 결과 레코드(데이터 객체)로는 동작 코드(?)를 넣을 이유가 없다.
대표적인 예로 단계 쪼개기(6.11)의 결과로 나온 중간 데이터 구조가 있다. 이런 데이터 구조는 불변(immutable)이다. <u>불변 필드는 굳이 캡슐화할 필요 없고, 불변 데이터로부터 나오는 정보는 게터를 통하지 않고 그냥 필드 자체를 공개해도 된다</u>

// chat-gpt 
**💡 데이터 클래스(Data Class)란?**
- **데이터 클래스는 말 그대로 데이터를 저장하는 데 집중된 클래스**입니다.
- **필드(속성)만 존재**하고, 동작(메서드)은 거의 없는 구조
- `getter`, `setter`, `equals`, `hashCode` 같은 기본 메서드만 있음
- 자바에서는 `record`(Java 14+) 또는 `Lombok @Data`를 활용하면 쉽게 만들 수 있음
```java
// ✅ Java 14+ record (자동으로 getter, equals, hashCode 제공)
public record User(String name, int age) {}

// ✅ Lombok 활용 (@Data = getter, setter, equals, hashCode, toString 자동 생성)
@Data
public class User {
    private final String name;
    private final int age;
}
```


**🛑 데이터 클래스가 문제되는 경우 (냄새)**
 - **데이터만 가지고 있고, 관련 동작(메서드)이 외부에서 처리됨**
```java
class Order {
    private String product;
    private int quantity;
    
    public Order(String product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }

    // Getter만 있음
    public String getProduct() { return product; }
    public int getQuantity() { return quantity; }
}

// ❌ Order에 대한 로직이 외부에 있음
class OrderService {
    public int calculateTotal(Order order, int pricePerUnit) {
        return order.getQuantity() * pricePerUnit; 
    }
}
```
- `Order` 클래스에는 **데이터만 있고**, 핵심 로직(총 가격 계산)이 `OrderService`에 있음
- **데이터 클래스는 동작을 포함하지 않고 단순한 데이터 전달 역할만 함** → 유지보수 어려움

 **✅ 데이터 클래스의 동작을 내부로 이동 (리팩터링)**
- **데이터를 가지고 있는 클래스에 로직을 추가**하면 응집도가 높아짐
- **클라이언트 코드가 불필요하게 데이터를 꺼내 쓰지 않아도 됨** (캡슐화)
```java
class Order {
    private String product;
    private int quantity;
    private int pricePerUnit;
    
    public Order(String product, int quantity, int pricePerUnit) {
        this.product = product;
        this.quantity = quantity;
        this.pricePerUnit = pricePerUnit;
    }

    // ✅ 데이터를 가지고 있는 클래스에서 처리하도록 이동
    public int calculateTotal() {
        return quantity * pricePerUnit;
    }
}
```
- 이제 `Order` 인스턴스에서 `calculateTotal()`을 호출하면 총 가격을 얻을 수 있음
- **데이터와 동작이 한곳에 모여 있어서 응집도가 높아짐**


**💡 예외적인 경우 (불변 데이터 구조는 OK!)**
불변(immutable) 데이터 클래스는 예외적으로 괜찮음.
- 예를 들어, **단계 쪼개기(6.11)의 중간 데이터 구조**처럼 **순수한 데이터 전용 객체**
- 다른 메서드가 데이터를 수정할 일이 없으므로 **getter 없이 필드 공개 가능**
```java
record ProductInfo(String name, int price) {}  // ✅ 불변 데이터 객체

var info = new ProductInfo("Laptop", 2000);
System.out.println(info.name());  // getter 없이 필드 접근 가능
```
- `record`를 사용하면 **자동으로 불변 객체가 됨**
- 이 경우에는 동작을 추가할 필요 없이 **순수 데이터 저장소 역할만 해도 OK**

**🔹 정리**
✅ **데이터 클래스 냄새란?**
- 필드만 있고 동작이 없는 클래스
- 동작(메서드)이 외부 클래스에서 처리됨
- **데이터를 가지고 있는 클래스에 관련 동작을 추가하는 것이 더 좋음**

✅ **예외적으로 괜찮은 경우**
- **불변(Immutable) 데이터 객체** (ex. `record`, DTO)
- **단계 쪼개기(6.11)에서 나온 중간 데이터 구조**

**즉, 데이터를 조작하는 동작이 있다면, 해당 데이터를 소유한 클래스 내부로 옮기는 것이 좋다**

### 3.23 상속 포기(Refused Bequest)
부모의 유산을 원치 않거나 필요없다면 어떻게 할까? 수많은 유산 중에서 관심 있는 몇 개만 받고 끝내려는 경우는 얼마든지 있을 수 있다.

예전에는 아래 방식을 권장했음
- **메서드 내리기**(12.4)와 **필드 내리기**(12.5) : 같은 계층에 서브 클래스를 하나 만들고 부모 클래스에서 물려 받지 않을 필드와 메서드를 새로 만든 서브 클래스로 내린다
	- 그러면 부모에는 공통된 부분만 남는다
	- 한 걸음 더 나아가서 부모 클래스는 모두 추상 클래스여야 한다고 말하는 사람도 많다

우리는 이 방식을 권장하지 않는다. 아니, 항상 이렇게 해야 한다는 입장은 아니다
일부 동작을 재활용하기 위한 목적으로 상속을 활용하기도 하는데, 실무 관점에서 아주 유용한 방식이다. 솔직히 냄새를 풍기지만, 보통은 참을만한 경우가 많다. 
그래서 상속을 포기할 시 혼란과 문제가 생긴다면 앞에서 설명한 예전 방식을 따른다.
<u>단, 무조건 이래야 한다는 생각은 버리자. </u>

상속 포기 냄새는 서브 클래스가 부모의 동작은 필요로하지만 인터페이스는 따르고 싶지 않을 때 특히 심하게 난다. 구현을 따르지 않는 것은 이해할 수 있지만 인터페이스를 따르지 않는다는 것은 상당히 무례한 태도다.

이 경우 **서브 클래스를 위임으로 바꾸기**(12.10)나 **슈퍼 클래스를 위임으로 바꾸기**(12.11)를 활용해서 아예 상속 매커니즘에서 벗어나보자


### 3.24 주석(Comments)

> 주석을 남겨야겠다는 생각이 들면, 가장 먼저 주석이 필요 없는 코드로 리팩터링해본다.

- 특정 코드 블록이 하는 일을 주석으로 남기고 싶다면 **함수 추출하기**(6.1)
- 이미 추출되어 있는 함수임에도 여전히 설명이 필요하다면 **함수 선언바꾸기**(6.5)
- 시스템이 동작하기 위한 선행조건을 명시하고 싶다면 **어셔션 추가하기**(10.6)

뭘 할지 모를 때라면 주석을 달아두면 좋다. 현재 진행 상황뿐만 아니라 확실하지 않는 부분에 주석에 남긴다. 코드를 지금처럼 작성한 이유를 설명하는 용도로 달 수도 있다. 이런 정보는 나중에 코드를 수정해야 할 프로그래머에게, 특히 건망증이 심한 프로그래머에게 도움될 것이다.


참고. 클린코드 4장. https://dev-ljw1126.tistory.com/104

1.주석을 최대한 쓰지 말자

"주석은 나쁜 코드를 보완하지 못한다"
① 코드에 주석을 추가하는 일반적인 이유는 코드 품질이 나쁘기 때문이다.
② 이는 곧 작성자가 의도를 명확히 표현하지 못했다는 것을 뜻하기도 함
👉 난장판을 주석으로 설명하지 말고 개선하는데 시간을 보내자

"주석은 방치된다"
① 코드의 변화에 따라가지 못하고, 주석은 방치된다.
② 방치된 주석은 뒤에 읽는 사람에게 혼용 야기 할 수 있다.
👉 관리하지 못할 거면 자제하는 것이 낫다

2.좋은 주석
법적인 이유로 다는 주석
```java
//Copyright (C) 2003,2004,2005 by Object Mentor, Inc. All rights reserved 
//GNU General Public License 
```

정보를 제공하는 주석
```java
//휴대폰 000-000-0000 || 000-0000-0000
String PHONE_PATTERN = “^01(?:0|1|[6-9])-(?:\\d{3}|\\d{4})-\\d{4}$”;

//시분 HH24MI(0000~2359) 
String HH24MI_TIME_PATTERN = “^([01]\\d|2[0-3])([0-5])(\\d)$”;

//날짜 YYYY-MM-dd 
String DATE_PATTERN = “^([12]\\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01]))$”;
```

의도를 설명하는 주석
```java
//스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다 
for (int i=0;i<25000;i++) {  
   .... 
} 
```
- 주석 저자가 문제를 해결한 방식으로 저자의 의도가 분명하게 나타남

의미를 명료하게 밝히는 주석
```java
assertTrue(a.compareTo(a) == 0)  //a == a 
assertTrue(a.compareTo(b) != 0)  //a != b 
assertTrue(a.compareTo(ab) == 0) //ab == ab 
assertTrue(a.compareTo(b) != -1) //a < b 
```
- 그러나 이러한 주석은 주석이 올바른지 검증하기 쉽지 않다는 문제가 있다. 따라서 위와 같은 주석을 달 때는 더 나은 방법이 없는지 먼저 꼭 고민해야 한다.

결과를 경고하는 주석
```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오. 
public void _testWithReallyBigFile() { 
  writeLinesToFile(100000000); 
  ..... 
} 
```

TODO 주석
```java
//TODO: MdM 현재 필요하지 않다 
//체크아웃 모델을 도입하면 함수가 필요없다. 
protected VersionInfo makeVersion() throws Exception{ 
    return null; 
} 
```


3.주석보다 annotation
- annotaiton은 코드에 대한 메타 데이터를 나타낸다
- 코드의 실행 흐름에 간섭을 주기도 하고, 주석처럼 코드에 대한 정보를 줄 수 있다
- 예
	- @Deprecated 
	- @NotThreadSafe
	- @DisplayName

4.나쁜주석
• 주절거리는 주석  
• 같은 이야기를 중복하는 주석  
• 오해할 여지가 있는 주석  
• 의무적으로 다는 주석 → 나쁜 습관💩
• 이력을 기록하는 주석 → 형상관리(git, svn)로 과거 이력 확인 가능하므로 필요없음  
• 특정 위치를 표시하는 주석  
• 닫는 괄호에 다는 주석  
• 공로를 돌리거나 저자를 표시하는 주석  
• HTML 주석  
• 전역정보 (ex. Port 번호) → 정보 관리 못할 거면 안 다는게 나음  
• 너무 많은 정보  
• 모호한 관계 등



---
- 부록B를 참고하여 코드가 풍기는 냄새(악취)가 무엇인지 찾자 (예제는 없었다)
- 3.6을 작성하는 중에 든 생각이지만, 코드의 냄새를 찾으려면 경험과 감각이 있어야해보인다
- 코드를 읽는 습관이 연습이 된다는 느낌이 든다
- 3.7에서 서로 다른 **맥락(context)** 용어가 나왔는데, 그 맥락은 집합을 의미하는 걸까
- 무엇이 힘들었는가?
	- 기법과 상황을 상상하지 못함 -> chat-gpt와 리팩터링 카탈로그 활용
