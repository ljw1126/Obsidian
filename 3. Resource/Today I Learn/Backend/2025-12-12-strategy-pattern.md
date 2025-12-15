# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 12일`

## 카테고리
`#CSharp`, `#xUnit`, `#strategy pattern`, `#design pattern`, `전략 패턴`, `delegate`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- 외부 인프라 호출시 재시도 로직이 있었음 (아래 코드 참조)
- Retry Delay 시간이 하드 코딩으로 설정되어 있다보니, 단위 테스트 기반으로 검증시 지수함수적으로 실행 시간이 늘어남 
	- 전체 테스트 실행 시간 : `46s`
- 외부에서 제어 할 수 있도록 생성자 주입 받아 사용하도록 리팩터링 진행

```cs
public class CctvAnalysisRepo(
    ICosmosDbClientFactory factory)
    : ICctvAnalysisRepo
{
    // .. 

    public async Task UpsertDataAsync(bool isForService, JObject data)
    {
        var sk = data["sk"].ToString();

        const int maxRetries = 3;
        var retryCount = 0;

        while (true)
        {
            try
            {
                if (isForService)
                {
                    await _cctvAnalysisClient.UpdateAsync(sk, data);
                }
                else
                {
                    await _cctvAnalysisTestClient.UpdateAsync(sk, data);
                }

                return;
            }
            catch (CosmosException e) when (e.StatusCode == HttpStatusCode.RequestTimeout)
            {
                if (retryCount >= maxRetries)
                {
                    throw;
                }

                await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, retryCount + 1)));
                retryCount++;
            }
        }
    }
}
```

### 2. 핵심 내용 / 개념 정리
- 처음, Java의 Functional Interface를 생각하여 arrow function 형태로 테스트에서 쉽게 구현 및 할당 할 수 있을거라 생각함.
- 하지만 테스트 작성이 생성자에 arrow function 형태로 선언하는게 어렵다는 것을 확인
	- 결국 임의 클래스를 생성해 주입을 해야 함
	- 재활용성이 떨어진다고 생각
- 생성자에서 arrow function 형태로 선언하기 위해서는 C#의 delegate를 활용해야 한다는 것을 확인함
- 외부에서 Retry Delay 설정 가능해짐으로써 
	- 전체 테스트 실행 시간이 **46s 👉 5.6s로 개선**

**delegate 선언**
```cs
namespace CCTVAnalysis.RepositoryImplements.RetryDelay;

public delegate TimeSpan RetryDelayStrategy(int retryCount);
```

**재시도 딜레이 전략 구현**
```cs
namespace CCTVAnalysis.RepositoryImplements.RetryDelay;

/// <summary>
/// 재사용 가능한 미리 정의된 재시도 지연 전략 모음
/// </summary>
public static class RetryDelayStrategies
{
    public static TimeSpan ExponentialBackOff(int retryCount) => TimeSpan.FromSeconds(Math.Pow(2, retryCount + 1));
    public static TimeSpan FixedTwoMinutes(int _) => TimeSpan.FromMinutes(2);
}
```

`Program.cs`에서 스코프 설정시 사용
```cs
    _ = services.AddScoped<ICctvAnalysisRepository>(sp =>
    {
        var factory = sp.GetRequiredService<ICosmosDbClientFactory>();
        return new CctvAnalysisRepository(factory.GetClient(CosmosDatabaseNameCode.SHIP_DB, CosmosContainerNameCode.CCTV_ANALYSIS), RetryDelayStrategies.ExponentialBackOff);
    });
```

**단위 테스트 통해 재시도 로직 검증**
```cs
using Azure;
using CCTVAnalysis.RepositoryImplements.Blob;
using CCTVAnalysis.RepositoryInterfaces.Blob;
using Moq;
using Shouldly;
using Xunit.Abstractions;

namespace CCTVAnalysis.Tests.RepositoryImplements.Blob;
public class CctvImageRepositoryUnitTests
{
    private readonly ITestOutputHelper _output;
    private readonly IImageRepository _sut;
    private readonly Mock<IBlobClientWrapper> _mockBlobClient;
    private readonly Mock<IBlobContainerWrapper> _mockBlobContainer;
    private static TimeSpan immediateRetry(int retryCount) => TimeSpan.Zero;
    public CctvImageRepositoryUnitTests(ITestOutputHelper output)
    {
        _output = output;
        _mockBlobClient = new Mock<IBlobClientWrapper>();
        _mockBlobContainer = new Mock<IBlobContainerWrapper>();
        _sut = new CctvImageRepository(_mockBlobContainer.Object, immediateRetry);
    }


    [Fact(DisplayName = "최대 재시도 횟수 초과 검증")]
    public async Task LoadImageAsyncTest()
    {
        // Arrange
        const int maxRetries = 6;
        _ = _mockBlobClient.Setup(o => o.OpenReadAsync(It.IsAny<CancellationToken>()))
            .ThrowsAsync(CreateRequestFailedException(500));
        _ = _mockBlobContainer.Setup(o => o.GetBlobClient(It.IsAny<string>()))
           .Returns(_mockBlobClient.Object);

        // Act & Assert
        _ = await Should.ThrowAsync<RequestFailedException>(() => _sut.LoadImageAsync("testPath"));

        _mockBlobClient.Verify(o => o.OpenReadAsync(It.IsAny<CancellationToken>()), Times.Exactly(maxRetries));
    }

    [Fact(DisplayName = "Azure.RequestFailedException 외 복구 불가능한 예외(Non-Recoverable Exception) 발생 시 재시도 로직이 건너뛰어지고 즉시 종료")]
    public async Task LoadImageAsyncThrowInvalidOperationExceptionTest()
    {
        // Arrange
        _ = _mockBlobClient.Setup(o => o.OpenReadAsync(It.IsAny<CancellationToken>()))
            .ThrowsAsync(new InvalidOperationException("Image load failed unexpectedly."));
        _ = _mockBlobContainer.Setup(o => o.GetBlobClient(It.IsAny<string>()))
            .Returns(_mockBlobClient.Object);

        // Act & Assert
        _ = await Should.ThrowAsync<InvalidOperationException>(() =>
            _sut.LoadImageAsync("testPath")
        );

        _mockBlobClient.Verify(o => o.OpenReadAsync(It.IsAny<CancellationToken>()), Times.Once());
    }

    [Fact(DisplayName = "blobClient 요청시 500이 아닌 모든 오류는 복구 불가능한 오류로 간주되어 재시도 없이 바로 종료된다")]
    public async Task LoadImageAsyncThrow404StatusCodeTest()
    {
        // Arrange
        _ = _mockBlobClient.Setup(o => o.OpenReadAsync(It.IsAny<CancellationToken>()))
            .ThrowsAsync(CreateRequestFailedException(403)); // Forbidden
        _ = _mockBlobContainer.Setup(o => o.GetBlobClient(It.IsAny<string>()))
          .Returns(_mockBlobClient.Object);

        // Act & Assert
        _ = await Should.ThrowAsync<RequestFailedException>(() => _sut.LoadImageAsync("testPath"));

        _mockBlobClient.Verify(o => o.OpenReadAsync(It.IsAny<CancellationToken>()), Times.Once());
    }

    private RequestFailedException CreateRequestFailedException(int status)
    {
        var failedResponse = new Mock<Azure.Response>();
        _ = failedResponse.SetupGet(r => r.Status).Returns(status);
        return new RequestFailedException(failedResponse.Object);
    }
}

```

