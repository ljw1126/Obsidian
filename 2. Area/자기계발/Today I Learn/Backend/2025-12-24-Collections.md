# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 24일`

## 카테고리
`#CSharp`, `#Collection Framework`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- C# Collection Framework의 특징과 API에 대해 학습한다

### 2. 핵심 내용 / 개념 정리

📌 [공식 문서 - collections](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/collections)
📌 [Notebook LM](https://notebooklm.google.com/notebook/9374b9b0-6d52-434a-b8f7-b4399cbd22c5)


컬렉션은 객체 그룹을 유연하게 다루게 해준다 
- 1. 요소 접근 방식 (Element Access)
	- 모든 컬렉션은 기본적으로 각 요소에 순서대로 접근할 수 있도록 **열거(Enumeration)** 가 가능합니다. 하지만 특정 요소에 직접 접근하는 방식은 컬렉션 타입에 따라 다음과 같이 나뉩니다.
	- **인덱스(Index) 기반 접근**
		- `System.Collections.Generic.List<T>`와 같이 정렬된 컬렉션에서 **요소의 위치(번호)** 를 통해 접근하는 방식입니다. 인덱스는 **0**부터 시작하며, 이는 해당 요소 앞에 있는 요소의 개수를 의미합니다
	- **키(Key) 기반 접근**
		- `System.Collections.Generic.Dictionary<TKey, TValue>`와 같이 **특정 값(Value)을 고유한 키(Key)와 연결**하여 접근하는 방식입니다. 사용자는 인덱스 대신 키를 사용하여 원하는 항목을 빠르게 찾을 수 있습니다
- 2. 성능 프로필 (Performance Profile)
	- 모든 컬렉션은 수행하는 작업에 따라 서로 다른 성능 특성을 보입니다.
	- 컬렉션의 성능은 요소를 **추가, 검색, 또는 삭제**하는 동작에서 차이가 납니다
	- 작업별 특성 예시 
		- `List<T>`에서 인덱스를 통해 요소를 삭제(`RemoveAt`)하면, 삭제된 요소 뒤에 있는 모든 요소의 인덱스 값이 낮아지도록 조정되는 과정이 발생합니다
		- `Dictionary`는 키를 사용하여 특정 항목을 매우 빠르게 찾을 수 있는 `ContainsKey`, `TryGetValue` 등의 메서드를 제공합니다
- 3. 동적 확장과 축소 (Grow and shrink dynamically)
	- 대부분의 컬렉션은 요소를 동적으로 추가하거나 제거할 수 있지만, `Array`, `Span<T>`, `Memory<T>`와 같은 타입은 크기가 고정되어 있어 동적으로 늘어나거나 줄어들지 않는다는 성능적·구조적 차이가 있습니다

이 외에도 .NET은 다중 스레드 환경에서 안전한 접근을 제공하는 컬렉션이나, 요소의 수정을 방지하는 특수한 컬렉션 등을 통해 다양한 성능 및 보안 요구사항을 충족합니다


**📌 참고** 
- [.NET API Browser](https://learn.microsoft.com/en-us/dotnet/api/?term=collection)
- [Commonly used collection types](https://learn.microsoft.com/en-us/dotnet/standard/collections/commonly-used-collection-types)
- [Selecting a Collection Class](https://learn.microsoft.com/en-us/dotnet/standard/collections/selecting-a-collection-class)


> [!info]
> Beginning with C# 12, all of the collection types can be initialized using a [Collection expression](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/collection-expressions)
- Collection expression이 표현식 나타낸다
	- `{}`, `[]` 사용하여 초기화가 가능하다는 걸 말함 


```cs
[Fact]
public void RemoveAtOddNumberTest()
{
	// Arrange
	List<int> numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

	// Act
	for (var index = numbers.Count - 1; index >= 0; index--)
	{
		if (numbers[index] % 2 == 1)
		{
			// Remove the element by specifying
			// the zero-based index in the list.
			numbers.RemoveAt(index);
		}
	}

	// Assert
	numbers.ShouldBe(new[] {0, 2, 4, 6, 8});
}

[Fact]
public void RemoveAllOddNumberTest()
{
	// Arrange
	List<int> numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

	// Act
	numbers.RemoveAll(i => i % 2 == 1);

	// Assert
	numbers.ShouldBe(new[] {0, 2, 4, 6, 8});
}
```
- 컬렉션의 요소를 삭제할때 내부적으로 shifting이 발생하기 때문에 인덱스 마지막 부터 내림차순으로 삭제 처리한다. 
- 이때 RemoveAll은 “뒤에서부터 지우는 패턴”을  프레임워크가 안전하고 빠르게 캡슐화한 메서드다.

```cs
// 개념적 의사코드 RemoveAll
int write = 0;

for (int read = 0; read < Count; read++)
{
    if (!match(items[read]))
    {
        items[write++] = items[read];
    }
}

ClearRange(write, Count - write);
Count = write;
```
- RemoveAll은 shifting이 아니라 포인터를 이용한 **압축(compaction)** 이라 안전하다


```cs
[Fact]
public void KeyValuePairTest()
{
	// Arrange
	Dictionary<string, Element> elements = new ()
	{
		{ "K", new () { Symbol = "K", Name = "Potassium", AtomicNumber = 19}},
		{"Ca", new (){ Symbol = "Ca", Name = "Calcium", AtomicNumber = 20}},
		{"Sc", new (){ Symbol = "Sc", Name = "Scandium", AtomicNumber = 21}},
		{"Ti", new (){ Symbol = "Ti", Name = "Titanium", AtomicNumber = 22}}    
	};

	// Act & Assert
	elements.Count.ShouldBe(4);
	elements.ShouldContainKey("K");

	elements["K"].Name.ShouldBe("Potassium");
	elements["K"].AtomicNumber.ShouldBe(19);

	elements.TryGetValue("K", out var value).ShouldBeTrue();
	value.Symbol.ShouldBe("K");

	elements.TryGetValue("Na", out _).ShouldBeFalse();

	Should.Throw<KeyNotFoundException>(() => { var _ = elements["Na"]; });

	elements.Keys.ShouldBe(new[] { "K", "Ca", "Sc", "Ti" }); // 주의: 순서는 보장 X
	elements.Values.ShouldContain(e => e.Name == "Titanium");

	elements["K"].AtomicNumber.ShouldBe(19);
	elements["K"] = new Element { Symbol="K", Name="Potassium", AtomicNumber=999 };
	elements["K"].AtomicNumber.ShouldBe(999);

	elements.Remove("Sc").ShouldBeTrue();
	elements.ShouldNotContainKey("Sc");
}
```

📌 **Java처럼 “Map 인터페이스 + 구현체 교체”가 C#에도 있다**
- **`IDictionary<TKey,TValue>`** (Map에 해당)
- 구현체로
    - `Dictionary<TKey,TValue>` (기본 해시 맵)
    - `SortedDictionary<TKey,TValue>` (키 정렬, TreeMap 느낌)
    - `SortedList<TKey,TValue>` (정렬 + 배열 기반)
    - `ConcurrentDictionary<TKey,TValue>` (동시성)

```cs
IDictionary<string, Element> elements = new Dictionary<string, Element>();
// 또는
IDictionary<string, Element> elements = new SortedDictionary<string, Element>();
```


📌 `new () { ... }` 문법 명칭
- 두 가지의 문법이 결합된 형태
- **(1) Object Initializer (객체 이니셜라이저)**
	- C# 3.0 부터 존재 (Java에는 없음)
	- 생성자 호출 없이 프로퍼티를 초기화

```cs
new Element { Name = "Potassium", AtomicNumber = 19 }
```

- **(2) Target-Typed `new` (타깃-타입 추론 new)**
	- C# 9.0 도입
	- 좌변 타입(`Element`)으로 `new Element()`를 추론

```cs
Element e = new() { Name = "Potassium" };
```

- 그래서 전체 이름은 
	- Target-typed `new` expression + Object Initializer
	- 보통은` target-typed new` 또는 `객체 이니셜라이저(new() {})` 라고 부름
- 장점 
	- 타입 중복 제거 
	- 가독성 향상 
	- 테스트 코드/초기화 코드에서 특히 깔끔
		- 의미만 남기고 보일러플레이트 코드 제거가 목적이다
- 주의할 점 
	- 타입 문맥을 추론할 수 없으면 사용 불가 
	- `var x = new() { Name = "X" }; // ❌ 컴파일 에러`
- 결론 
	- `new() { ... }`는  **“타입 추론 기반 객체 이니셜라이저(target-typed new)”** 이다.


**iterator**, **yeild return**
- 모든 짝수를 연산해서 미리 메모리에 올리는게 아님 
- 필요한 요청이 올때마다 즉석에서 하나 응답 (상태 기억하고 멈춤)
	- 필요할때마다 항목을 만드는 설계도, 레시피 같은 것
- yeild return이 나오면서 대용량 컬렉션을 다룰 때 매우 효율적으로 다룰 수 있게 되었다.
	- ex. 1~ 10억 사이 소수를 구한다
	- 데이터 소유 관점에서 **데이터 생성**으로 관점 전환 

// 예제 생략

**LINQ**는 기본이 지연 실행이다
- C# 으로 데이터 다루는 근본적인 패러다임의 변화를 가져옴 
- for문과 같은 명령형(how) 방식에서 무엇을 원하는지 선언형(what) 방식으로 
- 장점 
	- 가독성 향상, 비즈니스 로직에 집중 가능해진다
	- yeild 처럼 지연 평가로 동작한다
```cs
[Fact]
public void LinqCollectionTest()
{
	// Arrange
	List<Element> elements = BuildList();

	// Act
	var subset = from theElement in elements
		where theElement.AtomicNumber < 22
		orderby theElement.Name
		select theElement;
	var actual = subset.ToList();

	// Assert
	actual.Count.ShouldBe(3);
  
	actual.All(e => e.AtomicNumber < 22).ShouldBeTrue();
  
	actual.Select(e => e.Name)
		.ShouldBe(new [] { "Calcium", "Potassium", "Scandium" });
	
	// 오름차순인지 검증
	actual.Select(e => e.Name)
		.SequenceEqual(actual.Select(e => e.Name).Order()) 
		.ShouldBeTrue();             

	// 순서 중요
	actual.Select(e => e.Symbol)
		.ShouldBe(new[] { "Ca", "K", "Sc" });    
}

private static List<Element> BuildList() => new()
{
	{ new(){ Symbol="K", Name="Potassium", AtomicNumber=19}},
	{ new(){ Symbol="Ca", Name="Calcium", AtomicNumber=20}},
	{ new(){ Symbol="Sc", Name="Scandium", AtomicNumber=21}},
	{ new(){ Symbol="Ti", Name="Titanium", AtomicNumber=22}}
};
```