# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 09일`

## 카테고리
`#CSharp`, `#xUnit`, `#MemoryCache`, `#캐시`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- TimerTrigger에서 10분 단위로 sql server 테이블 전체 조회해와 static한 컬렉션에 갱신하는게 비효율적으로 보여 `로컬 캐시` 도입을 고려
- EventGridTrigger에 대한 비즈니스 이해도가 높아지면서, 트래픽이 몰릴때 자체적으로 로컬 캐시 갱신하게 될 경우 `캐시 스탬피드 현상` 발생 가능할 거라고 생각
- TimerTrigger 프로세스에서 캐시 처리를 담당하고, EventGridTrigger에서 조회하는 현 레거시 방식을 유지하기로 결정 
- 로컬 캐시는 유의미 하기 때문에 이번 기회에 학습 테스트를 진행

### 2. 핵심 내용 / 개념 정리

**패키지 설치**
```shell
dotnet add package Microsoft.Extensions.Caching.Memory

// 통합 테스트를 할 경우 WebApplicationFactory 필요
dotnet add package Microsoft.AspNetCore.Mvc.Testing
```

**노트북 LM 요약**
- 서비스 익스텐션으로 등록하게 될 경우 글로벌한 싱글톤 객체로 등록되어 서드 파티 패키지에서도 공유해 사용하게 된다 
	- 이때 임의로 캐시 사이즈를 조정하게 되면, 글로벌 캐시를 사용하는 서드 파티 패키지에서 사이즈 초과시 오류를 발생시킬 수 있다.
	- 고로 캐시 커스터 마이징이 필요한 경우 따로 싱글톤 객체로 등록해 사용할 것을 권장
		- `services.Add*()`로 등록하는 방식이 글로벌 방식
		- 직접 `new MemoryCache()` 생성하여 싱글톤 등록해 사용
- 만료 시간 설정에는 `슬라이딩 만료`와 `절대 만료`가 있다
	- `슬라이딩 만료`
		- 캐시를 조회하게 되면 그때 만료 시간을 갱신하게 된다
			- 슬라이딩 만료를 통해 자주 쓰는 데이터를 보장할 수 있다. 
			- **문제**. DB 데이터는 갱신되었는데, 캐시에 접근해 슬라이딩 만료가 계속 연장됨
				- 그 결과 사용자는 계속 낡은 정보를 보게 된다
				- 서버 재시작할 때까지 갱신되지 않아 장애가 발생하는 경우도 종종 있다
			- **해결책**. `절대 만료`와 `슬라이딩 만료`를 함께 설정한다
	- `절대 만료`
		- 시한부 설정으로 반드시 정해진 시점에 만료가 보장된다
- 기타
	- 1) 외부 사용자 입력을 절대 key로 사용하지 말 것
		- 악의적인 공격을 통해 서버 자원 소비 시킬 수 있는 위험이 있다.
	- 2) 반드시 만료 정책 사용
		- `슬라이딩 만료` 사용시 `절대 만료` 도 함께 사용할 것
	- 3) 캐시 총량 결정할 경우
		- 글로벌한 캐시 객체의 캐시 총량을 제한하면 앱이 죽을 수 있다
			- 서드 파티 패키지에서 글로벌한 싱글톤 캐시 객체를 공유하여 사용하다보니
		- 그래서 커스터마이징이 필요한 경우 전용 캐시를 만들어 싱글톤으로 사용 권장
		- 이때 kb, mb 단위 말고도, 1개(개수), 길이와 같은 유연한 개념으로 저장 가능
	- 4) 그외 캐시가 가득찬 경우 LRU나 여러 방식 지원한다는 듯함


