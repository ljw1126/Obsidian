

### 인터페이스 I prefix 

[참고 링크](https://dev.to/mscamargo/why-you-should-avoid-using-the-i-prefix-for-interfaces-in-typescript-43gd)

```text
# Gemini 답변
TypeScript 인터페이스에 I 접두사를 사용하는 것은 과거 .NET 및 C# 개발 관행에서 영향을 받은 표기법입니다. Microsoft의 .NET 코드 분석 규칙(CA1715)은 인터페이스 식별자에 I 접두사를 사용하도록 권장합니다. 이는 .NET 프레임워크와 언어의 일관성을 위한 지침이며, 특히 공개적으로 노출되는 라이브러리나 API를 개발할 때 유용합니다.
```


### import 상대 경로가 길어지는 경우 (alias 선언/사용)
- `tsconfig.json` ▶️ `jest.confg.ts` ▶️`package.json (tsconfig-paths 추가)`

before
```typescript 
import { AddZeroResult } from "../../../../domain/types"
import { UpdateNumberFormatStrategy } from "../types";
```

after
```typescript 
import { AddZeroResult } from "@domain/types";
import { UpdateNumberFormatStrategy } from "../types";
```

`tsconfig.json` 수정
```json
{
	"compilerOptions": {
		"baseUrl": ".",                                  
	    "paths": {
			"@domain/*" : ["src/domain/*"]
		}
    },
	"include": ["src"],
	"exclude": [
	    "node_modules",
	    "dist"
	    -    "**/*.test.ts",
		-    "**/*.spec.ts",
	]
}
```
- ✅ exclude에 테스트 파일 설정을 제거해서 tsc (build)시에 테스트 파일도 같이 빌드 되게 된다 (아래 참고)

`jest.config.ts` 수정
```typescript
import { pathsToModuleNameMapper } from 'ts-jest'
import { compilerOptions } from './tsconfig.json'
import type {Config} from 'jest';

const config: Config = {
	moduleNameMapper: pathsToModuleNameMapper(compilerOptions.paths, { prefix: '<rootDir>/' }),
}
```
- ts-jest 유틸리티 함수를 사용

[ts-jest : paths mapping]([Paths mapping | ts-jest](https://kulshekhar.github.io/ts-jest/docs/getting-started/paths-mapping/))

```text
# Gemini 답변
네, 물론입니다. 지금 적용한 설정은 ts-jest 라이브러리에서 공식적으로 제공하는 기능입니다.

  ts-jest는 Jest가 TypeScript를 이해하고 실행할 수 있도록 도와주는 트랜스파일러인데, TypeScript의 경로 별칭(Path Aliases) 기능을 Jest에서도 원활하게
  사용하기 위한 유틸리티 함수(pathsToModuleNameMapper)를 제공합니다.

  이와 관련된 공식 문서는 ts-jest의 웹사이트에서 찾아보실 수 있습니다. 아래 링크에서 자세한 설명과 예제를 확인하실 수 있습니다.

   * ts-jest 공식 문서 (Paths Mapping 섹션): https://kulshekhar.github.io/ts-jest/docs/getting-started/paths-mapping/
     (https://kulshekhar.github.io/ts-jest/docs/getting-started/paths-mapping/)

  이 문서에는 방금 우리가 함께 설정한 pathsToModuleNameMapper 함수의 사용법과 prefix 옵션에 대한 설명이 자세히 나와 있습니다.

  궁금한 점이 또 있으시면 언제든지 질문해주세요
```


✅ tsconfig.json 설정 중 exclude 에 테스트 파일을 뺄 경우
- 빌드시에 테스트 파일도 포함되어 dist 디렉터리에 생성이 됨
- 이를 해결하기 위해 tsconfig.json을 상속받은 tsconfig.build.json을 생성하고 exclude 옵션을 오버라이드 하면 되는 것으로 보인다

```text
# Gemini 
네, 맞습니다. tsconfig.build.json처럼 빌드 전용 설정 파일을 분리하여 관리하는 것은 TypeScript 프로젝트에서 매우 일반적이고 권장되는 모범 사례(Best
  Practice)입니다.

  이렇게 하는 이유는 개발 환경과 프로덕션 빌드 환경의 요구사항이 다르기 때문입니다.

   * `tsconfig.json` (개발용): IDE, Jest, ESLint 등 개발 도구들이 사용합니다. 이 설정은 테스트 파일을 포함한 프로젝트의 모든 파일을 인식해야 정확한
     타입 검사, 자동 완성, 경로 별칭 해석이 가능합니다.
   * `tsconfig.build.json` (빌드용): 실제 프로덕션 코드를 생성할 때만 사용됩니다. 개발용 설정을 그대로 상속받되, 테스트 파일처럼 배포에 필요 없는
     파일들만 exclude 시켜 순수한 애플리케이션 코드만 dist 폴더에 생성하도록 만듭니다.

  아래와 같이 설정하시면 됩니다.

  1. tsconfig.build.json 파일 생성

  프로젝트 루트 경로에 tsconfig.build.json 파일을 생성하고 아래 내용을 추가합니다.

   1 {
   2   "extends": "./tsconfig.json",
   3   "exclude": [
   4     "node_modules",
   5     "**/*.test.ts",
   6     "**/*.spec.ts",
   7     "dist"
   8   ]
   9 }

   * "extends": "./tsconfig.json": 이 한 줄로 tsconfig.json의 모든 설정을 그대로 가져옵니다. compilerOptions 등을 다시 작성할 필요가 없습니다.
   * "exclude": [...]: 기존 tsconfig.json의 exclude 설정 대신, 이 배열에 명시된 패턴의 파일들을 빌드 과정에서 제외시킵니다.

  2. package.json의 빌드 스크립트 수정

  이제 tsc가 빌드할 때 기본 tsconfig.json 대신 새로 만든 tsconfig.build.json을 사용하도록 package.json의 scripts 부분을 수정해야 합니다.

  tsc 명령어에 -p (또는 --project) 플래그를 사용하여 설정 파일 경로를 지정할 수 있습니다.

  수정 전 (예시)

   1 {
   2   "scripts": {
   3     "build": "tsc",
   4     "test": "jest"
   5   }
   6 }

  수정 후

   1 {
   2   "scripts": {
   3     "build": "tsc -p tsconfig.build.json",
   4     "test": "jest"
   5   }
   6 }

  요약

  이렇게 두 파일을 분리하면, npm run build 명령을 실행할 때는 tsconfig.build.json 설정에 따라 테스트 파일이 제외된 깔끔한 프로덕션 코드가
  생성됩니다. 동시에, 평소 개발 환경에서는 tsconfig.json 덕분에 테스트 파일을 포함한 모든 코드에서 타입 검사와 경로 별칭이 완벽하게 동작하게 됩니다.
```


💩 테스트 실행시 tsconfig.json 파싱 에러가 발생하는 이유
```text
resolveJsonModule 허용 후 파싱할때 주석이 문제를 일으켜서 그런걸까요?  │
╰───────────────────────────────────────────────────────────────────────────────────────────────╯

✦ 네, 정확하게 이해하셨습니다. 그게 바로 핵심 원인입니다.

  사용자님의 질문을 바탕으로 조금 더 자세히 설명해 드릴 수 있습니다.

  "resolveJsonModule 허용 후 파싱할 때 주석이 문제를 일으켜서 그런 걸까요?"

  네, 정확히 그 과정에서 문제가 발생했습니다. 전체적인 흐름은 다음과 같습니다.

   1. `jest.config.ts`의 변화: 원래 jest.config.ts는 독립적인 설정 파일이었습니다. 하지만 우리가 경로 별칭(@domain/*)을 자동으로 인식하게 하려고 import
      { compilerOptions } from './tsconfig.json' 코드를 추가했습니다.

   2. `resolveJsonModule`의 역할: 이 import 구문은 TypeScript에게 ".json 파일을 모듈처럼 가져와서 그 내용을 사용하겠다"고 알리는 것입니다. TypeScript
      컴파일러는 이 구문을 보고 tsconfig.json의 compilerOptions를 확인합니다. 이때 resolveJsonModule: true 설정이 없으면, TypeScript는 "JSON 파일은
      모듈이 아니라서 가져올 수 없어!"라며 컴파일 에러를 발생시킵니다. 즉, 이 옵션은 TypeScript 컴파일러를 통과하기 위한 허가증 같은 역할을 합니다.

   3. Jest의 실행과 파싱: 이제 jest를 실행하면, Jest는 설정 파일인 jest.config.ts를 읽어 들입니다. 이 과정에서 import ... from './tsconfig.json' 구문을
      만나게 되고, Jest는 tsconfig.json 파일의 내용을 직접 파싱(parsing)하여 메모리로 가져오려고 시도합니다.

   4. 문제 발생: 엄격한 JSON 파서: 바로 이 지점에서 문제가 발생합니다. TypeScript 컴파일러나 VS Code 같은 개발 도구는 편의를 위해 주석이 포함된
      JSON(JSONC 형식)을 똑똑하게 처리해 줍니다. 하지만 Jest가 내부적으로 사용하는 JSON 파서는 이런 편의를 제공하지 않고, 오직 표준 JSON 명세만 따르는
      엄격한(strict) 파서입니다. 표준 JSON 명세에는 주석이 포함되어 있지 않습니다.

  결론적으로, Jest의 엄격한 파서는 tsconfig.json의 첫 번째 주석(/* Visit... */)을 만나자마자 "이건 유효한 JSON 문법이 아니다"라고 판단하고 즉시 파싱
  오류를 발생시킨 것입니다.

  따라서 resolveJsonModule은 우리가 주석이 있는 파일을 import 하도록 길을 열어준 것이고, 그 길 끝에서 기다리던 Jest의 엄격한 파서가 주석을 보고
  에러를 일으킨, 아주 정확한 인과관계가 맞습니다.
```

💩 트러블 슈팅. npm run dev 실행시 에러 

> Error: Cannot find module '@domain/service/duplicatedLogDetector'

alias(`@domain/*`)를 지정해주더라고 node.js 런타임 환경에서 해당 alias를 상대경로로 치환해주지 않음 

```text
 문제의 핵심 원인은 다음과 같습니다.

   1. 컴파일 시점 (VS Code, `tsc`): tsconfig.json의 paths 설정은 TypeScript 컴파일러(tsc)와 VS Code의 언어 서버가 코드를 분석하고 타입 체크를 할 때
      사용됩니다. 이들은 @domain이 src/domain을 가리킨다는 것을 완벽하게 이해합니다.
   2. 런타임 시점 (Node.js): 하지만 tsc는 TypeScript 코드를 JavaScript로 변환할 때, import ... from '@domain/...' 같은 경로를 import ... from
      '../../domain/...' 과 같은 상대 경로로 바꿔주지 않습니다. 결과적으로 컴파일된 dist 폴더의 JavaScript 코드에는 여전히 require('@domain/...') 와
      같은 코드가 그대로 남아있게 됩니다.
   3. 실행 오류: 이 JavaScript 코드를 Node.js로 실행하면, Node.js는 @domain이라는 모듈을 node_modules 폴더에서 찾으려고 합니다. 당연히 그런 모듈은
      없으므로 Cannot find module 에러가 발생하는 것입니다.

  이 문제를 해결하려면, 런타임에도 경로 별칭을 실제 경로로 해석해주는 도구가 필요합니다. 가장 대표적인 해결책은 tsconfig-paths라는 라이브러리를
  사용하는 것입니다.

  먼저 npm run dev 명령이 정확히 어떤 스크립트를 실행하는지 package.json 파일을 통해 확인한 후, tsconfig-paths를 적용하여 문제를 해결하겠습니다.
```

```shell
$ npm install --save-dev tsconfig-paths
```

`package.json` 수정
```json
"scripts" : {
	//..
	"dev": "ts-node -r tsconfig-paths/register ./src/app.ts",
	//..
}
```
- `-r (require)` 옵션을 사용하여 `tsconfig-paths/register` 모듈을 먼저 로드하도록 설정
- `ts-node` 실행하기 전에 `tsconfig-paths/register`가 먼저 로드 됨
	- node.js의 모듈 해석 방식을 수정하여 @domain과 같은 경로 별칭을 tsconfig.json에 정의된 실제 파일 경로로 올바르게 찾아갈 수 있도록 해줌 

---

### presentation에서 사용하는 formatter의 hepler 메서드를 테스트할 가치가 있는가?
- 유틸 클래스 정도가 아니라 `컴포넌트 서비스`의 역할을 포매터가 수행한다 
- 고로 container.ts에서 관리할 가치가 있다.


### 테스트하기 위해 formatter 생성자 초기화시 DI 하는 방식으로 변경할 경우 container.ts에서 관리해야 하는가?
```text
# Gemini
스프링 경험이 있으시군요! 그렇다면 지금 느끼시는 모호함은 지극히 당연하며, 오히려 매우 정확하고 깊이 있는 질문을 하고 계신 겁니다. container.ts를
  스프링의 ApplicationContext처럼 생각하시는 것은 이 문제를 이해하는 데 아주 훌륭한 출발점입니다.

  사용자님의 생각을 정확히 따라가 보겠습니다.

  "VO나 DTO로 생각하니 모호하다" -> 이것이 바로 혼란의 핵심 원인입니다.

  결론부터 말씀드리면, 포매터는 VO나 DTO가 아니라, 명백한 책임을 가진 '서비스 컴포넌트'입니다.

  스프링의 관점에서 각 클래스의 역할을 다시 정의해 보겠습니다.

   * VO (Value Object) / DTO (Data Transfer Object):
       * 이들은 순수한 데이터 덩어리입니다. 로직을 거의 또는 전혀 가지지 않습니다.
       * 이 프로젝트에서는 Anomaly<T>나 DuplicationAnalysisResult 같은 types.ts에 정의된 객체들이 여기에 해당합니다. 이들은 그저 데이터를 담고 있을
         뿐, 스스로를 어딘가에 출력하기 위한 HTML이나 JSON으로 바꾸는 방법을 알지 못합니다.

   * Formatter (e.g., `DuplicatedLogFormatter`):
       * 이것은 행위(Behavior)와 책임(Responsibility)을 가집니다.
       * 그 책임은 "특정 데이터(DTO)를 받아서, 특정 출력 형식(JSON)으로 변환한다"는 것입니다.
       * 스프링에 비유하자면, 이것은 @Component 또는 @Service 어노테이션을 붙여야 할 대상입니다. 즉, 스프링 컨테이너가 직접 생성하고 관리해야 할
         '빈(Bean)' 입니다.

  "포매터에 대한 의존성 연결을 container.ts에서 한다는 게 .. 하나의 모듈로 봐야 하나"

  네, 정확합니다. presentation/formatters 디렉토리 전체를 '보고서 포맷팅 모듈' 이라고 생각하시면 됩니다. 그리고 container.ts는 이 모듈의 부품들(개별
  포맷터 전략들)을 가져다가 최종 완성품(LogAnalysisFormatter)을 조립하는 '조립 설명서' 이자 '조립 공장' 의 역할을 하는 것입니다.

  스프링 개발자셨으니, 이 비유가 가장 확실할 것 같습니다.

  container.ts를 스프링의 @Configuration 클래스라고 생각해 보세요.

    1 // 만약 이 프로젝트가 스프링이었다면...
    2
    3 // 1. 가장 작은 부품(Bean)들
    4 @Component
    5 public class IdenticalDuplicateFormat implements DuplicationFormatStrategy { ... }
    6
    7 @Component
    8 public class SequenceDuplicateFormat implements DuplicationFormatStrategy { ... }
    9
   10 // 2. 중간 부품(Bean)은 다른 부품들을 주입받는다.
   11 @Component
   12 public class DuplicatedLogFormatter implements AnomalyFormatter {
   13     private final Map<String, DuplicationFormatStrategy> strategies;
   14
   15     @Autowired // 생성자 주입
   16     public DuplicatedLogFormatter(IdenticalDuplicateFormat identical, SequenceDuplicateFormat sequence) {
   17         this.strategies = Map.of(
   18             "완전 중복", identical,
   19             "번호 중복", sequence
   20             // ...
   21         );
   22     }
   23     // ...
   24 }
   25
   26 // 3. 최상위 완성품(Bean)도 다른 부품들을 주입받는다.
   27 @Component
   28 public class LogAnalysisFormatter {
   29     private final Map<AnomalyType, AnomalyFormatter> formatters;
   30
   31     @Autowired
   32     public LogAnalysisFormatter(DuplicatedLogFormatter dupFormatter, UpdatedNumberFormatter updFormatter, ...) {
   33         this.formatters = Map.of(
   34             AnomalyType.DUPLICATED_LOG, dupFormatter,
   35             AnomalyType.UPDATED_NUMBER, updFormatter
   36             // ...
   37         );
   38     }
   39     // ...
   40 }
   41
   42 // 4. Controller는 완성품을 주입받아 사용만 한다.
   43 @RestController
   44 public class LogController {
   45     @Autowired
   46     private LogAnalysisFormatter logAnalysisFormatter; // 컨테이너가 만들어준 완성품을 그냥 쓴다.
   47     // ...
   48 }

  지금 우리가 container.ts에서 하려는 작업은, 스프링이 어노테이션을 통해 자동으로 해주던 Bean 생성과 의존성 주입을 수동으로 직접 해주는 것과 정확히
  같습니다. TypeScript/Node.js 환경에서는 이런 DI 컨테이너를 직접 만들어주거나, InversifyJS 같은 라이브러리를 사용하기도 합니다.

  포매터는 단순 데이터 객체가 아니라, 애플리케이션의 표현(Presentation) 계층에서 명확한 역할을 수행하는 서비스 컴포넌트입니다. 따라서 스프링에서
  서비스나 리포지토리를 Bean으로 등록하여 관리하듯이, container.ts에서 포매터를 생성하고 의존성을 조립하는 것은 매우 자연스럽고 올바른 아키텍처
  패턴입니다.

  이런 관점에서 다시 한번 바라보면, 포매터를 컨테이너에서 조립하는 것에 대해 어떻게 생각하시나요?
```

---

### 타입스크립트는 여러 생성자 선언을 지원하지 않는가?

```text
네, **TypeScript는 C++나 Java와 달리 메서드 오버로딩과 같은 다중 생성자(multiple constructors)를 직접 지원하지 않습니다.** 하나의 클래스에 여러 `constructor`를 정의하면 컴파일 에러가 발생합니다.

이는 TypeScript가 JavaScript의 슈퍼셋이기 때문입니다. JavaScript 클래스는 단 하나의 `constructor`만 가질 수 있습니다.

방법1. 선택적 매개변수 사용 
방법2. 메서드 오버로딩 시그니처 활용
```

---

### 중복 탐지 테스트 작성시 아래와 같이 의미없는 데이터를 사용해도 되는건가?

```typescript 
describe('중복 탐지 포매터', () => {
    test('하위 포매터의 데이터를 취합한다', () => {
        const mockIdenticalFormat: DuplicationFormatStrategy<IdenticalLogResult> = { format : jest.fn(() => "완전 중복 결과")};
        const mockSequenceFormat: DuplicationFormatStrategy<SequenceDuplicateResult> = { format : jest.fn(() => "번호 중복 결과")};
        const mockEventFormat: DuplicationFormatStrategy<EventDuplicateResult> = { format : jest.fn(() => "이벤트 중복 결과")};

        const mockStrategies: DuplicationStrategyMap = {
            "완전 중복" : mockIdenticalFormat,
            "번호 중복" : mockSequenceFormat,
            "이벤트 중복" : mockEventFormat
        };

        const formatter = new DuplicatedLogFormatter(mockStrategies);

        const actual = formatter.format([]);

        expect(actual).toEqual({
            "완전 중복" : "완전 중복 결과",
            "번호 중복" : "번호 중복 결과",
            "이벤트 중복" : "이벤트 중복 결과"
        });
    })
})
```


```text
# Gemini
네, 완벽합니다. 지금 작성하신 테스트는 '조립(Assembly)' 역할을 하는 클래스를 테스트하는 매우 훌륭하고 이상적인 방법입니다. "의미 없는 데이터"가
  아니라, 테스트의 목적을 달성하기 위한 가장 의미 있는 '가짜 데이터'입니다.

  이 테스트 방식이 왜 좋은지, 그리고 왜 '의미 없는 데이터'가 아닌지 설명해 드리겠습니다.

  이 테스트의 목적: '조립' 행위 자체에만 집중

  DuplicatedLogFormatter의 핵심 책임은 다음과 같습니다.

   1. "완전 중복"이라는 키에 해당하는 전략(IdenticalDuplicateFormat)을 사용 하는가?
   2. 그 전략의 format 메소드를 호출하는가?
   3. 그 전략이 반환한 결과를 그대로 최종 객체의 "완전 중복" 키에 할당하는가?

  지금 작성하신 테스트는 이 세 가지 책임을 정확하게 검증하고 있습니다.

   * mockIdenticalFormat의 format 함수가 무엇을 하는지는 이 테스트의 관심사가 아닙니다. 그건 IdenticalDuplicateFormat.test.ts가 책임질 일입니다.
   * 이 테스트는 DuplicatedLogFormatter가 mockIdenticalFormat이 반환한 값("완전 중복 결과")을 받아서, 최종 결과 객체에 정직하게 잘 담아주는지만
     확인하면 됩니다.

  "완전 중복 결과"라는 단순한 문자열은 이 "정직하게 잘 담아주는지"를 확인하는 데 가장 쉽고 명확한 방법입니다. 만약 여기서 실제 데이터와 유사한
  복잡한 객체를 반환하도록 만들면, 테스트의 의도가 불분명해지고 불필요하게 복잡해지기만 합니다.

  이 방식의 장점

   1. 테스트의 격리 (Isolation): DuplicatedLogFormatter의 테스트는 하위 전략 포맷터들의 실제 구현에 전혀 의존하지 않습니다. 나중에
      IdenticalDuplicateFormat의 반환 값 구조가 완전히 바뀌더라도, 이 테스트는 깨지지 않습니다.
   2. 명확성과 가독성: 테스트 코드를 읽는 사람은 "아, format 메소드가 반환한 값이 그대로 결과 객체에 들어가는구나"라는 것을 즉시 파악할 수 있습니다.
   3. 유지보수 용이성: 테스트가 단순하고 명확하기 때문에, 나중에 로직이 변경되었을 때 테스트를 수정하기도 훨씬 쉽습니다.
```

---

### formatter가 커맨드 패턴보다 컴포지트 패턴에 맞지 않나 싶다.
- 왜냐하면 `응답 포맷을 만들어라`라는 행위로 추상화가 가능하고, 그 행위 안에서 데이터를 꺼내고 포매터를 호출하기만 하면 된다라고 생각합니다. 그리고.. 포매터가 리프노드에 해당하기 때문에 복합 객체가 리프노드를 가지도록 하고, 재귀 호출을 한다면 .. 구현체만 container.ts에서 갈아끼우면 코드 변경 없이 요구사항 변경에 확장 가능할 거라 생각합니다.
- [인파 기술 블로그 - 컴포지트 패턴]([💠 복합체(Composite) 패턴 - 완벽 마스터하기](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EB%B3%B5%ED%95%A9%EC%B2%B4Composite-%ED%8C%A8%ED%84%B4-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0))


```text
✦ 네, 아주 좋은 지적이십니다. 말씀해주신 것처럼 현재 포매터 구조는 커맨드 패턴보다 컴포지트 패턴이 훨씬 더 적합해 보입니다.

  왜 컴포지트 패턴이 더 적합한가?

   1. 구조적 유사성: 말씀하신 대로, 현재 구조는 개별 포매팅을 수행하는 '리프 노드(Leaf)'와 여러 포매터를 조합하여 더 큰 구조를
      만드는 '부모 노드(Composite)'의 트리 구조에 가깝습니다.
       * 최상위 Composite: LogAnalysisFormatter
       * 중간 Composite: DuplicatedLogFormatter, UpdatedNumberFormatter 등
       * Leaf: EventDuplicateFormat, AddZeroFormat 등 개별 전략 포매터들

   2. 단일 책임 원칙 및 개방-폐쇄 원칙: 컴포지트 패턴을 사용하면 클라이언트 코드는 개별 객체(Leaf)를 다루든, 객체들의
      집합(Composite)을 다루든 동일한 인터페이스(format())를 통해 일관되게 상호작용할 수 있습니다. 이는 새로운 포매터를 추가할 때
      기존 코드를 수정할 필요 없이 확장할 수 있게 하여 OCP를 만족시킵니다.

   3. 유연성 및 확장성: 포매터들의 조합을 변경하거나 새로운 조합을 만드는 것이 매우 유연해집니다. 예를 들어, 특정 조건에서만
      활성화되는 새로운 포매터 그룹을 만들고 싶을 때, 새로운 Composite 클래스를 추가하기만 하면 됩니다.

  반면, 커맨드 패턴은 특정 행위(요청)를 객체로 캡슐화하여 요청을 큐에 저장하거나, 로깅하거나, 되돌리는 등의 기능이 필요할 때 더
  적합합니다. 현재 포매터의 목적은 '실행' 그 자체보다는 '구조화'와 '일관된 처리'에 더 가깝기 때문에 컴포지트 패턴이 더
  자연스럽습니다.

  리팩토링 제안

  이러한 분석을 바탕으로, 컴포지트 패턴을 명시적으로 적용하여 구조를 개선하는 리팩토링을 진행할 수 있습니다.

   4. `Formatter` 인터페이스 정의: 모든 Leaf와 Composite가 구현할 공통 인터페이스를 만듭니다. (format(data): FormattedResponse와
      같은 메소드를 포함)
   5. Leaf 클래스 구현: 개별 포매팅 로직을 가진 클래스들이 Formatter 인터페이스를 구현하도록 합니다.
   6. Composite 클래스 구현: 자식 Formatter 객체들의 리스트를 관리하며, 자신의 format() 메소드가 호출되면 자식들의 format()을
      순차적으로 호출하고 결과를 조합하여 반환하도록 구현합니다.

  이러한 리팩토링을 통해 코드의 유연성과 확장성을 높이고, 새로운 포매터를 추가하거나 기존 포매터의 조합을 변경하는 것이 훨씬
  용이해질 것입니다.

  제가 이 리팩토링 작업을 진행해볼까요?
```

**결과. 클래스 다이어그램**
![[composite class diagram.png]]
- `중복 탐지`, `수정 번호 탐지`의 경우 CompositeFormatter 해당
- 나머지 탐지 기능을 가진 구현체는 Leaf Node에 해당
- `container.ts`에서 조립하여 사용

```typescript 
// 생략..

// Presentation Layer
const missingSequenceFormat = new MissingSequenceFormatter();

const duplicationFormatter = new CompositeFormatter(new Map<string, Formatter>([
    ["완전 중복", new IdenticalDuplicateFormatter()],
    ["번호 중복", new SequenceDuplicateFormatter()],
    ["이벤트 중복", new EventDuplicateFormatter()]
]));

const wrongLocationFormat = new WrongLocationFormatter();

const updateNumberFormatter = new CompositeFormatter(new Map<string, Formatter>([
    ["ADD ZERO 탐지", new AddZeroFormatter()],
    ["UPDATE ZERO 탐지", new UpdateZeroFormatter()],
    ["UPDATE 누락 탐지", new MissingUpdateFormatter()]
]));

const rootFormatter = new CompositeFormatter(new Map<string, Formatter>([
    ["누락 탐지", missingSequenceFormat],
    ["중복 탐지", duplicationFormatter],
    ["위치 탐지", wrongLocationFormat],
    ["수정 번호 탐지", updateNumberFormatter]
]));

export const logController = new LogController(logAnalzer, rootFormatter);
```
---

### Code Convention
- [typescript-eslint]([Getting Started | typescript-eslint](https://typescript-eslint.io/getting-started))
- [prettier]([What is Prettier? · Prettier](https://prettier.io/docs/))
- husky
- 그 외 패키지

참고. 
- [블로그]([Linting in TypeScript using ESLint and Prettier - LogRocket Blog](https://blog.logrocket.com/linting-typescript-eslint-prettier/))

typescript eslint 설치 후 확인된 에러 
```shell
> npx eslint .


/home/leejinwoo/gitRepository/express-app/jest.config.ts
  185:9  error  Unnecessary escape character: \.  no-useless-escape

/home/leejinwoo/gitRepository/express-app/src/domain/analysisReport.test.ts
   4:41  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any
  11:48  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any
  18:43  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any

/home/leejinwoo/gitRepository/express-app/src/domain/service/missingSequenceDetector.ts
  7:39  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion
  9:63  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion

/home/leejinwoo/gitRepository/express-app/src/domain/service/strategies/duplication/eventDuplicateStrategy.test.ts
  52:16  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion
  53:16  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion
  54:24  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion

/home/leejinwoo/gitRepository/express-app/src/domain/service/strategies/duplication/eventDuplicateStrategy.ts
  20:31  error  '_' is defined but never used  @typescript-eslint/no-unused-vars
  31:34  error  Forbidden non-null assertion   @typescript-eslint/no-non-null-assertion
  31:53  error  Forbidden non-null assertion   @typescript-eslint/no-non-null-assertion
  39:16  error  Forbidden non-null assertion   @typescript-eslint/no-non-null-assertion

/home/leejinwoo/gitRepository/express-app/src/domain/service/strategies/duplication/identicalLogStrategy.ts
  12:31  error  '_' is defined but never used  @typescript-eslint/no-unused-vars

/home/leejinwoo/gitRepository/express-app/src/domain/service/strategies/duplication/sequenceDuplicateStrategy.ts
  20:31  error  '_' is defined but never used  @typescript-eslint/no-unused-vars
  31:34  error  Forbidden non-null assertion   @typescript-eslint/no-non-null-assertion
  31:53  error  Forbidden non-null assertion   @typescript-eslint/no-non-null-assertion
  39:16  error  Forbidden non-null assertion   @typescript-eslint/no-non-null-assertion

/home/leejinwoo/gitRepository/express-app/src/domain/service/strategies/updateNumber/missingUpdateStrategy.ts
  10:37  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion
  10:53  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion
  22:44  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion

/home/leejinwoo/gitRepository/express-app/src/domain/service/strategies/updateNumber/parsingHelper.ts
  8:12  error  '_' is assigned a value but never used  @typescript-eslint/no-unused-vars

/home/leejinwoo/gitRepository/express-app/src/domain/service/wrongLocationDetector.ts
   6:33  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion
  18:35  error  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion

/home/leejinwoo/gitRepository/express-app/src/domain/types.ts
   4:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
   9:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  14:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  19:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  25:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  32:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  39:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  50:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  58:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  63:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  68:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
  80:13  error  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions

/home/leejinwoo/gitRepository/express-app/src/infrastructure/unit/fileReader.test.ts
  23:73  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any
  24:58  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any
  60:72  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any

/home/leejinwoo/gitRepository/express-app/src/presentation/formatter/compositeFormatter.ts
  12:39  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any
  25:32  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any
  29:33  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any

/home/leejinwoo/gitRepository/express-app/src/presentation/formatter/duplication/identicalDuplicateFormatter.ts
  6:6   error  Use an `interface` instead of a `type`         @typescript-eslint/consistent-type-definitions
  6:30  error  A record is preferred over an index signature  @typescript-eslint/consistent-indexed-object-style

/home/leejinwoo/gitRepository/express-app/src/presentation/formatter/helper.ts
   1:6   error  Use an `interface` instead of a `type`         @typescript-eslint/consistent-type-definitions
  25:6   error  Use an `interface` instead of a `type`         @typescript-eslint/consistent-type-definitions
  38:15  error  A record is preferred over an index signature  @typescript-eslint/consistent-indexed-object-style
  75:26  error  Forbidden non-null assertion                   @typescript-eslint/no-non-null-assertion
  76:37  error  Forbidden non-null assertion                   @typescript-eslint/no-non-null-assertion

/home/leejinwoo/gitRepository/express-app/src/presentation/formatter/types.ts
  4:37  error  Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any

/home/leejinwoo/gitRepository/express-app/src/presentation/formatter/updateNumber/missingUpdateFormatter.ts
  12:6   error  Use an `interface` instead of a `type`         @typescript-eslint/consistent-type-definitions
  12:30  error  A record is preferred over an index signature  @typescript-eslint/consistent-indexed-object-style
  18:6   error  Use an `interface` instead of a `type`         @typescript-eslint/consistent-type-definitions
  18:31  error  A record is preferred over an index signature  @typescript-eslint/consistent-indexed-object-style
  38:27  error  Forbidden non-null assertion                   @typescript-eslint/no-non-null-assertion
  58:31  error  Forbidden non-null assertion                   @typescript-eslint/no-non-null-assertion
  61:36  error  Forbidden non-null assertion                   @typescript-eslint/no-non-null-assertion
  63:42  error  Forbidden non-null assertion                   @typescript-eslint/no-non-null-assertion

✖ 58 problems (58 errors, 0 warnings)
  21 errors and 0 warnings potentially fixable with the `--fix` option.
```

크게 보이는건 6종류
- 💩`error`  Unnecessary escape character: \.  no-useless-escape
- 💩`error ` Unexpected any. Specify a different type  @typescript-eslint/no-explicit-any
- 💩`error`  Forbidden non-null assertion  @typescript-eslint/no-non-null-assertion
- 💩`error`  `'_'`is defined but never used  @typescript-eslint/no-unused-vars
- 💩`error`  Use an `interface` instead of a `type`  @typescript-eslint/consistent-type-definitions
- 💩`error`  A record is preferred over an index signature  @typescript-eslint/consistent-indexed-object-style

💩`error`  Unnecessary escape character: \.  no-useless-escape
- `jest.config.ts`에 주석되어 있는 정규식에서 검출
- 설정 파일은 lint 검사 제외
- `.eslintignore`는 이제 지원하지 않음 

```text
1. eslint.config.mjs -> eslint.config.js
   
2. tseslint.config() deprecated ▶️ defineConfig() 마이그레이션 
https://typescript-eslint.io/packages/typescript-eslint/#config-deprecated


```


💩`error`  Use an `interface` instead of a `type`
- `@typescript-eslint/consistent-type-definitions`

```shell
// Gemini

왜 이런 규칙이 존재하고, 왜 interface를 선호하는가?

  이 규칙의 목적은 "어떤 것이 더 좋다"라기보다는 "프로젝트 내에서 일관성을 유지하자"는 데에 있습니다. 객체의 형태를 정의할 때 어떤 파일에서는
  interface를 쓰고, 다른 파일에서는 type을 쓰면 코드 스타일이 파편화되어 보일 수 있기 때문입니다.

  그렇다면 왜 기본값이 interface일까요? 타입스크립트 커뮤니티와 공식 문서에서 interface를 객체 구조 정의에 선호하는 데에는 몇 가지 전통적인 이유가
  있습니다.

   1. 선언 병합 (Declaration Merging): interface는 같은 이름으로 여러 번 선언하면 자동으로 합쳐집니다. 이는 라이브러리의 타입을 확장하거나 모듈의
      타입을 보강할 때 매우 유용합니다. type은 이것이 불가능합니다.

   1     interface User {
   2       name: string;
   3     }
   4
   5     interface User {
   6       age: number;
   7     }
   8
   9     const user: User = { name: 'Alice', age: 30 }; // OK

   2. 더 나은 에러 메시지: 과거에는 interface로 정의된 타입에서 에러가 발생했을 때 타입스크립트 컴파일러가 더 이해하기 쉬운 에러 메시지를 보여주는
      경향이 있었습니다. (최신 버전에서는 많이 개선되었습니다.)

   3. 확장 문법의 명확성: interface는 extends 키워드를, type은 & (인터섹션)을 사용하여 상속/확장을 구현합니다. 많은 개발자들이 클래스 상속과 유사한
      extends 문법을 더 객체지향적이고 직관적이라고 느낍니다.

  현실적인 접근 방법: 규칙 조정하기

  말씀하신 대로, 이 규칙은 때로는 너무 강하게 느껴질 수 있습니다. 다행히도 ESLint 규칙은 얼마든지 우리 프로젝트에 맞게 조정할 수 있습니다.

  해결책은 두 가지입니다.

   4. 규칙을 따르기: 프로젝트의 모든 객체 형태 정의를 interface로 통일합니다. 이는 일관성 측면에서 가장 좋은 방법입니다.
   5. 규칙을 수정하거나 끄기: type과 interface를 의도에 따라 혼용하고 싶다면, 이 규칙을 비활성화할 수 있습니다. 또는, "객체 리터럴 형태일 때만
      interface를 사용하고, 유니온이나 유틸리티 타입을 쓸 때는 type을 허용"하도록 옵션을 조정할 수도 있습니다.

  규칙을 끄는 방법 (`eslint.config.js`):

    1 // eslint.config.js
    2
    3 module.exports = tseslint.config(
    4   // ... 다른 설정들
    5   {
    6     rules: {
    7       // ... 다른 규칙들
    8       '@typescript-eslint/consistent-type-definitions': 'off', // 규칙 비활성화
    9     },
   10   }
   11 );

  제안

  제 생각에는, DTO나 순수한 데이터 구조는 `interface`로 정의하는 것에 큰 단점이 없으므로, 일단은 일관성을 위해 규칙을 따르는 방향으로 진행해 보시는
  것을 추천합니다. type은 유니온(|), 인터섹션(&), 또는 Record나 Omit 같은 유틸리티 타입을 사용하여 복잡한 타입을 "별칭(alias)"으로 만들 때 사용하는
  것으로 역할을 분리하면 좋습니다.

  예시:

   1 // 객체의 "모양"은 interface로 정의
   2 interface User {
   3   id: number;
   4   name: string;
   5 }
   6
   7 // 여러 타입을 조합하거나 별칭을 만들 땐 type 사용
   8 type UserOrGuest = User | { isGuest: true };
   9 type UserId = User['id'];

  이 규칙에 대해 어떻게 생각하시나요? 규칙을 비활성화할까요, 아니면 일단 규칙에 맞춰 코드를 수정하는 방향으로 진행할까요?
```


---
## Reference.
- [tsconfig.json 완전 정복]([🚀 tsconfig.json 완전 정복! 타입스크립트 설정 옵션 A to Z 가이드 🚀](https://www.jaenung.net/tree/15794))
- [tsconfig 공식 reference]([TypeScript: TSConfig Reference - Docs on every TSConfig option](https://www.typescriptlang.org/tsconfig/#paths))
- [How to User TSCOnfig Path Aliases to Improve Your Code]([How to Use TSConfig Path Aliases to Improve Your Code](https://webreaper.dev/posts/tsconfig-paths-setup/))



