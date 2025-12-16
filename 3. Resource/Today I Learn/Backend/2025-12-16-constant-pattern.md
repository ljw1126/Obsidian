# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 16일`

## 카테고리
`#CSharp`, `#constant pattern`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- if 조건문에 `is true`, `is false`와 같은 표기가 있었음
- 자바/스프링 개발자로서 이러한 문법은 지원하지 않았기 때문에 평소하던데로 조건문의 논리연산자를 읽기 쉽게 개선
- 이후 피드백 받았을때 `C# .NET` 개발을 할때 팀 컨벤션이라 하여 무엇인지 찾아보게 됨

### 2. 핵심 내용 / 개념 정리

**Constant Pattern이란?**
- `switch` 나 `is` 패턴 매칭에서 **값을 상수와 직접 비교하는 패턴**
- 즉, `'값 자체가 특정 상수인가?'`를 패턴으로 표현하는 방식이다

```cs
x is 0
x switch { 1 => ..., 2 => ... }
```


**도입 시기**
- **C# 7.0 (2017)** 에서 처음 도입
- 이후 C# 8 ~ 12에서 
	- switch expression
	- relational pattern
	- logical pattern (`and`, `or`) 
		- 과 결합되며 핵심 문법으로 성장 

> Java에는 이 문법이 없다 
- C#은 분기를 **statement** ❌아닌 **expression** ✅으로 봄 

```cs
if (x is null) { } // null 패턴
if (x is int i) { } // type 패턴
if (x is > 0) { } // relational 패턴
```


ex. **enum + constant pattern(실무 유용)** 
```cs
var state = order.Status switch
{
    OrderStatus.Pending => "대기",
    OrderStatus.Paid    => "결제 완료",
    OrderStatus.Canceled => "취소",
    _ => throw new ArgumentOutOfRangeException()
};
```


>[!note]  정리
>Constant pattern은 "값 비교를 패턴의 한 종류로 승격시킨 C#의 분기 문법"이다

**참고.** 
- https://learn.microsoft.com/ko-kr/dotnet/csharp/language-reference/operators/patterns