
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
