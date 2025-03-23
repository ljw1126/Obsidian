
### 단위 테스트와 통합 테스트 차이

|         | 단위 테스트                | 통합 테스트                          |
| ------- | --------------------- | ------------------------------- |
| 목적      | 개별 모듈 동작 확인           | 여러 모듈을 연동했을 때 동작 확인             |
| 범위      | 특정 메서드나 클래스 정도의 작은 단위 | 서비스, 데이터베이스, 외부 API 등 다양한 모듈 조합 |
| 의존성     | 테스트 더블을 주로 사용         | 실제 시스템과 직접 상호 작용                |
| 환경      | 독립적이고 가벼운 환경          | 테스트 대상의 실제 의존성이 포함된 환경          |
| 속도      | 빠름(메모리 내에서 실행)        | 상대적으로 느림(네트워크, 디스크 I/O 포함)      |
| 테스트 용이성 | 문제 원인은 모두 로컬 코드로 한정   | 문제 원인이 여러 모듈간 연동 범위로 확대         |
| 방법      | Spock                 | Spock with Spring Boot Test     |

### 테스트 더블

| 유형    | 설명                                                                                           |
| ----- | -------------------------------------------------------------------------------------------- |
| Dummy | 테스트 실행을 위해 최소한의 인터페이스만 갖춘 객체, 별다른 동작 없이 기본값(0, null, false 등)만 반환                            |
| Stub  | 특정 입력에 대해 미리 정의된 값을 반환하는 객체, 테스트 환경에서 의존성을 제거하는 데 사용                                         |
| Mock  | 예상된 메서드 호출이 이루어졌는지 검증할 수 있는 객체, 테스트 수행 후 호출 여부와 인자를 검증하는 데 활용                                |
| Spy   | 기존 객체의 동작을 유지하면서, 특정 메서드 호출을 감시하거나 변경할 수 있는 객체, 테스트를 위해 일부 메서드의 동작을 오버라이드 할 수 있다             |
| Fake  | 실제 객체의 축소된 버전, 간단한 로직을 포함하여 테스트 환경에서 일부 기능을 대체하는 객체, 보통 인터페이스를 구현하거나 베이스 객체를 상속하여 페이크 객체를 작성 |

>[!note] dummy != fake , fake가 간단하게 기능을 구현하거에 가깝고, dummy는 의미없는 기본값만 반환


### Groovy
- `groovy4` 부터 `자바17` 지원

![[스크린샷 2025-03-23 오전 10.44.40.png]]

### Groovy Console
- 인텔리제이 기본 지원
- `[Tools > Groovy Console]` 생성

```text
plugins {
	id 'java'
	id 'groovy'
}

dependencies {
	implementation 'org.apache.groovy:groovy:4.0.24' // 추가
	//..
}
```
- `groovy` 플러그인을 추가할 경우
	- gradle 창에서 `groovydoc`과 `compileGroovy` 생성됨

```groovy
println GroovySystem.version // 콘솔에 버전 출력됨
```

```groovy
record People(String name, int age){}
People p = new People('tester', 20)
println(p)
```
- `groovy4` 버전 부터 실행됨
- 패키지에 `record` 클래스를 만들고 import 해서 사용할 경우 
	- `build`가 되어 있어야 import 인식이 올바르게 동작한다