---
#### Gemini
📝 C# `delegate` 및 Spring `Functional Interface` 비교 요약
- Java의 Functional Interface가 등장하기 전부터 존재했던 개념
- 메서드를 참조하는 형식의 안전한 포인터 역할을 함
- 현대 C#에서는 `delegate`를 기반으로 `Func<T>`, `Action<T>`와 같은 내장 타입이 제공되어 함수형 프로그래밍 스타일을 지원함

📝 Spring `Functional Interface`와 C# `delegate`의 비교
- Spring/Java의 Function Interface는 Single Abstract Method 원칙을 따르며, C#의 delegate와 **개념적으로 동일한 역할을 수행**함
- 둘다 함수를 **일급 객체(First-Class Citizen)** 로 취급하게 만드는 핵심 요소

| **특징**         | **Java/Spring Functional Interface**              | **C# delegate (주로 Func/Action)**                                     |
| -------------- | ------------------------------------------------- | -------------------------------------------------------------------- |
| **기반 기술**      | **인터페이스** (`@FunctionalInterface` 어노테이션)          | **타입** (명시적 `delegate` 키워드 또는 내장 `Func/Action` 타입)                   |
| **콜백 구현**      | **람다식** (`->` 기호)                                 | **람다식** (`=>` 기호)                                                    |
| **전략 패턴 구현**   | `StrategyInterface`를 정의하고 `Service` 클래스가 이를 주입받음. | `Func<T>` 또는 `Action<T>`를 클래스가 주입받음. **DI에서 더욱 자연스럽게 함수형 전략 주입 가능.** |
| **예시 코드 (전략)** | `BiFunction<Integer, Integer, Integer>`           | `Func<int, int, int>`                                                |


📝 C# `delegate`의 개념과 역할

| **구분**    | **내용**                                                                                      |
| --------- | ------------------------------------------------------------------------------------------- |
| **정의**    | **메서드에 대한 참조(포인터)**를 저장하는 타입 안전한 객체입니다. C++의 함수 포인터와 유사하지만, 타입 안정성과 객체 지향 특성(캡슐화)이 추가되었습니다. |
| **목적**    | **메서드를 다른 메서드의 인자로 전달**하거나, 콜백(Callback) 메커니즘을 구현하여 **실행될 코드를 나중에 결정**할 수 있게 합니다.           |
| **사용 예**  | 이벤트 핸들링, 쓰레드 콜백, **전략 패턴 구현** (실행 로직을 인자로 전달).                                              |
| **핵심 문법** | `public delegate int MathOperation(int x, int y);`                                          |


📝 C#의 과거 한계

|**관점**|**과거 C#의 복잡성 (C# 1.0 ~ 2.0)**|**현대 C# (C# 3.0+ 이후)의 해결책**|
|---|---|---|
|**콜백 정의**|메서드를 전달하려면 **명시적으로 `delegate` 타입을 선언**해야 했음.|`Func<T>` (반환 값 O)와 `Action<T>` (반환 값 X)라는 **범용 델리게이트**가 도입되어 명시적 선언 필요성이 대폭 감소.|
|**코드 간결성**|매번 콜백 메서드를 따로 정의해야 했음.|**람다식(`=>`)** 및 **익명 메서드** 도입으로 인라인 코드 작성이 가능해져 코드가 간결해짐.|
|**전략 패턴 구현**|`delegate`를 필드로 가진 클래스를 정의해야 했음.|**`Func<T>`** 타입을 클래스의 생성자나 메서드 인자로 받아 바로 전략을 주입하는 **함수형 스타일**이 가능해짐.|

**✅ 결론**
현대 C#에서는 `Func<T>`와 람다식을 통해 Spring의 `Functional Interface` 스타일의 **함수형 전략 패턴**을 매우 유사하고 간결하게 구현할 수 있습니다. 과거의 복잡성은 거의 사라졌으며, `delegate`는 여전히 C#의 함수형 프로그래밍의 근간을 이룹니다.