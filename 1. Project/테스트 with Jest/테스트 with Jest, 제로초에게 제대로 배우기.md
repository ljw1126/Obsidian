- `2025-09-08` 구입

## 섹션1. Jest 배워보기

**jest** 
- js + 테스트, 농담이란 뜻
- 페이스북에서 만든 테스트 프레임워크
	- 타입스크립트도 가능
	- Bable / React / Node.js 전부 다 지원
	- Vitest(Vite), Jasmine, Mocha + Sinon + Chai 등의 대체재도 존재

**테스트를 안 하는 이유**
1. 테스트 효과에 대한 불확실성 
2. 귀찮음
3. 스트레스
4. 시간이 오래 걸림

**해야 하는 이유** 
1. 예전에 났던 에러가 또 나는 경우 (`회귀 테스트` 강추✅)
2. 코드가 복잡한데 많이 바꿔야하는 경우
	1. 하나의 코드를 수정했더니 import 한 다른 곳에서 에러가 나는 경우
		1. 이때 변경에 대한 불안감을 해결하는게 `테스트`다
		2. 테스트를 통해 안심하고 수정 가능
		3. <font color="#c00000">테스트가 없으면 런타임에 확인하거나 배포 후 터지고 확인하게 되는데 최악</font>☠️
3. 장기간에 걸쳐 내가 유지보수를 해야 하는 경우 

> 테스트 커버리지보다 쓸모있는 테스트를 작성해야 한다.

**원칙**
- ✅ 테스트는 틈틈이 추가하자 
- ✅ 진짜 도움이 되는 테스트를 작성하자

---

### Jest 설치 및 ESM 세팅

```shell
$ mkdir zerocho-jest

$ npm init -y

$ npm i jest -D
```

window에서 jest 실행시 환경 변수를 추가해서 실행을 해줘야 한다
```shell
$ npm i -D cross-env

$ npx cross-env NODE_OPTIONS="$NODE_OPTIONS --experimental-vm-modules" jest
```
- `package.json`
	- `"type" : "module"` 설정

### Typescript 셋팅

```shell
$ npm i -D ts-jest @types/jest

$ npx ts-jest config:init

```
- `jest.config.js` 생성됨
	- 테스트 파일 확장자는 `*.spec.ts` , `*.test.ts` 둘 중 하나 선택해 사용
	- 옵션 중에 `testRegex` 설정 있는데 찾아보기 (`강의 참고`)


VSCode에서 Jest 익스텐션이랑 같이 사용하기 위해 작성
`settings.json`
```json
{
	"jest.pathToJest": "**/node_modules/.bin/jest",
	"jest.pathToConfig": "**/jest.config.js",
	"jest, showCoverageOnLoad": true
}
```


`not`
```typescript 
import { sum } from './toBe';

test('두 숫자를 합산한다', () => {
	expect(sum(1, 2)).toBe(3);
	expect(sum(1, 2)).not.toBe(2);
});

```

`toStrictEqual`
```typescript 
export function obj() {
	return { a : 'hello' };
}
```


```typescript 
import { obj } from './toStrictEqual';

test('객체는 toStrictEqual로 비교한다', () => {
	expect(obj()).toStrictEqual({ a : 'hello' });
	expect(obj()).not.toBe({ a : 'hello' });
})

test('배열끼리도 toStrictEqual을 써야 한다', () => {
	expect([1, 2, 3]).toStrictEqual([1, 2, 3]);
	expect([1, 2, 3]).not.toBe([1, 2, 3]);
});

```
- 객체 끼리 비교할 때 주로 사용

`toMatchObject`
```typescript 
class TestObj {
	a: string;
	constructor(str) {
		this.a = str;
	}
}

export function obj(str: string) {
	return new TestObj(str);
}
```

```typescript 
import { obj } from './toStrictEqual';

test('객체 생성자까지 비교한다, 클래스 비교는 toMatchObject로 해야 한다', () => {
	expect(obj('hello')).not.toStrictEqual({ a: 'hello' });
	expect(obj('hello')).toMatchObject(new TestObj('hello'));
})

```
- 클래스까지 다르면 사용

---

### toHaveBeenCalled() 시리즈와 jest.fn(), jest.spyOn()

jest.fn()을 스파이 함수라고 부르기도 함
```typescript 

test('sum 함수가 호출되었다', () => {
	const sumSpy = jest.fn(sum);
	sumSpy(1, 2);
	expect(sumSpy).toHaveBeenCalled();
})

test('sum 함수가 1번 호출되었다', () => {
	const sumSpy = jest.fn(sum);
	sumSpy(1, 2);
	expect(sumSpy).toHaveBeenCalledTimes(1);
})

test('sum 함수가 1,2와 함께 호출되었다', () => {
	const sumSpy = jest.fn(sum);
	sumSpy(1, 2);
	expect(sumSpy).toHaveBeenCalledWith(1, 2);
})

```

jest.spyOn()의 경우 
- 객체의 메서드 일부를 스파이할 수 있다. 
- 나머지는 실제 동작하게 하면서

```typescript 
import {obj} from "...";

test('obj.minus 함수가 1번 호출되었다(spy 함수 생성)', () => {
	const minusSpy = jest.fn(obj.minus);
	minusSpy(1, 2);
	expect(minusSpy).toHaveBeenCalledTimes(1);
})

test('obj.minus 함수가 1번 호출되었다(spy 삽입)', () => {
	jest.spyOn(obj, 'minus');
	obj.minus(1, 2);
	
	console.log(obj.minus); // 확인해보기 ✅
	expect(obj.minus).toHaveBeenCalledTimes(1);
})
```

