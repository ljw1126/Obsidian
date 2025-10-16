
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