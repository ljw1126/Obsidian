# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 27일`

## 카테고리
`#csharp`, `#dotnet`, `#LINQ`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- 실무에서 LINQ를 활용하는 코드가 자주 보이기 시작하고 있다 

### 2. 핵심 내용 / 개념 정리

**📌 공식 문서**
- https://learn.microsoft.com/en-us/dotnet/csharp/linq/
- https://learn.microsoft.com/ko-kr/dotnet/csharp/linq/get-started/introduction-to-linq-queries

---

**📌 Notebook LM 요약 정리**
- 문제 - 데이터 소스마다 다른 언어를 써야 하는 번거로움이 있다. 
- `쿼리가 언어의 정식 구성원으로 인정 받았다 (일급 시민)`
- 나중에 이런 질문을 해야 하는 구나 라고 실행 스케쥴만 잡는다 
	- `지연 실행`
		- `쿼리의 결과가 요구되는 순간에 실행이 된다`
		- 결과를 눈으로 확인하기 전까지 하나도 실행하지 않겠다! (현명한 게으름)
- 개발자는 최적화에 대한 고민을 안하고 비즈니스에 집중 가능하다
- LINQ는 성능 차이 없는데 두 가지 방식을 지원한다 (메서드 구문과 쿼리 구문)
	- 모든 기능을 SQL 같은 문장으로만 처리하기 어렵다
	- 쿼리 구문으로 선택하고 메서드 구문으로 집계한다
	- 실제 코드에서는 상황에 맞게 간결한 도구를 선택할 수 있게 해준다
	- `특정 방식만 고집하지 않은 실용적인 문법`
- 프로그램내 인메모리 data : `IEnumerable<T>` - 컴파일시 C# 지시서(Delegate) 생성
	- 이 데이터로 이렇게 정렬, 집계 해주세요 라고 지시서를 생성함
- 프로그램 외부 데이터 : `IQueryable<T>`
	- DB 서버는 SQL만 알아 듣는다 
	- C#으로된 레시피를 보내 봐야 알 수 없다 
	- 쿼리의 의도만 뽑아내서 데이터 구조로 만든다
		- 의도만 담겨 있어 특정 언어에 묶여 있지 않다.
	- 번역을 누가 해준다.
		- IQueryable LINQ Provider는 여러가지가 있다
- IQueryable LINQ Provider에 3가지 단계로 복잡도를 다룰 수 있다.
	- 마지막 복잡도가 제일 높은거를 처리하는 공급자는 EF Core provider가 있다
		- LINQ -> SQL로 번역을 해준다

> LINQ는 데이터 번역 프레임워크라고 할 수 있다.
- 동일 언어로 데이터 소스 다룰 수 있다.
- 지연 전략으로 불필요한 연산 막아 최적화
- IEnumrable, IQueryable 통해 인메모리 데이터든 원격 데이터든 가리지 않고 대화 가능한 번역 지원

---

> Language-Integrated Query (LINQ)
> LINQ는 “쿼리 언어”가 아니라  “데이터 변환 파이프라인을 표현하는 C# 언어 기능”이다.


**1️⃣ “같은 LINQ 문법으로 모든 데이터 소스를 다룬다”**
>You use the same query expression patterns to query and transform data from any type of data source
- LINQ는 
	- 컬렉션, xml, db, 메모리 객체를 같은 모양의 문법으로 다룰 수 있게 해준다
	- LINQ는 `“데이터 소스 추상화 계층”`이다.


**2️⃣ “하나의 쿼리가 여러 데이터 변환을 할 수 있다”**
> a single query can retrieve data from a SQL database and produce an XML stream as output
- LINQ는 
	- 단순 조회가 아니라 변환(transform)도 포함한다
	- 하나의 쿼리 파이프라인으로 데이터 조회/객체 맵핑 가능
		- "조회 + 가공 + 변환"을 하나의 흐름으로 표현


**3️⃣ “C# 문법을 그대로 쓴다 (SQL 흉내가 아님)”**
> Query expressions use many familiar C# language constructs

> LINQ는 DSL이 아니라 “C# 내부 문법”이다.


**4️⃣ “LINQ 변수는 모두 강타입이다”**
> The variables in a query expression are all strongly typed

```csharp
var q = from e in elements
        select e;
```
- `e`는 `object` ❌
- **정확한 타입 `Element`** ⭕️
- IDE에서 전부 지원됨
	- 자동완성
	- 컴파일 타임 체크
	- 리팩터링 안정성
- Java Stream과 동일하지만, **C#은 익명 타입도 강타입**


**5️⃣ “LINQ는 실행되지 않는다 (지연 실행)”**
> A query isn't executed until you iterate over the query variable

```csharp
// 이 시점에 실행도 안되고, 데이터도 안 읽는다
var query = from e in elements
            where e.Id > 10
            select e;
            
// 실행되는 순간 (예시)
foreach (var e in query) { }
query.ToList();
query.Count();
```
- Java Stream의 **lazy execution**과 동일한 개념


**6️⃣ “query syntax와 method syntax는 컴파일 타임에 같다”**
> the compiler converts query expressions to standard query operator method calls

- 아래 두 코드는 완전히 **동일**하다
- 성능, 의미 차이 ❌
```csharp
// Query syntax
var q =
    from e in elements
    where e.Age > 20
    select e.Name;
    
// Method syntax
var q = elements
    .Where(e => e.Age > 20)
    .Select(e => e.Name);
```

> 취향 + 가독성의 차이다


**7️⃣ “모든 연산이 query syntax로 되는 건 아니다”**

> Some query operations, such as Count or Max, have no equivalent query expression clause

```csharp
// 이건 안 됩니다 ❌
from e in elements
count e

// 이건 됨 ⭕️
var count =
    (from e in elements
     where e.Age > 20)
    .Count();
```
-  **query syntax + method syntax는 섞어서 사용 가능**
	- query syntax는 `“필터/변환”`
	- method syntax는 `“집계/종결”`


**8️⃣ “IEnumerable vs IQueryable의 결정적 차이”**

> compiled to expression trees or to delegates
- 이 문장이 `LINQ 전체의 핵심`이다

`IEnumerable<T>`
- 대상: List, Array
- 컴파일 결과: **delegate**
- 실행 위치: **메모리(C# 코드)**

```csharp
Func<T, bool>
```

`IQueryable<T>`
- 대상: EF Core, DB
- 컴파일 결과: **expression tree**
- 실행 위치: **외부 시스템(DB)**
```csharp
Expression<Func<T, bool>>
```

📌 이 차이 때문에:
- LINQ to Objects ≠ LINQ to Entities
- 지원되는 메서드가 다름
- DB에서 실행 불가능한 LINQ도 존재