#### 단위 테스트 
- Set, TryGetValue, 슬라이딩 만료, 절대 만료, GetOrCreateAsync의 thread-safe 검증
```cs
using Microsoft.Extensions.Caching.Memory;
using Shouldly;

namespace CCTVAnalysis.Tests.Examples.InMemoryCache;
/*
* services.AddMemoryCache() 통해 활성화하게 될 경우 싱글톤(전역 객체)으로 캐시 객체가 등록되어 서드 파티 패키지에서도 공유해서 사용 
* 이로인해 캐시 사이즈를 설정하게 될 경우 다른 서드 파티에도 영향을 끼쳐, 최악의 경우 서버 다운될 수 있다함
* 고로 커스터 마이징이 필요한 경우, 싱글톤으로 자체적으로 등록해 사용할 것을 공식 권장 
*/
public class MemoryCacheUnitTests
{
    private readonly IMemoryCache _cache;

    public MemoryCacheUnitTests()
    {
        _cache = new MemoryCache(new MemoryCacheOptions());
    }

    [Fact]
    public void SetTest()
    {
        // Arrange
        const string key = "testKey";
        const string expectedValue = "Hello World!";

        // Act
        _cache.Set(key, expectedValue);

        // Assert
        var isSuccess = _cache.TryGetValue(key, out string? actualValue);

        isSuccess.ShouldBeTrue();
        actualValue.ShouldBe(expectedValue);
    }

    [Fact]
    public void AbsoluteExpirationTest()
    {
        // Arrange
        var options = new MemoryCacheEntryOptions()
            .SetAbsoluteExpiration(TimeSpan.FromMicroseconds(200)); // 만료 시간 200ms

        _cache.Set("testKey", "expireAfter1Sec", options);

        // Act
        Thread.Sleep(300); // 300ms
        var isSuccess = _cache.TryGetValue("testKey", out _);

        // Assert
        isSuccess.ShouldBeFalse();
    }

    // NOTE. [이진우]
    // 슬라이딩 만료가 무기한 갱신될 경우, 해당 항목은 "좀비 데이터"처럼 캐시에서 영원히 퇴출되지 않는 문제가 발생 가능함
    // AbsoluteExpriation을 같이 설정하는 것을 공식 권장함
    [Fact(DisplayName = "2초 동안 접근이 없으면 만료된다. 반대로 2초 사이 조회하면 리셋(갱신) 된다", Skip = "슬라이딩 만료 시간 확인용")]
    public void SlidingExpirationTest()
    {
        // Arrange
        const string key = "testSlidingKey";
        const string value = "SlidingCacheValue";

        var options = new MemoryCacheEntryOptions()
            .SetSlidingExpiration(TimeSpan.FromSeconds(2));

        _cache.Set(key, value, options);

        // Act
        Thread.Sleep(1000); // 1초 대기
        _cache.TryGetValue(key, out _); // 접근하여 만료 시간 리셋(갱신)

        Thread.Sleep(1000); // 다시 1초 대기
        var isSuccess = _cache.TryGetValue(key, out _);

        // Assert
        isSuccess.ShouldBeTrue();

        Thread.Sleep(2000); // 2초 더 대기
        _cache.TryGetValue(key, out _).ShouldBeFalse(); // 만료 확인
    }

    // NOTE. 두번째 인자 factory의 경우 키에 해당하는 값이 없을때 동시 요청시 한번만 호출이 된다
    [Fact]
    public async Task GetOrCreateAsyncConcurrencyTest()
    {
        // Arrange
        const string key = "testKeyForStampede";
        var executionCount = 0;

        Func<ICacheEntry, Task<string>> dataFactory = async entry =>
        {
            // 오직 한번만 실행되는지 카운팅
            Interlocked.Increment(ref executionCount);

            // 만료 시간 설정
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromSeconds(5);

            return $"DATA_{executionCount}";
        };

        // Act
        // 100개 동시 요청 시뮬레이션
        var tasks = new List<Task<string>>();
        for (int i = 1; i <= 100; i++)
        {
            tasks.Add(_cache.GetOrCreateAsync(key, dataFactory)!);
        }

        var results = await Task.WhenAll(tasks);

        // Assert
        executionCount.ShouldBe(1);

        results.Length.ShouldBe(100);
        results.ShouldAllBe(r => r == "DATA_1");
    }
}

```

#### 레거시 TimerTrigger 캐싱 방식의 장단점 (gemini 답변)

네, 기존에 사용하셨던 **TimerTrigger 기반의 백그라운드 갱신 전략**은 동시성 환경에서 캐시 스탬피드를 방지하는 효과적인 방법이지만, 그 자체로 몇 가지 장단점을 가집니다.

---

**🟢 장점 (안정성 및 단순성)**

1. 🛡️ 캐시 스탬피드 원천 차단 (가장 큰 장점)
캐시 갱신을 오직 **TimerTrigger**라는 **단일하고 통제된 주체**가 담당합니다. EventGridTrigger 요청이 아무리 폭주해도, 사용자 요청 스레드는 DB 갱신 작업에 관여하지 않고 항상 캐시에서 데이터를 읽기만 합니다. 이는 **DB 부하를 일정하게 유지**하고 서비스의 **안정성을 극대화**합니다. 

2. 🚀 단순한 구현 및 성능 예측 가능
갱신 로직이 **읽기(Reader)** 로직과 **쓰기(Writer)** 로직으로 명확히 분리됩니다.
* 쓰기: TimerTrigger (단독 실행)
* 읽기: EventGridTrigger (동시 실행)
이 분리 덕분에 코드가 단순해지고, 캐시 만료 및 갱신 시점이 예측 가능해집니다.

3. 👍 높은 가용성 (High Availability)
만료 시점에도 TimerTrigger가 새 데이터를 준비하는 동안, 서비스는 **이전 데이터(Stale Data)**를 계속 제공하여 요청에 대한 **응답 지연이나 실패 없이** 서비스를 지속할 수 있습니다.

---

**🔴 단점 (신선도 및 비효율성)**

1. 📉 데이터 신선도 저하 (Latency)
캐시가 10분 주기로 갱신된다면, 데이터는 **최대 10분** 동안 오래된 상태일 수 있습니다. 데이터의 신선도(Freshness)가 중요한 애플리케이션에는 적합하지 않습니다. 데이터가 실제 변경된 시점과 캐시에 반영되는 시점 사이에 불필요한 지연이 발생합니다.

2. 🔋 불필요한 갱신 (Inefficiency)
TimerTrigger는 **데이터가 변경되었는지 여부와 관계없이** 10분마다 DB를 조회하고 캐시를 갱신하는 작업을 수행합니다.
* 만약 10분 동안 데이터 변경이 없었다면, DB 조회 및 캐시 갱신 비용은 **낭비**입니다.
* 이는 클라우드 환경에서 불필요한 컴퓨팅 자원 및 DB 트래픽 비용을 발생시킵니다.

3. 🚨 초기 로딩 문제
애플리케이션이 처음 시작되거나 캐시가 완전히 비었을 때, 첫 TimerTrigger 실행(예: 10분 후) 전까지는 캐시 데이터가 없습니다. 이 시간 동안 들어오는 요청은 **캐시 미스**를 경험하고 데이터 부재 상태를 처리해야 합니다.

4. 🔗 인프라 의존성
TimerTrigger (Azure Function Timer, Cron Job 등) 인프라에 직접 의존하게 됩니다. 캐싱 로직 자체가 애플리케이션의 핵심 비즈니스 로직과 함께 실행되는 것이 아니라, **외부 스케줄러**의 제어를 받게 됩니다.