### Groovy 기본 문법
- [Apache Groovy](https://groovy-lang.org/structure.html)


**println**
```groovy
System.out.println("hello");

println("hello")
println "hello"
println 'hello'
```
- 파라미터가 하나일때 괄호 생략 가능
- `groovy`에서 세미콜론(`;`) 생략 가능


**Array**
```groovy
int[] arr = new int[] {1, 2, 3};

def gArr = {1, 2, 3} as int[]

println arr
println gArr
println arr.equals(gArr) // true
println arr == gArr      // true , equals 호출함
println arr.is(gArr)     // false
```


**List**
```groovy
List<Integer> jList = List.of(1, 2, 3);
var jList2 = List.of(1, 2, 3);
List<Integer> gList = [1, 2, 3]
def gList2 = [1, 2, 3]

println jList
println jList2
println gList
println gList2
println gList.getClass()   // class java.util.ArrayList
println gList2.getClass()  // class java.util.ArrayList

// 데이터 꺼내기 
println jList.get(0)
println gList[0]
println gList[-1]    // 3
println gList[-3]    // 1
```
- java의 `var`는 정적 타입 추론 
- groovy의 `def`는 동적 타입 추론


**Map**
```groovy
// java
Map<String, Object> jMpa = Map.of("name", "pro", "age", 20)
println jMap.get("name")
println jMap.get("age")

// groovy에서 Map 생성하는 다양한 방법
def gMap = [name: 'pro', age: 20]
println gMap.get('name')
println gMap.get('age')
gMap.getClass() // LinkedHashMap 기본

def cMap = new ConcurrentHashMap([name: 'pro', age: 20])
println cMap.getClass()

def cMap2 = [name: 'pro', age: 20] as ConcurrentHashMap
println cMap2.getClass()

```
- groovy에서는 `LinkedHashMap`을 기본 생성


**String format**
```groovy 
// java
String world = "world";
println String.format("hello + %s", world)

// groovy
println "hello + $world"
println "hello + ${world + '!'}"
```
- 표현식이 없다면 중괄호 생략 가능


**range**
```groovy
IntStream.rangeClosed(1, 10).forEach(v -> System.out.println(v))
println()
(1..10).each { print it}
println()
(1..10).each { v -> print v} // 12345678910
println()
```
it이라는 클러저 ? 


**if 조건문**
```groovy
def value = 20
if(value in 10..30) {
	println "$value is between 10 ~ 30"
}
```


**for 반복문**
```groovy
// java
for(int i = 0; i < 5; i++) {
	println "idx : " + i
}
println()

//groovy
for(int i in 0..5) {
	println "idx : " + i 
}

for(def i in 0..<5) {
	println "idx : " + i     // 4까지
}

for(i in 0..5) { // int, def 생략 가능
	println "idx : " + i
}
```


**for-each문**
```groovy
// java
var jList = Arryas.asList(1, 2, 3)
for(Integer a : jList) {
	println a
}

// groovy
for(a : jList) {
	println a
}

for(def a : [1, 2, 3]) {
	println a
}
```


**switch 문**
```groovy
// java 
int value = 12
var jResult = switch(value) {
	case 10, 11, 12, 13, 14 -> {
		println value          //12
		yield value * 2
	}
	default -> 0
}
println jResult //24


// groovy
def gResult = switch(value) {
	case 10..14 -> {
		println value          //12
		value * 2
	}
	default -> 0
}
println gResult //24


def res = 'ok'
switch(res) {
	case ~/ok|yes|sure/ -> { // 정규식 사용 가능
		println 'positive'
	}
	default -> 'none'
}
```


**boolean형 결과 비교**
```groovy
// java
"".isEmpty() ? println('empty') : println('non empty') // empty

var list = Collections.emptyList()
list.isEmpty() ? println('empty') : println('non empty') // empty

var obj = null
obj == null ? println('null') : println('non null') // null

class Person {
	String name;
}
Person person = null
println person != null ? person.name : 'invalid'  // invalid


//groovy
"" ? println('non empty') : println('empty') // empty
list ? println('non empty') : println('empty') // empty
obj ? println('none null') : println('null') // null
println person?.name ?: 'invalid'  // invalid
println person?.name // null
```
- `null, empty, ''`은 **false** 처리 된다


**덧셈**
```groovy
println 5 + 2

println 5.plus(2)
```


**람다, 클러저**
```groovy 
// java
BiFunction<Long, Long, Long> jSum = (Long a, Long b) -> a + b;
println jSum.apply(5, 5); // 10

// groovy 
def gSum = { a, b -> a + b}
println gSum(5, 5)
```
- 함수형 인터페이스가 필요없이 좀 더 간결하게 표현가능


**함수 합성**
```groovy
// java
Function<Long, Long> addFive = a -> a + 5;
Function<Long, Long> mulByTwo = a -> a * 2;
FUnction<Long, Long> combined = addFive.andThen(mulByTwo)
println combined.apply(2) // 14

// groovy
def addFive = { a -> a + 5 }
def mulByTwo = { a -> a * 2 }
def combined = addFive >> mulByTwo
println combined(2) // 14
```


**커링(부분함수)**
```groovy
// java 
Function<Long, Function<Long, Long>> jSumForCurry = a -> (Long b) -> a + b
Function<Long, Long> jAddFive = jSumForCurry(5)
println jAddFive.apply(5) // 10


// groovy
def gSumAB = {a, b -> a + b}
def gSumB = gSumAB.curry(5)
println gSumB(5) // 10
```
- 자바는 커링 지원하지 않아서 함수형 인터페이스로 구현 가능
- groovy는 언어상 지원