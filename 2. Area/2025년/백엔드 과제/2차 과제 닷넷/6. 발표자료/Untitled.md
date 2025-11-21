
## 2. 진행절차 
- flowchart 작성 

```text
---
config:
  theme: mc
---
flowchart LR
    A[요구사항 & 테이블 명세 <br/> 분석] --> B[**단위 테스트** 기반 <br/> 도메인, 레이어 로직 구현]
    B --> C[**통합 테스트** 기반 <br/> Service / Repository 검증]
    C --> D[전체 연결 설정 후<br/> API 호출]
    D --> E[리팩터링]
```

<img src="02-flowchart.png">


## 3. 요구사항 분석
`플로우 차트(251023)`
```text
---
config:
  theme: mc
  layout: dagre
  look: neo
---
flowchart TD
    A(["API 요청"]) --> B{"AIS Toggle On ?"}
    B -- Yes --> C{"Exist Service?"}
    C -- Yes --> G{"GPS Toggle On ?"}
    C -- No --> E["Add SHIP_SERVICE"]
    B -- No --> D{"Exist Service?"}
    D -- Yes --> F["Delete SHIP_SERVICE"]
    D -- No --> G
    E --> G
    F --> G
    G -- Yes --> H{"Exist Service ?"}
    H -- Yes --> W["Update SHIP_INFO, SHIP_SATELLITE"]
    W --> K{"Use SK Tellink?"}
    H -- No --> J["Add SHIP_SERVICE, SHIP_SATELLITE"]
    J --> K
    K -- Yes --> L["Add SK_TELINK_COMPANY_SHIP"]
    K -- No --> T["Delete SK_TELINK_COMPANY_SHIP"]
    L --> O["Upsert SHIP_INFO"]
    T --> O
    G -- No --> I{"Exist Service ?"}
    I -- Yes --> M["Delete SHIP_SERVICE, SHIP_SATELLITE, SK_TELINK_COMPANY_SHIP"]
    I -- No --> O
    M --> O
    O --> P["Upsert REPLACE_SHIP_NAME"]
    P --> Q["Upsert SHIP_MODEL_TEST"]
    Q --> R["Upsert SHIP_SATELLITE"]
    R --> S["Update SHIP_SERVICE"]
    S --> Z(["종료"])

```

<img src="04-flow chart.png"><

`클래스 다이어그램(251023`
```text
---
config:
  theme: neo
---
classDiagram
direction TB
	class ShipInfo {
		+long Id
		+string ShipKey 
    +string Callsign
    +string ShipName
    +ShipTypes ShipType
    +string ShipCode
    +string ExternalShipId
    +bool IsUseKtsat
    +bool IsService
    +bool IsUseAis
    +ICollection~ShipService~ ShipServices
    +ShipSatellite ShipSatellite
    +SkTelinkCompanyShip SkTelinkCompanyShip
    +ReplaceShipName ReplaceShipName
    +ShipModelTest ShipModelTest
    
    +From(ShipInfoDetails)$ ShipInfo
    +UpdateDetails(ShipInfoDetails) ShipInfo
    +ActiveAisService() void 
    +DeactiveAisService() void
    +HasSatAisService() bool
    +ActiveGpsService(SatelliteDetails, userId) void
    +DeactiveGpsService() void
	}
  
	class ShipService {
		+long Id
		+string ShipKey
    +ServiceNameTypes ServiceName
    +bool IsCompleted
    +Of(shipKey, ServiceNameTypes)$ ShipService
	}
	class ShipSatellite {
		+long Id
		+string ShipKey
    +SatelliteTypes SatelliteType
    +string SatelliteId
    +bool IsUseSatellite
    +string CreateUserId
    +string UpdateUserId
    +Of(shipKey, satelliteId, satelliteType, userId)$ ShipSatellite
    +Update(satelliteId, satelliteType, userId) void
    +IsSkTelink() bool
	}
	class SkTelinkCompanyShip {
		+long Id
		+string ShipKey
    +string CompanyName
    +Of(shipKey, companyName)$ SkTelinkCompanyShip
    +Update(companyName) void
	}
	class ReplaceShipName {
		+long Id
		+string ShipKey
    +string ReplacedShipName
    +From(shipKey, ReplaeShipNameDetails)$ ReplaceShipName
    +Update(ReplaceShipNameDetails) void
	}
	class ShipModelTest {
		+long Id
		+string ShipKey
    +생략..
    +From(shipKey, ShipModelTestDetails)$ ShipModelTest
    +Update(ShipModelTestDetails) void
	}
	
	
	class BaseEntity {
	 <<abstract>>
		+DateTime CreateDateTime
    +DateTime UpdateDateTime
	}
	
	ShipInfo --* ShipService
	ShipInfo --* ShipSatellite
	ShipInfo --* SkTelinkCompanyShip
	ShipInfo --* ReplaceShipName
	ShipInfo --* ShipModelTest
	
	ShipSatellite --|> BaseEntity
```

<img src="04-class diagram.png">


## 7. Reference.
- 종류별로 구분해서 나열
- 한장더 추가해서 도서 추천

```text
.NET & E.F Core  
- [공식 .NET 문서](https://learn.microsoft.com/ko-kr/dotnet/)
- [자습서: ASP.NET Core를 사용하여 컨트롤러 기반 웹 API 만들기](https://learn.microsoft.com/ko-kr/aspnet/core/tutorials/first-web-api?view=aspnetcore-8.0&tabs=visual-studio) 
- [Entity Framework Core official docs](https://learn.microsoft.com/en-us/ef/core/)
- [What is Entity Framework](https://www.luisllamas.es/en/what-is-entity-framework/)

Testing 
- [xUnit.net v2](https://xunit.net/docs/getting-started/v2/getting-started)
- [Moq Framework](https://github.com/devlooped/moq)
- [FluentAssertions](https://fluentassertions.com/)
- [aspnet core integration-tests official docs](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-8.0&pivots=xunit)
- [SQLite Memory Testing](https://github.com/dotnet/EntityFramework.Docs/blob/live/samples/core/Testing/TestingWithoutTheDatabase/SqliteInMemoryBloggingControllerTest.cs#L50
)

Tools
- ERD : Draw.io
- Flow Chart : Mermaid
- Code Image : Carbon

```


---

## 기타. 아쉬운점 
- 커스텀 Exception을 마지막에 넣어서 예외 케이스를 테스트하지 못한거
- DB에 대한 주도권이 개발자한테 있는데 DB 통합 테스트를 SQLite로 한거 
	- `TestContainer + MSSQL` 
- Controller 테스트를 작성해보지 못한거
	- 현재 서비스 레이어 기준으로 통합 테스트, 단위 테스트로 동작을 확인하다보니 
	- http 스크립트를 작성해서 API 호출 확인하는 것으로 넘어감
- 엔티티에 접근 제어자..

✅ 언급할만한 이미지..
- 테스트와 생산성에 관한
- TDD 사이클 이미지


