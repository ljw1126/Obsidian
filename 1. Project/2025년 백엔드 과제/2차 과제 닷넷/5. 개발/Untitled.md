
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