---

### mockImplementation, mockReturnValue
- 원해 함수가 실행되지 않고, stubbing 함
- 예제. `mockFuntion.ts`, `mockFunction.test.ts`

유닛테스트에서 함수를 테스트하는데, 제어하기 힘든 외부 의존성을 제어할 수 있다.
```typescript 

test('obj.minus에 spy를 심고 리턴값을 바꾼다', () => {
	jest.spyOn(obj, 'minus').mockImplementation((a, b) => a + b);
	const result = obj.minus(1, 2);
	
	expect(obj.minus).toHaveBeenCalledTimes(1);
	expect(result).toBe(5);
})
```

`mockImplementationOnce`, `mockReturnValue`
```typescript

test('obj.minus에 spy를 심고 mockImplementationOnce로 구현을 한번 바꾼다', () => {
	jest.spyOn(obj, 'minus')
		.mockImplementationOnce((a, b) => a + b)
		.mockImplementation(() => 5);
	
	const result1 = obj.minus(1, 2);
	const result2 = obj.minus(1, 2);
	const result3 = obj.minus(1, 2);
	
	expect(obj.minus).toHaveBeenCalledTimes(3);
	expect(result1).toBe(3);
	expect(result2).toBe(5);
	expect(result3).toBe(-1);
})

test('obj.minus에 spy를 심고 mockReturnValue로 반환 값을 고정한다', () => {
	jest.spyOn(obj, 'minus')
		.mockReturnValue(5);
	
	const result = obj.minus(1, 2);

	expect(obj.minus).toHaveBeenCalledTimes(1);
	expect(result1).toBe(5);
})

```

---

### 비동기  함수 테스트 

`asyncFunction.ts`
```typescript 
export function okPromise() {
	return Promise.resolve('ok');
}

export function noPromise() {
	return Promise.reject('no');
}

// 둘다 자동으로 Promise 감싸준다
export async fnction okAsync() {
	return 'ok';
}

export async function noAsync() {
	throw 'no';
}
```

`asyncFunction.test.ts`
```typescript 
import { ... }

test('okPromise 테스트 💩', () => {
	const okSpy = jest.fn(okPromise);
	expect(okSpy()).resolves.toBe('no');
})

test('okPromise 테스트1', () => {
	const okSpy = jest.fn(okPromise);
	return expect(okSpy()).resolves.toBe('no');
})

test('okPromise 테스트2', () => {
	const okSpy = jest.fn(okPromise);
	return okSpy().then((result) => {
		expect(result).toBe('ok');
	})
})

test('okPromise 테스트3', async () => {
	const okSpy = jest.fn(okPromise);
	const result = await okSpy();
	expect(okSpy()).toBe('ok');
})

test('noPromise 테스트', () => {
	const noSpy = jest.fn(noPromise);
	return noSpy().catch((result) => {
		expect(result).toBe('no');
	})
})

test('noPromise 테스트2', () => {
	const noSpy = jest.fn(noPromise);
	return noSpy().rejects.toBe('no');
})

test('noPromise 테스트3', () => {
	const noSpy = jest.fn(noPromise);
	try {
		const result = await noSpy();
	} catch(err) {
		expect(err).toBe('no');
	}
})

// okPromise랑 동일함
test('okAsync 테스트', () => {
	const okSpy = jest.fn(okAsync);
	expect(okSpy()).resolves.toBe('no');
})

test('okAsync 테스트1', () => {
	const okSpy = jest.fn(okAsync);
	return expect(okSpy()).resolves.toBe('no');
})

test('okAsync 테스트2', () => {
	const okSpy = jest.fn(okAsync);
	return okSpy().then((result) => {
		expect(result).toBe('ok');
	})
})
```
- 테스트 성공으로 표기되고 나중에 실패 로그가 뜬다. 💩
	- ✅ `return`을 꼭 붙여줘서 해결한다
	- 또는 method chaining을 해서 해결
	- 또는 await 키워드 사용하는 방법

```typescript 
import * as fns from './asyncFunction';

test('okPromise 테스트1', () => {
	jest.spyOn(fns, 'okPromise').mockResolveValue('ok');
	return expect(fns.okPromise()).resolves.toBe('no');
})

test('noPromise 테스트2', () => {
	//jest.spyOn(fns, 'noPromise').mockReturnValue(Promise.reject('no'));
	jest.spyOn(fns, 'noPromise').mockRejectedValue('no');
	return expect(fns.noPromise()).rejects.toBe('no');
})
```

> 강의 스타일인지 싶지만, 명시적으로 안하고 이러하다라고 있는 예시에 수정을 하다보니 헷갈리네

---
### 콜백함수 테스트 

`callback.ts`
```typescript
export function timer(callback) {
	setTimeout(() => {
		callback('success');
	}, 3000);
}
```

```typescript
import { timer } from './callback';

test('타이머 잘 실행되나?', (done) => {
	timer((message:string) => {
		expect(message).toBe('success');
		done();
	})
})
```
- setTimeout과 같은 콜백함수 테스트시에는 done 매개변수를 사용해서 종료해줘야 한다

> ✅ 콜백 함수는 테스트 안하는게 낫다 (실행시간 길어지니), 왠만해서 promise 처리 권장📌