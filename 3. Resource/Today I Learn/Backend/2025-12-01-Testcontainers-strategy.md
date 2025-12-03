# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 01일`

## 카테고리
`#CSharp`, `#Testcontainers`, `#ContainerPerTest`, `#ContainerPerClass`, `#ContainerPerCollection`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- SQLite 할용해 초기 개발 진행했지만, 운영 DB(SQL Server)와 같은 형태로 작업을 할 필요성을 느낌
- 닷넷 진영에서도 Testcontainers 테스트를 지원한다는 것을 알게 됨
- 서비스 레이어 기준 통합 테스트를 수행하는 것을 목표로 3가지 전략 학습 테스트 진행

### 2. 핵심 내용 / 개념 정리

#### Reference 
https://blog.jetbrains.com/dotnet/2023/10/24/how-to-use-testcontainers-with-dotnet-unit-tests/#container-per-test-strategy

#### 패키지 설치 

**단위 테스트 관련**
```text
xunit //2.9.3
xunit.runner.visualstudio // 3.1.4
Microsoft.NET.Test.Sdk // 17.14.1
```
- 버전이 상이해서 LTS 버전으로 설치
- [Getting Started with xUnit.net v2 [2025 July 4] | xUnit.net](https://xunit.net/docs/getting-started/v2/getting-started)
	- 진짜 위에 3개 설치한다 
	- xUnit.net v2를 지금 설치한거고 v3가 나온 상태인듯하다

**Testcontainers 관련**
```shell
dotnet add package Testcontainers.Xunit
dotnet add package Testcontainers.MsSql
```

> [!warning] 테스트시 docker 실행 중이여야 한다.

💩`Fluent Assertions 라이센스 이슈 확인`
```text
 Warning:
 The component "Fluent Assertions" is governed by the rules defined in the Xceed License Agreement and
 the Xceed Fluent Assertions Community License. You may use Fluent Assertions free of charge for
 non-commercial use only. An active subscription is required to use Fluent Assertions for commercial use.
 Please contact Xceed Sales mailto:sales@xceed.com to acquire a subscription at a very low cost.
 A paid commercial license supports the development and continued increasing support of
 Fluent Assertions users under both commercial and community licenses. Help us
 keep Fluent Assertions at the forefront of unit testing.
 For more information, visit https://xceed.com/products/unit-testing/fluent-assertions/
```
- 상업용 사용시 법적 분쟁 발생 가능 
	- `Shouldy` 패키지 사용 권장✅ (`gemini`)
	- [shouldly/LICENSE.txt at master · shouldly/shouldly](https://github.com/shouldly/shouldly/blob/master/LICENSE.txt)


#### ContainerPerTest (테스트 메서드당 하나의 컨테이너)
- 테스트 메서드당 컨테이너를 매번 재시작한다. 
- **실행 속도는 제일 느리지만, 격리성은 뛰어나다**

```cs
namespace ShipParticularsApi.Tests.Tests.Testcontainers
{
    public class DatabaseContainerPerTestcs(ITestOutputHelper output)
        : IAsyncLifetime
    {
        private readonly MsSqlContainer _dbContainer = new MsSqlBuilder()
            .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
            .WithPassword("qwer1234!@#$")
            .WithCleanUp(true)
            .Build();

        private DbContextOptions<ShipParticularsContext> _options;

        private ShipParticularsContext CreateContext() => new(_options);

        private const string reason = "Study purpose";

        public async Task DisposeAsync()
        {
            await _dbContainer.StopAsync();
        }

        public async Task InitializeAsync()
        {
            await _dbContainer.StartAsync();

            _options = new DbContextOptionsBuilder<ShipParticularsContext>()
                           .UseSqlServer(_dbContainer.GetConnectionString())
                           .UseLazyLoadingProxies()
                           .Options;

            var context = new ShipParticularsContext(_options);
            context.Database.Migrate();

            output.WriteLine($"Container Id : {_dbContainer.Id}");
        }


        [Fact(DisplayName = "DB에서 조회한 엔티티는 DbContext가 상태 추적 한다.")]
        public async Task AsTracking()
        {
            // Arrange
            await using var context = CreateContext();
            await using var transaction = await context.Database.BeginTransactionAsync();

            const string shipKey = "SHIP_KEY";

            context.ShipInfos.Add(NoService(shipKey).Build());
            await context.SaveChangesAsync();

            // Act & Assert
            var repository = new ShipInfoRepository(context);
            var actual = await repository.GetByShipKeyAsync(shipKey);

            actual.Should().NotBeNull();

            var entry = context.Entry(actual!);
            entry.State.Should().Be(EntityState.Unchanged);

            // 🌟 await using 블록 종료 시, transaction이 롤백되어 데이터 자동 정리.
        }
        
		// 이하 생략
}
```


총 4개 메서드 실행시 Container Id가 전부 다른 걸 확인✅
```text
Container Id : 8d00c66ebaba57ef2e1293d67c4b51d24c766591cddfe98b0c60246315ce9590

Container Id : fad7ee1e9e9797b8a6ae88ddef74b0b7afc78dfaeeae5eea54808bc5382fb6e6

Container Id : 8ae5997de38ddf5f80cdedb4bf4c164bb96e32ab0f7014d2ff5428cccb242f0c

Container Id : 46322121d4606166d25c073f9076d52133d88a9d2a4967cdb6a8b6ab8f60cf40
```

#### ContainerPerClass (테스트 클래스당 하나의 컨테이너)
- 테스트 클래스당 하나의 컨테이너를 공유해서 사용한다. 
	- 그러다보니 테스트간의 격리성을 보장하기 위해 초기화를 잘해줘야 한다. 
	- 다행스럽게도 현재 repository 테스트를 하고 있고, context와 트랜잭션에 대한 제어권을 가지고 있기 때문에 rollback 처리가 수행된다.
		- 반면 서비스 통합 테스트에서는 Db 초기화 처리를 위해 `respawn` 패키지를 사용함


```cs
public class DatabaseFixture : IAsyncLifetime
{
    private readonly MsSqlContainer _dbContainer = new MsSqlBuilder()
        .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
        .WithPassword("qwer1234!@#$")
        .WithCleanUp(true)
        .Build();

    public string ConectionString => _dbContainer.GetConnectionString();
    public string ContainerId => $"{_dbContainer.Id}";

    private DbContextOptions<ShipParticularsContext> _options;
    public ShipParticularsContext CreateContext() => new(_options);

    public async Task InitializeAsync()
    {
        await _dbContainer.StartAsync();

        _options = new DbContextOptionsBuilder<ShipParticularsContext>()
                       .UseSqlServer(_dbContainer.GetConnectionString())
                       .UseLazyLoadingProxies()
                       .Options;

        var context = new ShipParticularsContext(_options);
        context.Database.Migrate();
    }

    public Task DisposeAsync() => _dbContainer.DisposeAsync().AsTask();
}
```


```cs
public class DatabaseContainerPerTestClass(DatabaseFixture fixture, ITestOutputHelper output)
    : IClassFixture<DatabaseFixture>, IDisposable
{
    public void Dispose()
    {
        output.WriteLine($"Container Id = {fixture.ContainerId}");
    }

    [Fact(DisplayName = "DB에서 조회한 엔티티는 DbContext가 상태 추적 한다.")]
    public async Task AsTracking()
    {
        // Arrange
        await using var context = fixture.CreateContext();
        await using var transaction = await context.Database.BeginTransactionAsync();

        const string shipKey = "SHIP_KEY";

        context.ShipInfos.Add(NoService(shipKey).Build());
        await context.SaveChangesAsync();

        // Act & Assert
        var repository = new ShipInfoRepository(context);
        var actual = await repository.GetByShipKeyAsync(shipKey);

        actual.Should().NotBeNull();

        var entry = context.Entry(actual!);
        entry.State.Should().Be(EntityState.Unchanged);

        // 🌟 await using 블록 종료 시, transaction이 롤백되어 데이터 자동 정리.
    }
    
    // 이하 생략 
    
}
```
- `IClassFixture<DatabaseFixture>`가 실행이 되면서 테스트 컨테이너가 초기화 된다


```text
// DatabaseContainerPerTestClass 4개다 같은 Container Id 출력
Container Id = 13d9ec8d98c651d0fbf1b442eab2621157598eea914e33d2f965153101bba167

// DatabaseContainerPerTestClass2 의 경우 
Container Id = 51ad8b7cce076d4f4ea91ae3f219b100402ebed44becb2788305f58d7697a2a0
```

#### ContainerPerCollection (Collection 당 하나의 컨테이너)
- 여러 클래스가 동일한 컨테이너를 공유한다 
- `[CollectionDefinition("Database Collection")]`을 선언해두고 `[Collection("Database Collection")]`을 표기하게 되면 동일한 `ICollectionFixture<DatabaseFixture>`를 공유하 된다
	- 그룹핑 하는 느낌이다.

>[!info] CollectionDefinition 선언시 nameof(..) 사용할 경우 컴파일 시점에 클래스마다 다르게 해석되어 컨테이너가 각각 생성됨 (유의) => 아래와 같이 문자열로 지정하는게 안정적

```cs
[CollectionDefinition("Database Collection")]
public class DatabaseCollection : ICollectionFixture<DatabaseFixture>;

[Collection("Database Collection")]
public class DatabaseContainerPerCollection(DatabaseFixture fixture, ITestOutputHelper output)
    : IDisposable
{
    public void Dispose()
    {
        output.WriteLine($"Collection Container Id = {fixture.ContainerId}");
    }

    [Fact(DisplayName = "DB에서 조회한 엔티티는 DbContext가 상태 추적 한다.")]
    public async Task AsTracking()
    {
        // Arrange
        await using var context = fixture.CreateContext();
        await using var transaction = await context.Database.BeginTransactionAsync();

        const string shipKey = "SHIP_KEY";

        context.ShipInfos.Add(NoService(shipKey).Build());
        await context.SaveChangesAsync();

        // Act & Assert
        var repository = new ShipInfoRepository(context);
        var actual = await repository.GetByShipKeyAsync(shipKey);

        actual.Should().NotBeNull();

        var entry = context.Entry(actual!);
        entry.State.Should().Be(EntityState.Unchanged);

        // 🌟 await using 블록 종료 시, transaction이 롤백되어 데이터 자동 정리.
    }
    
    // 이하 생략 
}
        
```



```cs
 [Collection("Database Collection")]
 public class DatabaseContainerPerCollection2(DatabaseFixture fixture, ITestOutputHelper output)
     : IDisposable
 {
     public void Dispose()
     {
         output.WriteLine($"Collection Container Id = {fixture.ContainerId}");
     }
     
     // 테스트 메서드 
}


[Collection("Database Collection")]
public class DatabaseContainerPerCollection3(DatabaseFixture fixture, ITestOutputHelper output)
    : IDisposable
{
    public void Dispose()
    {
        output.WriteLine($"Collection Container Id = {fixture.ContainerId}");
    }
	 
     // 테스트 메서드   
}
```


```text
// 클래스 3개다 container id 가 동일함 확인
Collection Container Id = b0da8a3449780cf76f4d22435d4a68e279cd72b80b536b9769fea57fe9876d19
```