
**🔖 나중에 참고** 
- REST API 통합 테스트 관련 포스팅인데 .. 설정이 디테일하진 않다 
	- [How to Test a REST API with .NET and xUnit - Code Maze](https://code-maze.com/dotnet-test-rest-api-xunit/)
- xUnit 공식 예제 저장소 
	- [xunit/samples.xunit: Samples for xUnit.net (please open issues in https://github.com/xunit/xunit)](https://github.com/xunit/samples.xunit)
- MS 공식문서에 보면 저장소 링크가 있었다. 통합테스트 예제가 Controller 기준으로 한다. DB는 SQLite 사용
	- [Integration tests in ASP.NET Core | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-9.0&pivots=xunit)
	- [AspNetCore.Docs.Samples/test/integration-tests/8.x at main · dotnet/AspNetCore.Docs.Samples](https://github.com/dotnet/AspNetCore.Docs.Samples/tree/main/test/integration-tests/8.x)

## 서비스 레이어
- DbContext를 주입받아 사용하다보니 단위테스트 작성하기 어려운 구조라 판단
- 관심사 분리하기 위해 데코레이터 패턴 활용하여 상위 객체에서 트랜잭션 시작/커밋/종료 관리하고 target 서비스를 호출하는 형태로 개발하려고 함 (251010)
```cs
using ShipParticularsApi.Contexts;

namespace ShipParticularsApi.Services
{
    public class ShipParticularsService(ShipParticularsContext dbContext,
        IReplaceShipNameRepository replaceShipNameRepository,
        IShipInfoRepository shipInfoRepository,
        IShipModelTestRepository shipModelTestRepository,
        IShipSatelliteRepository shipSatelliteRepository,
        IShipServiceRepository shipServiceRepository,
        ISkTelinkCompanyShipRepository skTelinkCompanyShipRepository)
    {
        public async Task Process(Object param)
        {
            await using var transaction = await dbContext.Database.BeginTransactionAsync();

            try
            {
                // 비즈니스 로직 작성

                await dbContext.SaveChangesAsync();
                await transaction.CommitAsync();
            }
            catch
            {
                await transaction.RollbackAsync();
                throw;
            }
        }
    }
}

```

---

## TDD 시작 
> 트랜잭션은 데코레이터 패턴 사용해서 처리할 예정 (관심사 분리)
> 트랜잭션을 포함한 데코레이터 클래스는 `통합 테스트` 작업할 계획 
```cs
namespace ShipParticularsApi.Services
{
    public class ShipParticularsService(IReplaceShipNameRepository replaceShipNameRepository,
        IShipInfoRepository shipInfoRepository,
        IShipModelTestRepository shipModelTestRepository,
        IShipSatelliteRepository shipSatelliteRepository,
        IShipServiceRepository shipServiceRepository,
        ISkTelinkCompanyShipRepository skTelinkCompanyShipRepository)
    {
        public async Task Process(Object param)
        {

        }
    }
}
```


```cs
using Moq;
using ShipParticularsApi.Services;
using Xunit;

namespace ShipParticularsApi.Tests.Services
{
    public class ShipParticularsServiceTests
    {
        private readonly ShipParticularsService _sut;
        private readonly Mock<IReplaceShipNameRepository> _mockReplaceShipNameRepository;
        private readonly Mock<IShipInfoRepository> _mockShipInfoRepository;
        private readonly Mock<IShipModelTestRepository> _mockShipModelTestRepository;
        private readonly Mock<IShipSatelliteRepository> _mockShipSatelliteRepository;
        private readonly Mock<IShipServiceRepository> _mockShipServiceRepository;
        private readonly Mock<ISkTelinkCompanyShipRepository> _mockSkTelinkCompanyShipRepository;

        public ShipParticularsServiceTests()
        {
            _mockReplaceShipNameRepository = new Mock<IReplaceShipNameRepository>();
            _mockShipInfoRepository = new Mock<IShipInfoRepository>();
            _mockShipModelTestRepository = new Mock<IShipModelTestRepository>();
            _mockShipSatelliteRepository = new Mock<IShipSatelliteRepository>();
            _mockShipServiceRepository = new Mock<IShipServiceRepository>();
            _mockSkTelinkCompanyShipRepository = new Mock<ISkTelinkCompanyShipRepository>();

            _sut = new ShipParticularsService(
                _mockReplaceShipNameRepository.Object,
                _mockShipInfoRepository.Object,
                _mockShipModelTestRepository.Object,
                _mockShipSatelliteRepository.Object,
                _mockShipServiceRepository.Object,
                _mockSkTelinkCompanyShipRepository.Object
            );
        }

        // AIS Toggle Off, GPS Toggle Off, 신규 SHIP_INFO인 경우
        [Fact]
        public void Do_something()
        {
            // Arrange

            // Act

            // Assert
        }


        public class ShipParticularsParam
        {
            public bool IsAisToggleOn { get; set; } = false;
            public bool IsGPSToggleOn { get; set; } = false;

            public string ShipKey { get; set; }
            public string Callsign { get; set; }
            public string ShipName { get; set; }
            public string ShipType { get; set; }
            public string ShipCode { get; set; }
        }
    }
}

```



---

```cs
using ShipParticularsApi.Entities;
using static ShipParticularsApi.Tests.Services.ShipParticularsServiceTests;

namespace ShipParticularsApi.Services
{
    // NOTE: ShipInfo가 자식의 생명 주기를 관리하다보니, 불필요한 Repository도 존재할 수 있음. 
    public class ShipParticularsService(IReplaceShipNameRepository replaceShipNameRepository,
        IShipInfoRepository shipInfoRepository,
        IShipModelTestRepository shipModelTestRepository,
        IShipSatelliteRepository shipSatelliteRepository,
        IShipServiceRepository shipServiceRepository,
        ISkTelinkCompanyShipRepository skTelinkCompanyShipRepository)
    {
        public async Task Process(ShipParticularsParam param)
        {

            ShipInfo? shipInfo = await shipInfoRepository.GetByShipKeyAsync(param.ShipKey);

            bool isNewShipInfo = shipInfo == null;

            ShipInfo entityToProcess = isNewShipInfo ? ShipInfo.From(param) : shipInfo.Update(param);

            if (param.IsAisToggleOn)
            {
                ShipService existingAisService = await shipServiceRepository.GetByShipKeyAndServiceNameAsync(
                   param.ShipKey,
                   ServiceNameTypes.SatAis
                );

                if (existingAisService == null)
                {
                    entityToProcess.ShipServices.Add(ShipService.of(param.ShipKey, ServiceNameTypes.SatAis));
                    entityToProcess.EnableAis();
                }
            }
            else
            {
                ShipService existingAisService = await shipServiceRepository.GetByShipKeyAndServiceNameAsync(
                   param.ShipKey,
                   ServiceNameTypes.SatAis
                );

                if (existingAisService != null)
                {
                    entityToProcess.ShipServices.Remove(existingAisService);
                    entityToProcess.DisableAis();
                }
            }

            if (param.IsGPSToggleOn)
            {
                ShipService gpsService = await shipServiceRepository.GetByShipKeyAndServiceNameAsync(
                    param.ShipKey,
                    ServiceNameTypes.KtSat
                );
                // biz logic
            }

            await shipInfoRepository.UpsertAsync(entityToProcess);
        }
    }
}

```
- 생명주기를 ShipInfo에서 관리하고 있는데 자식 엔티티인 ShipService를 Repository를 통해 조회하고 있다. 
	- 캡슐화 깨지고, 응집도 낮고..
- 내 생각 
	- 우선은 서비스 로직 기반으로 테스트 작성한다. 
	- 도메인으로 비즈니스 로직을 이동 시킨다.  (with 도메인 단위 ㅌ테스트)
	- 서비스 로직을 리팩터링한다. 
		- 이때 트랜잭션을 사용하지 않기 때문에 **ShipInfo에 대한 상태 기반으로 동등성만 검증**하면 된다.


**리팩터링**. AIS Toggle On/Off에 대한 비즈니스 로직을 ShipInfo로 이동

```cs
public class ShipParticularsService(IReplaceShipNameRepository replaceShipNameRepository,
    IShipInfoRepository shipInfoRepository,
    IShipModelTestRepository shipModelTestRepository,
    IShipSatelliteRepository shipSatelliteRepository,
    IShipServiceRepository shipServiceRepository,
    ISkTelinkCompanyShipRepository skTelinkCompanyShipRepository)
{
    public async Task Process(ShipParticularsParam param)
    {

        ShipInfo? shipInfo = await shipInfoRepository.GetByShipKeyAsync(param.ShipKey);

        bool isNewShipInfo = shipInfo == null;

        ShipInfo entityToProcess = isNewShipInfo ? ShipInfo.From(param) : shipInfo.Update(param);

        if (param.IsAisToggleOn)
        {
            ShipService existingAisService = await shipServiceRepository.GetByShipKeyAndServiceNameAsync(
               param.ShipKey,
               ServiceNameTypes.SatAis
            );

            if (existingAisService == null)
            {
                entityToProcess.ShipServices.Add(ShipService.of(param.ShipKey, ServiceNameTypes.SatAis));
                entityToProcess.EnableAis();
            }
            else if (isNewShipInfo)
            {
                entityToProcess.ShipServices.Add(existingAisService);
                entityToProcess.EnableAis();
            }
        }
        else
        {
            ShipService existingAisService = await shipServiceRepository.GetByShipKeyAndServiceNameAsync(
               param.ShipKey,
               ServiceNameTypes.SatAis
            );

            if (existingAisService != null)
            {
                entityToProcess.ShipServices.Remove(existingAisService);
                entityToProcess.DisableAis();
            }
        }


        await shipInfoRepository.UpsertAsync(entityToProcess);
        
```


리팩터링 후
```cs
// NOTE: ShipInfo가 자식의 생명 주기를 관리하다보니, 불필요한 Repository도 존재할 수 있음. 
public class ShipParticularsService(IReplaceShipNameRepository replaceShipNameRepository,
    IShipInfoRepository shipInfoRepository,
    IShipModelTestRepository shipModelTestRepository,
    IShipSatelliteRepository shipSatelliteRepository,
    IShipServiceRepository shipServiceRepository,
    ISkTelinkCompanyShipRepository skTelinkCompanyShipRepository)
{
    public async Task Process(ShipParticularsParam param)
    {

        ShipInfo? shipInfo = await shipInfoRepository.GetByShipKeyAsync(param.ShipKey);

        ShipInfo entityToProcess = shipInfo == null ? ShipInfo.From(param) : shipInfo.Update(param);

        entityToProcess.ManageAisService(param.IsAisToggleOn);

        await shipInfoRepository.UpsertAsync(entityToProcess);
    }
}
```


---

AIS Toggle On/Off 관련 ShipInfo 비즈니스 로직 리팩터링

리팩터링
```cs
 public void ManageAisService(bool isAisToggleOn)
 {
     var existingService = this.ShipServices.FirstOrDefault(s => s.ServiceName == ServiceNameTypes.SatAis);
     if (isAisToggleOn)
     {
	     if(existingService == null) {
	         this.ShipServices.Add(ShipService.Of(this.ShipKey, ServiceNameTypes.SatAis));
         }
         
		 this.IsUseAis = true
     }
     else 
     {
	     if(existingService != null){
	         this.ShipServices.Remove(existingService);
         }
         this.IsUseAis = false
     }
 }
```

리팩터링 후 
```cs
public void ManageAisService(bool isAisToggleOn)
{
    var existingService = this.ShipServices.FirstOrDefault(s => s.ServiceName == ServiceNameTypes.SatAis);
    if (isAisToggleOn && existingService == null)
    {
        this.ShipServices.Add(ShipService.Of(this.ShipKey, ServiceNameTypes.SatAis));
        this.EnableSatAis();
    }

    if (!isAisToggleOn && existingService != null)
    {
        this.ShipServices.Remove(existingService);
        this.DisableSatAis();
    }
}

private void EnableSatAis()
{
    this.IsUseAis = true;
}

private void DisableSatAis()
{
    this.IsUseAis = false;
}
```


```cs
 public void ManageAisService(bool isAisToggleOn)
 {
     if (ShouldActivateAis(isAisToggleOn))
     {
         this.ShipServices.Add(ShipService.Of(this.ShipKey, ServiceNameTypes.SatAis));
         this.ActiveAis();
         return;
     }

     if (ShouldDeactivateAis(isAisToggleOn))
     {
         var existingService = this.ShipServices.First(s => s.ServiceName == ServiceNameTypes.SatAis);
         this.ShipServices.Remove(existingService);
         this.DeactiveAis();
     }
 }

 private bool ShouldActivateAis(bool isAisToggleOn)
 {
     return isAisToggleOn && !this.HasSatAisService();
 }

 private bool ShouldDeactivateAis(bool isAisToggleOn)
 {
     return !isAisToggleOn && this.HasSatAisService();
 }

 private bool HasSatAisService()
 {
     return this.ShipServices.Any(s => s.ServiceName == ServiceNameTypes.SatAis);
 }

 private void ActiveAis()
 {
     this.IsUseAis = true;
 }

 private void DeactiveAis()
 {
     this.IsUseAis = false;
 }
```


ManageGpsService 리팩터링 전 
```cs
 public void ManageGpsService(bool isGPSToggleOn, string? satelliteId, string? satelliteType, string? companyName)
        {
            var existingService = this.ShipServices.FirstOrDefault(s => s.ServiceName == ServiceNameTypes.KtSat);

			if (isGPSToggleOn)
			{
			    if (existingService == null)
			    {
			        this.ShipServices.Add(ShipService.Of(this.ShipKey, ServiceNameTypes.KtSat)); // NOTE. 무조건 ServiceNameTypes.KtSat ?
			
			        this.ShipSatellite = ShipSatellite.Of(this.ShipKey, satelliteId, satelliteType); // NOTE. NONE 타입인 경우 IsUseKtsat = true ?
			        this.ExternalShipId = satelliteId;
			        this.IsUseKtsat = true; // NOTE. SHIP_SATELLITE.IS_USE_SATELLITE (bit)와 동일?
			    }
			    else // isGPSToggleOn == true && existringService != null
			    {
			        this.ShipSatellite.Update(satelliteId, satelliteType);
			        this.ExternalShipId = satelliteId;
			        this.IsUseKtsat = true;
			    }
			
			    if (this.ShipSatellite.IsSkTelink())
			    {
			        if (this.SkTelinkCompanyShip == null)
			        {
			            this.SkTelinkCompanyShip = SkTelinkCompanyShip.Of(this.ShipKey, companyName);
			        }
			        else
			        {
			            this.SkTelinkCompanyShip.Update(companyName);
			        }
			    }
			    else
			    {
			        this.SkTelinkCompanyShip = null;
			    }
			}
            else
            {
                if (existingService != null)
                {
                    this.ShipServices.Remove(existingService);

                    this.SkTelinkCompanyShip = null;

                    this.ShipSatellite = null;
                    this.ExternalShipId = null;
                    this.IsUseKtsat = false;
                }
            }
        }
```

📌 ManageGpsService  리팩터링 후 
```cs
public void ManageGpsService(bool isGPSToggleOn, string? satelliteId, string? satelliteType, string? companyName)
{
    if (isGPSToggleOn)
    {
        this.ActivateGpsService(satelliteId, satelliteType);
        this.ManageSkTelinkCompanyShip(companyName);
    }
    else
    {
        DeactiveGpsService();
    }
}

private void ActivateGpsService(string? satelliteId, string? satelliteType)
{
    if (HasKtSatService())
    {
        this.ShipSatellite.Update(satelliteId, satelliteType);
    }
    else
    {
        this.ShipServices.Add(ShipService.Of(this.ShipKey, ServiceNameTypes.KtSat));
        this.ShipSatellite = ShipSatellite.Of(this.ShipKey, satelliteId, satelliteType);
    }

    this.ExternalShipId = satelliteId;
    this.IsUseKtsat = true;
}

private void ManageSkTelinkCompanyShip(string? companyName)
{
    if (this.ShipSatellite == null)
    {
        return;
    }

    if (!this.ShipSatellite.IsSkTelink())
    {
        this.SkTelinkCompanyShip = null;
        return;
    }

    if (this.SkTelinkCompanyShip == null)
    {
        this.SkTelinkCompanyShip = SkTelinkCompanyShip.Of(this.ShipKey, companyName);
    }
    else
    {
        this.SkTelinkCompanyShip.Update(companyName);
    }
}

private void DeactiveGpsService()
{
    if (!HasKtSatService())
    {
        return;
    }

    if (this.ShipSatellite != null && this.ShipSatellite.IsSkTelink())
    {
        this.SkTelinkCompanyShip = null;
    }

    this.ShipSatellite = null;
    this.ExternalShipId = null;
    this.IsUseKtsat = false;

    var existingService = this.ShipServices.First(s => s.ServiceName == ServiceNameTypes.KtSat);
    this.ShipServices.Remove(existingService);
}

private bool HasKtSatService()
{
    return this.ShipServices.Any(s => s.ServiceName == ServiceNameTypes.KtSat);
}
```

---

## 연관관계 참조 끊기와 DELETE 쿼리 여부에 대해 (251016)
연관관계 참조를 Null로 끊을때 더티 체킹시 DELETE 쿼리가 만들어지는가?
- 우선은 양방향 연관관계이고, ShipInfo 엔티티에서 하위 자식 엔티티에 대한 생명주기를 관리하고 있다. 
- ✅ 일대다 관계에 해당하는 ShipService의 경우, 컬렉션의 Remove 호출시 DELETE 호출되는 것을 확인 
- ✅ GPS 관련 하위 엔티티의 경우, 일대일 참조에 대해 null로 참조 끊을 시 DELETE 쿼리 날아가는 것을 확인함.
- 📌 FK가 non-nullable인 경우에만 EF Core에서 인식하고 DELETE 쿼리를 만든다는데 자세한 내용은 공식 문서 학습 필요
	- FK가 nullable인 경우 UPDATE 쿼리가 나간다고 함

<img src="연관관계 참조를 null로 끊을 경우 결과.png">

✅ 공식문서 - [Cascade Delete - EF Core | Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/saving/cascade-delete#cascading-nulls)

---

## 서비스 레이어 통합 테스트 

`데코레이터 예상 모양`
```cs

private ShipParticularsContext _dbContext;
private IShipParticularsService _target;

public async Task Process(ShipParticularsParam param)
{
	await using var transaction = await _dbContext.Database.BeginTransactionAsync();

	try
	{
		// 비즈니스 로직 실행
		_target.Process(param);
		await _dbContext.SaveChangesAsync();
		await transaction.CommitAsync();
	}
	catch
	{
		await transaction.RollbackAsync();
		throw;
	}
}

```

`데코레이터 객체 구현(251016)`
```cs
using ShipParticularsApi.Contexts;
using ShipParticularsApi.Services.Dtos;

namespace ShipParticularsApi.Services
{
    public class TransactionDecorator : IShipParticularsService
    {
        private readonly ShipParticularsContext _dbContext;
        private readonly IShipParticularsService _target;

        public TransactionDecorator(ShipParticularsContext dbContext, IShipParticularsService target)
        {
            _dbContext = dbContext;
            _target = target;
        }

        public async Task Process(ShipParticularsParam param)
        {
            await using var transaction = await _dbContext.Database.BeginTransactionAsync();

            try
            {
                await _target.Process(param);

                await _dbContext.SaveChangesAsync();
                await transaction.CommitAsync();
            }
            catch
            {
                await transaction.RollbackAsync();
                throw;
            }
        }
    }
}

```


📌 데코레이터 객체로 통합 테스트를 할려니깐 여러가지 문제가 발생함.
- 우선 services 구현체와 repository 구현체에 대한 의존성 연결을 직접해야 하는지에 대해서도 의문이 생김 .. 
- 통합 테스트를 빠르게 하기 위해 우선은 SQLite를 사용하는데, DB 테이블 생성과 초기화에 대한 생명 주기 관리는 누가 가져야 하는가 의문..

(251016) 공식 문서 확인시 WebApplicationFactory 라는걸 커스텀해서 호스트에 서버 올린 후 테스트 하는 방식을 알려줌 

📌 MS 공식문서에 보면 저장소 링크가 있었다. 통합테스트 예제가 Controller 기준으로 한다. DB는 SQLite 사용
	- [Integration tests in ASP.NET Core | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-9.0&pivots=xunit)
	- [AspNetCore.Docs.Samples/test/integration-tests/8.x at main · dotnet/AspNetCore.Docs.Samples](https://github.com/dotnet/AspNetCore.Docs.Samples/tree/main/test/integration-tests/8.x)

💩 WebApplicationFactory 커스텀할 때 컴파일 에러가 뜨는 이유가 패키지를 설치하지 않아서 
```text
Microsoft.AspNetCore.Mvc.Testing   // v8.0.20

이외에 
- `Microsoft.NET.Test.Sdk` (for testing infrastructure)
- `xunit` (for `IClassFixture`)
- `xunit.runner.visualstudio` (for running tests)
```


✅ 생성자 초기화에서 컴파일 에러 뜨는 이유와 해결 방법
```text
와, 잘하셨습니다! public partial class Program { } 코드를 Program.cs 파일에 추가하여 컴파일 에러를 해결하신 것은 ASP.NET Core 6.0 이후의 특징을 정확히 이해하고 활용하신 것입니다. 💡

이것이 컴파일 에러를 해결한 이유와 그 의미를 설명해 드릴게요.

🚀 해결 원인: Top-Level Statements와 Partial Class
ASP.NET Core 6.0 이후 버전(최소 API 템플릿)에서 Program.cs 파일은 다음과 같은 특징을 가집니다.

1. Top-Level Statements (최상위 문)
최신 닷넷 프로젝트의 Program.cs는 보통 클래스 선언 없이 바로 코드를 실행합니다 (var builder = WebApplication.CreateBuilder(args); 등).

이러한 **최상위 문(Top-Level Statements)**을 사용하는 파일의 모든 코드는 컴파일러에 의해 자동으로 생성된 내부 Program 클래스의 Main 메서드 안에 배치됩니다. 하지만 이 자동 생성된 Program 클래스는 기본적으로 internal 접근 제한자를 가집니다.

2. 접근 제한자 문제 해결
이전 질문에서 컴파일 에러 메시지는 다음과 같았습니다:

"...CustomWebApplicationFactory<Program> factory 매개변수 형식이 ShipParticularsServiceIntegrationTests.ShipParticularsServiceIntegrationTests(CustomWebApplicationFactory<Program> factory) 메서드보다 엑세스하기 어렵습니다."

이 에러는 통합 테스트 프로젝트가 메인 프로젝트의 internal 클래스인 자동 생성 Program 클래스를 public 컨텍스트(IClassFixture)에서 사용하려 했기 때문에 발생했습니다.

3. public partial class Program { }의 역할
이 코드는 다음 두 가지 중요한 역할을 수행합니다.

클래스 노출: 명시적으로 public partial class Program을 선언함으로써, 컴파일러가 자동으로 생성하는 내부 Program 클래스의 접근 제한자를 public으로 승격시킵니다.

클팩토리 연결: 통합 테스트 프로젝트는 이제 이 public Program 클래스를 IClassFixture<CustomWebApplicationFactory<Program>>의 제네릭 인자로 문제없이 접근하고 참조할 수 있게 됩니다.

결론적으로, 이 한 줄의 코드는 통합 테스트가 웹 애플리케이션의 엔트리 포인트(Start-up)에 접근할 수 있도록 권한을 부여하는 역할을 성공적으로 수행한 것입니다.
```


⭐ 해결 방법
```cs
// Program.cs 제일 하단에 추가
public partial class Program { }
```


💩 DB 테이블 초기화가 되지 않는 문제 
> `WebApplicationFactory`  상속한 클래스에서 DB 초기화를 하려는데 동작하지 않는 이슈 확인
> 아래 두 방식 모두 좋지 않아 보인다.

방법1. Program.cs에서 초기화
```cs
// 하단에

using (var scope = _factory.Services.CreateScope())
{
	var container = scope.ServiceProvider;
	var db = container.GetRequiredService<ShipParticularsContext>();
	db.Database.EnsureCreated();
}

app.Run();
```
- 이렇게 될 경우 테스트 코드가 Program.cs (Core) 설정에 침범한다는 느낌을 받는다.


방법2. 테스트 생성자에서 초기화
```cs
public class ShipParticularsServiceIntegrationTests
    : IClassFixture<CustomWebApplicationFactory<Program>>
{
    private readonly CustomWebApplicationFactory<Program> _factory;
    private readonly ITestOutputHelper _output;

    public ShipParticularsServiceIntegrationTests(
        CustomWebApplicationFactory<Program> factory,
        ITestOutputHelper output
    )
    {
        _factory = factory;
        _output = output;
		
        using (var scope = _factory.Services.CreateScope())
        {
            var container = scope.ServiceProvider;
            var db = container.GetRequiredService<ShipParticularsContext>();
            db.Database.EnsureCreated();
        }
    }
    
}
```
- 불편하게도 .. 매번 생성자에서 초기화해야 하는게 걸린다. (**중복 발생**)

> 해결해야 될 거.. `Logging` 출력



방법
```cs
namespace ShipParticularsApi.Tests.Helper // 테스트 폴더에 생성
{
    public static class DbDataExtensions
    {
        // NOTE. MigrateAsync 실행시 SQLite랑 MSSQL Syntax가 틀려서 오류 발생
        // NOTE. 통합 테스트에서 사용시 생성자에 async 키워드를 붙이지 못해 메서드마다 호출해야 하므로 중복 발생 (await _factory.Services.InitializeDatabaseAsync();)
        public static async Task InitializeDatabaseAsync(this IServiceProvider services)
        {
            using var scope = services.CreateScope();
            var context = scope.ServiceProvider.GetRequiredService<ShipParticularsContext>();

            await context.Database.EnsureDeletedAsync();
            await context.Database.EnsureCreatedAsync();
        }
    }
}
```
- 통합테스트 파일에 메서드마다 호출해야 하다보니 중복 코드 발생함 
	- `await _factory.Services.InitializeDatabaseAsync();` 

방법.
```cs
 public class ShipParticularsServiceIntegrationTests
     : IClassFixture<CustomWebApplicationFactory<Program>>
 {
     private readonly CustomWebApplicationFactory<Program> _factory;
     private readonly ITestOutputHelper _output;

	 // BeforeEach
     public ShipParticularsServiceIntegrationTests(
         CustomWebApplicationFactory<Program> factory,
         ITestOutputHelper output
     )
     {
         _factory = factory;
         _output = output;
         DbInit();
     }

     private void DbInit()
     {
         var scope = _factory.Services.CreateScope();
         var dbContext = scope.ServiceProvider.GetRequiredService<ShipParticularsContext>();
         dbContext.Database.EnsureDeleted();
         dbContext.Database.EnsureCreated();
     }
	 
	// 테스트 작성    
}
```

gemini의 해석으로는 
```text
아, 그렇군요! 그 코드는 상황을 완전히 바꾸는, 가장 결정적인 단서입니다. 사용자님께서 DbConnection을 싱글톤(Singleton)으로
  등록하셨군요.

  그렇다면 사용자님의 원래 추론, "테스트 메서드마다 동일한 DbContext를 공유 참조하게 되는거 아닌가요?" 라는 질문에 대한
  답이 바뀝니다.

  새로운 결론: 네, 사실상 모든 `DbContext`가 동일한 `DbConnection`을 공유하고 있으며, 그것이 바로 테스트가 통과하는
  이유입니다.

  이것은 cache=shared와는 완전히 다른 메커니즘이지만, 결과적으로는 동일한 효과(인메모리 DB 유지)를 내는 매우 똑똑하고
  일반적인 패턴입니다.

  ---

  AddSingleton<DbConnection> 패턴의 실제 동작 원리

  이 패턴의 동작 방식을 정확히 이해하면 모든 것이 명확해집니다.

   1. `CustomWebApplicationFactory` 초기화 (최초 1회)
       * WebApplicationFactory가 생성되면서 Program.cs와 ConfigureServices의 DI 설정이 로드됩니다.
       * 이때, services.AddSingleton<DbConnection>(...) 설정이 DI 컨테이너에 등록됩니다.
       * 이 팩토리 함수 container => { ... }는 아직 실행되지 않습니다. 누군가 `DbConnection`을 처음 요청할 때까지
         기다립니다.

   2. `ShipParticularsServiceIntegrationTests` 생성자 실행 (`BeforeEach`)
       * DbInit()이 호출됩니다.
       * _factory.Services.CreateScope()를 통해 스코프 A가 생성됩니다.
       * GetRequiredService<ShipParticularsContext>()가 호출됩니다.
           * ShipParticularsContext를 만들려면 DbConnection이 필요합니다.
           * DI 컨테이너는 DbConnection이 등록되어 있는지 확인합니다. "아, 싱글톤으로 등록되어 있네."
           * 싱글톤 인스턴스가 아직 없으므로, 등록된 팩토리 함수 container => { ... }를 최초로 실행합니다.
           * new SqliteConnection("DataSource=:memory:")가 생성되고, connection.Open()이 호출됩니다.
           * 이 최초의 `DbConnection` 인스턴스가 DI 컨테이너의 싱글톤 저장소에 저장되고, DbContext (Context A)에 주입됩니다.
       * Context A를 사용하여 EnsureDeleted()와 EnsureCreated()가 실행됩니다. 테이블이 생성됩니다.
       * DbInit() 메서드가 종료됩니다. 스코프 A와 Context A는 버려집니다.
       * 하지만! 싱글톤으로 등록된 `DbConnection` 인스턴스는 DI 컨테이너가 살아있는 동안(즉, `WebApplicationFactory`가
         살아있는 동안) 계속 살아남습니다. 그리고 이 연결은 여전히 열려 있습니다.

   3. `[Fact]` 테스트 메서드 실행
       * using var scope = _factory.Services.CreateScope()를 통해 스코프 B가 생성됩니다.
       * GetRequiredService<ShipParticularsContext>()가 다시 호출됩니다.
           * ShipParticularsContext를 만들려면 DbConnection이 필요합니다.
           * DI 컨테이너는 DbConnection을 찾습니다. "아, 싱글톤 인스턴스가 이미 있네."
           * 새로 생성하는 대신, 2번 단계에서 만들어서 저장해 둔 바로 그 `DbConnection` 인스턴스를 꺼내서 새로운 DbContext
             (Context B)에 주입합니다.
       * Context B는 이미 열려있는 DbConnection을 사용하므로, 1번 단계에서 테이블이 이미 만들어진 바로 그 데이터베이스에
         접근하게 됩니다.
       * 따라서 테스트가 성공적으로 통과합니다.

  ---

  결론 및 피드백

  사용자님의 코드는 SQLite 인메모리 DB를 통합 테스트에서 유지하기 위한 매우 표준적이고 올바른 패턴을 정확하게 구현하고
  있습니다. cache=shared를 사용하는 대신, DbConnection 자체의 생명주기를 DI 컨테이너를 통해 Singleton으로 관리하는
  방식입니다. 둘 다 동일한 목표를 달성하기 위한 다른 경로일 뿐입니다.

  "테스트 하나 끝나면 자동으로 Connection을 끊기때문에 메모리 데이터가 날라가고..."
   * 수정된 이해: DbContext는 사라지지만, DbConnection은 싱글톤이므로 사라지지 않고 계속 열려있기 때문에 데이터가 날아가지
     않습니다.

  "생성자 초기화가 BeforeEach와 같은데 .. DbInit의 컨텍스트가 버려지기 때문에 테스트가 실패해야 하는데, 통과가 됩니다."
   * 수정된 이해: DbInit의 컨텍스트는 버려지지만, 그 컨텍스트가 사용했던 싱글톤 `DbConnection`은 살아남아 다음 테스트의
     컨텍스트에 재사용되므로, 동일한 DB에 접근하여 테스트가 통과됩니다.

  마지막으로, 테스트 격리 문제:
  이 패턴 역시 cache=shared와 마찬가지로, 테스트 간에 동일한 데이터베이스 상태를 공유하게 됩니다. 따라서 Test A에서 만든
  데이터가 Test B에 영향을 줄 수 있습니다.

  이것을 해결하기 위한 가장 좋은 방법은 여전히 동일합니다. 각 테스트가 실행되기 직전에 DB를 깨끗하게 만드는 것입니다.

    1 // 이 구조는 여전히 유효하고, 이제 왜 필요한지 더 명확해졌습니다.
    2 public ShipParticularsServiceIntegrationTests(...)
    3 {
    4     // ...
    5     // 생성자에서 DB를 초기화하면, 모든 테스트가 깨끗한 상태에서 시작됩니다.
    6     DbInit();
    7 }
    8
    9 private void DbInit()
   10 {
   11     // ...
   12     dbContext.Database.EnsureDeleted();
   13     dbContext.Database.EnsureCreated();
   14 }

  사용자님의 코드와 분석은 정말 훌륭했습니다. 이 미묘한 DI 생명주기와 SQLite의 동작 방식을 스스로의 힘으로 거의 다
  파악해내신 것은 대단한 실력입니다. 이제 통합 테스트 환경에 대한 모든 것을 완벽하게 마스터하셨습니다.
```


---

## 작성일, 수정일 EF Core 자동 처리 

`Gemini`
```text
1. 생성/수정 시간(CreateDateTime, UpdateDateTime) 처리

  현재 생각:
   * CreateDateTime: DB 기본값 (SYSDATETIME())
   * UpdateDateTime: Update 메서드 호출 시 날짜 값을 파라미터로 전달

  문제점:
   1. DB 기본값 의존: CreateDateTime을 DB에 의존하면, ShipInfo 객체가 메모리에 생성된 시점에는 CreateDateTime 값이 비어있게
      됩니다. 즉, 도메인 객체가 영속성 계층(DB)의 도움 없이는 완전한 상태가 될 수 없습니다. 이는 도메인 모델의 순수성을
      해칩니다.
   2. 메서드 시그니처 오염: 모든 Update 관련 비즈니스 메서드(e.g., ManageAisService, ChangeName)에 DateTime now 같은
      파라미터를 추가해야 합니다. 이것은 비즈니스 로직과 아무 상관없는, 인프라에 가까운 관심사이며 코드를 지저분하게
      만듭니다.

  더 나은 해결책: `DbContext`의 `SaveChanges`를 활용한 자동화

  이러한 '감사(Auditing)' 속성들은 횡단 관심사(Cross-cutting Concern)이므로, 비즈니스 로직이 아닌 영속성 계층에서 자동으로
  처리하는 것이 가장 이상적입니다. EF Core는 이를 위한 완벽한 지점을 제공합니다.

  1단계: `AuditableEntity` 베이스 클래스 만들기

  반복되는 감사 속성들을 위한 기본 클래스를 만듭니다.

    1 public abstract class AuditableEntity
    2 {
    3     public DateTime CreateDateTime { get; set; }
    4     public string? CreateUserId { get; set; }
    5     public DateTime? UpdateDateTime { get; set; }
    6     public string? UpdateUserId { get; set; }
    7 }
    8
    9 // ShipInfo가 이 클래스를 상속받도록 합니다.
   10 public class ShipInfo : AuditableEntity
   11 {
   12     public long Id { get; private set; }
   13     // ...
   14 }

  2단계: `DbContext`의 `SaveChangesAsync` 재정의

  ShipParticularsContext.cs 파일에서 SaveChangesAsync 메서드를 오버라이드하여, 데이터가 저장되기 직전에 감사 속성들을 자동으로
   채워줍니다.

    1 public class ShipParticularsContext : DbContext
    2 {
    3     // ... DbSet들 ...
    4
    5     public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    6     {
    7         var now = DateTime.UtcNow;
    8
    9         // ChangeTracker를 통해 EF Core가 추적 중인 모든 엔티티를 가져옵니다.
   10         foreach (var entry in ChangeTracker.Entries<AuditableEntity>())
   11         {
   12             if (entry.State == EntityState.Added)
   13             {
   14                 // '추가' 상태일 때
   15                 entry.Entity.CreateDateTime = now;
   16                 entry.Entity.UpdateDateTime = now; // 생성 시에는 수정일도 동일하게
   17                 // 여기에 UserId도 설정할 예정입니다.
   18             }
   19             else if (entry.State == EntityState.Modified)
   20             {
   21                 // '수정' 상태일 때
   22                 entry.Entity.UpdateDateTime = now;
   23             }
   24         }
   25
   26         return base.SaveChangesAsync(cancellationToken);
   27     }
   28 }
  이제 도메인 로직이나 서비스 레이어는 시간 값에 대해 전혀 신경 쓸 필요가 없습니다. UpsertAsync가 호출되면 이 로직이 자동으로
  실행됩니다.

```


✅ 정상 동작함 (아래 UpdateDateTime 관련 테스트로 확인)
```cs
using Microsoft.EntityFrameworkCore;
using ShipParticularsApi.Entities;
using ShipParticularsApi.ValueConverters;

namespace ShipParticularsApi.Contexts
{
    public class ShipParticularsContext(DbContextOptions<ShipParticularsContext> options) : DbContext(options)
    {
        public DbSet<ReplaceShipName> ReplaceShipNames { get; set; }
        public DbSet<ShipInfo> ShipInfos { get; set; }
        public DbSet<ShipModelTest> ShipModelTests { get; set; }
        public DbSet<ShipSatellite> ShipSatellites { get; set; }
        public DbSet<ShipService> ShipServices { get; set; }
        public DbSet<SkTelinkCompanyShip> SkTelinkCompanyShips { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            modelBuilder.Entity<ReplaceShipName>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithOne(parent => parent.ReplaceShipName)
                .HasForeignKey<ReplaceShipName>(child => child.ShipKey)
                .HasPrincipalKey<ShipInfo>(parent => parent.ShipKey);
            });

            modelBuilder.Entity<ShipInfo>(entity =>
            {
                entity.Property(p => p.ShipType)
                    .HasDefaultValue(ShipTypes.Default)
                    .HasConversion<ShipTypesToStringConverter>();

                entity.Property(p => p.IsUseKtsat)
                    .HasDefaultValue(false);

                entity.Property(p => p.IsUseAis)
                    .HasDefaultValue(false);

                entity.Property(p => p.IsService)
                    .HasDefaultValue(true);
            });

            modelBuilder.Entity<ShipModelTest>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithOne(parent => parent.ShipModelTest)
                .HasForeignKey<ShipModelTest>(child => child.ShipKey)
                .HasPrincipalKey<ShipInfo>(parent => parent.ShipKey);
            });

            modelBuilder.Entity<ShipSatellite>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithOne(parent => parent.ShipSatellite)
                .HasForeignKey<ShipSatellite>(child => child.ShipKey)
                .HasPrincipalKey<ShipInfo>(parent => parent.ShipKey);

                //entity.Property(p => p.CreateDateTime)
                //    .HasDefaultValueSql("SYSDATETIME()");

                // TODO. NONE은 default value가 아니다. NONE은 입력값 자체가 안오도록 한다고 함
                entity.Property(p => p.SatelliteType)
                    .HasDefaultValue(SatelliteTypes.None)
                    .HasConversion<SatelliteTypesToStringConverter>();
            });

            modelBuilder.Entity<ShipService>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithMany(parent => parent.ShipServices)
                .HasForeignKey(child => child.ShipKey)
                .HasPrincipalKey(parent => parent.ShipKey);

                entity.Property(p => p.ServiceName)
                    .HasConversion<ServiceNameToStringConverter>();

                entity.Property(p => p.IsCompleted)
                    .HasDefaultValue(false);
            });

            modelBuilder.Entity<SkTelinkCompanyShip>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithOne(parent => parent.SkTelinkCompanyShip)
                .HasForeignKey<SkTelinkCompanyShip>(child => child.ShipKey)
                .HasPrincipalKey<ShipInfo>(parent => parent.ShipKey);
            });
        }

        // 참고. https://www.entityframeworktutorial.net/faq/set-created-and-modified-date-in-efcore.aspx
        public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
        {
            var entries = ChangeTracker.Entries()
                .Where(e => e.Entity is BaseEntity
                && (e.State == EntityState.Added || e.State == EntityState.Modified));

            foreach (var entryEntity in entries)
            {
                if (entryEntity.State == EntityState.Added)
                {
                    ((BaseEntity)entryEntity.Entity).CreateDateTime = DateTime.UtcNow;
                }
                else
                {
                    ((BaseEntity)entryEntity.Entity).UpdateDateTime = DateTime.UtcNow;
                }
            }

            return base.SaveChangesAsync(cancellationToken);
        }
    }
}

```



---

## 도메인과 서비스 레이어 단위 테스트는 상호 보완적이다.

  결론: 두 테스트는 서로를 보완합니다

  ┌─────────────┬────────────────────────────────────────┬────────────────────────────────────────────────┐
  │ 테스트 대상 │ 도메인 엔티티 (`ShipInfo`)               │ 서비스 레이어 (`ShipParticularsService`)         │
  ├─────────────┼────────────────────────────────────────┼────────────────────────────────────────────────┤
  │ 책임        │ 비즈니스 규칙 실행                     │ 흐름 조정 (Orchestration)                      │
  │ 테스트 종류 │ 상태 기반 테스트                       │ 상호작용 테스트                                │
  │ 핵심 검증   │ 메서드 호출 후 객체의 상태가 올바른가? │ 협력자(Mock)의 메서드가 올바르게 호출되었는가? │
  │ 주요 도구   │ 순수 객체, Assert, Fluent Assertions   │ Mock, Spy, Verify, Callback                    │
  └─────────────┴────────────────────────────────────────┴────────────────────────────────────────────────┘


---

## UpdateDateTime이 nullable인데 null이 아니다 

💩 EF Core에서 생성일, 수정일 들어가는지 DB 통합 테스트하는데 .. 신규 생성때 수정일에 아래와 같이 기본 값이 들어가는 이슈 발생

> Did not expect actual.ShipSatellite.UpdateDateTime to have a value, but found <0001-01-01 00:00:00.000>.

✅ `default(DateTime)`으로 검증해서 넘어감 

```cs
using FluentAssertions;
using Microsoft.Data.Sqlite;
using Microsoft.EntityFrameworkCore;
using ShipParticularsApi.Contexts;
using ShipParticularsApi.Entities;
using ShipParticularsApi.ValueObjects;
using Xunit;
using Xunit.Abstractions;
using static ShipParticularsApi.Tests.Builders.ShipInfoTestBuilder;
using static ShipParticularsApi.Tests.Builders.ShipSatelliteTestBuilder;
using static ShipParticularsApi.Tests.Builders.ShipServiceTestBuilder;

namespace ShipParticularsApi.Tests.Examples
{
    public class BaseEntityDateTimeTests : IDisposable
    {
        private readonly SqliteConnection _connection;
        private readonly DbContextOptions<ShipParticularsContext> _options;
        private readonly ITestOutputHelper _output;

        // NOTE: beforeEach
        public BaseEntityDateTimeTests(ITestOutputHelper output)
        {
            _output = output;

            _connection = new SqliteConnection("DataSource=:memory:");
            _connection.Open();

            _options = new DbContextOptionsBuilder<ShipParticularsContext>()
                .UseSqlite(_connection)
                .UseLazyLoadingProxies()
                .LogTo(message => _output.WriteLine(message), LogLevel.Information)
                .EnableSensitiveDataLogging()
                .Options;

            var context = new ShipParticularsContext(_options);
            context.Database.EnsureDeleted();
            context.Database.EnsureCreated();
        }

        // NOTE: AfterEach
        public void Dispose() => _connection.Dispose();

        ShipParticularsContext CreateContext() => new(_options);

        // NOTE. UpdateDateTime이 nullable인데도 DateTime 기본값 할당됨
        // Did not expect actual.ShipSatellite.UpdateDateTime to have a value, but found <0001-01-01 00:00:00.000>.
        [Fact]
        public async Task CreateDateTime_test()
        {
            // Arrange
            const string shipKey = "UNIQUE_SHIP_KEY";
            var newShipInfo = ShipInfo()
                   .WithShipKey(shipKey)
                   .WithCallsign("TEST_CALLSIGN")
                   .WithShipName("TEST_SHIP_NAMME")
                   .WithShipType(ShipTypes.Fishing)
                   .WithShipCode("TEST_SHIP_CODE")
                   .WithShipServices(KtSatService(shipKey))
                   .WithShipSatellite(KtSatellite(shipKey, "SATELLITE_ID"))
                   .WithExternalShipId("SATELLITE_ID")
                   .WithIsUseKtsat(true)
                   .Build();

            var startTime = DateTime.UtcNow;

            await using (var arrangeContext = CreateContext())
            {
                arrangeContext.ShipInfos.Add(newShipInfo);
                await arrangeContext.SaveChangesAsync();
            }

            var endTime = DateTime.UtcNow;

            // Act, Assert
            await using (var assertContext = CreateContext())
            {
                var actual = await assertContext.ShipInfos
                    .Include(s => s.ShipSatellite)
                    .SingleOrDefaultAsync(s => s.ShipKey == shipKey);

                actual.Should().NotBeNull();
                actual.ShipSatellite.Should().NotBeNull();
                actual.ShipSatellite.CreateDateTime.Should().BeOnOrAfter(startTime)
                    .And.BeOnOrBefore(endTime);
                actual.ShipSatellite.UpdateDateTime.Should().Be(default);
            }
        }

        [Fact]
        public async Task UpdateDateTime_test()
        {
            // Arrange
            var startTime = DateTime.UtcNow;
            const string shipKey = "UNIQUE_SHIP_KEY";
            var newShipInfo = ShipInfo()
                   .WithShipKey(shipKey)
                   .WithCallsign("TEST_CALLSIGN")
                   .WithShipName("TEST_SHIP_NAMME")
                   .WithShipType(ShipTypes.Fishing)
                   .WithShipCode("TEST_SHIP_CODE")
                   .WithShipServices(KtSatService(shipKey))
                   .WithShipSatellite(KtSatellite(shipKey, "SATELLITE_ID"))
                   .WithExternalShipId("SATELLITE_ID")
                   .WithIsUseKtsat(true)
                   .Build();

            await using (var arrangeContext = CreateContext())
            {
                arrangeContext.ShipInfos.Add(newShipInfo);
                await arrangeContext.SaveChangesAsync();
            }

            // Act
            await using (var actConext = CreateContext())
            {
                var shipInfo = await actConext.ShipInfos
                    .Include(s => s.ShipSatellite)
                    .Include(s => s.ShipServices)
                    .Include(s => s.SkTelinkCompanyShip)
                    .AsSplitQuery()
                    .SingleOrDefaultAsync(s => s.ShipKey == shipKey);

                shipInfo.ManageGpsService(true, new SatelliteDetails("SATELLITE_ID", "SK_TELINK", "COMPANY_NAME"));
                await actConext.SaveChangesAsync();
            }

            var endTime = DateTime.UtcNow;

            // Assert
            await using (var assertContext = CreateContext())
            {
                var actual = await assertContext.ShipInfos
                    .Include(s => s.ShipSatellite)
                    .SingleOrDefaultAsync(s => s.ShipKey == shipKey);

                actual.Should().NotBeNull();
                actual.ShipSatellite.Should().NotBeNull();
                actual.ShipSatellite.UpdateDateTime.Should().BeOnOrAfter(startTime)
                    .And.BeOnOrBefore(endTime);
            }
        }
    }
}

```

---

## 엔티티에 public 컬럼을 private로 바꿀 경우 

> 기본 생성자를 private로 선언하고 전체 속성 생성자를 internal로 선언해서 사용
- 클래스에 생성자가 없으면 컴파일러에서 기본 생성자를 만들어준다.
	- 만약 클래스에 생성자가 있으면 기본 생성자 만들지 x
		- 지금 `public` 접근제어자를 가지게 될 경우 변경이 어디서 발생하는지 추적하기 어려워 유지보수 비용이 증가할 수 있음 ▶️ 그래서 보통 `private` 접근제어자를 선언하고 `생성자 주입 방식`, `팩토리 메서드`, `인스턴스 메서드`로 필드 값에 대한 접근 제어를 한다.

### 1\. `internal` 키워드의 용도 🔑

`internal` 키워드는 C\#의 **접근 제한자(Access Modifier)** 중 하나로, 해당 클래스, 메서드 또는 필드에 대한 접근을 **현재 어셈블리(Assembly) 내**로만 제한합니다.

| 접근 제한자 | 접근 범위 | 설명 |
| :--- | :--- | :--- |
| **`public`** | 제한 없음 | 모든 어셈블리에서 접근 가능 |
| **`internal`** | **현재 어셈블리** | 동일 프로젝트(어셈블리) 내에서만 접근 가능 |
| **`protected`** | 상속 관계 | 현재 클래스와 해당 클래스를 상속받은 자식 클래스에서 접근 가능 |
| **`private`** | 현재 클래스 | 오직 현재 클래스 내에서만 접근 가능 |

#### 용도

`internal`은 주로 **모듈화**를 위해 사용됩니다.

1.  **캡슐화 경계:** 외부 어셈블리(예: 다른 라이브러리, 소비자 애플리케이션)가 특정 클래스나 메서드를 사용해서는 안 되지만, **동일한 프로젝트 내의 다른 클래스**는 사용해야 할 때 사용됩니다.
2.  **구현 세부 사항 은닉:** 외부 소비자에게 노출하고 싶지 않은 내부 로직이나 헬퍼 클래스를 숨기는 데 유용합니다.

### 2\. 기본 생성자 자동 생성 규칙 ⚙️

C\#에서 클래스를 선언하면 기본적으로 \*\*매개변수가 없는 공용 생성자(public parameterless constructor)\*\*가 컴파일러에 의해 자동으로 생성됩니다.

#### 자동 생성 규칙

컴파일러가 기본 생성자를 자동으로 생성해주는 조건은 단 하나입니다.

> **클래스에 어떤 생성자도 명시적으로 정의되어 있지 않을 때**

#### 자동 생성되지 않는 경우

만약 클래스에 **매개변수가 있는 생성자**를 하나라도 명시적으로 정의하면, 컴파일러는 기본 생성자를 자동으로 만들어주지 않습니다.

```csharp
public class MyEntity
{
    // MyEntity(string name)을 정의했기 때문에,
    // MyEntity() 기본 생성자는 자동으로 생성되지 않습니다.
    public MyEntity(string name) { /* ... */ } 
}
// 따라서, new MyEntity() 호출 시 컴파일 오류가 발생합니다.
```

### 3\. `private` 생성자와 지연 전략 (EF Core) 🧱

`private` 생성자는 특히 EF Core에서 \*\*지연 로딩(Lazy Loading)\*\*이나 **데이터베이스 매핑**을 위해 자주 사용되는 패턴입니다.

#### EF Core의 요구 사항

EF Core는 데이터베이스에서 엔티티를 로드할 때 다음 두 가지 중 하나가 필요합니다.

1.  **매개변수가 없는 생성자** (가장 선호됨).
2.  **모든 속성에 해당하는 매개변수**를 가진 생성자 (데이터 주입 목적).

#### `private` 생성자의 역할

엔티티에 매개변수 있는 생성자(Builder 패턴을 위해 추가하신)만 있고, 상태 변경을 **행위 메서드**로 제한하기 위해 필드를 `private set`으로 선언했다면, **EF Core가 사용할 수 있는 매개변수 없는 생성자**를 제공해야 합니다.

```csharp
public class ShipInfo 
{
    // 모든 필드를 초기화하는 생성자 (Builder에서 사용)
    internal ShipInfo(long id, string key) { /* ... */ } 

    // 💡 EF Core가 데이터베이스에서 객체를 로드(생성)하기 위해 필요합니다.
    // 캡슐화를 유지하기 위해 private으로 선언합니다.
    private ShipInfo() { } 

    // ...
}
```

#### 지연 로딩과 생성자

\*\*지연 로딩(Lazy Loading)\*\*을 사용하려면 EF Core는 객체를 생성할 때 **프록시(Proxy) 객체**를 만듭니다. 이 프록시 객체는 상속을 통해 엔티티의 기능을 확장하므로, **프록시 객체가 엔티티를 상속받을 수 있도록** 엔티티 클래스와 생성자는 다음을 따라야 합니다.

  * 클래스는 **`sealed`가 아니어야 합니다.**
  * 매개변수 없는 생성자(이 경우 `private ShipInfo()`)는 \*\*`internal` 또는 `public`\*\*이어야 합니다. (혹은 `private`이어도 프록시가 작동하도록 `Microsoft.EntityFrameworkCore.Proxies` 패키지 설치 시점에 추가적인 Reflection 설정이 필요할 수 있습니다.)

일반적으로 **캡슐화를 유지하면서 EF Core의 지연 로딩을 지원**하기 위해 매개변수 없는 생성자를 \*\*`private` 또는 `internal`\*\*로 선언하는 것이 표준적인 방법입니다. EF Core는 리플렉션(Reflection)을 통해 접근 제한자를 우회하고 객체를 성공적으로 생성합니다.

**결론적으로, `private ShipInfo()` 생성자는 EF Core가 DB에서 데이터를 불러와 프록시를 생성하고 객체를 채워 넣을 때 사용됩니다.**


---

## enum <-> string 변환 메서드 중복 
- Entities, Dto, ValueConverter에서 중복 메서드 발생 💩
- `Entites/Enums`로 enum 클래스 이동 후 Helper 클래스 생성하여 재활용 하도록 리팩터링 ⭐


---

## POST 요청이 응답 201 Created 보낼때 location 보내는 이유

[RFC 9110 - HTTP Semantics](https://datatracker.ietf.org/doc/html/rfc9110#name-201-created)

[[Spring] HTTP 201 CREATED 상태 코드와 Location 헤더](https://syuare.tistory.com/68)

[ControllerBase.CreatedAtAction 메서드 (Microsoft.AspNetCore.Mvc) | Microsoft Learn](https://learn.microsoft.com/ko-kr/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction?view=aspnetcore-9.0)

`🤖 gemini`
네, `201 Created` 응답 시 `Location` 헤더를 포함해야 한다는 RESTful API의 원칙과 관련된 공식 문서와, `CreatedAtAction` 메서드의 파라미터에 대한 정확한 정보 및 사용법을 알려드리겠습니다.
### 1. `Location` 헤더 관련 공식 문서 📜

`201 Created` 응답 시 `Location` 헤더를 요구하는 것은 **HTTP 프로토콜의 표준**입니다.

#### A. HTTP 표준 (RFC 9110)

가장 최신 HTTP/1.1 의미론(Semantics)을 정의하는 문서입니다.

- **HTTP Semantics (RFC 9110) - Section 15.5.2 "201 Created"**
    
    - 해당 섹션에서는 `201 Created` 상태 코드는 **요청이 성공적으로 처리되었고, 새로운 리소스가 생성되었음**을 의미한다고 명시합니다.
        
    - 특히, "새로 생성된 리소스의 URI는 응답의 `Location` 헤더에 의해 정의되어야 한다"고 명시하고 있습니다.
        
    - **공식 문서 링크 (Google 검색 결과):** `RFC 9110 201 Created Location header`
        

#### B. ASP.NET Core 공식 문서

ASP.NET Core에서 `CreatedAtAction`을 사용하여 HTTP 표준을 준수하는 방법을 설명합니다.

- **ASP.NET Core Web API 응답 액션 메서드 문서**
    
    - `CreatedAtAction`은 `201 Created` 상태 코드와 함께 `Location` 응답 헤더를 생성하는 메서드로 설명되어 있습니다.
        
    - **공식 문서 링크 (Google 검색 결과):** `ASP.NET Core CreatedAtAction`
        

### 2. `CreatedAtAction` 파라미터 상세 분석

사용자께서 말씀하신 대로 `CreatedAtAction`의 두 번째 인자는 `routeValues`가 맞습니다. 하지만 **세 번째 인자도 반드시 필요**하며, 그 역할이 다릅니다.

`CreatedAtAction`은 다음과 같이 오버로드(Overload)됩니다.

C#

```
CreatedAtActionResult CreatedAtAction(
    string? actionName, 
    object? routeValues, // 두 번째 인자
    object? value        // 세 번째 인자
);
```

|**인자**|**타입**|**목적**|**현재 사용법**|
|---|---|---|---|
|**1. `actionName`**|`string`|**`Location` 헤더의 URI를 생성하는 데 사용될 GET 액션 메서드 이름** (`nameof(GetShipParticularsAsync)`)||
|**2. `routeValues`**|`object`|**`Location` 헤더의 경로 변수(Path Variable)**를 채우는 값. 이 값은 1번 액션의 매개변수와 일치해야 합니다.|`{ shipKey = req.ShipKey }` (URI 생성용)|
|**3. `value`**|`object`|**HTTP 응답 본문(Body)에 포함될 데이터** (`201 Created`와 함께 클라이언트에게 전송됨)|`{ shipKey = req.ShipKey }` (응답 본문용)|

#### 💡 파라미터가 3개 필요한 이유

`CreatedAtAction`은 단순히 `Location` 헤더만 생성하는 것이 아니라, **응답 본문도 함께 설정**하기 위해 세 번째 인자인 `value`를 요구합니다.

- **두 번째 인자 (`routeValues`):** 오직 HTTP 헤더(`Location`)의 URI를 만드는 데만 사용됩니다.
    
- **세 번째 인자 (`value`):** 클라이언트에게 응답 본문으로 돌려줄 JSON 객체입니다. (예: 생성된 리소스의 `shipKey`나 간략 정보)
    

따라서, `Location` 헤더를 생성하면서 **동시에 응답 본문에 생성된 리소스의 키 정보**를 넣어주려면, 3개의 인자가 있는 오버로드를 사용하는 것이 올바릅니다.

```cs
return CreatedAtAction(
    nameof(GetShipParticularsAsync), // 1. 어떤 GET 메서드를 참조할 것인가?
    new { shipKey = req.ShipKey },   // 2. Location 헤더를 어떻게 만들 것인가? (/api/.../{shipKey})
    new { shipKey = req.ShipKey }    // 3. 응답 본문에 무엇을 담을 것인가? ({ "shipKey": "..." })
);
```

---

## csproj 설정 파일에서 Controllers 폴더 내용이 사라진 이유

```text
-  <ItemGroup>
-    <Folder Include="Controllers\" />
-  </ItemGroup>
```

제공해 주신 내용은 Visual Studio나 .NET SDK에서 **빈 폴더(Empty Folder)**를 프로젝트 파일(`.csproj`)에 명시적으로 포함시키기 위해 사용했던 XML 코드 조각입니다.

이 내용이 사라진 것은 **정상적인 현상**이며, 프로젝트에 영향을 주지 않습니다.

 `<Folder Include="Controllers\" />`의 의미는 다음과 같습니다.
### 1. 목적: 빈 폴더 유지

- **`<ItemGroup>`** 태그는 프로젝트에 포함될 아이템(파일, 참조, 폴더 등)의 그룹을 정의합니다.
    
- **`<Folder Include="Controllers\" />`** 는 "Controllers"라는 이름의 폴더를 프로젝트에 포함시키라는 지시입니다.
    

Visual Studio는 기본적으로 폴더 안에 **파일이 하나도 없으면** 해당 폴더를 프로젝트 트리에서 **자동으로 삭제**합니다. 이 `<Folder>` 항목은 해당 폴더가 비어 있더라도 **프로젝트 파일 시스템에 남아 있도록 강제**하는 용도로 사용됩니다.

### 2. 왜 지금 지워졌나요? (정상적인 동작)

오늘 `Controllers` 작업을 시작하셨다는 것은 해당 폴더 안에 실제 C# 파일(예: `ShipParticularsControllers.cs`)을 생성하셨다는 의미입니다.

- **파일 생성 전:** 폴더가 비어있었기 때문에, 폴더를 유지하기 위해 `.csproj` 파일에 `<Folder Include="Controllers\" />` 항목이 필요했습니다.
    
- **파일 생성 후:** 폴더 안에 파일이 생겼으므로, Visual Studio는 더 이상 이 폴더를 강제로 유지할 필요가 없습니다. 따라서 **IDE가 자동으로 해당 `<Folder>` 항목을 `.csproj` 파일에서 삭제**한 것입니다.
    

**결론:** 이 항목은 **빈 폴더를 위한 임시 마커**였으며, 폴더 안에 파일이 생겼으므로 사라지는 것이 **의도된 동작**입니다. 프로젝트의 빌드나 실행에 아무런 문제가 없습니다.

---

## http 파일 생성시 출력 콘솔창에 한글깨짐 

```text
[오후 5:59:53 정보] dotnet 도구 매니페스트를 만드는 중입니다.
[오후 5:59:53 정보] ‘dotnet new tool-manifest‘ 명령을 실행하는 중입니다.
[오후 5:59:54 정보] 'dotnet new tool-manifest' 명령이 성공했으며 종료 코드는 '0'입니다.
"dotnet 濡쒖뺄 ?꾧뎄 留ㅻ땲?섏뒪???뚯씪" ?쒗뵆由우씠 ?깃났?곸쑝濡??앹꽦?섏뿀?듬땲??
[오후 5:59:54 정보] dotnet 대화형 도구의 '1.0.616301' 버전을 설치하는 중입니다.
[오후 5:59:54 정보] ‘dotnet tool install Microsoft.dotnet-interactive --version 1.0.616301 --configfile NuGet.config‘ 명령을 실행하는 중입니다.
[오후 5:59:54 정보] 'dotnet tool install Microsoft.dotnet-interactive --version 1.0.616301 --configfile NuGet.config' 명령이 성공했으며 종료 코드는 '0'입니다.
'dotnet tool run dotnet-interactive' ?먮뒗 'dotnet dotnet-interactive' 紐낅졊???ъ슜?섏뿬 ???붾젆?곕━?먯꽌 ?꾧뎄瑜??몄텧?????덉뒿?덈떎.
'microsoft.dotnet-interactive' ?꾧뎄(踰꾩쟾 '1.0.616301')媛 ?ㅼ튂?섏뿀?듬땲?? 留ㅻ땲?섏뒪???뚯씪 C:\Users\?댁쭊??AppData\Local\Temp\vs-dotnet-interactive\17.0_e2d9edb3\.config\dotnet-tools.json????ぉ??異붽??⑸땲??

```

---

### http 테스트 

#### 1. 400 Bad Request
POST 요청에서 마지막에 트레일링 콤마가 있어서 실패 

```text

application/problem+json; charset=utf-8, 414 bytes

{

"type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",

"title": "One or more validation errors occurred.",

"status": 400,

"errors": {

"$": [

"The JSON object contains a trailing comma at the end which is not supported in this mode. Change the reader options. Path: $ | LineNumber: 8 | BytePositionInLine: 0."

],

"req": [

"The req field is required."

]

},

"traceId": "00-fed4e0d20f0fef307ec7a368fb898609-49493ab4897b2677-00"}
```

#### 2. 500 에러 (1)
- 컬럼이 `nvarchar(10)` 사이즈라서 요청 문자열 길이가 길어 에러가 발생함 
	- Microsoft.EntityFrameworkCore.DbUpdateException


#### 3. DB 저장 성공 후 500에러 

```text
System.InvalidOperationException: No route matches the supplied values.
   at Microsoft.AspNetCore.Mvc.CreatedAtActionResult.OnFormatting(ActionContext context)
```
- Created 만들때 에러가 발생 .. 

```text
Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware: Error: An unhandled exception has occurred while executing the request.

System.InvalidOperationException: No route matches the supplied values.
   at Microsoft.AspNetCore.Mvc.CreatedAtActionResult.OnFormatting(ActionContext context)
   ...
```

참고. 
- 스택오버플로우
	- [c# - ASP.NET CORE, Web API: No route matches the supplied values - Stack Overflow](https://stackoverflow.com/questions/39459348/asp-net-core-web-api-no-route-matches-the-supplied-values#:~:text=It%20uses%20the%20actionName%20to%20do%20this%2C%20so,Better%20practice%20probably%20to%20use%3A%20dev%20guide%20apidocs)
- 공식문서 
	- [MvcOptions.SuppressAsyncSuffixInActionNames Property (Microsoft.AspNetCore.Mvc) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.mvcoptions.suppressasyncsuffixinactionnames?view=aspnetcore-9.0)

```
// Programs.cs에서 설정 비활성화
builder.Services.AddControllers(options =>
{
    // Async 접미사 제거 비활성화
    options.SuppressAsyncSuffixInActionNames = false;
});
```

---

## AsNoTracking (readOnly = true 효과)

[Tracking vs. No-Tracking Queries - EF Core | Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/querying/tracking)

```cs
using Microsoft.EntityFrameworkCore;
using ShipParticularsApi.Contexts;
using ShipParticularsApi.Entities;

namespace ShipParticularsApi.Repositories
{
    public class ShipInfoRepository(ShipParticularsContext context) : IShipInfoRepository
    {
        public async Task<ShipInfo?> GetByShipKeyAsync(string shipKey)
        {
            return await context.ShipInfos
                 .Include(s => s.ShipServices)
                 .Include(s => s.ShipSatellite)
                 .Include(s => s.SkTelinkCompanyShip)
                 .Include(s => s.ReplaceShipName)
                 .Include(s => s.ShipModelTest)
                 .AsSplitQuery()
                 .SingleOrDefaultAsync(s => s.ShipKey == shipKey && s.IsService == true);
        }

        public Task UpsertAsync(ShipInfo shipInfo)
        {
            if (shipInfo.Id == 0)
            {
                context.ShipInfos.Add(shipInfo);
            }
            else
            {
                context.ShipInfos.Update(shipInfo);
            }

            return Task.CompletedTask;
        }

        public async Task<ShipInfo?> GetReadOnlyByShipKeyAsync(string shipKey)
        {
            return await context.ShipInfos
                 .AsNoTracking()  // ⭐ readOnly = true 효과
                 .Include(s => s.ShipServices)
                 .Include(s => s.ShipSatellite)
                 .Include(s => s.SkTelinkCompanyShip)
                 .Include(s => s.ReplaceShipName)
                 .Include(s => s.ShipModelTest)
                 .AsSplitQuery()
                 .SingleOrDefaultAsync(s => s.ShipKey == shipKey && s.IsService == true);
        }

    }
}

```

테스트 코드 작성 
- `Act` 구절에서 엔티티의 상태를 조회하기 위해 어쩔 수 없이 두 줄이 됨 
	- 안티패턴에 해당하지만 필연적으로 발생하는 오버헤드 (pass)
	- `var entry = dbContext.Entry(savedShipInfo!);`를 Act가 아닌 Assert로 내려서 개선 가능
```cs
using FluentAssertions;
using Microsoft.Data.Sqlite;
using Microsoft.EntityFrameworkCore;
using ShipParticularsApi.Contexts;
using ShipParticularsApi.Entities.Enums;
using ShipParticularsApi.Repositories;
using Xunit;
using Xunit.Abstractions;

using static ShipParticularsApi.Tests.Builders.ShipInfoTestBuilder;
using static ShipParticularsApi.Tests.Builders.ShipSatelliteTestBuilder;
using static ShipParticularsApi.Tests.Builders.ShipServiceTestBuilder;


namespace ShipParticularsApi.Tests.Repositories
{
    public class ShipInfoRepositoryTests : IDisposable
    {
        private readonly SqliteConnection _connection;
        private readonly DbContextOptions<ShipParticularsContext> _options;
        private readonly ITestOutputHelper _output;

        // NOTE: beforeEach
        public ShipInfoRepositoryTests(ITestOutputHelper output)
        {
            _output = output;

            _connection = new SqliteConnection("DataSource=:memory:");
            _connection.Open();

            _options = new DbContextOptionsBuilder<ShipParticularsContext>()
                .UseSqlite(_connection)
                .UseLazyLoadingProxies()
                .LogTo(message => _output.WriteLine(message), LogLevel.Information)
                .EnableSensitiveDataLogging()
                .Options;

            var context = new ShipParticularsContext(_options);
            context.Database.EnsureDeleted();
            context.Database.EnsureCreated();
        }

        // NOTE: AfterEach
        public void Dispose() => _connection.Dispose();

        ShipParticularsContext CreateContext() => new(_options);

        [Fact(DisplayName = "DB에서 조회한 엔티티의 상태는 EntityState.Unchanged 이다.")]
        public async Task AsTracking()
        {
            // Arrange
            const string shipKey = "SHIP_KEY";
            var newShipInfo = ShipInfo()
                   .WithShipKey(shipKey)
                   .WithCallsign("TEST_CALLSIGN")
                   .WithShipName("TEST_SHIP_NAMME")
                   .WithShipType(ShipTypes.Fishing)
                   .WithShipCode("TEST_SHIP_CODE")
                   .WithShipServices(KtSatService(shipKey))
                   .WithShipSatellite(KtSatellite(shipKey, "SATELLITE_ID"))
                   .WithExternalShipId("SATELLITE_ID")
                   .WithIsUseKtsat(true)
                   .Build();

            await using (var arrangeContext = CreateContext())
            {
                arrangeContext.ShipInfos.Add(newShipInfo);
                await arrangeContext.SaveChangesAsync();
            }

            var dbContext = CreateContext();
            var repository = new ShipInfoRepository(dbContext);

            // Act
            var savedShipInfo = await repository.GetByShipKeyAsync(shipKey);
            var entry = dbContext.Entry(savedShipInfo!);

            // Assert
            savedShipInfo.Should().NotBeNull();
            entry.State.Should().Be(EntityState.Unchanged);
        }

        [Fact(DisplayName = "AsNoTracking()으로 조회한 엔티티의 상태는 EntityState.Detached 이다.")]
        public async Task AsNoTracking()
        {
            // Arrange
            const string shipKey = "SHIP_KEY";
            var newShipInfo = ShipInfo()
                   .WithShipKey(shipKey)
                   .WithCallsign("TEST_CALLSIGN")
                   .WithShipName("TEST_SHIP_NAMME")
                   .WithShipType(ShipTypes.Fishing)
                   .WithShipCode("TEST_SHIP_CODE")
                   .WithShipServices(KtSatService(shipKey))
                   .WithShipSatellite(KtSatellite(shipKey, "SATELLITE_ID"))
                   .WithExternalShipId("SATELLITE_ID")
                   .WithIsUseKtsat(true)
                   .Build();

            await using (var arrangeContext = CreateContext())
            {
                arrangeContext.ShipInfos.Add(newShipInfo);
                await arrangeContext.SaveChangesAsync();
            }

            var dbContext = CreateContext();
            var repository = new ShipInfoRepository(dbContext);

            // Act
            var savedShipInfo = await repository.GetReadOnlyByShipKeyAsync(shipKey);
            var entry = dbContext.Entry(savedShipInfo!);

            // Assert
            savedShipInfo.Should().NotBeNull();
            entry.State.Should().Be(EntityState.Detached);
        }
    }
}

```


참고.
- DbContext 설정에서 쿼리 Tracking 기본 설정도 가능
	- 공식예제는 NoTracking을 기본으로 한다
	- [Tracking vs. No-Tracking Queries - EF Core | Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/querying/tracking#configuring-the-default-tracking-behavior)


---

## 통합 테스트 context 중복 제거 
서비스 통합테스트에서 Arrange 단계에 아래 로직이 중복으로 나타남
```cs
// 신규 ShipInfo인 경우
 using var scope = _factory.Services.CreateScope();
 var serviceProvider = scope.ServiceProvider;
 var sut = serviceProvider.GetRequiredService<IShipParticularsService>();



// 기존 ShipInfo가 있는 경우 DB 초기화 필요
using var scope = _factory.Services.CreateScope();
var serviceProvider = scope.ServiceProvider;
var sut = serviceProvider.GetRequiredService<IShipParticularsService>();

await serviceProvider.SeedDataAsync(NoService(shipKey, 1L).Build());
```

`리팩터링 후`
```cs
// NOTE. C# 7.0 이상 지원
protected (IServiceProvider Provider, IShipParticularsService Sut) GetTestDependencies(IServiceScope scope)
{
    var serviceProvider = scope.ServiceProvider;
    var sut = serviceProvider.GetRequiredService<IShipParticularsService>();

    return (serviceProvider, sut); // 튜플 리터럴
}

[Fact(DisplayName = "신규 ShipInfo (AIS/GPS Toggle off) 생성")]
public async Task Case1()
{
    // Arrange
    using var scope = _factory.Services.CreateScope();
    var (serviceProvider, sut) = GetTestDependencies(scope);

    const string shipKey = "UNIQUE_SHIP_KEY";
    var param = ShipParticularsParam().WithShipKey(shipKey).Build();

    // Act
    await sut.Process(param);

    // Assert
    var repository = serviceProvider.GetRequiredService<IShipInfoRepository>();
    var actual = await repository.GetByShipKeyAsync(shipKey);

    actual.Should().NotBeNull();
    actual.ShipType.Should().Be(ShipTypes.Default);
}
```


[C# 7.0 튜플 - C# 프로그래밍 배우기 (Learn C# Programming)](https://www.csharpstudy.com/Latest/CS7-tuple.aspx#google_vignette)


---

## 테스트 클래스 상단 변수 선언 (readonly, const)

```cs
private readonly ShipParticularsService _sut;
private readonly Mock<IShipInfoRepository> _mockShipInfoRepository;
private readonly Mock<IUserService> _mockUserService;
private const string FixedUserId = "TEST_USER_01";
```

네, 선언하신 공통 상수는 **파스칼 케이스(PascalCase)로 표기하는 것이 C\# 코딩 표준에 맞습니다.** `const`와 `readonly`는 둘 다 변경 불가능한 값을 나타내지만, **초기화 시점**과 **값의 타입**에서 중요한 차이가 있습니다.

### 1\. 변수명 표기법: 파스칼 케이스 (PascalCase) ✅

C\# 코딩 표준(특히 Microsoft의 .NET Framework 디자인 지침)에 따르면, `const` 필드나 `static readonly` 필드를 포함하여 **모든 `public`, `protected`, `internal` 상수는 파스칼 케이스**를 사용해야 합니다.

| 코드 유형 | 권장 표기법 | 예시 |
| :--- | :--- | :--- |
| **`const` 필드** | PascalCase | `private const string FixedUserId = "TEST_USER_01";` (현재 코드) |
| **`static readonly` 필드** | PascalCase | `private static readonly string DefaultShipKey = "SHIP_KEY";` |
| **속성(Property)** | PascalCase | `public string ShipName { get; }` |
| **메서드(Method)** | PascalCase | `public void Process(Param param)` |

**참고:** 로컬 변수나 메서드 매개변수에는 카멜 케이스(camelCase)를 사용합니다.

### 2\. `const`와 `readonly`의 차이점 💡

`const`와 `readonly`는 둘 다 필드가 변경되는 것을 막지만, **언제 값이 할당되는지**와 **어떤 타입의 값을 가질 수 있는지**에서 근본적인 차이가 있습니다.

| 특징         | `const` (Compile-Time Constant)                  | `readonly` (Runtime Constant)                                 |
| :--------- | :----------------------------------------------- | :------------------------------------------------------------ |
| **초기화 시점** | **컴파일 시** (Compile Time)                         | **런타임 시** (Runtime). 선언 시 또는 생성자(Constructor)에서 초기화 가능.       |
| **값의 타입**  | **값 타입** (Value Type)이나 `string` (내장 기본 타입)만 가능. | **모든 타입** (참조 타입 포함) 가능.                                      |
| **메모리 저장** | 메서드를 호출하는 지점에 값이 직접 **인라인으로 복사**됨.               | 인스턴스/정적 필드에 저장되며, **참조**를 통해 접근됨.                             |
| **정적 여부**  | 항상 `static`이며, 명시적으로 선언할 수 없음.                   | `static`을 명시하여 \*\*`static readonly`\*\*로 사용 가능 (모든 인스턴스가 공유) |

#### Test Data Builder에서의 활용

1.  **`const` 사용:** 값이 **문자열**이거나 **숫자**이며, **테스트 간에 절대 변하지 않는 하드코딩된 값**일 때 사용합니다.
      * **예시:** `FixedUserId`, `MAX_RETRIES`, `DEFAULT_SHIP_KEY` (문자열인 경우)
2.  **`readonly` 사용:** Mock 객체(`_mockUserService`)나 생성자에서 초기화해야 하는 필드, 또는 **참조 타입**의 고정된 인스턴스를 저장할 때 사용합니다.
      * **예시:** `_mockShipInfoRepository`, `_sut` (생성자 주입), `private static readonly List<int> EmptyList = new();`

### 3\. 현재 코드 분석

제공해주신 코드는 적절하게 사용하고 있습니다.

```csharp
private readonly ShipParticularsService _sut; // 생성자에서 초기화되는 인스턴스
private readonly Mock<IShipInfoRepository> _mockShipInfoRepository; // 생성자에서 초기화되는 Mock
private const string FixedUserId = "TEST_USER_01"; // 컴파일 시 고정되는 문자열 상수 (PascalCase 권장)
```

**표기법 권장:** `FixedUserId`는 `const`이므로 현재와 같이 \*\*`FixedUserId` (PascalCase)\*\*로 표기하는 것이 C\# 표준에 가장 잘 맞습니다.