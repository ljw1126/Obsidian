
🏠 [Installing Entity Framework Core - EF Core | Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/get-started/overview/install)

### 💣 마이그레이션 생성 및 DB 업데이트 
- 개발/테스트 환경에서는 괜찮지만, 운영 환경에서는 재앙을 일으킬 수 있다. ☠️☠️
	- 마친 JPA의 ddl-auto와 같은 역할인 듯 하다
- `powershell` 로 명령어 실행하는데 우선은 중요하지 않은 듯 하다.


### 패키지 설치
- `[도구 > NuGet 패키지 관리자 > 솔루션용 NuGet 패키지 관리...]` 실행
- 닷넷 코어 버전이 v8.x 라서, 동일한 버전대로 설치.
```text
Microsoft.EntityFrameworkCore.SqlServer  // 8.0.20
Microsoft.EntityFrameworkCore.Tools
Microsoft.EntityFrameworkCore.Design
Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore  //추가 설치 
```

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

**FluentAssertions**
- 🏠 [Fluent Assertions - Fluent Assertions](https://fluentassertions.com/)

> 우선 보류 .. 테스트 작성한 다음에 나중에 도입 고려해보는 형태로 가자


**SQLite** 
- SQLite는 Spring으로 치면 H2 DB에 해당

```shell
> dotnet add package Microsoft.EntityFrameworkCore.Sqlite // 8.0.20
```

`Progame.cs`에서 우선은 하드코딩 형태로 변경
```c#
builder.Services.AddDbContext<ShipParticularsContext>(options
=> options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
```

```shell
> dotnet ef migrations add InitialCreateSqlite

> dotnet ef migrations remove
```
- 💩 앞에 MSSQL 작업했던게 남아서 `nvarchar(max)` 문법이 SQLite 문법으로 변환되지 않아 실패 발생 
	- migrations가 앞에 작업에 영향을 받는다함 .. 
	- 그래서 디렉터리 삭제후 다시 생성 및 업데이트하여 처리

SQLite는 단순한 동적 타입 시스템을 사용, 주로 5가지 타입만 존재함
 ┌─────────────┬────────────────────────┬────────────────────────────────────────────────┐
  │ SQLite 타입 │ C# 타입과 맵핑되는 예  │ 설명                                           │
  ├─────────────┼────────────────────────┼────────────────────────────────────────────────┤
  │ `TEXT`        │ string, DateTime, Guid │ 문자열, 날짜, GUID 등을 저장합니다.            │
  │ `INTEGER`     │ int, long, bool        │ 정수, 참/거짓 값을 저장합니다.                 │
  │ `REAL`        │ float, double          │ 부동 소수점 숫자를 저장합니다.                 │
  │ `NUMERIC`     │ decimal                │ decimal과 같이 정확한 숫자 값을 저장합니다.    │
  │ `BLOB`        │ byte[]                 │ 바이너리 데이터(이미지, 파일 등)를 저장합니다. │
  └─────────────┴────────────────────────┴────────────────────────────────────────────────┘
- 프로젝트 루트 디렉터리에 `ShipParticulars.db` 생성됨 
	- 버전마다 경로가 다를 수 있다하네 (이전: `{프로젝트 루트}/bin/Debug/net8.0/`)

SQLite Browser 설치 
- [Downloads - DB Browser for SQLite](https://sqlitebrowser.org/dl/)
- 루트 디렉터리에 생성된 `ShipParticulars.db` 열면 GUI 도구 통해 확인 가능

<img src="sqlite browser 설치.png">

---

### 테스트 콘솔 출력
- xUnit 테스트 실행 중에 `Console.WriteLine(..)` 출력은 기본적으로 숨기도록 동작 
- 콘솔 출력 확인할 수 있는 방법 2가지
	- 1. `dotnet test` 명령어를 직접 실행하여 실패한 테스트에 한해서 콘솔 출력이 확인가능 
		- `--logger` 옵션 추가해 상세하게 확인 가능 
		- `dotnet test --logger "console;verbosity=detailed"`
	- 2. `ITestOutputHelper` 사용 (xUnit에서 권장하는 방식)
		- xUnit은 콘솔에 직접 출력하는 대신, 테스트별로 격리된 출력 버퍼를 제공함
		- 생성자 주입 방식으로 초기화하여 사용 (아래 코드 참고)
		  
```c#
using Microsoft.Data.Sqlite;
using Microsoft.EntityFrameworkCore;
using ShipParticularsApi.Contexts;
using Xunit;
using Xunit.Abstractions;

namespace ShipParticularsApi.Tests
{
    public class BasicCrudTests : IDisposable
    {
        private readonly ITestOutputHelper _output; // ✅
        private readonly SqliteConnection _connection;
        private readonly ShipParticularsContext _context;

        // NOTE: beforeEach
        public BasicCrudTests(ITestOutputHelper output)
        {
            _output = output;
            _connection = new SqliteConnection("DataSource=:memory:");
            _connection.Open();

            var options = new DbContextOptionsBuilder<ShipParticularsContext>()
                .UseSqlite(_connection)
                .Options;

            _context = new ShipParticularsContext(options);
            _context.Database.EnsureCreated();

            _output.WriteLine("초기화(beforeEach)");
        }

        [Fact]
        public void Test1()
        {
            int a = 1;
            int b = 2;

            int actual = a + b;

            Assert.Equal(3, actual);
            _output.WriteLine("Test1");
        }

        [Fact]
        public void Test2()
        {
            int a = 1;
            int b = 2;

            int actual = b - a;

            Assert.Equal(1, actual);
            _output.WriteLine("Test2");
        }

        // NOTE: AfterEach
        public void Dispose()
        {
            _context.Dispose();
            _output.WriteLine("자원 해제(afterEach)");
        }
    }
}

```
- "테스트 탐색기"에서 "테스트 세부 정보 요약" 창에 출력이됨

---

### 테스트 데이터 관련..


---

C# 기본값과 "값이 제공되지 않음"을 의미하는 null과 구분되지 않아 발생하는 문제를 "Sentinel Value 문제"가 확인됨 

```c#
// ShipInfo 
[Column("IS_SERVICE")]
public bool IsService { get; set; } // bool 기본값이 false라서 0이 할당됨 (Fluent API 기본값 설정 적용 x)

[Column("IS_SERVICE")]
public bool? IsService { get; set; } // 초기화 하지 않을 경우 null 할당되서 1 기본 저장
```


```c#
 [Fact]
 public async Task CacheTest()
 {
     await using var context = CreateContext();

     var newShip = new ShipInfo
     {
         ShipKey = "CREATE01",
         ShipName = "New Vessel",
         Callsign = "CALL01"
     };

     context.ShipInfos.Add(newShip);
     await context.SaveChangesAsync();

	 // Id 값으로 맵핑되어 있는 캐시 객체를 조회
     var savedShip = await context.ShipInfos.SingleAsync(s => s.Id == newShip.Id);

     Assert.Same(newShip, savedShip); // 주소 값이 같다. (캐싱을 읽어옴)
     Assert.Equal("-", savedShip.ShipType); // 기본값 할당
     Assert.True(savedShip.IsService); // 마찬가지로 기본값 할당
 }
```
- 기본적으로 저장시 PK값을 DB에서 읽어와 할당하는 형태
- `ShipInfo.IsService`, `ShipInfo.ShipType`을 nullable로 선언해둠
	- ▶️ 저장시 쿼리에서 null 필드를 제외하고, DB에서는 기본값이 할당됨
	- ▶️ SaveChangesAsync()는 DB로부터 반환받은 값들을 사용하여, 추적하고 있던 원본 C# 객체의 속성을 업데이트한다.
	- ▶️ 고로 nullable 선언된 `ShipInfo.IsService = true(1)`, `ShipInfo.ShipType = '-'` 기본값이 업데이트 된다.
		- nullable이고, Fluent API로 기본값이 설정되어 있으면 저장 후 불러와 업데이트 해준다!

> Fluent API를 사용하지 않는다고 했는데 .. 테이블에 default랑 nullable 설정이 잘되어 있고, 엔티티 모델에도 nullable (?) 표기 및 초기화만 잘 한다면 데이터 오류가 발생하거나 하지는 않을거 같다.

---
### SQLite 테스트 골격 
🏠[EntityFramework.Docs/samples/core/Testing/TestingWithoutTheDatabase/SqliteInMemoryBloggingControllerTest.cs at live · dotnet/EntityFramework.Docs](https://github.com/dotnet/EntityFramework.Docs/blob/live/samples/core/Testing/TestingWithoutTheDatabase/SqliteInMemoryBloggingControllerTest.cs#L50)

```c#
namespace ShipParticularsApi.Tests
{
    public class BasicCrudTests : IDisposable
    {
        private readonly SqliteConnection _connection;
        private readonly DbContextOptions<ShipParticularsContext> _options;
        private readonly ITestOutputHelper _output;
        // NOTE: beforeEach
        public BasicCrudTests(ITestOutputHelper output)
        {
            _output = output;

            _connection = new SqliteConnection("DataSource=:memory:");
            _connection.Open();

            _options = new DbContextOptionsBuilder<ShipParticularsContext>()
                .UseSqlite(_connection)
                .Options;

            var context = new ShipParticularsContext(_options);
            context.Database.EnsureCreated();
        }

        // NOTE: AfterEach
        public void Dispose() => _connection.Dispose();

        ShipParticularsContext CreateContext() => new(_options);
        
        // 테스트 작성
	}
}
```
- memory를 사용하기 때문에 테스트마다 초기화하게 된다.
- 처음에 Migrations/Update 명령어를 실행했을때 `*.db`가 생성 되었는데, 로컬에 H2 디비 올려서 실행하는 것과 비슷하다고 생각하면 되는 듯하다.

---

### 쿼리 출력 

```c#
public class NavigationPropertyTests
{
    private readonly SqliteConnection _connection;
    private readonly DbContextOptions<ShipParticularsContext> _options;
    private readonly ITestOutputHelper _output;

    // NOTE: beforeEach
    public NavigationPropertyTests(ITestOutputHelper output)
    {
        _output = output;

        _connection = new SqliteConnection("DataSource=:memory:");
        _connection.Open();

        _options = new DbContextOptionsBuilder<ShipParticularsContext>()
            .UseSqlite(_connection)
            // 모든 로그를 _output(xUnit의 테스트별 출력)으로 보내도록 설정합니다.
            .LogTo(message => _output.WriteLine(message), LogLevel.Information)
            // (선택사항) 민감한 데이터(파라미터 값) 로깅 활성화. 디버깅 시에만 사용하세요.
            .EnableSensitiveDataLogging()
            .Options;

        var context = new ShipParticularsContext(_options);
        context.Database.EnsureCreated();
    }

    // NOTE: AfterEach
    public void Dispose() => _connection.Dispose();

    ShipParticularsContext CreateContext() => new(_options);
    
    // 테스트 .. 
}
```

---

### Include 사용 여부
- ShipInfo (부모)
	- ReplaceShipName (자식) : 일대일
	- ShipService(자식) : 일대다

DbContext에서 Include 사용하지 않는 경우 null로 할당된다.
```text
SELECT "s"."id",  
       "s"."callsign",  
       "s"."external_ship_id",  
       "s"."is_service",  
       "s"."is_use_ais",  
       "s"."is_use_ktsat",  
       "s"."ship_code",  
       "s"."ship_key",  
       "s"."ship_name",  
       "s"."ship_type"  
FROM   "ship_info" AS "s"  
WHERE  "s"."ship_key" = 'SHIP01'  
LIMIT  2     // SingleAsync() 의 안전 장치, 2개 이상이면 예외 던진다
```

DbContext에서 Include 사용하는 경우 
```text
SELECT "t"."ID", "t"."CALLSIGN", "t"."EXTERNAL_SHIP_ID", "t"."IS_SERVICE", "t"."IS_USE_AIS", "t"."IS_USE_KTSAT", "t"."SHIP_CODE", "t"."SHIP_KEY", "t"."SHIP_NAME", "t"."SHIP_TYPE", "t"."ID0", "s0"."ID", "s0"."IS_COMPLETED", "s0"."SERVICE_NAME", "s0"."SHIP_KEY", "t"."REPLACED_SHIP_NAME", "t"."SHIP_KEY0"
      FROM (
          SELECT "s"."ID", "s"."CALLSIGN", "s"."EXTERNAL_SHIP_ID", "s"."IS_SERVICE", "s"."IS_USE_AIS", "s"."IS_USE_KTSAT", "s"."SHIP_CODE", "s"."SHIP_KEY", "s"."SHIP_NAME", "s"."SHIP_TYPE", "r"."ID" AS "ID0", "r"."REPLACED_SHIP_NAME", "r"."SHIP_KEY" AS "SHIP_KEY0"
          FROM "SHIP_INFO" AS "s"
          LEFT JOIN "REPLACE_SHIP_NAME" AS "r" ON "s"."SHIP_KEY" = "r"."SHIP_KEY"
          WHERE "s"."SHIP_KEY" = 'SHIP01'
          LIMIT 2
      ) AS "t"
      LEFT JOIN "SHIP_SERVICE" AS "s0" ON "t".
... <잘림>
```


---
### xUnit 테스트 동등성, 동일성
- `동등성`은 주소값이 다르더라도 정의된 필드 값이 같으면 같은 객체 (equality)
	- Assert.Equal(..)
- `동일성`은 값(주소 값)이 같다면 동일하다고 판변
	- Assert.Same(..)

🏠 [xUnit Assert basics: True/False, Equal, Same, Matches](https://www.roundthecode.com/dotnet-tutorials/xunit-assert-basics-true-false-equal-same-other-methods)
- 이외에도 여러 메서드를 지원하는 것으로 보인다.


---
### Migration, Update 명령어 
code-first 방식으로 C# 기반 엔티티 클래스 생성 후 
```shell
// 프로젝트 루트 디렉터리로 이동
> dotnet ef migrations add initialCreate

> dotnet ef database update

> dotnet ef migrations remove
```

엔티티 모델을 업데이트 후에 동기화하려고 할때
- Migrations 폴더와 디비 테이블 기반으로 히스토리를 관리한다
- 그래서 수동으로 Migrations 폴더 지우고 업데이트를 할 경우 히스토리가 맞지 않아 에러가 출력된다. 
- ✅ 최초 동기화한 이후에 누적해서 쌓아가야 한다.


---


### E.F(Entity Framework)의 특징 

 1. **LINQ (Language-Integrated Query) 사용**: SQL 문자열 대신, C# 언어에 통합된 쿼리 문법(LINQ)을 사용하여 데이터를 조회, 추가, 수정, 삭제할 수 있습니다. 이는 컴파일 시점에 타입 체크가 가능하게
      하여 오류를 줄여주고, 자동 완성을 지원하여 생산성을 높여줍니다.

```c# 
// SQL: SELECT * FROM Ships WHERE ShipName = 'Titanic'
// C# (LINQ):
var ship = context.Ships.FirstOrDefault(s => s.ShipName == "Titanic");
```

   2. **마이그레이션 (Migrations)**: Code-First 방식의 핵심 기능입니다. C#으로 작성된 엔티티 모델(클래스)의 변경사항을 추적하여, 데이터베이스의 스키마(테이블 구조)를 자동으로 업데이트하는 SQL 스크립트를 생성하고 실행해 줍니다. 이를 통해 DB 변경 이력을 코드로 관리할 수 있습니다.

   3. **변경 추적 (Change Tracking)**: DbContext는 조회된 엔티티의 상태를 계속 추적합니다. 개발자가 엔티티의 속성을 변경하거나, 새로운 엔티티를 추가/삭제한 후 SaveChanges()를 호출하면, EF가 변경된 내용만 감지하여 알아서 INSERT, UPDATE, DELETE SQL 구문을 생성하고 실행합니다.

   4. **다양한 로딩 전략 지원**: 관계가 있는 데이터를 가져올 때, 한 번에 모두 가져올지(즉시 로딩, Eager Loading), 필요할 때 따로 가져올지(지연 로딩, Lazy Loading) 등을 선택할 수 있어 성능 최적화에 유리합니다.

---

### 세 가지 개발 방식 (Code / DB / Model First)

✅ `Code-First` 
- 코드 우선 방식, 가장 권장되는 현대적인 방식
- 작업 순서: `C# 클래스(엔티티) 작성 → 마이그레이션(Migrations) 실행 → DB 테이블 자동 생성`
- 설명: 개발자가 먼저 C# 코드로 데이터 모델을 정의하면, EF가 이 코드를 기반으로 데이터베이스 스키마를 생성하고 관리합니다. 데이터베이스의 구조가 C# 코드에 의해 결정됩니다.
- 언제 사용하는가?
	- 새로운 프로젝트를 시작할 때
	- 데이터베이스 스키마를 포함한 모든 것을 코드로 관리하고 싶을 때
	- Git과 같은 버전 관리 시스템으로 DB 스키마 변경 이력을 관리하고 싶을 때

✅ `Database-First`
- DB 우선 방식
- 작업 순서: 기존 DB 테이블 → EF 도구(Scaffolding) 실행 → C# 클래스(엔티티) 자동 생성
- 설명: 이미 존재하는 데이터베이스를 기반으로 EF가 C# 엔티티 클래스와 DbContext를 자동으로 생성해 줍니다. 데이터베이스가 중심이 되고, 코드는 이를 따라갑니다.
- 언제 사용하는가?
	- 이미 운영 중인 레거시(legacy) 데이터베이스에 연결해야 할 때
	- 데이터베이스 스키마를 DBA(데이터베이스 관리자)가 별도로 관리하고, 개발자는 이를 가져다 쓰기만 하면 될 때

🪦 `Model-First`
- 모델 우선 방식 
- 작업 순서: Visual Studio의 비주얼 디자이너로 모델(.edmx 파일) 그리기 → C# 클래스 및 DB 테이블 동시 생성
- 설명: 개발자가 GUI 디자이너를 사용해 엔티티와 관계를 시각적으로 디자인하면, EF가 이 디자인을 바탕으로 C# 코드와 데이터베이스 스키마를 모두 생성해 줍니다.
- 언제 사용하는가?
	- 과거 EF 초기 버전에서 사용되던 레거시 방식으로 **새 프로젝트는 비권장**💩
	- 현재는 유연성이 높은 Code-First 방식에 밀려 거의 사용 X

> 개인적으로 용어 자체를 처음 접해서 흥미롭고, TDD 교육 받을때 Code-First 방식을 기반으로 설명했던거 같다. 경우에 따라 DB-First도 필요로 하지 않나 싶다. Model-First는 말로만 들었던거 같은데 .. 보통 러프하게 모델링하고 마지막에 ERD로 그리는 것을 권장한다고만 이해하고 있었음. 왜냐하면 개발 초반에는 변경이 잦기 때문에 



