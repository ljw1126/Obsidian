# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 18일`

## 카테고리
`#CSharp`, `#const`, `#readonly`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- `const`, `readonly`의 차이에 대한 질문을 받았고, 사용하면서 느낀 것과 찾아본 내용 기록

### 2. 핵심 내용 / 개념 정리

**Reference.**
- https://learn.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/readonly
- https://learn.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/const


>[!hint]
>`const`와 `readonly`의 차이는 재할당 가능 여부 보다 


| 구분        | `const`       | `readonly`     |
| --------- | ------------- | -------------- |
| 값 결정 시점   | **컴파일 타임**    | **런타임**        |
| 메모리 위치    | IL에 값이 직접 삽입  | 객체/타입 필드       |
| static 여부 | **항상 static** | 인스턴스 or static |
| 할당 시점     | 선언 시만 가능      | 선언 시 또는 생성자    |
| 참조 타입     | ❌ (string 제외) | ✅              |
| 값 변경      | 절대 불가         | 생성자까지만 가능      |
| 상속 시      | 값이 **고정 복사**  | 자식도 동일 필드 참조   |

**const 가능** 
- 숫자 타입 
- `bool`
- `char`
- `string`
- `enum`

**const 불가능**
```cs
const DateTime Now = DateTime.Now; // ❌
const object Obj = new();         // ❌
```
- 이유 : 컴파일 타임에 값을 확정할 수 없기 때문


✅ readonly는 "불변 객체"와 잘 맞는다 
```cs
public class User
{
    public readonly Guid Id;

    public User()
    {
        Id = Guid.NewGuid();
    }
}
```
- 생성 시점에만 값 결정 (생성자 기준)
- 이후 절대 변경이 불가하다
	- **DDD / 불변 설계 / 테스트 친화성**에 매우 중요


✅ static readonly의 진짜 사용처
```cs
public static readonly TimeSpan DefaultTimeout =
    TimeSpan.FromSeconds(30);
```
- `const`를 못 쓰는 이유
	- `TimeSpan.FromSeconds()` ➡️ 런타임 계산 
	- 참조 타입/struct 초기화 필요


✅ JS의 `const`와 C#의 `const`는 다르다 (중요)
- `JS const ≠ C# const`
- C#의 "readonly reference"에 더 가깝다

| JavaScript  | C#          |
| ----------- | ----------- |
| 재할당 금지      | 컴파일 타임 상수   |
| 객체 내부 변경 가능 | 객체 자체 생성 불가 |
| 런타임 개념      | 컴파일 타임 개념   |


✅ 실무에서 선택 기준
- `const`를 사용해도 되는 경우 
	- private
	- 값이 절대 안 바뀜 
	- primitive, enum, string
	- **같은 어셈블리 내부에서만 사용**
- `readonly`를 사용해도 되는 경우 
	- public / protected
	- 라이브러리 API
	- 생성자에서 결정되는 값 
	- 참조 타입 / struct


> [!info] 요약
> “const는 컴파일 타임 상수라서 호출부에 값이 인라인되고, readonly는 런타임 상수라서 필드를 참조합니다.
그래서 public API에는 보통 readonly를 씁니다.”

---

**추가 . 🤔 어셈블리에서 const를 조심해야 하는 이유**

const는 컴파일 타임의 상수이다
```cs
// ProjectA
public const int Timeout = 30;

// ProjectB
var t = Config.Timeout; // 컴파일 시점에 30이 복사됨
```

이후 ProjectA만 상수 값을 수정하고, ProjectB를 재컴파일 하지 않을 경우
```cs
// ProjectA
public const int Timeout = 60;

// ProjectB를 재컴파일 하지 않으면 여전히 30을 사용
var t = Config.Timeout; 
```
- 어셈블리 경계에서 const는 값이 `"복사"` 되기 때문에 이러한 현상이 발생


반면 readonly의 경우 런타임에 필드를 참조하기 때문에 ProjectA만 다시 배포해도 값 변경이 반영됨 
- 메모리를 참조하니깐 

```cs
// ProjectA
public static readonly int Timeout = 30;

// ProjectB
var t = Config.Timeout;
```