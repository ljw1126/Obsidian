
**📚 TS 예시 포함된 개발 도서**
- 단위 테스트의 기술
- 멀티패러다임 프로그래밍

참고.
- [랜덤 이미지 제공 사이트]([https://picsum.photos/](https://picsum.photos/))

### 타입스크립트란?
- Statically Typed (정적으로 결정되는 타입)
- compile errors
- class /  interface / generic 지원
- TS는 transcompiles 통해 JS로 변환된다
	- 대표적인 컴파일러는 BABEL

참고. JS
- Prototype-based
- Constructor Functions
- dynamically typed (동적으로 결정되는 타입)
- runtime errors
- ES6 부터 class 문법 지원


>[!info]
>컴파일 시점에 타입이 결정되면 `statically typed` 이라 부르고 런타임에 타입이 결정되면 `dynamically typed`이라 부른다


참고. [자바스크립트 기본 가이드 영상]([https://www.youtube.com/playlist?list=PLv2d7VI9OotTVOL4QmPfvJWPJvkmv6h-2](https://www.youtube.com/playlist?list=PLv2d7VI9OotTVOL4QmPfvJWPJvkmv6h-2))


### Setup
- VSCode
- Node.js
- Terminal : iTerm2(mac), cmder (window)

>[!info] Node.js
>JavaScript runtime environment(framework) that executes JS code outside a web browser
>"JavaScript everywhere"

>[!info] npm package manager
>publish and share course code of Node.js packages simplify installation, updating and uninstallation of packags

**TS 설치**
- https://www.typescriptlang.org/download/
- 노드가 설치되어 있는 상태에서 글로벌로 typescript 설치

```shell
$ node -v

$ npm install -g typescript

$ tsc -v  // typescript compiler
```

참고. VSCode 관련
- [설치 및 좋은 익스텐션 소개]( [https://youtu.be/bS9yTI2fC0w](https://youtu.be/bS9yTI2fC0w))
- [단축키]([https://youtu.be/EVxCdenPbFs](https://youtu.be/EVxCdenPbFs))
- [https://youtu.be/m7wsrVQsVjI](https://youtu.be/m7wsrVQsVjI)

### 북마크 해둬야 할 사이트 
- [공식 사이트](https://www.typescriptlang.org/) 
	- 커뮤니티, 플레이그라운드, 핸드북, 툴즈 등
- [깃허브](https://github.com/microsoft/TypeScript)


> [!note] 공식 사이트의 Playground 활용 
> TS 를 JS로 변환된 결과를 확인할 수 있다. 그리고 Target 통해 버전 정보 변경하면 문법 호환에 대한 부분을 알 수 있다. 예로 클래스는 ES6에서 지원하기 때문에 ES5 선택할 경우 함수형으로 변환되는 것을 확인할 수 있다.


### 타입 스크립트 컴파일러 툴
타입스크립트 코드를 자바스크립트 코드로 컴파일해서 실행하야 한다
```shell
# 컴파일 후 스크립트 실행
$ tsc main.ts
$ node main.js
```

아래 패키지를 통해 간편하게 타입스크립트 파일을 실행할 수 있다
```shell
$ npm install -g ts-node

# 바로 컴파일 해서 실행해준다
$ ts-node main.ts

# watch 옵션 통해 저장할 때마다 즉시 실행해준다
$ ts-node main.ts -w
```

---

## 2. 기본 타입
### 기본 타입 정리
- `let`, `const` : es6 도입
	- 이전에는 `var` 사용 ➡️ 호이스팅이나 다른게 문제가 됨
- JS 타입 분류
	- **Primitive Type**: number, string, boolean, bigint, symbol, null, undefined
	- **Object Type**: function, array ...

```typescript
// number
const num:number = 1; // 정수 소수 실수


// string
const str:string = 'hello';


// boolean
const boal:boolean = false;


// undefined: 값이 있는지 없는지 결정되지 않은 상태 (초기화 되지 않은 상태)
let age: number | undefined;
age = undefined;
age = 1;

function find() : number | undefined {
	return undefined;



// null : 비었다고 결정된 상태
let person: string | null;
person = null;
person = 'jason'


// unknonw 💩, JS에서 타입이 불확실한 경우 때문에, 비 권장
let notSure: unknown = 0;
notSure = 'he';
notSure = true;


// any 💩, 비 권장
let anything: any = 0;
anything = 'hello';


// void, 생략 가능
function print(): void {
	console.log('hello');
	return; // 생략 되어 있는 상태
}

function print() {
	console.log('world');
}


// never, 리턴하는 값이 없다는 것, return을 적으면 에러
function throwError(): never {
	// throw new Error(message);
	// 또는 while(true) { .. }
}


// object 💩, 원시 타입을 제외한 모든 타입을 넣을 수 있기 때문에 명시적으로 하는게 좋다
let obj: object;
function acceptSomeObject(obj: object) {
	// do something;	
}

```


### 함수에서 타입 이용하기 (JS -> TS)
타입을 정의함으로써 안정적으로 정의하고 사용할 수 있고, 문서화의 효과도 제공

```typescript
{
	// JS
	function jsAdd(num1, num2) {
		return num1 + num2;
	}

	// TS
	function add(num1: number, num2: number):number {
		return num1 + num2;
	}
}

```


```typescript
{
	// JS
	function fetchNum(id) {
		// do something..
		return new Promis((resolve, reject) => {
			resolve(100);
		});
	}

	// TS
	function fetchNum(id: string): Promise<number> {
		// do something..
		return new Promis((resolve, reject) => {
			resolve(100);
		});
	}
}

```


### spread, default, optional

```typescript
// Optional Parameter
function printName(firstName: string, lastName?: string) {
	console.log(firstName);
	console.log(lastName);
}

printName('tester'); // ? 추가 이후로 에러가 사라짐


function printName(firstName: string, lastName: string | undefined) {
	console.log(firstName);
	console.log(lastName);
}

printName('tester', undefined); // 이 경우 항상 파라미터 입력해줘야함


// Default Parameter
function printName(message: string = 'default message') {
	console.log(message);
}

printMessage(); 


// Rest Parameter
function addNumber(...numbers: number[]):number {
	return numbers.reduce((a, b) => a + b);
}

console.log(addNumber(1, 2));
console.log(addNumber(1, 2, 3, 4));
```

### 배열과 튜플

```typescript
{
	// Array
	const fruits: string[] = ['apple', 'banana'];
	const scores: Array<number> = [1, 3, 4];

	// readonly 추가하면 오브젝트의 수정이 불가하다
	function printArray(fruits: readonly string[]) {
		// do something
	}


	// Tuple, 사용 권장을 하진 않음
	let student: [string, number];
	student = ['name', 123];
	student[0]
	student[1]
	const [name, age] = student;
}

```
- 튜플을 사용하기 보다는 interface, type alias, class 권장 
	- 인덱스로 접근하는게 가독성 떨어짐💩
	- 💡참고로 React의 useState API를 사용하면 튜플 형태로 값을 반환한다 (유용한 경우✨)
		- 동적으로 사용자가 묶어서 쓰는 경우 튜플이 유용할 수 있음(의도가 있어야 한다는 의미인듯)

### Type Alias 🌹
새로운 타입을 정의할 수 있다 (powerful)
```typescript
{
	type Text = string;
	const name: Text = 'tester';
	const address: Text = 'korea';

	type Num = number;

	type Student = {
		name: string;
		age: number;
	}

	const student: Student = {
		name: 'tester', 
		age: 20
	};

	// String Literal Types, 타입의 기본 값으로 고정되네
	type Name = 'name';
	let testerName:Name;

	type JSON = 'json';
	const json: JSON = 'json';

	type Boal = true;
}
```

### Union 타입(OR)
- 활용도가 
```typescript
{
	type Direction = 'left' | 'right' | 'up' | 'down';
	function move(direction: Direction) {
		console.log(direction);
	}	

	move('down');


	type TileSize = 8 | 16 | 32;
	const tile: TileSize = 16;


	type SuccessState = {
		response : {
			body: string;
		}
	}
	type FailState = {
		reason: string;
	}
	type LoginState = SuccessState | FailState;
	function login(): Promise<LoginState> {
		return {
			response: {
				body: 'logged in!'
			}
		}
	}


	
}
```

### Discriminated Union

```typescript
{

	// Quiz
	// printLoginState(state)
	// success -> 🎉 body print
	// fail -> 😂 reason
	type SuccessState = {
		response: {
			body: string;
		}
	}
	type FailState = {
		reason: string;
	}
	type LoginState = SuccessState | FailState;

	function printLoginState(state: LoginState) {
		if('response' in state) {
			console.log(`🎉 ${state.response.body}`);
		} else {
			console.log(`😂 ${state.reason}`);
		}
	}


	// Discriminated Union 사용
	type SuccessState = {
		result: 'success';
		response: {
			body: string;
		}
	}
	type FailState = {
		result: 'fail';
		reason: string;
	}
	type LoginState = SuccessState | FailState;

	function printLoginState(state: LoginState) {
		iif(state.result === 'success') {
			console.log(`🎉 ${state.response.body}`);
		} else {
			console.log(`😂 ${state.reason}`);
		}
	}
}
```
- `Discriminated`: 차별받았다, 구별되었다
- `result` 타입을 둘 다 추가함
	- 공통 property를 가지게 함으로써 구분하기 쉽게 만든다

### Intersection 타입 (AND)

```typescript
{
	type Student = {
		name: string;
		score: number;
	}

	type Worker = {
		empolyeeId: number;
		work: () => void;
	}

	function internWork(person: Student & Worker) {
		console.log(person.name, person.employeeId, person.work());
	}

	internWork({
		name: 'tester',
		socre: 1,
		employeeId: 123
		work: () => console.log('work🛠️');
	});
}
```
- 다양한 타입을 하나로 묶어서 사용가능

### Enum 
```typescript
{
	// 이런 방식으로도 사용 가능하다
	const DAYS_ENUM = Object.freeze({"MONDAY": 0, "SUNDAY" : 6});	


	// TS
	// 숫자를 할당하면 나머지 순차적으로 할당됨
	// 문자열은 할당하려면 직접 다 선언해줘야 함
	enum Days {
		Monday = 1, // 0
		Tuesday, // 1
		Wednesday,
		Thursday,
		Friday,
		Satarday,
		Sunday
	}

	let day: Days = Days.Sunday;
	day = Days.Monday;
	day = 10; // 💩 아무 이슈 없이 동작하게 됨

	// ✨ 유니온 타입을 권장, 타입이 보장되니깐
	let dayOfWeek: DaysOfWeek = 'Monday' | 'Tuesday' | 'Sunday';
}

```
- 관련있는 상수를 그룹으로 묶어서 관리 및 사용
- TS에서 enum은 사용하지 않는 것을 권장 ➡️ 단, 다른 클라이언트와 통신할 때 언어가 다르면 Union을 지원하지 않기 때문에 enum을 사용하기도 함 
	- 예시와 같이 타입이 보장되지 않음
	- 차라리 **유니온 타입**을 사용하는게 나음🎉

### 타입 추론(Type Inference)

```typescript
{
	// 선언과 동시에 값을 할당했기 때문에 타입을 추론 가능
	let text = 'hello';
	text = 'world';

	// 암묵적으로 any 타입이 할당됨 💩
	function print(message) {
		console.log(message);
	}
	
	// 타입을 명시하는게 좋다, default 값이나
	function print(message: string) {
		console.log(message);
	}


	// 함수의 number 타입이 추론됨
	function add(x: number, y:number) {
		return x + y;
	}
}
```
- TS가 알아서 타입을 추론해주지만, 실무는 복잡하기 때문에 정확하게 타입 명시하는게 좋다
	- `void`는 생략 가능
	- 팀의 스타일 가이드를 명시적으로 정해, 일관성있게 작성하는 것이 좋다👍


### Type Assertion

```typescript 
{
	function jsStrFunc(): any {
		return 'hello';
	}

	const result = jsStrFunc(); // any이기 때문에 문자열 메서드 사용 못함
	console.log((result as string).length);
	console.log((<string>result).length);  


	const wrong: any = 5;
	console.log((wrong as Array<number>).push(5)); // 💩


	function findNumbers(): number | undefined {
		return undefined;
	}
	const numbers = findNumbers()!; //!: number라고 100% 확신
	numbers.push(2); // 💣
}
```
- 100% 장담할때 써야 함
	- <font color="#c00000">그렇지 않으면 런타임에 에러가 발생해 어플리케이션이 종료됨</font>💣

---

### 기본 타입 챌린지

**계산기 함수**
```typescript 
{
	type Command = 'add' | 'substract' | 'multiply' | 'divide' | 'remainder'

	function calculate(command: Command, a: number, b: number) {
		if(command === 'add') return a + n2;
		else if(command === 'substract') return a - b;
		else if(command === 'multiply') return a * b;
		else if(command === 'divide') return a / b;
		else if(command === 'remainder') return a % b;

		throw new Error('unknown error');
	}

	function calculate(command: Command, a: number, b: number) {
		switch(command) {
			case 'add': return a + b;
			case 'substract': return a - b;
			case 'multiply': return a * b;
			case 'divide': return a / b;
			case 'remainder': return a % b;
			default: throw new Error('unknown');
		}
	}
}

```

**좌표 게임**
- `{x:0, y : 0}` 좌표에서 시작으로 `move('up')`과 같이 커맨드 입력하면 한 칸씩 이동
```typescript 
{

	const position = { x : 0, y : 0};
	function move(direction: 'up' | 'down' | 'left' | 'right') {
		switch(direction) {
			case 'up' : postion.x -= 1; break; 
			case 'down' : position.x += 1; break;
			case 'left' :  position.y -= 1; break;
			case 'right' :  position.y += 1; break;
			default: throw new Error('unknown direction');
		}
	}

}
```


**로딩 상태 표시**

```typescript
{
	type SuccessState = {
		state: 'success';
		response : {
			body: string;
		}
	}

	type FailState = {
		state: 'fail';
		reason: string;
	}

	type LoadingState = {
		state: 'loading';
	}

	// Union 타입이라 한다
	type ResourceLoadState = LoadingState | SuccessState | FailState;

	function printLoginState(state : ResourceLoadState) {
		switch(state.state) {
			case 'loading': console.log(`👀loading...`); break;
			case 'success': console.log(`🙌 ${state.response.body}`); break;
			case 'fail':  console.log(`💣${state.reason}`); break; 
			default: throw new Error(`unknown : ${state.state}`);
		}
	}

}
```

---

## 객체 지향 프로그래밍(OOP)
- 프로그래밍 패러다임 중 하나

> [!info] a programming paradiam based on the concept of "objects" which can contain `data` and `code`


**🔄 Imperative and Procedural Programming** (명령과 절차적 프로그래밍)
- 절차지향적 프로그래밍 단점
	- 변경에 취약하다 (사이드 이펙트 발생 가능)
	- 유지보수 및 확장이 어렵다
- 객체지향 프로그래밍
	- 서로 관련있는 객체끼리 메시지를 통해 상호작용
	- 재사용 가능
	- 장점
		- Productivity : 생산성을 높여줌
		- Faster
		- higher-quality : 높은 퀄리티 


✅ **class vs object**
- `class`
	- template
	- declare once
	- no data in
- `object`
	- instance of a class
	- created many times
	- data in

### 객체지향 원칙 (4가지)
**✅ 캡슐화 (Encapsulation)**
- 서로 관련있는 데이터와 함수를 한 군데에 모음 
- + 외부로 노출 시킬 필요없는 의존관계를 숨김으로써 클라이언트와의 결합도를 낮추고, 객체의 내부의 상태 변경에 대한 접근 제어를 제공할 수 있다. #리팩터링2판 

**✅ 추상화 (Abstraction)**
- 내부의 복잡한 기능을 알 필요없이, 외부로 노출된 인터페이스 통해 사용할 수 있도록 함

**✅ 상속 (Inheritance)**
- `IS-A` 관계
- 예. animal 클래스를 상속 
	- dog
	- cat
- +자식 클래스는 부모 클래스의 속성과 메서드를 사용할 수 있다.
	- 단, 이로인해 부모 클래스오의 결합도가 높아지고 부모 클래스의 변경이 자식 클래스로 전파된다
	- 또한, 자식 클래스에서 부모 클래스의 메서드를 임의로 오버라이딩 하게 되면 캡슐화가 깨질 수 있다
	- 자바에서는 단일 상속만을 지원하므로 `상속보다는 위임을 사용하라`는 말을 흔히 함

**✅ 다형성 (Polymorphism)**
- 다양한(poly) 형태(morphism)


### 절차 지향적으로 커피 기계 만들기 💩
```typescript
{
	type CoffeeCup = {
		shots: number;
		hashMilk: boolean;
	};

	const BEANS_GRAMM_PER_SHOT:number = 7; // 커피 한잔당 필요한 원두 개수
	let coffeeBeans: number = 0;
	function makeCoffee(shots: number):CoffeeCup {
		if(coffeBeans < shots * BEANS_GRAMM_PER_SHOT) {
			throw new Error('Not enough coffee beans!🫘');
		}

		coffeeBeans -= shots * BEANS_GRAMM_PER_SHOT;
		return {
			shots: shots,
			hasMilk: false
		}
	};
}

coffeeBeans = 15;
const coffee = makeCoffee(2);
console.log(coffee);
```

```shell
// 실행
$ ts-node {*.ts}
```


### 객체 지향적으로 커피 기계 만들기 💡
```typescript
{
	type CoffeeCup = {
		shots: number;
		hashMilk: boolean;
	};

	class CoffeeMaker {
		static BEANS_GRAMM_PER_SHOT:number = 7; // class level
		coffeeBeans: number = 0; // instance(object) level

		constructor(beans:number) {
			this.coffeeBeans = beans;
		}

		static makeMachine(beans: number): CoffeeMaker {
			return new CoffeeMaker(beans);
		}
		
		makeCoffee(shots: number):CoffeeCup {
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
	
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
			return {
				shots: shots,
				hasMilk: false
			}
		};
	}	
}

const maker = new CoffeeMaker(100);
const coffee = maker.makeCoffee(2);
console.log(coffee);


const staticMaker = CoffeeMaker.makeMachine(10);
```
- 클래스는 템플릿이고, 데이터가 없는 상태
	- 인스턴스는 데이터가 있는 상태


### 캡슐화 시켜보기 (to. 커피머신)
- ex. 고양이가 느끼는 `감정`들은 모두 고양이의 `상태`
- `CoffeeMaker`를 초기화할 때 음수가 들어올 수 있다
	- 제약 조건 필요
	- 특별한 키워드가 없으면 접근제어자가 `public`으로 지정됨

```typescript
{
	type CoffeeCup = {
		shots: number;
		hashMilk: boolean;
	};

	class CoffeeMaker {
		private static BEANS_GRAMM_PER_SHOT:number = 7;  // ✨ 접근제어자 수정
		private coffeeBeans: number = 0; // ✨

		private constructor(beans:number) { // ✨
			this.coffeeBeans = beans;
		}

		static makeMachine(beans: number): CoffeeMaker {
			return new CoffeeMaker(beans);
		}

		fillCoffeeBeans(beans:number) {
			if(bean < 0) {
				throw new Error('value of beans should be greater than 0');
			}
			this.coffeeBeans = beans;
		}
		
		makeCoffee(shots: number):CoffeeCup {
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
	
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
			return {
				shots: shots,
				hasMilk: false
			}
		};
	}	
}

```
- 커피 콩을 외부에서 직접 필드에 접근해서 변경할 수 없다 
- 오로지 팩터리, 생성자, fillConffeeBeans() 를 통해서 상태 변경에 대한 접근 제어를 제공
	- **캡슐화**💊
	- 상태 변경되는 곳이 메서드로 제약 되기 때문에 문제 원인 파악이나 유지보수 용이

> [!info] 생성자가 private로 선언되어 있으면 정적 팩터리 메서드 같은게 있지 않나라는 생각으로 유연하게 이어질 수 있구나



### Abstraction 추상화 

```typescript

{
	type CoffeeCup = {
		shots: number;
		hashMilk: boolean;
	};

	interface CoffeeMaker { // prefix로 I*를 붙이기도 한다
		makeCoffee(shots: number): CoffeeCup;
	}

	class CoffeeMachine implements CoffeeMaker {
		private static BEANS_GRAMM_PER_SHOT:number = 7;
		private coffeeBeans: number = 0;

		private constructor(beans:number) {
			this.coffeeBeans = beans;
		}

		static makeMachine(beans: number): CoffeeMachine {
			return new CoffeeMachine(beans);
		}

		fillCoffeeBeans(beans:number) {
			if(bean < 0) {
				throw new Error('value of beans should be greater than 0');
			}
			this.coffeeBeans = beans;
		}

		private grindBeans(shots:number) {
			console.log(`grinding beans for ${shots}`);
			
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
			
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
		}

		private preheat() {
			console.log('heating up ... 🔥');
		}

		private extract(shots:number): CoffeeCup {
			console.log(`Pulling ${shots} shots... ☕️`);
			return {
				shots,
				hasMilk: false
			}
		}
		
		makeCoffee(shots: number):CoffeeCup {
			this.grindBeans(shots);
			this.preheat();
			return this.extract(shots);
		};
	}	
}


const maker:CoffeeMaker = CoffeeMachine.makeMachine(32);
maker.makeCoffee(2);
```
- 추상화는 복잡한 객체 내부 메서드를 노출할 필요없이 인터페이스를 간단하게 만듦으로써 클라이언트가 심플하게 사용할 수 있도록 지원 가능
	- `makeCoffee(..)`만 호출하면 커피를 만들 수 있다
- `CoffeeMaker` 인터페이스를 정의
	- 인터페이스에 없는 함수는 클라이언트가 호출 못함
	- 인터페이스를 통해 정보 은닉이랑 추상화 제공 가능
		- 정보 은닉: CoffeeMachine의 메서드 정보 숨김 
		- 추상화: 커피를 만들기 위해 grindBeans, preheat, extract 호출 및 순서 알 필요없이 인터페이스에 노출된 `makeCoffee`만 호출하면 됨
			- 단, `makeCoffee`를 구현할 책임이 있다 


### Interface, 모든 것의 시작
- CommercialCoffeeMaker 인터페이스 추가 

```typescript

{
	type CoffeeCup = {
		shots: number;
		hashMilk: boolean;
	};

	interface CoffeeMaker {
		makeCoffee(shots: number): CoffeeCup;
	}

	interface CommercialCoffeeMaker {
		makeCoffee(shots: number): CoffeeCup;
		fillCoffeeBeans(beans:number): vodi;
		clean(): void;
	}

	class CoffeeMachine implements CoffeeMaker, CommercialCoffeeMaker {
		private static BEANS_GRAMM_PER_SHOT:number = 7;
		private coffeeBeans: number = 0;

		private constructor(beans:number) {
			this.coffeeBeans = beans;
		}

		static makeMachine(beans: number): CoffeeMachine {
			return new CoffeeMachine(beans);
		}

		fillCoffeeBeans(beans:number) {
			if(bean < 0) {
				throw new Error('value of beans should be greater than 0');
			}
			this.coffeeBeans = beans;
		}

		private grindBeans(shots:number) {
			console.log(`grinding beans for ${shots}`);
			
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
			
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
		}

		private preheat() {
			console.log('heating up ... 🔥');
		}

		private extract(shots:number): CoffeeCup {
			console.log(`Pulling ${shots} shots... ☕️`);
			return {
				shots,
				hasMilk: false
			}
		}
		
		makeCoffee(shots: number):CoffeeCup {
			this.grindBeans(shots);
			this.preheat();
			return this.extract(shots);
		};

		clean(): void {
			console.log('cleaning...🧼');
		}
	}	
}


const maker:CoffeeMaker = CoffeeMachine.makeMachine(32);
maker.makeCoffee(2);


const maker:CommercialCoffeeMaker = CoffeeMachine.makeMachine(32);
maker.clean(); // 인터페이스에서 제공되는 메서드만 호출 가능
```


```typescript
{
	class AmateurUser {
		constructor(private machine: CoffeeMaker) {}
		makeCoffee() {
			const coffee = this.machine.makeCoffee(2);
			console.log(coffee);
		}
	}


	class ProBarista {
		constructor(private machine: CommercialCoffeeMaker) {}
		makeCoffee() {
			const coffee = this.machine.makeCoffee(2);
			console.log(coffee);
			this.machine.fillCoffeeBeans(45);
			this.machine.clean();
		}
	}

}


```
- 생성자에 인터페이스 규격화해서 기능 제약 가능

### Inheritance, 상속으로 다양한 커피 기계 만들기 

```typescript
{
	type CoffeeCup = {
		shots: number;
		hashMilk: boolean;
	};

	interface CoffeeMaker {
		makeCoffee(shots: number): CoffeeCup;
	}

	class CoffeeMachine implements CoffeeMaker {
		private static BEANS_GRAMM_PER_SHOT:number = 7;
		private coffeeBeans: number = 0;

		private constructor(beans:number) {
			this.coffeeBeans = beans;
		}

		static makeMachine(beans: number): CoffeeMachine {
			return new CoffeeMachine(beans);
		}

		fillCoffeeBeans(beans:number) {
			if(bean < 0) {
				throw new Error('value of beans should be greater than 0');
			}
			this.coffeeBeans = beans;
		}

		private grindBeans(shots:number) {
			console.log(`grinding beans for ${shots}`);
			
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
			
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
		}

		private preheat() {
			console.log('heating up ... 🔥');
		}

		private extract(shots:number): CoffeeCup {
			console.log(`Pulling ${shots} shots... ☕️`);
			return {
				shots,
				hasMilk: false
			}
		}
		
		makeCoffee(shots: number): CoffeeCup {
			this.grindBeans(shots);
			this.preheat();
			return this.extract(shots);
		};
	}	

	// ✅ 상속, Inheritance
	class CaffeeLatteMachine extends CoffeeMachine {
		constructor(beans:number, public readyonly serialNumber: string) {
			super(beans);
		}

		private steamMilk(): void {
			console.log('Steaming some milk...');
		}
		
		makeCoffee(shots: number): CoffeeCup {
			const coffee = super.makeCoffee(shots);
			this.steamMilk();
			return {
				...coffeee.shots,
				hasMilk: true
			}
		};		
	}
}

const machine = new CoffeeMachine(23);
const latteMachine = new CaffeLatteMachine(23, 'SSSS');
const coffee = latteMachine.makeCoffee(1);
console.log(coffee);
console.log(latteMachine.serialNumber);

```


### Polymorphism 다형성을 적용한 커피머신
```typescript
{
	type CoffeeCup = {
		shots: number;
		hashMilk?: boolean;
		hasSugar?: boolean; // ? : optional
	};

	interface CoffeeMaker {
		makeCoffee(shots: number): CoffeeCup;
	}

	class CoffeeMachine implements CoffeeMaker {
		private static BEANS_GRAMM_PER_SHOT:number = 7;
		private coffeeBeans: number = 0;

		private constructor(beans:number) {
			this.coffeeBeans = beans;
		}

		static makeMachine(beans: number): CoffeeMachine {
			return new CoffeeMachine(beans);
		}

		fillCoffeeBeans(beans:number) {
			if(bean < 0) {
				throw new Error('value of beans should be greater than 0');
			}
			this.coffeeBeans = beans;
		}

		private grindBeans(shots:number) {
			console.log(`grinding beans for ${shots}`);
			
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
			
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
		}

		private preheat() {
			console.log('heating up ... 🔥');
		}

		private extract(shots:number): CoffeeCup {
			console.log(`Pulling ${shots} shots... ☕️`);
			return {
				shots,
				hasMilk: false
			}
		}
		
		makeCoffee(shots: number): CoffeeCup {
			this.grindBeans(shots);
			this.preheat();
			return this.extract(shots);
		};
	}	

	class CaffeeLatteMachine extends CoffeeMachine {
		constructor(beans:number, public readyonly serialNumber: string) {
			super(beans);
		}

		private steamMilk(): void {
			console.log('Steaming some milk...');
		}
		
		makeCoffee(shots: number): CoffeeCup {
			const coffee = super.makeCoffee(shots);
			this.steamMilk();
			return {
				...coffeee.shots,
				hasMilk: true
			}
		};		
	}

	class SweetCoffeeMaker extends CoffeeMachine {
		makeCoffee(shots:number): CoffeeCup {
			const coffee = super.makeCoffee(shots);
			return {
				...coffee,
				hasSugar: true
			}
		}
	}
}

const machines: CoffeeMaker = [
	new CoffeeMachine(16),
	new CaffeeLatteMachine(16, '1'),
	new SweetCoffeeMaker(16),
	new CoffeeMachine(16),
	new CaffeLatteMachine(16, '1'),
	new SweetCoffeeMaker(16)
];

machines.forEach(machine => {
	console.log('---------');
	machine.makeCoffee(1);
});
```
- 다형성을 통해 다양한 인스턴스(같은 인터페이스)에 있는 공통 API를 호출 가능하다
	- 인터페이스 규격에 대한 세부 내용은 각 클래스별로 다양하게 구현 가능
	- 클라이언트는 내부 구현에 알 필요없이 인터페이스에 정의된 규약에 따라 호출하면 이용 가능하다 

### 상속의 문제점💩
- 상속은 수직적으로 관계가 형성된다
- 상속의 깊이가 깊어질 수록 디버깅이 어려워짐
- 부모와 자식이 강결합하고 있어서 변경에 취약해짐 
	- 부모에 추상 메서드나 필드 추가시 자식 클래스에 변경 전파 
	- 자식 클래스가 부모 클래스 내용을 다 알기 때문에 오버라이딩 잘못해서 캡슐화 깨질 수 있다
	- 무분별한 상속 남용으로 클래스 폭발 💥
		- 다중 상속은 지원하지 않음 
		- 조합을 하려면 결국 클래스가 추가될 수 밖에 없음

### Composition, 위임
- `Favor Composition over Inheritance`
	- 상속보다는 위임을 사용해라
- 상속이 무조건 나쁜건 아니다 
	- +너무 상속만 사용해서 깊어지면 디버깅도 힘들고 정말 상속이 필요한 경우에 변경이 어려워질 수 있다.


```typescript
{

	// 싸구려 우유 거품기
	class CheapMilkSteamer {
		private steamMilk(): void {
			console.log('Steaming some milk ... 🥛');
		}

		makeMilk(cup: CoffeeCup): CoffeeCup {
			this.steamMilk();
			return {
				...cup,
				hasMilk: true
			}
		}
	}

	// 설탕 제조기
	class AutomaticSugarMixer {
		private getSugar() {
			console.log('Getting some sugar from candy 🍬');
		}

		addSugar(cup: CoffeeCup): CoffeeCup {
			const sugar = this.getSugar();
			return {
				...cup,
				hasSugar: sugar;
			}
		}
	}

	interface CoffeeMaker {
		makeCoffee(shots: number): CoffeeCup;
	}

	class CoffeeMachine implements CoffeeMaker {
		private static BEANS_GRAMM_PER_SHOT:number = 7;
		private coffeeBeans: number = 0;

		private constructor(beans:number) {
			this.coffeeBeans = beans;
		}

		static makeMachine(beans: number): CoffeeMachine {
			return new CoffeeMachine(beans);
		}

		fillCoffeeBeans(beans:number) {
			if(bean < 0) {
				throw new Error('value of beans should be greater than 0');
			}
			this.coffeeBeans = beans;
		}

		private grindBeans(shots:number) {
			console.log(`grinding beans for ${shots}`);
			
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
			
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
		}

		private preheat() {
			console.log('heating up ... 🔥');
		}

		private extract(shots:number): CoffeeCup {
			console.log(`Pulling ${shots} shots... ☕️`);
			return {
				shots,
				hasMilk: false
			}
		}
		
		makeCoffee(shots: number): CoffeeCup {
			this.grindBeans(shots);
			this.preheat();
			return this.extract(shots);
		};
	}	

	class CaffeeLatteMachine extends CoffeeMachine {
		constructor(beans:number, 
		public readyonly serialNumber: string,
		private milkFother: CheapMilkSteamer) { // ✅
			super(beans);
		}

		private steamMilk(): void {
			console.log('Steaming some milk...');
		}
		
		makeCoffee(shots: number): CoffeeCup {
			const coffee = super.makeCoffee(shots);
			return this.milkFother.makeMilk(coffee); // ✅
		};		
	}

	class SweetCoffeeMaker extends CoffeeMachine {
		constructor(private beans: number, 
		private sugar: AutomaticSugarMixer) {
			super(beans);
		}

		makeCoffee(shots:number): CoffeeCup {
			const coffee = super.makeCoffee(shots);
			return sugar.addSugar(coffee); // ✅
		}
	}

	// ✅
	class SweetCaffeLatteMachine extends CoffeMachine {
		constructor(
			private beans: number,
			private milk: CheapMilkSteamer,
			private sugar: AutomaticSugarMixer
		) {
			super(beans);
		}

		makeCoffee(shots:number): CoffeeCup {
			const coffee = super.makeCoffee(shots);
			const sugarAdded = this.suagr.addSugar(coffee); // ✅
			
			return milk.makeMilk(sugarAdded); // ✅
		}
	}

}
```
- `CaffeLatteMachine`
	- DI 통해 외부에서 객체를 주입하여 사용
	- 코드의 재사용성을 높여줌
		- 지금 코드의 단점 ➡️ 커플링이 높아짐 (연관관계를 멤버 필드로 가지게 되니깐 변경에 취약해질수도, 추상화에 의존하도록 하자! 하이레벨의존하고, 로우 레벨에 의존하지 않도록)

### 강력한 interface
- `상속보다 컴포지션을 선호해라`
	- DI 받는 타입을 인터페이스(고수준) 추상화에 의존하도록 한다💡

```typescript 

{

	interface MilkFrother {
		makeMilk(cup: CoffeeCup): CoffeeCup;
	}

	interface SugarProvider {
		addSugar(cup: CoffeeCup): CoffeeCup;
	}


	// 싸구려 우유 거품기
	class CheapMilkSteamer implements MilkFrother {
		private steamMilk(): void {
			console.log('Steaming some milk ... 🥛');
		}

		makeMilk(cup: CoffeeCup): CoffeeCup {
			this.steamMilk();
			return {
				...cup,
				hasMilk: true
			}
		}
	}

	// 설탕 제조기
	class AutomaticSugarMixer implements SugarProvider {
		private getSugar() {
			console.log('Getting some sugar from candy 🍬');
		}

		addSugar(cup: CoffeeCup): CoffeeCup {
			const sugar = this.getSugar();
			return {
				...cup,
				hasSugar: sugar;
			}
		}
	}

	// ✅ DI 받는 타입을 인터페이스로 변경, 구현체에 의존하지 않고 추상화에 의존
	class SweetCaffeLatteMachine extends CoffeMachine {
		constructor(
			private beans: number,
			private milk: MilkFrother, 
			private sugar: SugarProvider
		) {
			super(beans);
		}

		makeCoffee(shots:number): CoffeeCup {
			const coffee = super.makeCoffee(shots);
			const sugarAdded = this.suagr.addSugar(coffee);
			
			return milk.makeMilk(sugarAdded); 
		}
	}

}
```
- `인터페이스 == 규격서`
- `MilkFrother`, `SuagrProvider` 인터페이스 구현체를 갈아가면서 사용가능
	- 사용하는 커피 머신 입장에서는 구현체가 무엇인지 알 필요 없이 인터페이스 규격에 따라 호출하여 사용하기만 하면 된다✨
	- 이름이 복잡한 상속 클래스가 필요없어지고, `CoffeeMachine`  통해서 처리 가능 해짐

```typescript 
{
	interface MilkFrother {
		makeMilk(cup: CoffeeCup): CoffeeCup;
	}

	interface SugarProvider {
		addSugar(cup: CoffeeCup): CoffeeCup;
	}

	// Null Object Pattern
	class NoMilk implements MilkFrother {
		makeMilk(cup:CoffeeCup): CoffeeCup {
			return cup;
		}
	}

	// Null Object Pattern
	class NoSugar implements SugarProvider {
		addSugar(cup: CoffeeCup): CoffeeCup {
			return cup;
		}
	}

	class CoffeeMachine implements CoffeeMaker {
		private static BEANS_GRAMM_PER_SHOT:number = 7;
		private coffeeBeans: number = 0;

		private constructor(beans:number,
			private milk: MilkFrother,
			private sugar: SugarProvider) {
			this.coffeeBeans = beans;
		}

		fillCoffeeBeans(beans:number) {
			if(bean < 0) {
				throw new Error('value of beans should be greater than 0');
			}
			this.coffeeBeans = beans;
		}

		private grindBeans(shots:number) {
			console.log(`grinding beans for ${shots}`);
			
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
			
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
		}

		private preheat() {
			console.log('heating up ... 🔥');
		}

		private extract(shots:number): CoffeeCup {
			console.log(`Pulling ${shots} shots... ☕️`);
			return {
				shots,
				hasMilk: false
			}
		}
		
		makeCoffee(shots: number): CoffeeCup {
			this.grindBeans(shots);
			this.preheat();
			const coffee this.extract(shots);
			const sugarAdded = this.suagr.addSugar(coffee);
			return milk.makeMilk(sugarAdded); 
		};
	}	

}

```
- `Null Object Pattern` 적용
	- `NoMilk` 클래스
	- `NoSugar` 클래스


> [!info] 오버 엔지니어링 하지 마라 ! 어느정도 중간점을 지키면서 개발하는 것도 필요


### Abstract 클래스 사용 예시
- **추상 클래스**
- 상속 관계에서 반복되는 필드나 메서드가 있으면 추상 클래스를 만들어서 상속할 수 있다
	- 추상 클래스는 자체적으로 생성자 초기화할 수 없다 
	- 자식 클래스에서 super(..)

```typescript 
{
	// ✅ abstract 키워드 추가
	abstract class CoffeeMachine implements CoffeeMaker {
		private static BEANS_GRAMM_PER_SHOT:number = 7;
		private coffeeBeans: number = 0;

		private constructor(beans:number) {
			this.coffeeBeans = beans;
		}

		fillCoffeeBeans(beans:number) {
			if(bean < 0) {
				throw new Error('value of beans should be greater than 0');
			}
			this.coffeeBeans = beans;
		}

		private grindBeans(shots:number) {
			console.log(`grinding beans for ${shots}`);
			
			if(this.coffeBeans < shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT) {
				throw new Error('Not enough coffee beans!🫘');
			}
			
			this.coffeeBeans -= shots * CoffeeMaker.BEANS_GRAMM_PER_SHOT;
		}

		private preheat() {
			console.log('heating up ... 🔥');
		}

		// ✅ 추상 메서드 선언, 자식 클래스에서 무조건 구현해줘야 함
		protected abstract extract(shots:number): CoffeeCup;
		
		makeCoffee(shots: number): CoffeeCup {
			this.grindBeans(shots);
			this.preheat();
			return this.extract(shots);
		};
	}	


	class SweetCoffeeMaker extends CoffeeMachine {
		constructor(beans:number) {
			super(beans);
		}

		protected extract(shots:number): CoffeeCup {
			return {
				shots,
				hasSugar: true
			};
		}
	}

}
```

---

### 객체지향 챌린지
- Stack 만들어보기

**단일 연결 리스트**로 구현
```typescript
{
	interface Stack {
		readonly size: number;
		push(value: string): void;
		pop(): string;
	}

	type StackNode = {
		readonly value: string; // 불변성 유지 위해 readonly 
		readonly next?: StackNode; // StackNode | undefined
	}

	class StackImpl implements Stack {
		 // _ : 내부에서만 사용하는 값, 동일한 public 변수가 있다는걸 유추 가능
		private _size: number; // 요소의 개수만 카운팅, 링크드 리스트이기 때문에
		private head?: StackNode;  // ? : optional

		constructor(private capacity: number) {}
		
		get size() {
			return this._size;
		}

		push(value: string) {
			if(this.size === this.capacity) {
				throw new Error('Stack is full');
			}

			const node: StackNode = {
				value,
				next: this.head
			};

			this.head = node;
			this._size += 1;
		}

		pop(): string {
			// null == undefined(true), null !== undefined (true)
			if(this.head == null) {
				throw new Error('Stack is Empty');
			}
			
			const node = this.head;
			this.head = node.next;
			this._size -= 1;

			return node.value;
		}
	}

	const stack = new StackImpl(10);
	stack.push('1');
	stack.push('2');
	stack.push('3');

	while(stack.size !== 0) {
		console.log(stack.pop()); // 3 2 1
	}

}

```

---
## 제네릭(Generic)

### 함수에 제네릭 선언
```typescript
{
	function checkNotNull(arg: number | null): number {
		if(arg == null) {
			throw new Error('now valid number!');
		}

		return arg;
	}

	const result = checkNoNull(123);
	console.log(result);
	checkNotNull(null); // throw Error

	// 💩 타입 정보가 없어서 안정성도 줄어듦
	function checkNotNull(arg: any | null): any {..}


	// 💡 제네릭 선언, 함수가 사용될 때 타입 결정된다
	function checkNotNull<T>(arg: T | null): T {
		if(arg == null) {
			throw new Error('now valid number!');
		}

		return arg;
	}


	const boal:boolean = checkNotNull(true);
}
```
- `number` 타입만 받을 수 있는 상태 💩
	- 다른 타입도 지원하려면 ?


### 클래스에 제네릭 선언

```typescript
{
	interface Employee {
		pay(): void;
	}

	class FullTimeEmployee implements Employee {
		pay() {
			console.log('full time!!');
		}

		workFullTime() {}
	}

	class PartTimeEmployee implements Employee {
		pay() {
			console.log('part time!!');
		}

		workPartTime() {}
	}

	// 세부적인 타입을 인자로 받아서 정말 추상적인 타입으로 다시 리턴하는 함수 💩
	function payBad(employee: Employee): Employee {
		employee.pay();
		return employee;
	}

	// 💡
    function pay<T extends Employee>(employee: T): T {
	    employee.pay();
	    return employee;
    }

	const shelly = new FullTimeEmployee();
	const bob = new PartTimeEmployee();
	shelly.workFullTime();
	bob.workPartTime();

	const shellyAfterPay = payBad(shelly);
	const bobAfterPay = payBad(bob);

	shellyAfterPay.workFullTime(); // ❌ 타입 정보 잃어비림
	bob.workPartTime(); // ❌, 인터페이스 메서드만 노출


	const shellyAfterPay = pay(shelly);
	const bobAfterPay = pay(bob);

	shellyAfterPay.workFullTime();
	bob.workPartTime();
}
```


오브젝트와 키값을 파라미터로 전달되었을 때 타입이 보장되면서 값을 리턴하는 함수 생성
```typescript
{

	const obj = {
		name: 'shelly',
		age: 20 
	};

	const obj2 = {
		animal: '🦔'
	}

	// K : T 오브젝트에 포함된 키만 허용💡
	function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
		return obj[key];
	}

	console.log(getValue(obj, 'name'));
	console.log(getValue(obj, 'age'));
	console.log(getValue(obj2, 'animal'));
}

```

### 제네릭 연습
string 타입만 지원하는 Stack을 제네릭 지원하도록 변경
```typescript
{
	interface Stack<T> {
		readonly size: number;
		push(value: T): void;
		pop(): T;
	}

	type StackNode<T> = {
		readonly value: T; // 불변성 유지 위해 readonly 
		readonly next?: StackNode<T>; // StackNode | undefined
	}

	class StackImpl<T> implements Stack<T> {
		 // _ : 내부에서만 사용하는 값, 동일한 public 변수가 있다는걸 유추 가능
		private _size: number; // 요소의 개수만 카운팅, 링크드 리스트이기 때문에
		private head?: StackNode<T>;  // ? : optional

		constructor(private capacity: number) {}
		
		get size() {
			return this._size;
		}

		push(value: T) {
			if(this.size === this.capacity) {
				throw new Error('Stack is full');
			}

			// 타입 추론
			const node = {
				value,
				next: this.head
			};

			this.head = node;
			this._size += 1;
		}

		pop(): T {
			// null == undefined(true), null !== undefined (true)
			if(this.head == null) {
				throw new Error('Stack is Empty');
			}
			
			const node = this.head;
			this.head = node.next;
			this._size -= 1;

			return node.value;
		}
		
	}

	const stack = new StackImpl<string>(10);
	stack.push('1');
	stack.push('2');
	stack.push('3');

	while(stack.size !== 0) {
		console.log(stack.pop()); // 3 2 1
	}

	const stack = new StackImpl<number>(10);
	stack.push(123);
	stack.push(456);
	stack.push(789);

	while(stack.size !== 0) {
		console.log(stack.pop()); // 789 456 123
	}
}
```


---

## API 읽어보기  (여기부터 조금 어렵네.. 학습 테스트 필요)
- https://github.com/microsoft/TypeScript/blob/main/src/lib/es5.d.ts
- `as` : type assertion, 타입이 확실할 때 지정 가능
- `value is S` : S 타입인지 

```typescript
class Animal {}

class Cat extends Animal {
	isCat: boolean = true;
}

class Dog extends Animal {
	isDog: boolean = false; 
}

const animals: Animal[] = [new Cat(), new Cat(), new Dog()];
function isCat(animal: Animal): animal is Cat {
	return (animal as Cat).isCat !== undefined;
}

console.log(animals.every<Cat>(isCat)); // 전부 맞으면 true
```

### 오픈 소스 프로젝트 활용하기
- 프로그래밍 언어를 공부한다고 해서 실력이 오르진 않는다 
- 공식사이트와 API를 잘 활용
- 잘 작성되어 있는 코드를 보기만 해도 실력 향상 가능
	- 오픈 소스 프로젝트를 활용💡 (제품에 사용된 걸 보는게 더 좋다)
	- https://github.com/microsoft/vscode
		- `common` 디렉터리나 찾아보기
	- https://github.com/microsoft/TypeScript


---

## 에러 처리(Exception Handling)
- ERROR : expected
- Exception: unexpected
	- 반대 아닌가?? 에러는 걸리면 안되는 거고, Exception은 잡거나 던지거나 둘 중 하나

```markdown
**특정한 경로의 파일을 읽어서 데이터를 보여주는 앱을 예로 들면:**

해당 경로가 존재한다고 100% 확신하고, 데이터를 보여주죠? 여기서 try { 파일 읽기 } catch(error) 는 우리가 100% 존재 한다고 확신했지만, 혹시 만약의 경우를 대비해서, 파일을 읽지 못하는 예외 (exception)이 발생하면 catch에서 에러 처리를 해주죠.

여기서 발생하는 에러는 **예****외 상황**이예요. 앱이 정상적으로 동작하지 않을때 발생할 수 있는 예외상황. 그쵸?

제가 강의에서 언급한 에러는, 앱에서 발생할 수 있는 시스템 적인 메모리 부족이나, 배터리 부족, 보안 에러 등과 같은 **심각한 에러를 (예상하지 못하는, 시스템 적인 에러 등)** 말하는 것이 아니라, **우리 앱에서 발생할 수 있는 예상하는 실패 케이스들을 말한답니다.**

예를 들어, 로그인 실패는 예외 상황 (exception)이 아니라, 앱을 사용하는 use case중 발생할 수 있는 케이스중 하나이기 때문에 exeption으로 처리 하는게 아니라, Error State(LoginFail과 같은)를 만들어서 처리 하자는거죠
```


> JS에서는 Error가 있음 (Java는 Exception)


```typescript
{
	function move(direction: 'u' | 'd' | 'l' | 'r')
		swtich(direction) {
			case 'u':
				position.y += 1;
				break;
			case 'd':
				position.y -= 1;
				break;
			case 'l':
				postion.x -= 1;
				break;
			case 'r':
				position.x += 1;
				break; 
			default:
				const invalid: never = direction; // ✅
				throw new Error(`unknown : {invalid}`);	
		}
	}
}
```
- 모든 케이스를 다 처리했을때 default에는 `never`가 할당됨
	- 이때 케이스를 전부 처리하지 않으면 default에 never 변수에서 컴파일 에러 발생


### try, catch, finally
```typescript
{
	function readFile(fileName: string): string {
		if(fileName === 'not exist!💩') {
			throw new Error(`file not exist ! ${fileName}`);
		}
		return 'file contents';
	}

	function closeFile(fileName: string) {
		console.log(fileName);
	}

	const fileName = 'not exist!💩';
	try {
		console.log(readFile(fileName));
	} catch(error) {
		console.log(`catched`);
	} finally {
		closeFile('finally');
	}

}
```


```typescript
{
	class NetworkClient {
		tryConnect(): void {
			throw new Error('no network');
		}
	}

	class UserService {
		constructor(private client: NetworkClient){}

		login() {
			this.client.tryConnect();
			// login...
		}
	}

	class App {
		constructor(private userService: UserService){}

		run() {
			try {
				userService.login();
			} catch(error) { // any 타입
				// ..
			}
		}
	}


	const app = new App(new UserService(new NetworkClient));
	app.run(); // 에러 로그 출력 
}
```
- try, catch문은 
	- `UserService`
		- 우아하게, 정확하게 처리하지 못한다면 외부로 던지는게 나을 수 있다
	- `App`
		- 여기서 의미있는 에러 처리를 가능하다

> ✅ 여기서 처리하는게 의미가 있는가? 우아하게 처리할 수 있는 곳에서 try catch를 선언한다


### Error State
- Error를 catch하면 any 타입이 되서 타입 정보가 잃어버리게 된다

```typescript
{
	type NetworkErrorState = {
		result: 'fail';
		reason: 'offline' | 'down' | 'timeout';
	}

	type SuccessState = {
		result: 'success';
	}
	
	type ResultState = SuccessState | NetworkErrorState;

	class NetworkClient {
		tryConnect(): ResultState {
			//
		}
	}

	class UserService {
		constructor(private client: NetworkClient){}

		login() {
			this.client.tryConnect();
			// login...
		}
	}

	class App {
		constructor(private userService: UserService){}

		run() {
			const result = userService.login();
			// ResultState 타입을 반환하는 걸 알기에 조건문 분기 처리 가능
		}
	}
}
```


**💡 예외와 에러**
```text
좋은 질문 이예요 :) 

말씀하신 것처럼 예외(Exception)는 프로그램에서 발생할 수 있는 예외적인 상황에 대해 얘기해요. 파일을 정상적으로 읽어 오지 못했거나, 네트워크에 접속이 안된다던지요.

에러는 시스템 에러, 메모리 에러, 문법 에러 등 예외적인 상황을 포함하는 조금더 포괄적인 것을 말할 수 있을 것 같아요.

제가 말하고 싶은 포인트는 이런 용어 적인 정의보다는, 간혹 개발자 분들이 성공적인 케이스만 (Happy Path 라고 부르지요) 생각해서 프로그램을 작성하고 그 외적인 것들은 다 예외로 간주하는 경우가 많은데요. 그렇게 프로그래밍을 하면 프로그램의 안정성과 사용성이 떨어지고 유지보수도 어려워요.

발생할 수 있는 예외에 대해 무작정 (Try-Catch) 또는 throw new Error() 예외 처리를 하기 보다는, 예상 가능한 예외 상황이라면 제가 영상에서 보여드린 예제처럼 에러 상태를 정의해서 예외 적인 상황이 아니라, 우리가 예상하고 있는 에러 상황(상태)로 간주해서 각기 다른 처리를 해주는것이 좋다고 생각해요 :)


// 다른 질문 글에서 
각각 장단점이 존재 하기 때문에 **이 함수를 사용하는 사람이 알고 싶은것은(return) 무엇인가? 내가 꼭 무엇을 알려줘야 하는가?** 이 질문들에 대답을 찾다보면 어떤 값을 return해 줘야 하는지 좋은 답을 얻을 수 있다고 생각해요 :)

```


---

## 타입스크립트의 핵심
### Type Alias와 interface 중에 뭘 써야 할까 (기술 측면)
- 두 개의 차이점을 인지하고 사용해야 함

**구현 차이 살펴보기**
```typescript
type PositionType = {
	x: number;
	y: number;
}

interface PositionInterface {
	x: number;
	y: number;
}
```


```typescript 
// object
const obj1: PositionType {
	x: 1, 
	y: 1
}

const obj1: PositionInterface {
	x: 1, 
	y: 1
}


// class
class Pos1 implements PositionType {
	x: number;
	y: number;
}

class Pos2 implements PositionInterface {
	x: number;
	y: number;
}


// Extends 가능
interface ZPositionInterface extends PositionInterface {
	z: number;
}

// inter secation
type ZPositionType = PositionType & { z: number };


```

`interface`에서만 가능한 것들
```typescript
// only interfaces can be merged 👀 (최신 문서 찾아보기)
interface PositionInterface {
	x: number;
	y: number;
}

interface PositionInterface {
	z: number;
}

const obj: PositionInterface {
	x: 1, 
	y: 1,
	z: 1
}
```


`type`만 가능한 것들
```typescript 
type Person = {
	name: string,
	age: number
}

type Name = Person['name']; // string type이 된다
type NumberType = number;
type Direction = 'left' | 'right'; // union type 

```


### Type Alias와 interface 중에 뭘 써야 할까 (개념 측면)

`interface`
- 특정한 **규격사항** 정의
- API는 서로 간의 소통을 위한 약속

```typescript
interface CoffeeMaker {
	coffeeBeans: number;
	makeCoffee: (shots: number) => Coffee;
}

class CoffeeMachine implements CoffeeMaker {
	coffeeBeans: number;
	makeCoffee(shots: number) {
		return {};
	}
}
```


`types`
- 어떠한 데이터를 담을지 타입을 결정
- 구현 목적이 아니라 **데이터를 담을 목적**이라면 인터페이스보다 타입이 적절
	- dto, record 느낌이네👀

```typescript 
type Position = {
	x: number;
	y: number;
}

const pos: Position = { x: 0, y: 0 };
printPosition(pos);
```

### Utility Type이란
- 타입스크립트에서는 type을 변환(transform)하는게 가능하다
- 제공되는 유틸리티 타입을 사용할 수 도 있지만 원리를 이해하고 사용하는게 중요

### Index Type
9-2-index.ts
```typescript
{
	const obj = {
		name: 'tester'
	}

	obj.name; // tester
	obj['name']; // tester

	type Animal = {
		name: string;
		age: number;
		gender: 'male' | 'female';
	}

	type Name = Animal['name']; // string 타입으로 결정
	const text:Name = 'temp';

	type Gender = Animal['gender']; // 'male' | 'female'

	type Keys = keyof Animal; // 'name' | 'age' | 'gender', 문자열만
	const key: keys = 'gender';

	type Person = {
		name: string;
		gender: Animal['gender'];
	};
	const person: Person = {
		name: 'tester',
		gender: 'male'
	};
	
}
```
- type도 인덱스 기반으로 결정 가능
	- 인덱스로 키값을 호출하면 키의 타입이 할당된다는 거네
	- 위에 Person 예제를 보면 키의 값이 할당되네


### Mapped Type
- 기존 타입을 다른 타입으로 맵핑하는 타입이라는듯
- 재사용성이 높다

9-3-map.ts
```typescript 
type Video = {
	title: string;
	author: string;
	descrition: string;
}

// 💩
type VideoOptional = {
	title?: string;
	author?: string;
	descrition?: string;
}

// ✨
type Optional<T> = {
	[P in keyof T]?: T[P] // for...in과 비슷
};
type VidoeOptional = Optional<Video>;


type Animal = {
	name: string;
	age: number;
}
const animal: Optional<Animal> = {
	name: 'dog';
}

// 읽기 전용
type ReadOnly<T> = {
	readonly [P in keyof T]: T[P];
}

```
- Video가 변경되면 VideoOptional도 변경이 일어나야함 (코드의 변경 발생, 불편함)
- type안에서 `[]`기호 사용하면 for...in 처럼 key 조회할 수 있다


```typescript 
{
	// null을 허용함
	type Nullable<T> = {
		[P in keyof T]: T[P] | null;
	}

	const obj: Nullable<Video> = {
		title: 'hi',
		author: null
	}


	type Proxy<T> = {
		get(): T;
		set(value: T): void;
	}

	type Proxify<T> = {
		[P in keyof T]: Proxy<T[P]>;
	}
}
```


임의 예제 
```typescript 
{
	type ProxyWrapper = <T>(value: T) => Proxy<T>

	const wrappedProxy: ProxyWrapper = (value: T) => {
		let _v = value;
		return {
			get: () => _v,
			set:(value: T) => { _v = value }
		}
	}

	const videoProxy: Proxyfy<Video> {
		title: wrapperProxy('영상 제목'),
		author: wrapperProxy('영상 제작자'),
		description: wrapperProxy('영상 설명'),
	}

	// 속성별로 getter, setter 호출 가능
}
```

### Conditional Type
- 조건으로 타입을 결정 가능하네

9-4-condition.ts
```typescript
type Check<T> = T extends string? boolean : number;

type Type = Check<string>; // string을 상속하기 때문에 boolean 타입이 됨

// 공식 예제
type TypeName<T> = T extends string
	? 'string'
	: T extends number
	? 'number'
	: T extends boolean
	? 'boolean'
	: T extends undefined
	? 'undefined'
	? T extends Function
	? 'function'
	: 'object';

type T0 = TypeName<string>; // string
type T1 = TypeName<'a'>; // string
type T2 = TypeName<() => void>; // function
```

### ReadOnly 
- 속성의 불변성을 보장하도록 Readonly 타입을 지원
- 공식 문서를 보면 **유틸 클래스** 통해 미리 정의된 `type`  지원
	- 많이 사용하는 것들 위주로 설명

9-5-readonly.ts
```typescript 
{
	type Todo = {
		title: string;
		description: string;
	}

	function display(todo: Readonly<Todo>) {
		//..
	}
}
```


### Partial Type
- 기존 타입 중에 부분적으로 허용하고 싶은 경우 사용 

9-6-partial.ts
```typescript
{
	type Todo = {
		title: string;
		description: string;
		label: string;
		priority: 'high' | 'low';
	}

	function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>): Todo {
		return { ...todo, ...fieldsToUpdate };	
	}

	const todo: Todo = {
		title: 'learn ts',
		description: 'study hard',
		label: 'study',
		priority: 'high'
	};

	const updated = updateTodo(todo, { priority: 'low'});
	console.log(updated);
}
```
- `Partial<Todo>` 
	- Todo의 속성만 
### Pick Type
- 기존에 있는 타입에서 원하는 것만 골라서 제한된 타입을 재정의할 때 사용

9-7-pick.ts

```typescript
{
	type Video = {
		id: string;
		title: string;
		url: string;
		data: string;
	}

	function getVideo(id: string): Video {
		return {
			// 전체 속성 주저리주저리
		}
	}

	// 제한된 속성만 뽑아서 사용, Video 타입에 있는 키 중 선택
	type VideoMetadata = Pick<Video, 'id' | 'title'>;

	function getVideoMetadata(id: string): VideoMetadata {
		return {
			id: id,
			title: 'title'
		};
	}

	
}
```

### Omit Type
- pick과 반대로 원하는 것을 제외 가능

```typescript
{
	type Video = {
		id: string;
		title: string;
		url: string;
		data: string;
	}

	function getVideo(id: string): Video {
		return {
			// 전체 속성 주저리주저리
		}
	}

	// Pick과 반대, 원하는 속성을 제외 처리
	// 이때 두번째 타입에 any를 받아서 없는 속성도 받음 (의미는 없음)
	// 문서에 제니릭 해석해보기
	type VideoMetadata = Omit<Video, 'url' | 'data'>;

	function getVideoMetadata(id: string): VideoMetadata {
		return {
			id: id,
			title: 'title'
		};
	}
		
}
```

### Record ? 
- 두 타입을 묶을 때 사용 
- 이때 앞에 타입이 키가 되고, 뒤에 타입이 키의 value 타입이 된다

9-9-record.ts
```typescript 
{
	type PageInfo = {
		title: string;
	}
	type Page = 'home' | 'about' | 'contact';

	// Page를 키로 삼고, PageInfo를 value로 삼음
	const nav: Record<Page, PageInfo> = {
		home: {title: 'home'},
		about: {title: 'about'},
		contact: {title: 'contact'}
	}
}
```


### 기타
- 지금까지 유틸리티 타입을 살펴봄
	- 이외에도 많은 유틸리티 타입이 많음 
	- ReturnType, Cap.. 등등


---

## JavaScript 정리

### 프로토타입 (Prototype) 
- **TS** = superset of JavaScript
- ES6
	- class-like
- JS
	- proto-based

> 자바스크립트의 프로토타입은 상속을 위해서 사용


**Prototype-based programming**
- a style of OOP
	- behavior reuse (inheritance)
- be resuing existing objects
	- that serve as prototype

**데모**
`10-1-proto.js`
```javascript
const x = {}
const y = {}

console.log(x);
console.log(y);

console.log(x.__proto__ === y.__proto___); // true


const array = [];
console.log(array); // __proto__ : Array(0) 오브젝트

```
- js의 모든 오브젝트는 `__proto__` 오브젝트를 상속한다
	- 콘솔에서 살펴보면 상속되는 메서드는 모두 사용 가능
- Array 프로토 안에는 Object 프토로를 가지고 있다
	- array ➡️ Array ➡️ Object


```javascript

function CoffeeMachine(beans) {
	this.beans = beans;

	// Instance member level
	/*
	this.makeCoffee = shots => {
		console.log('making...');
	};
	*/
}

// Prototype member level
CoffeeMachine.prototype.makeCoffee = () => {
	console.log('making...');
}

const machine1 = new CoffeeMachine(10);
const machine2 = new CoffeeMachine(20);

console.log(machine1);
console.log(machine2);



function LatteMachine(milk) {
	this.milk = milk;
}

// 👀 라떼머신 ➡️ 커피머신 ➡️ 오브젝트
LatteMachine.prototype = Object.create(CoffeeMachine.prototype);

const latteMachine = new LatteMachine(123);
console.log(latteMachine);
latteMachine.makeCoffee(); // 커피머신 상속했으므로 
```
- 콘소 출력을 보면서 proto 타입이 어디 있는지 확인
- 공통적으로 Object 프로토 상속
- machine ➡️ CoffeeMachine ➡️ Object
### This
- 생성된 객체 그 자신을 뜻함
- 하지만 JS의 경우 호출한 문맥에 따라 동적으로 달라짐

10-2-this.js
```javascript
console.log(this); // 브라우저에서 확인 시 Window 객체

function simpleFunc() {
	console.log(this); // Window 객체 출력
}
simpleFunc();


class Counter {
	count = 0;
	increase = function() {
		console.log(this); // Counter
	};
}

const counter = new Counter();
counter.increase();

const caller = counter.increase;
caller(); // undefined, 할당되면서 this 정보 잃어버림


```

개발자 도구에서 선언한 함수는 글로벌 객체 통해 호출 가능 
```text
function hello() { console.log('hello'); }

window.hello();
hello

// 상수/변수는 글로벌(window) 객체에서 접근 못함
const name = 'tester';
name
tester
```
- `var`로 선언하는 변수는 글로벌 객체에서 접근 가능 💩
	- 호이스팅 문제 💩

```javascript
class Bob {}
const bob = new Bob();
bob.run = counter.increase;
bob.run(); // this에 Bob 출력됨


// this 정보를 잃지 않으려면 bind 사용
const counter = new Counter();
counter.increase();
const caller = counter.increase.bind(counter);
caller(); // this에 Counter 출력 

```

> this 정보를 다른 곳에 할당하는 순간 정보를 잃어 버릴 수 있다
> js에서 this는 부르는 문맥에 따라 달라질 수 있다


```javascript
class Counter {
	count = 0;
	increase = function() {
		console.log(this); // Counter
	};
}

// arrow function을 사용하면 bind 호출하지 않고도 문맥 유지
class Counter {
	count = 0;
	increase = () => {
		console.log(this); // Counter
	};
}

```
###  모듈✨
- 한 파일안에 작성되어 있는 코드를 모듈이라 한다
	- 분산되어 작성하는 경우 window/global에 등록된다
	- 이로인해 다른 파일에 add 함수가 있으면 충돌날 수 있다 
- 모듈을 통해 그 파일 내부로 스코프를 한정 가능
	- 기본적으로 다른 파일에 접근 불가하지만 export, import 활용해 협력 가능하다
- 모듈을 사용함으로써
	- 파일 간에 중복 이름으로 인한 충돌 방지
	- 모듈 간에 스코프 제한되어 모듈성과 재사용성 높여준다
		- 글로블 스코프로 등록되지 않음
``

```html
//..
<script src=""/>
<script src=""/>

```

10-3-module1.js
```javascript 
function add(a, b) {
	return a + b;
}
```

10-3-module2.js
```javascript
add(1, 2);
```

> 글로벌 스코프로 등록되어 add() 호출됨



module로 타입을 지정하면 해당 파일내에 스코프를 정의함
- 한 파일 내에 default는 하나만
- import 할때 default는 중괄호 필요없지만 다른거는 중괄호 필요👀

```html
//..
<script type = "module" src=""/>
<script type = "module" src=""/>

```

10-3-module1.js
```javascript 
export default function add(a, b) {
	return a + b;
}

export function print() {
	console.log('print');
}

export const number = 10;
```

10-3-module2.js
```javascript
import add, {print as printMessage} from './10-3-module1.js'
add(1, 2);
printMessage();


```

아래와 같이 전체 별칭을 지정하여 사용 가능
```javascript
import * as calculator from './10-3-module1.js'
console.log(calculator.add(1, 2));
calculator.print();
calculator.number;
```


