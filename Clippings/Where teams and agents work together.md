---
title: "Where teams and agents work together"
source: "https://app.notion.com/p/3-MVC-DispatcherServlet-3df5f76d265480e08c80ed1eeab65b21"
author:
published:
created: 2026-10-04
description: "A collaborative AI workspace, built on your company context. Build and orchestrate agents right alongside your team's projects, meetings, and connected apps."
tags:
  - "clippings"
---
Loopers의 게스트이십니다. 전체 사용 권한을 얻으려면 멤버가 되거나 내 워크스페이스를 탐색하세요.

멤버십을 요청하세요.

워크스페이스 추가하기

## 🦁 3주차 스프링 MVC: DispatcherServlet 을 직접 완성한다

> 발제([week3-spring-mvc.md](https://claude.ai/chat/week3-spring-mvc.md))에서 본 "인터페이스 리스트를
> 
> supports()
> 
> 로 훑어 첫 매칭을 실행"을 직접 코드로 완성한다. 보일러플레이트에는 프론트 컨트롤러 뼈대(
> 
> DispatcherServlet
> 
> )와 전략 인터페이스, 예제 컨트롤러가 주어지고, 비어 있는 전략 구현체(TODO) 를 채워 실패하는 인수 시나리오를 GREEN 으로 만든다.

### 🎓 실습 목표

> 🎓 이번 실습을 마치면
> 
> @RequestMapping
> 
> 스캔 →
> 
> (URL+METHOD)→HandlerMethod
> 
> 레지스트리를 직접 만들고, 매핑 없음(404)/메서드 불일치(405)를 코드로 가른다.
> 
> HandlerMethodArgumentResolver
> 
> 를 Composite(first-match) 로 조립해, 컨트롤러 파라미터가 채워지는 "마법"을 스스로 구현한다.
> 
> HttpMessageConverter
> 
> (Jackson)로
> 
> @RequestBody
> 
> ↔
> 
> @ResponseBody
> 
> 객체↔JSON 변환을 구현하고,
> 
> Content-Type
> 
> /
> 
> Accept
> 
> 가
> 
> canRead
> 
> /
> 
> canWrite
> 
> 를 어떻게 가르는지 버그로 재현한다.
> 
> throw
> 
> 된 예외를
> 
> @ControllerAdvice
> 
> /
> 
> @ExceptionHandler
> 
> 로 JSON 에러 응답으로 바꾸고, 커스텀 리졸버/인터셉터를 추가하는 것만으로 기능이 붙는(OCP) 확장 지점을 체험한다.

### 🎯 Summary

프론트 컨트롤러(

DispatcherServlet

) 뼈대는 주어진다. 학생은 그 뼈대가 호출하는 전략 구현체들(HandlerMapping / ArgumentResolver / MessageConverter / ReturnValueHandler / ExceptionResolver)을 채운다.

모든 STEP 은 하나의 인수 시나리오(GREEN 목표) 와 짝이다. 빨간 curl 을 초록으로 바꾸며 전진한다.

각 전략은 같은 패턴(리스트를

supports()

/

canRead

/

canWrite

로 훑어 첫 매칭)이라, 하나를 이해하면 나머지가 딸려 온다.

마지막 심화는 컨트롤러/DispatcherServlet 을 건드리지 않고 커스텀 리졸버/인터셉터를 "리스트에 추가"만으로 붙이는 것 — 스프링이 확장에 강한 이유를 손으로 확인한다.

### 📌 Keywords

프론트 컨트롤러

DispatcherServlet.doDispatch()

— 배분만 한다HandlerMapping —

@RequestMapping

스캔 →

RequestMappingInfo

레지스트리, 404 vs 405

ArgumentResolver(Composite) — Strategy + Composite, first-match

HttpMessageConverter(Jackson) —

@RequestBody

/

@ResponseBody

,

canRead

/

canWrite

HandlerExceptionResolver —

@ControllerAdvice

/

@ExceptionHandler

, 상속 기반 최근접

확장 지점(OCP) — 커스텀 리졸버 / HandlerInterceptor 추가

### 🧠 Learning

### 🚧 실습이 겨누는 문제 — "프레임워크가 알아서 해준다"를 열어본다

직접 구현해서 몸으로 이해한다. 채워야 할 전략들은 하나같이 동일한 패턴이라, 이 실습의 진짜 목표는 코드량이 아니라 "누가 이 일을 하는가"의 지도를 머릿속에 새기는 것이다.

### 📦 케이스 — 무엇이 주어지고, 무엇을 채우나

apps/spring-mvc-diy

보일러플레이트는 뼈대(주어짐) / 빈칸(TODO) 로 나뉜다. 뼈대는 손대지 말고, 빈칸만 채운다.

\[주어짐 ✅ — 건드리지 않는다\] DispatcherServlet 프론트 컨트롤러 doDispatch 오케스트레이션 (완성) ApplicationContext / BeanScanner 컴포넌트 스캔·DI (완성) TomcatWebServer 내장 톰캣 기동(8080) (완성) 전략 인터페이스들 HandlerMapping / HandlerAdapter / HandlerMethodArgumentResolver / HttpMessageConverter / HandlerExceptionResolver (시그니처만) 애노테이션 @RestController(@Controller+@ResponseBody) / @RequestMapping / @RequestBody / @ResponseBody / @ControllerAdvice / @ExceptionHandler 예제 앱 LectureRestController(GET·POST) / LectureExceptionAdvice / Lecture \[빈칸 🔧 — 여기를 채운다 (STEP 1~5)\] RequestMappingHandlerMapping @RequestMapping 스캔 → 레지스트리, getHandlerInternal (STEP 1) HandlerMethodArgumentResolverComposite selectResolver(first-match) + resolveArgument (STEP 2) RequestResponseBodyMethodProcessor @RequestBody read / @ResponseBody write (컨버터 위임) (STEP 3/4) MappingJackson2HttpMessageConverter canRead/canWrite/read/write (Jackson) (STEP 3) ExceptionHandlerExceptionResolver @ControllerAdvice 스캔 → 예외타입→메서드, resolveException (STEP 5) \[심화 🚀 — 리스트에 "추가"만으로 (STEP 6~7)\] RequestParamMethodArgumentResolver @RequestParam 처리 리졸버 신규 (STEP 6) RequestTimingInterceptor 등 preHandle/postHandle/afterCompletion (STEP 7)

> 👉 핵심 규칙: 컨트롤러와
> 
> DispatcherServlet
> 
> 은 절대 수정하지 않는다. 오직 전략 구현체만 채우거나 추가한다. 이 제약을 지키는 것 자체가 "프론트 컨트롤러 + 전략 패턴"을 체득했다는 증거다.

### 🧱 실습 준비 — 클론 → 실행 → 스모크 테스트

\# 1) 보일러플레이트 진입 (모노레포에 옮겨온 self-contained 프로젝트) cd apps/spring-mvc-diy # 2) 실행 — 내장 톰캣이 8080 에 뜬다 (com.diy.app.LectureApplication)./gradlew run # 또는 IDE 에서 LectureApplication#main 실행 # 3) 스모크 — 아직 STEP 을 안 채웠으면 대부분 실패(빨강)한다. 이게 출발점. curl -i localhost:8080/api/lectures # 목표: 이 요청부터 아래 ✅ 표의 10개가 전부 초록이 되게 만든다.

> ❗ 시작 상태는 의도적으로 빨강이다.
> 
> NullPointerException
> 
> /
> 
> IllegalStateException
> 
> ("적절한 리졸버 없음")/500 이 STEP 별로 사라져 간다.

### 🪜 단계별 과제 — 빨강을 초록으로

각 STEP 은 목표 / 채울 곳 / 구현 힌트 / 완료 기준(GREEN) 으로 구성된다. 순서대로 하면

doDispatch

흐름을 앞에서 뒤로 완성하게 된다.

#### STEP 1 — HandlerMapping: URL 을 자바 메서드로

목표:

GET /api/lectures

가

LectureRestController#list

로 라우팅되도록, 부팅 시

@RequestMapping

을 스캔해 레지스트리를 만든다.채울 곳:

RequestMappingHandlerMapping

—

initApplicationContext

(스캔·등록),

getHandlerInternal

(조회).

구현 힌트:

BeanFactoryUtils.beansOfAnnotated(context, Controller.class)

로

@RestController

빈을 모은다(

@RestController

는

@Controller

메타를 갖는다).클래스 레벨 경로 + 메서드 레벨 경로를 합성하고(슬래시 보정),

methods()

로 HTTP 메서드 집합을 만든다 →

RequestMappingInfo

.요청 시

getHandlerInternal

은

(requestURI, HTTP메서드)

로 레지스트리를 조회만 한다(재스캔 금지).완료 기준:

curl -i localhost:8080/api/lectures

가 500/NPE 대신 핸들러를 찾는다(아직 바디는 비어도 됨). 없는 경로는

null

→ 404.

#### STEP 2 — ArgumentResolver(Composite): 파라미터를 채운다

목표: 컨트롤러 파라미터(

HttpServletRequest

,

@RequestBody Lecture

등)를

HandlerMethod.invokeForRequest

가 채우도록, 리졸버 조립을 완성한다.채울 곳:

HandlerMethodArgumentResolverComposite

—

selectResolver

(first-match) /

resolveArgument

. (구체 리졸버

ServletRequestMethodArgumentResolver

는 참고용으로 주어짐.)

구현 힌트:

argumentResolvers

리스트를 순회하며

supportsParameter(p)

가 참인 첫 리졸버를 고른다. 없으면

IllegalStateException

("적절한 리졸버 없음").

리스트 순서 = 우선순위. 더 구체적인 리졸버를 앞에 둔다.

완료 기준:

GET /api/lectures

가 200 + JSON 배열 2건을 반환(STEP 3·4 의 write 가 준비됐다는 전제 — 아래와 함께 완성). "적절한 리졸버 없음" 예외가 사라진다.

#### STEP 3 — HttpMessageConverter(Jackson): 객체 ↔ JSON

목표: 요청 바디(JSON) → 객체(

@RequestBody

), 객체 → 응답 바디(JSON)(

@ResponseBody

)를 Jackson 으로 구현한다.채울 곳:

MappingJackson2HttpMessageConverter

(

canRead

/

canWrite

/

read

/

write

),

RequestResponseBodyMethodProcessor.readWithMessageConverters

.

구현 힌트:

canRead

:

Content-Type

에

application/json

이 있을 때만 true(없으면 false — 이게 뒤 STEP 의 버그 재현 포인트).

canWrite

:

Accept

가 비었거나

application/json

/

/\*

이면 true.

read

\=

objectMapper.readValue(req.getInputStream(), clazz)

,

write

\=

objectMapper.writeValue(resp.getOutputStream(), body)

+

Content-Type: application/json;charset=UTF-8

.완료 기준:

POST /api/lectures

(

Content-Type: application/json

)가 바디를 객체로 읽어 id=100 부여된 JSON 을 반환.

#### STEP 4 — ReturnValueHandler: 반환값을 응답으로

목표:

@ResponseBody

(및

@RestController

메타)인 메서드의 반환 객체를 컨버터로 직접 write 하고, 뷰 단계를 건너뛴다.채울 곳:

RequestResponseBodyMethodProcessor

—

supportsReturnType

(메타 애노테이션까지 탐색),

handleReturnValue

/

writeWithMessageConverters

.

구현 힌트:

supportsReturnType

: 선언 클래스/메서드에

@ResponseBody

가 메타 포함 존재하는지(→

@RestController

안의

@ResponseBody

를 찾아냄).

handleReturnValue

:

canWrite

인 첫 컨버터로 write 후

null

반환(→

doDispatch

가

mv==null

이라 render skip).완료 기준: 응답

Content-Type

이

application/json;charset=UTF-8

, 바디가 올바른 JSON. (STEP 2~4 가 함께

GET

을 완성한다.)

#### STEP 5 — ExceptionHandler: throw 를 JSON 에러로

목표: 서비스가

throw

한 예외를

@ControllerAdvice

/

@ExceptionHandler

로 잡아 깔끔한 JSON 에러 + 정확한 status 로 응답한다.채울 곳:

ExceptionHandlerExceptionResolver

—

initApplicationContext

(스캔: 예외타입→메서드),

resolveException

(최근접 탐색·invoke·JSON write).

구현 힌트:

부팅:

@ControllerAdvice

빈의

@ExceptionHandler

메서드를

Map<예외타입, 핸들러>

로 등록.요청: 예외 클래스에서 시작해

getSuperclass

로 상향하며 first-match(→ 하위 예외도 상위 핸들러가 받음).핸들러 인자(예외/

req

/

resp

)를 타입 보고 채워

invoke

, 반환값을 컨버터로 JSON write,

ModelAndView

반환(→ 재전파 안 함). 아무도 못 잡으면

null

→ 톰캣 기본 500.완료 기준:

POST /api/lectures

로

name

공백 전송 시 400 +

{"error":"BAD\_REQUEST","message":"name is required"}

.

#### STEP 6 (심화) — 커스텀 ArgumentResolver 추가: 확장 지점 체험

목표:

@RequestParam

을 처리하는 새 리졸버를 만들어 리스트에 추가만 하고, 컨트롤러에

@RequestParam int page

파라미터가 채워지는지 확인한다.채울 곳:

RequestParamMethodArgumentResolver

(신규) + 설정에서 리졸버 리스트에 등록. 컨트롤러/DispatcherServlet 은 수정 금지.구현 힌트:

supportsParameter

\=

@RequestParam

존재,

resolveArgument

\=

request.getParameter(name)

을 타입 변환(

Integer.parseInt

등). 필수/기본값 처리까지 하면 만점.완료 기준: 새 엔드포인트에

?page=2

를 주면 파라미터가 꽂힌다. 핵심 감상: "리스트에 구현체 추가"만으로 기능이 붙었다(OCP).

#### STEP 7 (심화) — HandlerInterceptor: 횡단 관심사

목표: 요청 처리 시간을 로깅하는

RequestTimingInterceptor

(pre/post/afterCompletion)를 붙인다.구현 힌트:

preHandle

에서 시작시각 저장(

req.setAttribute

),

afterCompletion

에서 경과 로깅(예외가 나도 실행됨).

InterceptorRegistry

에 등록.완료 기준: 모든 요청에 처리 시간 로그가 남고,

preHandle

이

false

면 컨트롤러가 실행되지 않는다.

### 🧭 예외 → 응답 매핑 분류 — 무엇을 몇 번으로 응답하나

> Round 7 의 "예외 4축"이 분산 트랜잭션의 대응을 갈랐듯, 스프링 MVC 에서도 예외를 종류별로 갈라 다른 HTTP status 로 응답해야 한다. STEP 5 의
> 
> @ExceptionHandler
> 
> 설계가 곧 이 분류다.

| 축 | 정체 | 예 | 응답 | 어디서 처리 |
| --- | --- | --- | --- | --- |
| 비즈니스(검증) | 도메인 규칙 위반 | IllegalArgumentException  ("name is required") | 400 + JSON | @ExceptionHandler  (우리  LectureExceptionAdvice  ) |
| 리소스 없음 | 매핑/엔티티 부재 | 매핑 없음,  NoSuchElementException | 404 | HandlerMapping  null  /  @ExceptionHandler |
| 메서드 불일치 | 경로는 맞고 METHOD 다름 | GET  \-only 에  POST | 405 | HandlerMapping(심화) |
| 요청 파싱 실패 | 바디/헤더 문제 | Content-Type  없음 →  @RequestBody  null | 400 | 컨버터  canRead=false  → 검증 예외 |
| 시스템 | 우리/인프라 결함 | NPE, 미처리 예외 | 500 | resolver 미처리 → 톰캣 기본 |

비즈니스 예외는 흡수하지 말고 정확한 4xx 로 전파한다("조용한 실패" 금지 — Round 6·7 과 동일한 교훈).

메서드 불일치(405) 는 현재 DIY 레지스트리가 404 로 뭉갠다 → STEP 1 심화로 "경로는 매칭·메서드만 불일치"를 감지해 405 로 분리하면 만점.

### 🎬 시퀀스 다이어그램 — 성공/예외 경로

> 정합성이 실패 경로에서 갈리듯, MVC 의 이해도 예외 경로에서 갈린다. 두 경로를 못 박는다.

#### ① 성공 — POST /api/lectures 가 JSON 을 되돌리기까지

<svg id="mermaid-33662061-5330-47a9-aa3d-e236082d265c" width="100%" xmlns="http://www.w3.org/2000/svg" style="max-width: 1864.5px;" viewBox="-50 -10 1864.5 721" role="graphics-document document" aria-roledescription="sequence"><g><rect x="1552.5" y="635" fill="#eaeaea" stroke="#666" width="212" height="65" name="RV" rx="3" ry="3"></rect><text x="1658.5" y="667.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1658.5" dy="0">ReturnValueHandler (STEP4)</tspan></text></g> <g><rect x="1335.5" y="635" fill="#eaeaea" stroke="#666" width="167" height="65" name="K" rx="3" ry="3"></rect><text x="1419" y="667.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1419" dy="0">LectureRestController</tspan></text></g> <g><rect x="1109.5" y="635" fill="#eaeaea" stroke="#666" width="176" height="65" name="MC" rx="3" ry="3"></rect><text x="1197.5" y="667.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1197.5" dy="0">Jackson 컨버터 (STEP3)</tspan></text></g> <g><rect x="781.5" y="635" fill="#eaeaea" stroke="#666" width="202" height="65" name="AR" rx="3" ry="3"></rect><text x="882.5" y="667.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="882.5" dy="0">ArgumentResolver (STEP2)</tspan></text></g> <g><rect x="546.5" y="635" fill="#eaeaea" stroke="#666" width="185" height="65" name="HM" rx="3" ry="3"></rect><text x="639" y="667.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="639" dy="0">HandlerMapping (STEP1)</tspan></text></g> <g><rect x="337" y="635" fill="#eaeaea" stroke="#666" width="150" height="65" name="DS" rx="3" ry="3"></rect><text x="412" y="667.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="412" dy="0">DispatcherServlet</tspan></text></g> <g><rect x="0" y="635" fill="#eaeaea" stroke="#666" width="150" height="65" name="C" rx="3" ry="3"></rect><text x="75" y="667.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="75" dy="0">클라이언트(curl)</tspan></text></g> <g><line id="actor18" x1="1658.5" y1="65" x2="1658.5" y2="635" stroke-width="0.5px" stroke="#999" name="RV" data-et="life-line" data-id="RV"></line><g id="root-18" data-et="participant" data-type="participant" data-id="RV"><rect x="1552.5" y="0" fill="#eaeaea" stroke="#666" width="212" height="65" name="RV" rx="3" ry="3"></rect><text x="1658.5" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1658.5" dy="0">ReturnValueHandler (STEP4)</tspan></text></g></g> <g><line id="actor17" x1="1419" y1="65" x2="1419" y2="635" stroke-width="0.5px" stroke="#999" name="K" data-et="life-line" data-id="K"></line><g id="root-17" data-et="participant" data-type="participant" data-id="K"><rect x="1335.5" y="0" fill="#eaeaea" stroke="#666" width="167" height="65" name="K" rx="3" ry="3"></rect><text x="1419" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1419" dy="0">LectureRestController</tspan></text></g></g> <g><line id="actor16" x1="1197.5" y1="65" x2="1197.5" y2="635" stroke-width="0.5px" stroke="#999" name="MC" data-et="life-line" data-id="MC"></line><g id="root-16" data-et="participant" data-type="participant" data-id="MC"><rect x="1109.5" y="0" fill="#eaeaea" stroke="#666" width="176" height="65" name="MC" rx="3" ry="3"></rect><text x="1197.5" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1197.5" dy="0">Jackson 컨버터 (STEP3)</tspan></text></g></g> <g><line id="actor15" x1="882.5" y1="65" x2="882.5" y2="635" stroke-width="0.5px" stroke="#999" name="AR" data-et="life-line" data-id="AR"></line><g id="root-15" data-et="participant" data-type="participant" data-id="AR"><rect x="781.5" y="0" fill="#eaeaea" stroke="#666" width="202" height="65" name="AR" rx="3" ry="3"></rect><text x="882.5" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="882.5" dy="0">ArgumentResolver (STEP2)</tspan></text></g></g> <g><line id="actor14" x1="639" y1="65" x2="639" y2="635" stroke-width="0.5px" stroke="#999" name="HM" data-et="life-line" data-id="HM"></line><g id="root-14" data-et="participant" data-type="participant" data-id="HM"><rect x="546.5" y="0" fill="#eaeaea" stroke="#666" width="185" height="65" name="HM" rx="3" ry="3"></rect><text x="639" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="639" dy="0">HandlerMapping (STEP1)</tspan></text></g></g> <g><line id="actor13" x1="412" y1="65" x2="412" y2="635" stroke-width="0.5px" stroke="#999" name="DS" data-et="life-line" data-id="DS"></line><g id="root-13" data-et="participant" data-type="participant" data-id="DS"><rect x="337" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" name="DS" rx="3" ry="3"></rect><text x="412" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="412" dy="0">DispatcherServlet</tspan></text></g></g> <g><line id="actor12" x1="75" y1="65" x2="75" y2="635" stroke-width="0.5px" stroke="#999" name="C" data-et="life-line" data-id="C"></line><g id="root-12" data-et="participant" data-type="participant" data-id="C"><rect x="0" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" name="C" rx="3" ry="3"></rect><text x="75" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="75" dy="0">클라이언트(curl)</tspan></text></g></g> <g></g><defs><symbol id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-computer" width="24" height="24"><path transform="scale(.5)" d="M2 2v13h20v-13h-20zm18 11h-16v-9h16v9zm-10.228 6l.466-1h3.524l.467 1h-4.457zm14.228 3h-24l2-6h2.104l-1.33 4h18.45l-1.297-4h2.073l2 6zm-5-10h-14v-7h14v7z"></path></symbol></defs><defs><symbol id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-database" fill-rule="evenodd" clip-rule="evenodd"><path transform="scale(.5)" d="M12.258.001l.256.004.255.005.253.008.251.01.249.012.247.015.246.016.242.019.241.02.239.023.236.024.233.027.231.028.229.031.225.032.223.034.22.036.217.038.214.04.211.041.208.043.205.045.201.046.198.048.194.05.191.051.187.053.183.054.18.056.175.057.172.059.168.06.163.061.16.063.155.064.15.066.074.033.073.033.071.034.07.034.069.035.068.035.067.035.066.035.064.036.064.036.062.036.06.036.06.037.058.037.058.037.055.038.055.038.053.038.052.038.051.039.05.039.048.039.047.039.045.04.044.04.043.04.041.04.04.041.039.041.037.041.036.041.034.041.033.042.032.042.03.042.029.042.027.042.026.043.024.043.023.043.021.043.02.043.018.044.017.043.015.044.013.044.012.044.011.045.009.044.007.045.006.045.004.045.002.045.001.045v17l-.001.045-.002.045-.004.045-.006.045-.007.045-.009.044-.011.045-.012.044-.013.044-.015.044-.017.043-.018.044-.02.043-.021.043-.023.043-.024.043-.026.043-.027.042-.029.042-.03.042-.032.042-.033.042-.034.041-.036.041-.037.041-.039.041-.04.041-.041.04-.043.04-.044.04-.045.04-.047.039-.048.039-.05.039-.051.039-.052.038-.053.038-.055.038-.055.038-.058.037-.058.037-.06.037-.06.036-.062.036-.064.036-.064.036-.066.035-.067.035-.068.035-.069.035-.07.034-.071.034-.073.033-.074.033-.15.066-.155.064-.16.063-.163.061-.168.06-.172.059-.175.057-.18.056-.183.054-.187.053-.191.051-.194.05-.198.048-.201.046-.205.045-.208.043-.211.041-.214.04-.217.038-.22.036-.223.034-.225.032-.229.031-.231.028-.233.027-.236.024-.239.023-.241.02-.242.019-.246.016-.247.015-.249.012-.251.01-.253.008-.255.005-.256.004-.258.001-.258-.001-.256-.004-.255-.005-.253-.008-.251-.01-.249-.012-.247-.015-.245-.016-.243-.019-.241-.02-.238-.023-.236-.024-.234-.027-.231-.028-.228-.031-.226-.032-.223-.034-.22-.036-.217-.038-.214-.04-.211-.041-.208-.043-.204-.045-.201-.046-.198-.048-.195-.05-.19-.051-.187-.053-.184-.054-.179-.056-.176-.057-.172-.059-.167-.06-.164-.061-.159-.063-.155-.064-.151-.066-.074-.033-.072-.033-.072-.034-.07-.034-.069-.035-.068-.035-.067-.035-.066-.035-.064-.036-.063-.036-.062-.036-.061-.036-.06-.037-.058-.037-.057-.037-.056-.038-.055-.038-.053-.038-.052-.038-.051-.039-.049-.039-.049-.039-.046-.039-.046-.04-.044-.04-.043-.04-.041-.04-.04-.041-.039-.041-.037-.041-.036-.041-.034-.041-.033-.042-.032-.042-.03-.042-.029-.042-.027-.042-.026-.043-.024-.043-.023-.043-.021-.043-.02-.043-.018-.044-.017-.043-.015-.044-.013-.044-.012-.044-.011-.045-.009-.044-.007-.045-.006-.045-.004-.045-.002-.045-.001-.045v-17l.001-.045.002-.045.004-.045.006-.045.007-.045.009-.044.011-.045.012-.044.013-.044.015-.044.017-.043.018-.044.02-.043.021-.043.023-.043.024-.043.026-.043.027-.042.029-.042.03-.042.032-.042.033-.042.034-.041.036-.041.037-.041.039-.041.04-.041.041-.04.043-.04.044-.04.046-.04.046-.039.049-.039.049-.039.051-.039.052-.038.053-.038.055-.038.056-.038.057-.037.058-.037.06-.037.061-.036.062-.036.063-.036.064-.036.066-.035.067-.035.068-.035.069-.035.07-.034.072-.034.072-.033.074-.033.151-.066.155-.064.159-.063.164-.061.167-.06.172-.059.176-.057.179-.056.184-.054.187-.053.19-.051.195-.05.198-.048.201-.046.204-.045.208-.043.211-.041.214-.04.217-.038.22-.036.223-.034.226-.032.228-.031.231-.028.234-.027.236-.024.238-.023.241-.02.243-.019.245-.016.247-.015.249-.012.251-.01.253-.008.255-.005.256-.004.258-.001.258.001zm-9.258 20.499v.01l.001.021.003.021.004.022.005.021.006.022.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.023.018.024.019.024.021.024.022.025.023.024.024.025.052.049.056.05.061.051.066.051.07.051.075.051.079.052.084.052.088.052.092.052.097.052.102.051.105.052.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.048.144.049.147.047.152.047.155.047.16.045.163.045.167.043.171.043.176.041.178.041.183.039.187.039.19.037.194.035.197.035.202.033.204.031.209.03.212.029.216.027.219.025.222.024.226.021.23.02.233.018.236.016.24.015.243.012.246.01.249.008.253.005.256.004.259.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.021.224-.024.22-.026.216-.027.212-.028.21-.031.205-.031.202-.034.198-.034.194-.036.191-.037.187-.039.183-.04.179-.04.175-.042.172-.043.168-.044.163-.045.16-.046.155-.046.152-.047.148-.048.143-.049.139-.049.136-.05.131-.05.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.053.083-.051.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.05.023-.024.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.023.01-.022.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.127l-.077.055-.08.053-.083.054-.085.053-.087.052-.09.052-.093.051-.095.05-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.045-.118.044-.12.043-.122.042-.124.042-.126.041-.128.04-.13.04-.132.038-.134.038-.135.037-.138.037-.139.035-.142.035-.143.034-.144.033-.147.032-.148.031-.15.03-.151.03-.153.029-.154.027-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.01-.179.008-.179.008-.181.006-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.006-.179-.008-.179-.008-.178-.01-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.027-.153-.029-.151-.03-.15-.03-.148-.031-.146-.032-.145-.033-.143-.034-.141-.035-.14-.035-.137-.037-.136-.037-.134-.038-.132-.038-.13-.04-.128-.04-.126-.041-.124-.042-.122-.042-.12-.044-.117-.043-.116-.045-.113-.045-.112-.046-.109-.047-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.05-.093-.052-.09-.051-.087-.052-.085-.053-.083-.054-.08-.054-.077-.054v4.127zm0-5.654v.011l.001.021.003.021.004.021.005.022.006.022.007.022.009.022.01.022.011.023.012.023.013.023.015.024.016.023.017.024.018.024.019.024.021.024.022.024.023.025.024.024.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.052.11.051.114.051.119.052.123.05.127.051.131.05.135.049.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.044.171.042.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.022.23.02.233.018.236.016.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.012.241-.015.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.048.139-.05.136-.049.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.051.051-.049.023-.025.023-.024.021-.025.02-.024.019-.024.018-.024.017-.024.015-.023.014-.023.013-.024.012-.022.01-.023.01-.023.008-.022.006-.022.006-.022.004-.021.004-.022.001-.021.001-.021v-4.139l-.077.054-.08.054-.083.054-.085.052-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.044-.118.044-.12.044-.122.042-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.035-.143.033-.144.033-.147.033-.148.031-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.009-.179.009-.179.007-.181.007-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.007-.179-.007-.179-.009-.178-.009-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.031-.146-.033-.145-.033-.143-.033-.141-.035-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.04-.126-.041-.124-.042-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.051-.093-.051-.09-.051-.087-.053-.085-.052-.083-.054-.08-.054-.077-.054v4.139zm0-5.666v.011l.001.02.003.022.004.021.005.022.006.021.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.024.018.023.019.024.021.025.022.024.023.024.024.025.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.051.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.043.171.043.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.021.23.02.233.018.236.017.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.013.241-.014.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.049.139-.049.136-.049.131-.051.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.049.023-.025.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.022.01-.023.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.153l-.077.054-.08.054-.083.053-.085.053-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.048-.105.048-.106.048-.109.046-.111.046-.114.046-.115.044-.118.044-.12.043-.122.043-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.034-.143.034-.144.033-.147.032-.148.032-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.024-.161.024-.162.023-.163.023-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.01-.178.01-.179.009-.179.007-.181.006-.182.006-.182.004-.184.003-.184.001-.185.001-.185-.001-.184-.001-.184-.003-.182-.004-.182-.006-.181-.006-.179-.007-.179-.009-.178-.01-.176-.01-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.023-.162-.023-.161-.024-.159-.024-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.032-.146-.032-.145-.033-.143-.034-.141-.034-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.041-.126-.041-.124-.041-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.048-.105-.048-.102-.048-.1-.05-.097-.049-.095-.051-.093-.051-.09-.052-.087-.052-.085-.053-.083-.053-.08-.054-.077-.054v4.153zm8.74-8.179l-.257.004-.254.005-.25.008-.247.011-.244.012-.241.014-.237.016-.233.018-.231.021-.226.022-.224.023-.22.026-.216.027-.212.028-.21.031-.205.032-.202.033-.198.034-.194.036-.191.038-.187.038-.183.04-.179.041-.175.042-.172.043-.168.043-.163.045-.16.046-.155.046-.152.048-.148.048-.143.048-.139.049-.136.05-.131.05-.126.051-.123.051-.118.051-.114.052-.11.052-.106.052-.101.052-.096.052-.092.052-.088.052-.083.052-.079.052-.074.051-.07.052-.065.051-.06.05-.056.05-.051.05-.023.025-.023.024-.021.024-.02.025-.019.024-.018.024-.017.023-.015.024-.014.023-.013.023-.012.023-.01.023-.01.022-.008.022-.006.023-.006.021-.004.022-.004.021-.001.021-.001.021.001.021.001.021.004.021.004.022.006.021.006.023.008.022.01.022.01.023.012.023.013.023.014.023.015.024.017.023.018.024.019.024.02.025.021.024.023.024.023.025.051.05.056.05.06.05.065.051.07.052.074.051.079.052.083.052.088.052.092.052.096.052.101.052.106.052.11.052.114.052.118.051.123.051.126.051.131.05.136.05.139.049.143.048.148.048.152.048.155.046.16.046.163.045.168.043.172.043.175.042.179.041.183.04.187.038.191.038.194.036.198.034.202.033.205.032.21.031.212.028.216.027.22.026.224.023.226.022.231.021.233.018.237.016.241.014.244.012.247.011.25.008.254.005.257.004.26.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.022.224-.023.22-.026.216-.027.212-.028.21-.031.205-.032.202-.033.198-.034.194-.036.191-.038.187-.038.183-.04.179-.041.175-.042.172-.043.168-.043.163-.045.16-.046.155-.046.152-.048.148-.048.143-.048.139-.049.136-.05.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.05.051-.05.023-.025.023-.024.021-.024.02-.025.019-.024.018-.024.017-.023.015-.024.014-.023.013-.023.012-.023.01-.023.01-.022.008-.022.006-.023.006-.021.004-.022.004-.021.001-.021.001-.021-.001-.021-.001-.021-.004-.021-.004-.022-.006-.021-.006-.023-.008-.022-.01-.022-.01-.023-.012-.023-.013-.023-.014-.023-.015-.024-.017-.023-.018-.024-.019-.024-.02-.025-.021-.024-.023-.024-.023-.025-.051-.05-.056-.05-.06-.05-.065-.051-.07-.052-.074-.051-.079-.052-.083-.052-.088-.052-.092-.052-.096-.052-.101-.052-.106-.052-.11-.052-.114-.052-.118-.051-.123-.051-.126-.051-.131-.05-.136-.05-.139-.049-.143-.048-.148-.048-.152-.048-.155-.046-.16-.046-.163-.045-.168-.043-.172-.043-.175-.042-.179-.041-.183-.04-.187-.038-.191-.038-.194-.036-.198-.034-.202-.033-.205-.032-.21-.031-.212-.028-.216-.027-.22-.026-.224-.023-.226-.022-.231-.021-.233-.018-.237-.016-.241-.014-.244-.012-.247-.011-.25-.008-.254-.005-.257-.004-.26-.001-.26.001z"></path></symbol></defs><defs><symbol id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-clock" width="24" height="24"><path transform="scale(.5)" d="M12 2c5.514 0 10 4.486 10 10s-4.486 10-10 10-10-4.486-10-10 4.486-10 10-10zm0-2c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm5.848 12.459c.202.038.202.333.001.372-1.907.361-6.045 1.111-6.547 1.111-.719 0-1.301-.582-1.301-1.301 0-.512.77-5.447 1.125-7.445.034-.192.312-.181.343.014l.985 6.238 5.394 1.011z"></path></symbol></defs><defs><marker id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead" refX="7.9" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M -1 0 L 10 5 L 0 10 z"></path></marker></defs><defs><marker id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-crosshead" markerWidth="15" markerHeight="8" orient="auto" refX="4" refY="4.5"><path fill="none" stroke="#000000" stroke-width="1pt" d="M 1,2 L 6,7 M 6,2 L 1,7" style="stroke-dasharray: 0, 0;"></path></marker></defs><defs><marker id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-filled-head" refX="15.5" refY="7" markerWidth="20" markerHeight="28" orient="auto"><path d="M 18,7 L9,13 L14,7 L9,1 Z"></path></marker></defs><defs><marker id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber" refX="15" refY="15" markerWidth="60" markerHeight="40" orient="auto"><circle cx="15" cy="15" r="6"></circle></marker></defs><defs><marker id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-solidTopArrowHead" refX="7.9" refY="7.25" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M 0 0 L 10 8 L 0 8 z"></path></marker></defs><defs><marker id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-solidBottomArrowHead" refX="7.9" refY="0.75" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M 0 0 L 10 0 L 0 8 z"></path></marker></defs><defs><marker id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-stickTopArrowHead" refX="7.5" refY="7" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M 0 0 L 7 7" stroke="black" stroke-width="1.5" fill="none"></path></marker></defs><defs><marker id="mermaid-33662061-5330-47a9-aa3d-e236082d265c-stickBottomArrowHead" refX="7.5" refY="0" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M 0 7 L 7 0" stroke="black" stroke-width="1.5" fill="none"></path></marker></defs><text x="242" y="80" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">POST /api/lectures (Content-Type: json)</text> <line x1="82" y1="115" x2="408" y2="115" data-et="message" data-id="i1" data-from="C" data-to="DS" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="fill: none;"></line><line x1="75" y1="115" x2="75" y2="115" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="75" y="119" font-family="sans-serif" font-size="12px" text-anchor="middle">1</text> <text x="524" y="130" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">getHandler(req)</text> <line x1="419" y1="165" x2="635" y2="165" data-et="message" data-id="i2" data-from="DS" data-to="HM" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="fill: none;"></line><line x1="412" y1="165" x2="412" y2="165" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="412" y="169" font-family="sans-serif" font-size="12px" text-anchor="middle">2</text> <text x="527" y="180" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">HandlerMethod(create)</text> <line x1="644" y1="215" x2="416" y2="215" data-et="message" data-id="i3" data-from="HM" data-to="DS" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line><line x1="639" y1="215" x2="639" y2="215" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="639" y="219" font-family="sans-serif" font-size="12px" text-anchor="middle">3</text> <text x="646" y="230" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">resolveArgument(@RequestBody Lecture)</text> <line x1="419" y1="265" x2="878.5" y2="265" data-et="message" data-id="i4" data-from="DS" data-to="AR" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="fill: none;"></line><line x1="412" y1="265" x2="412" y2="265" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="412" y="269" font-family="sans-serif" font-size="12px" text-anchor="middle">4</text> <text x="1039" y="280" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">canRead(json)? → read(inputStream)</text> <line x1="889.5" y1="315" x2="1193.5" y2="315" data-et="message" data-id="i5" data-from="AR" data-to="MC" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="fill: none;"></line><line x1="882.5" y1="315" x2="882.5" y2="315" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="882.5" y="319" font-family="sans-serif" font-size="12px" text-anchor="middle">5</text> <text x="1042" y="330" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">Lecture 객체</text> <line x1="1202.5" y1="365" x2="886.5" y2="365" data-et="message" data-id="i6" data-from="MC" data-to="AR" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line><line x1="1197.5" y1="365" x2="1197.5" y2="365" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="1197.5" y="369" font-family="sans-serif" font-size="12px" text-anchor="middle">6</text> <text x="914" y="380" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">method.invoke(create, [lecture])</text> <line x1="419" y1="415" x2="1415" y2="415" data-et="message" data-id="i7" data-from="DS" data-to="K" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="fill: none;"></line><line x1="412" y1="415" x2="412" y2="415" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="412" y="419" font-family="sans-serif" font-size="12px" text-anchor="middle">7</text> <text x="917" y="430" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">return Lecture(id=100)</text> <line x1="1424" y1="465" x2="416" y2="465" data-et="message" data-id="i8" data-from="K" data-to="DS" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line><line x1="1419" y1="465" x2="1419" y2="465" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="1419" y="469" font-family="sans-serif" font-size="12px" text-anchor="middle">8</text> <text x="1034" y="480" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">handleReturnValue(lecture)</text> <line x1="419" y1="515" x2="1654.5" y2="515" data-et="message" data-id="i9" data-from="DS" data-to="RV" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="fill: none;"></line><line x1="412" y1="515" x2="412" y2="515" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="412" y="519" font-family="sans-serif" font-size="12px" text-anchor="middle">9</text> <text x="1430" y="530" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">canWrite? → write(lecture, resp)</text> <line x1="1663.5" y1="565" x2="1201.5" y2="565" data-et="message" data-id="i10" data-from="RV" data-to="MC" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="fill: none;"></line><line x1="1658.5" y1="565" x2="1658.5" y2="565" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="1658.5" y="569" font-family="sans-serif" font-size="12px" text-anchor="middle">10</text> <text x="638" y="580" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">200 + application/json { id:100,... }</text> <line x1="1202.5" y1="615" x2="79" y2="615" data-et="message" data-id="i11" data-from="MC" data-to="C" stroke-width="2" stroke="none" marker-end="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line><line x1="1197.5" y1="615" x2="1197.5" y2="615" stroke-width="0" marker-start="url(#mermaid-33662061-5330-47a9-aa3d-e236082d265c-sequencenumber)"></line><text x="1197.5" y="619" font-family="sans-serif" font-size="12px" text-anchor="middle">11</text></svg>

#### ② 예외 — 검증 실패가 400 JSON 이 되기까지

<svg id="mermaid-a23989ee-143c-403c-a486-6e746144913f" width="100%" xmlns="http://www.w3.org/2000/svg" style="max-width: 1583.5px;" viewBox="-50 -10 1583.5 650" role="graphics-document document" aria-roledescription="sequence"><g><rect x="1300.5" y="564" fill="#eaeaea" stroke="#666" width="183" height="65" name="A" rx="3" ry="3"></rect><text x="1392" y="596.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1392" dy="0">LectureExceptionAdvice</tspan></text></g> <g><rect x="897.5" y="564" fill="#eaeaea" stroke="#666" width="253" height="65" name="ER" rx="3" ry="3"></rect><text x="1024" y="596.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1024" dy="0">ExceptionHandlerResolver (STEP5)</tspan></text></g> <g><rect x="680.5" y="564" fill="#eaeaea" stroke="#666" width="167" height="65" name="K" rx="3" ry="3"></rect><text x="764" y="596.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="764" dy="0">LectureRestController</tspan></text></g> <g><rect x="270" y="564" fill="#eaeaea" stroke="#666" width="150" height="65" name="DS" rx="3" ry="3"></rect><text x="345" y="596.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="345" dy="0">DispatcherServlet</tspan></text></g> <g><rect x="0" y="564" fill="#eaeaea" stroke="#666" width="150" height="65" name="C" rx="3" ry="3"></rect><text x="75" y="596.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="75" dy="0">클라이언트(curl)</tspan></text></g> <g><line id="actor23" x1="1392" y1="65" x2="1392" y2="564" stroke-width="0.5px" stroke="#999" name="A" data-et="life-line" data-id="A"></line><g id="root-23" data-et="participant" data-type="participant" data-id="A"><rect x="1300.5" y="0" fill="#eaeaea" stroke="#666" width="183" height="65" name="A" rx="3" ry="3"></rect><text x="1392" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1392" dy="0">LectureExceptionAdvice</tspan></text></g></g> <g><line id="actor22" x1="1024" y1="65" x2="1024" y2="564" stroke-width="0.5px" stroke="#999" name="ER" data-et="life-line" data-id="ER"></line><g id="root-22" data-et="participant" data-type="participant" data-id="ER"><rect x="897.5" y="0" fill="#eaeaea" stroke="#666" width="253" height="65" name="ER" rx="3" ry="3"></rect><text x="1024" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="1024" dy="0">ExceptionHandlerResolver (STEP5)</tspan></text></g></g> <g><line id="actor21" x1="764" y1="65" x2="764" y2="564" stroke-width="0.5px" stroke="#999" name="K" data-et="life-line" data-id="K"></line><g id="root-21" data-et="participant" data-type="participant" data-id="K"><rect x="680.5" y="0" fill="#eaeaea" stroke="#666" width="167" height="65" name="K" rx="3" ry="3"></rect><text x="764" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="764" dy="0">LectureRestController</tspan></text></g></g> <g><line id="actor20" x1="345" y1="65" x2="345" y2="564" stroke-width="0.5px" stroke="#999" name="DS" data-et="life-line" data-id="DS"></line><g id="root-20" data-et="participant" data-type="participant" data-id="DS"><rect x="270" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" name="DS" rx="3" ry="3"></rect><text x="345" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="345" dy="0">DispatcherServlet</tspan></text></g></g> <g><line id="actor19" x1="75" y1="65" x2="75" y2="564" stroke-width="0.5px" stroke="#999" name="C" data-et="life-line" data-id="C"></line><g id="root-19" data-et="participant" data-type="participant" data-id="C"><rect x="0" y="0" fill="#eaeaea" stroke="#666" width="150" height="65" name="C" rx="3" ry="3"></rect><text x="75" y="32.5" dominant-baseline="central" alignment-baseline="central" style="text-anchor: middle; font-size: 16px; font-weight: 400;"><tspan x="75" dy="0">클라이언트(curl)</tspan></text></g></g> <g></g><defs><symbol id="mermaid-a23989ee-143c-403c-a486-6e746144913f-computer" width="24" height="24"><path transform="scale(.5)" d="M2 2v13h20v-13h-20zm18 11h-16v-9h16v9zm-10.228 6l.466-1h3.524l.467 1h-4.457zm14.228 3h-24l2-6h2.104l-1.33 4h18.45l-1.297-4h2.073l2 6zm-5-10h-14v-7h14v7z"></path></symbol></defs><defs><symbol id="mermaid-a23989ee-143c-403c-a486-6e746144913f-database" fill-rule="evenodd" clip-rule="evenodd"><path transform="scale(.5)" d="M12.258.001l.256.004.255.005.253.008.251.01.249.012.247.015.246.016.242.019.241.02.239.023.236.024.233.027.231.028.229.031.225.032.223.034.22.036.217.038.214.04.211.041.208.043.205.045.201.046.198.048.194.05.191.051.187.053.183.054.18.056.175.057.172.059.168.06.163.061.16.063.155.064.15.066.074.033.073.033.071.034.07.034.069.035.068.035.067.035.066.035.064.036.064.036.062.036.06.036.06.037.058.037.058.037.055.038.055.038.053.038.052.038.051.039.05.039.048.039.047.039.045.04.044.04.043.04.041.04.04.041.039.041.037.041.036.041.034.041.033.042.032.042.03.042.029.042.027.042.026.043.024.043.023.043.021.043.02.043.018.044.017.043.015.044.013.044.012.044.011.045.009.044.007.045.006.045.004.045.002.045.001.045v17l-.001.045-.002.045-.004.045-.006.045-.007.045-.009.044-.011.045-.012.044-.013.044-.015.044-.017.043-.018.044-.02.043-.021.043-.023.043-.024.043-.026.043-.027.042-.029.042-.03.042-.032.042-.033.042-.034.041-.036.041-.037.041-.039.041-.04.041-.041.04-.043.04-.044.04-.045.04-.047.039-.048.039-.05.039-.051.039-.052.038-.053.038-.055.038-.055.038-.058.037-.058.037-.06.037-.06.036-.062.036-.064.036-.064.036-.066.035-.067.035-.068.035-.069.035-.07.034-.071.034-.073.033-.074.033-.15.066-.155.064-.16.063-.163.061-.168.06-.172.059-.175.057-.18.056-.183.054-.187.053-.191.051-.194.05-.198.048-.201.046-.205.045-.208.043-.211.041-.214.04-.217.038-.22.036-.223.034-.225.032-.229.031-.231.028-.233.027-.236.024-.239.023-.241.02-.242.019-.246.016-.247.015-.249.012-.251.01-.253.008-.255.005-.256.004-.258.001-.258-.001-.256-.004-.255-.005-.253-.008-.251-.01-.249-.012-.247-.015-.245-.016-.243-.019-.241-.02-.238-.023-.236-.024-.234-.027-.231-.028-.228-.031-.226-.032-.223-.034-.22-.036-.217-.038-.214-.04-.211-.041-.208-.043-.204-.045-.201-.046-.198-.048-.195-.05-.19-.051-.187-.053-.184-.054-.179-.056-.176-.057-.172-.059-.167-.06-.164-.061-.159-.063-.155-.064-.151-.066-.074-.033-.072-.033-.072-.034-.07-.034-.069-.035-.068-.035-.067-.035-.066-.035-.064-.036-.063-.036-.062-.036-.061-.036-.06-.037-.058-.037-.057-.037-.056-.038-.055-.038-.053-.038-.052-.038-.051-.039-.049-.039-.049-.039-.046-.039-.046-.04-.044-.04-.043-.04-.041-.04-.04-.041-.039-.041-.037-.041-.036-.041-.034-.041-.033-.042-.032-.042-.03-.042-.029-.042-.027-.042-.026-.043-.024-.043-.023-.043-.021-.043-.02-.043-.018-.044-.017-.043-.015-.044-.013-.044-.012-.044-.011-.045-.009-.044-.007-.045-.006-.045-.004-.045-.002-.045-.001-.045v-17l.001-.045.002-.045.004-.045.006-.045.007-.045.009-.044.011-.045.012-.044.013-.044.015-.044.017-.043.018-.044.02-.043.021-.043.023-.043.024-.043.026-.043.027-.042.029-.042.03-.042.032-.042.033-.042.034-.041.036-.041.037-.041.039-.041.04-.041.041-.04.043-.04.044-.04.046-.04.046-.039.049-.039.049-.039.051-.039.052-.038.053-.038.055-.038.056-.038.057-.037.058-.037.06-.037.061-.036.062-.036.063-.036.064-.036.066-.035.067-.035.068-.035.069-.035.07-.034.072-.034.072-.033.074-.033.151-.066.155-.064.159-.063.164-.061.167-.06.172-.059.176-.057.179-.056.184-.054.187-.053.19-.051.195-.05.198-.048.201-.046.204-.045.208-.043.211-.041.214-.04.217-.038.22-.036.223-.034.226-.032.228-.031.231-.028.234-.027.236-.024.238-.023.241-.02.243-.019.245-.016.247-.015.249-.012.251-.01.253-.008.255-.005.256-.004.258-.001.258.001zm-9.258 20.499v.01l.001.021.003.021.004.022.005.021.006.022.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.023.018.024.019.024.021.024.022.025.023.024.024.025.052.049.056.05.061.051.066.051.07.051.075.051.079.052.084.052.088.052.092.052.097.052.102.051.105.052.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.048.144.049.147.047.152.047.155.047.16.045.163.045.167.043.171.043.176.041.178.041.183.039.187.039.19.037.194.035.197.035.202.033.204.031.209.03.212.029.216.027.219.025.222.024.226.021.23.02.233.018.236.016.24.015.243.012.246.01.249.008.253.005.256.004.259.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.021.224-.024.22-.026.216-.027.212-.028.21-.031.205-.031.202-.034.198-.034.194-.036.191-.037.187-.039.183-.04.179-.04.175-.042.172-.043.168-.044.163-.045.16-.046.155-.046.152-.047.148-.048.143-.049.139-.049.136-.05.131-.05.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.053.083-.051.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.05.023-.024.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.023.01-.022.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.127l-.077.055-.08.053-.083.054-.085.053-.087.052-.09.052-.093.051-.095.05-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.045-.118.044-.12.043-.122.042-.124.042-.126.041-.128.04-.13.04-.132.038-.134.038-.135.037-.138.037-.139.035-.142.035-.143.034-.144.033-.147.032-.148.031-.15.03-.151.03-.153.029-.154.027-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.01-.179.008-.179.008-.181.006-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.006-.179-.008-.179-.008-.178-.01-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.027-.153-.029-.151-.03-.15-.03-.148-.031-.146-.032-.145-.033-.143-.034-.141-.035-.14-.035-.137-.037-.136-.037-.134-.038-.132-.038-.13-.04-.128-.04-.126-.041-.124-.042-.122-.042-.12-.044-.117-.043-.116-.045-.113-.045-.112-.046-.109-.047-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.05-.093-.052-.09-.051-.087-.052-.085-.053-.083-.054-.08-.054-.077-.054v4.127zm0-5.654v.011l.001.021.003.021.004.021.005.022.006.022.007.022.009.022.01.022.011.023.012.023.013.023.015.024.016.023.017.024.018.024.019.024.021.024.022.024.023.025.024.024.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.052.11.051.114.051.119.052.123.05.127.051.131.05.135.049.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.044.171.042.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.022.23.02.233.018.236.016.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.012.241-.015.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.048.139-.05.136-.049.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.051.051-.049.023-.025.023-.024.021-.025.02-.024.019-.024.018-.024.017-.024.015-.023.014-.023.013-.024.012-.022.01-.023.01-.023.008-.022.006-.022.006-.022.004-.021.004-.022.001-.021.001-.021v-4.139l-.077.054-.08.054-.083.054-.085.052-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.044-.118.044-.12.044-.122.042-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.035-.143.033-.144.033-.147.033-.148.031-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.009-.179.009-.179.007-.181.007-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.007-.179-.007-.179-.009-.178-.009-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.031-.146-.033-.145-.033-.143-.033-.141-.035-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.04-.126-.041-.124-.042-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.051-.093-.051-.09-.051-.087-.053-.085-.052-.083-.054-.08-.054-.077-.054v4.139zm0-5.666v.011l.001.02.003.022.004.021.005.022.006.021.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.024.018.023.019.024.021.025.022.024.023.024.024.025.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.051.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.043.171.043.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.021.23.02.233.018.236.017.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.013.241-.014.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.049.139-.049.136-.049.131-.051.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.049.023-.025.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.022.01-.023.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.153l-.077.054-.08.054-.083.053-.085.053-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.048-.105.048-.106.048-.109.046-.111.046-.114.046-.115.044-.118.044-.12.043-.122.043-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.034-.143.034-.144.033-.147.032-.148.032-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.024-.161.024-.162.023-.163.023-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.01-.178.01-.179.009-.179.007-.181.006-.182.006-.182.004-.184.003-.184.001-.185.001-.185-.001-.184-.001-.184-.003-.182-.004-.182-.006-.181-.006-.179-.007-.179-.009-.178-.01-.176-.01-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.023-.162-.023-.161-.024-.159-.024-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.032-.146-.032-.145-.033-.143-.034-.141-.034-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.041-.126-.041-.124-.041-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.048-.105-.048-.102-.048-.1-.05-.097-.049-.095-.051-.093-.051-.09-.052-.087-.052-.085-.053-.083-.053-.08-.054-.077-.054v4.153zm8.74-8.179l-.257.004-.254.005-.25.008-.247.011-.244.012-.241.014-.237.016-.233.018-.231.021-.226.022-.224.023-.22.026-.216.027-.212.028-.21.031-.205.032-.202.033-.198.034-.194.036-.191.038-.187.038-.183.04-.179.041-.175.042-.172.043-.168.043-.163.045-.16.046-.155.046-.152.048-.148.048-.143.048-.139.049-.136.05-.131.05-.126.051-.123.051-.118.051-.114.052-.11.052-.106.052-.101.052-.096.052-.092.052-.088.052-.083.052-.079.052-.074.051-.07.052-.065.051-.06.05-.056.05-.051.05-.023.025-.023.024-.021.024-.02.025-.019.024-.018.024-.017.023-.015.024-.014.023-.013.023-.012.023-.01.023-.01.022-.008.022-.006.023-.006.021-.004.022-.004.021-.001.021-.001.021.001.021.001.021.004.021.004.022.006.021.006.023.008.022.01.022.01.023.012.023.013.023.014.023.015.024.017.023.018.024.019.024.02.025.021.024.023.024.023.025.051.05.056.05.06.05.065.051.07.052.074.051.079.052.083.052.088.052.092.052.096.052.101.052.106.052.11.052.114.052.118.051.123.051.126.051.131.05.136.05.139.049.143.048.148.048.152.048.155.046.16.046.163.045.168.043.172.043.175.042.179.041.183.04.187.038.191.038.194.036.198.034.202.033.205.032.21.031.212.028.216.027.22.026.224.023.226.022.231.021.233.018.237.016.241.014.244.012.247.011.25.008.254.005.257.004.26.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.022.224-.023.22-.026.216-.027.212-.028.21-.031.205-.032.202-.033.198-.034.194-.036.191-.038.187-.038.183-.04.179-.041.175-.042.172-.043.168-.043.163-.045.16-.046.155-.046.152-.048.148-.048.143-.048.139-.049.136-.05.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.05.051-.05.023-.025.023-.024.021-.024.02-.025.019-.024.018-.024.017-.023.015-.024.014-.023.013-.023.012-.023.01-.023.01-.022.008-.022.006-.023.006-.021.004-.022.004-.021.001-.021.001-.021-.001-.021-.001-.021-.004-.021-.004-.022-.006-.021-.006-.023-.008-.022-.01-.022-.01-.023-.012-.023-.013-.023-.014-.023-.015-.024-.017-.023-.018-.024-.019-.024-.02-.025-.021-.024-.023-.024-.023-.025-.051-.05-.056-.05-.06-.05-.065-.051-.07-.052-.074-.051-.079-.052-.083-.052-.088-.052-.092-.052-.096-.052-.101-.052-.106-.052-.11-.052-.114-.052-.118-.051-.123-.051-.126-.051-.131-.05-.136-.05-.139-.049-.143-.048-.148-.048-.152-.048-.155-.046-.16-.046-.163-.045-.168-.043-.172-.043-.175-.042-.179-.041-.183-.04-.187-.038-.191-.038-.194-.036-.198-.034-.202-.033-.205-.032-.21-.031-.212-.028-.216-.027-.22-.026-.224-.023-.226-.022-.231-.021-.233-.018-.237-.016-.241-.014-.244-.012-.247-.011-.25-.008-.254-.005-.257-.004-.26-.001-.26.001z"></path></symbol></defs><defs><symbol id="mermaid-a23989ee-143c-403c-a486-6e746144913f-clock" width="24" height="24"><path transform="scale(.5)" d="M12 2c5.514 0 10 4.486 10 10s-4.486 10-10 10-10-4.486-10-10 4.486-10 10-10zm0-2c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm5.848 12.459c.202.038.202.333.001.372-1.907.361-6.045 1.111-6.547 1.111-.719 0-1.301-.582-1.301-1.301 0-.512.77-5.447 1.125-7.445.034-.192.312-.181.343.014l.985 6.238 5.394 1.011z"></path></symbol></defs><defs><marker id="mermaid-a23989ee-143c-403c-a486-6e746144913f-arrowhead" refX="7.9" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M -1 0 L 10 5 L 0 10 z"></path></marker></defs><defs><marker id="mermaid-a23989ee-143c-403c-a486-6e746144913f-crosshead" markerWidth="15" markerHeight="8" orient="auto" refX="4" refY="4.5"><path fill="none" stroke="#000000" stroke-width="1pt" d="M 1,2 L 6,7 M 6,2 L 1,7" style="stroke-dasharray: 0, 0;"></path></marker></defs><defs><marker id="mermaid-a23989ee-143c-403c-a486-6e746144913f-filled-head" refX="15.5" refY="7" markerWidth="20" markerHeight="28" orient="auto"><path d="M 18,7 L9,13 L14,7 L9,1 Z"></path></marker></defs><defs><marker id="mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber" refX="15" refY="15" markerWidth="60" markerHeight="40" orient="auto"><circle cx="15" cy="15" r="6"></circle></marker></defs><defs><marker id="mermaid-a23989ee-143c-403c-a486-6e746144913f-solidTopArrowHead" refX="7.9" refY="7.25" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M 0 0 L 10 8 L 0 8 z"></path></marker></defs><defs><marker id="mermaid-a23989ee-143c-403c-a486-6e746144913f-solidBottomArrowHead" refX="7.9" refY="0.75" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M 0 0 L 10 0 L 0 8 z"></path></marker></defs><defs><marker id="mermaid-a23989ee-143c-403c-a486-6e746144913f-stickTopArrowHead" refX="7.5" refY="7" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M 0 0 L 7 7" stroke="black" stroke-width="1.5" fill="none"></path></marker></defs><defs><marker id="mermaid-a23989ee-143c-403c-a486-6e746144913f-stickBottomArrowHead" refX="7.5" refY="0" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto-start-reverse"><path d="M 0 7 L 7 0" stroke="black" stroke-width="1.5" fill="none"></path></marker></defs><g data-et="note" data-id="i9"><rect x="320" y="505" fill="#EDF2AE" stroke="#666" width="729" height="39"></rect><text x="685" y="510" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;"><tspan x="685">resolver 가 처리 → 재전파 안 함. 못 잡았으면 톰캣 기본 500.</tspan></text></g><text x="209" y="80" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">POST /api/lectures { name:"" }</text> <line x1="82" y1="115" x2="341" y2="115" data-et="message" data-id="i1" data-from="C" data-to="DS" stroke-width="2" stroke="none" marker-end="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-arrowhead)" style="fill: none;"></line><line x1="75" y1="115" x2="75" y2="115" stroke-width="0" marker-start="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber)"></line><text x="75" y="119" font-family="sans-serif" font-size="12px" text-anchor="middle">1</text> <text x="553" y="130" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">method.invoke(create, [lecture])</text> <line x1="352" y1="165" x2="760" y2="165" data-et="message" data-id="i2" data-from="DS" data-to="K" stroke-width="2" stroke="none" marker-end="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-arrowhead)" style="fill: none;"></line><line x1="345" y1="165" x2="345" y2="165" stroke-width="0" marker-start="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber)"></line><text x="345" y="169" font-family="sans-serif" font-size="12px" text-anchor="middle">2</text> <text x="556" y="180" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">throw IllegalArgumentException("name is required")</text> <line x1="769" y1="215" x2="349" y2="215" data-et="message" data-id="i3" data-from="K" data-to="DS" stroke-width="2" stroke="none" marker-end="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-crosshead)" style="stroke-dasharray: 3, 3; fill: none;"></line><line x1="764" y1="215" x2="764" y2="215" stroke-width="0" marker-start="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber)"></line><text x="764" y="219" font-family="sans-serif" font-size="12px" text-anchor="middle">3</text> <text x="683" y="230" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">processHandlerException(ex)</text> <line x1="352" y1="265" x2="1020" y2="265" data-et="message" data-id="i4" data-from="DS" data-to="ER" stroke-width="2" stroke="none" marker-end="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-arrowhead)" style="fill: none;"></line><line x1="345" y1="265" x2="345" y2="265" stroke-width="0" marker-start="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber)"></line><text x="345" y="269" font-family="sans-serif" font-size="12px" text-anchor="middle">4</text> <text x="1025" y="280" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">findHandlerMethod(IllegalArgumentException) — 상속 상향 탐색</text> <path d="M 1025,315 C 1085,305 1085,345 1025,335" data-et="message" data-id="i5" data-from="ER" data-to="ER" stroke-width="2" stroke="none" marker-end="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-arrowhead)" x1="1031" style="fill: none;"></path><line x1="1024" y1="315" x2="1024" y2="315" stroke-width="0" marker-start="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber)"></line><text x="1024" y="319" font-family="sans-serif" font-size="12px" text-anchor="middle">5</text> <text x="1207" y="360" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">invoke handleIllegalArgument(ex, resp)</text> <line x1="1031" y1="395" x2="1388" y2="395" data-et="message" data-id="i6" data-from="ER" data-to="A" stroke-width="2" stroke="none" marker-end="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-arrowhead)" style="fill: none;"></line><line x1="1024" y1="395" x2="1024" y2="395" stroke-width="0" marker-start="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber)"></line><text x="1024" y="399" font-family="sans-serif" font-size="12px" text-anchor="middle">6</text> <text x="1210" y="410" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">resp.status=400, return Map{error,message}</text> <line x1="1397" y1="445" x2="1028" y2="445" data-et="message" data-id="i7" data-from="A" data-to="ER" stroke-width="2" stroke="none" marker-end="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-arrowhead)" style="stroke-dasharray: 3, 3; fill: none;"></line><line x1="1392" y1="445" x2="1392" y2="445" stroke-width="0" marker-start="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber)"></line><text x="1392" y="449" font-family="sans-serif" font-size="12px" text-anchor="middle">7</text> <text x="551" y="460" text-anchor="middle" dominant-baseline="middle" alignment-baseline="middle" dy="1em" style="font-size: 16px; font-weight: 400;">400 + application/json { "error":"BAD_REQUEST",... }</text> <line x1="1029" y1="495" x2="79" y2="495" data-et="message" data-id="i8" data-from="ER" data-to="C" stroke-width="2" stroke="none" marker-end="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-arrowhead)" style="fill: none;"></line><line x1="1024" y1="495" x2="1024" y2="495" stroke-width="0" marker-start="url(#mermaid-a23989ee-143c-403c-a486-6e746144913f-sequencenumber)"></line><text x="1024" y="499" font-family="sans-serif" font-size="12px" text-anchor="middle">8</text></svg>

### ✅ 인수 기준 — curl 시나리오 10개 전수 GREEN

> 채점은 아래 10개다. 시작 상태는 전부 빨강, STEP 을 채우며 초록으로 바꾼다. (서버:
> 
> localhost:8080
> 
> )

| # | 시나리오 | 요청 | 기대 | 관련 STEP |
| --- | --- | --- | --- | --- |
| 1 | 목록 조회 | GET /api/lectures | 200, JSON 배열 2건 | 1·2·4 |
| 2 | 응답 콘텐츠 타입 | GET /api/lectures | Content-Type: application/json;charset=UTF-8 | 3·4 |
| 3 | 생성 정상 | POST /api/lectures  (json 바디,  Content-Type: application/json  ) | 200,  id=100  부여된 JSON | 2·3·4 |
| 4 | @RequestBody  역직렬화 | 3번의 바디 필드가 객체에 매핑 | name  /  price  정확히 반영 | 3 |
| 5 | 검증 실패(빈 이름) | POST  { "name":"", "price":1 } | 400  {"error":"BAD\_REQUEST","message":"name is required"} | 5 |
| 6 | 검증 실패(음수 가격) | POST  { "name":"x","price":-1 } | 400 BAD\_REQUEST | 5 |
| 7 | Content-Type 누락 버그 | POST  (헤더 없이)  {...} | canRead=false  → 바디 유실 → 400 | 3·5 |
| 8 | 매핑 없음 | GET /api/unknown | 404 | 1 |
| 9 | (심화) 메서드 불일치 | DELETE /api/lectures | 405 (기본 구현은 404) | 1-심화 |
| 10 | (심화) 커스텀 파라미터 | GET /api/lectures?page=2 | @RequestParam  이 채워짐 | 6 |

### 🧩 통합 — 다 채우고 나면 보이는 것

요청 ─▶ DispatcherServlet.doDispatch (주어진 뼈대: 배분만) ├─ STEP1 HandlerMapping (URL+METHOD) → HandlerMethod ├─ STEP2 ArgumentResolver 파라미터 first-match 채우기 │ └─ STEP3 Jackson.read (@RequestBody) ├─ controller.invoke ← 순수 비즈니스 (수정 금지) └─ STEP4 ReturnValueHandler → STEP3 Jackson.write 객체 → JSON 예외 ─▶ STEP5 ExceptionResolver → @ControllerAdvice → JSON 에러 확장 ─▶ STEP6 커스텀 리졸버 / STEP7 인터셉터 "리스트에 추가"만

뼈대는 안 고치고 전략만 채웠는데 요청→응답이 완성된다 — 이게 프론트 컨트롤러 + 전략 패턴의 힘.

다섯 전략이 전부 동일 패턴이었음을 되돌아보라(리스트를

supports()

/

canRead

/

canWrite

로 훑어 first-match). 하나를 이해하면 스프링 MVC 확장 지점 전체가 열린다.

심화(STEP6/7)에서 컨트롤러를 한 줄도 안 고치고 기능을 붙였다면, "왜 스프링은 확장에 강한가"를 코드로 답할 수 있게 된 것이다.