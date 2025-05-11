
**📚 TS 예시 포함된 개발 도서**
- 단위 테스트의 기술
- 멀티패러다임 프로그래밍

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