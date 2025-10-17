
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


`CASE1` 
- Toggle Off 이고, ShipInfo 신규 생성인 경우

`Case2` AIS Toggle : **ON** 인데 
- SHIP_SERVICE가 없는 경우 ▶️ 신규 SHIP_SERVICE 생성
- SHIP_SERVICE가 있는 경우 

`Case3` AIS Toggle : **Off** 인데
- SHIP_SERVICE가 없는 경우
- SHIP_SERVICE가 있는 경우 ▶️ SHIP_SERVICE 삭제

`Case4` GPS Toggle : **Off**
- SHIP_SERVICE가 없는 경우
- SHIP_SERVICE가 있는 경우  ▶️ DELETE  

`Case5` GPS Toggle : **On**
- SHIP_SERVICE가 없는 경우
- SHIP_SERVICE가 있는 경우

`Case6`
- ShipInfo가 이미 생성되어 있는 경우


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

### 연관관계 참조 끊기와 DELETE 쿼리 여부에 대해 (251016)
연관관계 참조를 Null로 끊을때 더티 체킹시 DELETE 쿼리가 만들어지는가?
- 우선은 양방향 연관관계이고, ShipInfo 엔티티에서 하위 자식 엔티티에 대한 생명주기를 관리하고 있다. 
- ✅ 일대다 관계에 해당하는 ShipService의 경우, 컬렉션의 Remove 호출시 DELETE 호출되는 것을 확인 
- ✅ GPS 관련 하위 엔티티의 경우, 일대일 참조에 대해 null로 참조 끊을 시 DELETE 쿼리 날아가는 것을 확인함.
- 📌 FK가 non-nullable인 경우에만 EF Core에서 인식하고 DELETE 쿼리를 만든다는데 자세한 내용은 공식 문서 학습 필요
	- FK가 nullable인 경우 UPDATE 쿼리가 나간다고 함

<img src="연관관계 참조를 null로 끊을 경우 결과.png">

✅ 공식문서 - [Cascade Delete - EF Core | Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/saving/cascade-delete#cascading-nulls)

---

### 서비스 레이어 통합 테스트 

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


