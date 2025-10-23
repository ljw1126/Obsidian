
**찾아보기** 
- base 키워드 (상속 관련, 자바로 치면 super)
- ':' 상속 키워드
- namespace
- using 
- 자동 구현 속성(Auto-Implemented Properties) 
- var 예약어

**📗 C#의 역사**
- [10-1 C#의 역사 - C# 프로그래밍 입문](https://wikidocs.net/227194)

**📜 MS 공식 문서 - C# 설명서**
- [프로그램의 일반 구조 - C# | Microsoft Learn](https://learn.microsoft.com/ko-kr/dotnet/csharp/fundamentals/program-structure/)
### ':' 
- 상속을 의미
- C#에서도 단일 상속만을 지원

### base(..)
- 상속받은 자식 클래스에서 부모 클래스 생성자 호출 
- 자바의 `super(..)`에 해당 
- 아래 DbContext 예시 참고

### DbContext 기본 설정 
```c#
namespace BasicEntityFrameworkExample.Data;

public class UserDb : DbContext
{
	public UserDb(DbContextOptions<UserDb> options) : base(options)
	{
		// 자식 클래스 멤버 필드 초기화 가능
	}

	// <T> : T 는 테이블에 매핑되는 도메인 모델 
	// Users : 속성명은 실제 테이블명 (쿼리에 사용되는 듯)
	public DbSet<User> Users { get; set; }
}
```
- `: base(options)` : UserDb의 부모 클래스인 DbContext의 생성자를 호출하면서 options 매개 변수를 전달
```text
이렇게 하는 이유는 DbContext가 데이터베이스 연결 정보, 캐싱 설정 등 중요한 정보를 DbContextOptions 객체를 통해 초기화하기 때문입니다. UserDb 클래스 내에서 직접 이 모든 초기화 로직을 구현하는 대신, 부모 클래스가 이미 가지고 있는 초기화 로직을 재사용하는 것입니다. 즉, UserDb는 DbContext의 기능을 물려받아 사용하면서 필요한 정보만 부모에게 전달하는 역할을 합니다.
```
- `public DbSet<User> Users { get; set; }`
	- DbSet 타입의 속성(Property)를 선언하는 것
	- DbSet은 DB의 특정 테이블을 나타내는 EF(Entity Framework) Core의 핵심 클래스이다.

```text
DbSet<User> Users
- <User> : 테이블에 맵핑 되는 도메인 모델
- Users : DbSet의 속성명, 이는 DB의 테이블 이름으로 매핑됨. 이 속성을 통해 User 모델의 컬렉션에 접근하여 CRUD 작업을 수행할 수 있다.
  
▶️ 따라서 `User` 모델은 `Users` 테이블의 데이터 구조를 정의하며, `Users` 속성은 그 테이블에 접근하는 통로 역할을 하는 것이 맞습니다.

`public DbSet<User> Users { get; set; }` ➡️ Maps to the table named **`Users`**.
```

☠️ 만약 속성명을 Useres로 변경하면 런타임에 API 호출시 에러 출력 (실제 테이블명은 Users)
```text
Microsoft.Data.SqlClient.SqlException (0x80131904): Invalid object name 'Useres'.
```


`Program.cs` 내용 중
```c#
builder.Services.AddDbContext<UserDb>(opt => opt.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")))
```
- UserDb 생성자 초기화시 옵션을 주입할만 한게 요 부분밖에 안보임 
- 런타임에 DbContext 상속 객체를 찾아서 주입을 해주는게 아닌가 싶음 (추측) 
- 🤖 Gemini
	- `builder.Services.AddDbContext<UserDb>(...)` 설정은 런타임에 **`UserDb` 클래스를 자동으로 찾아 `DbContextOptions<UserDb>`를 주입**해주는 역할을 합니다
	- 1. 서비스 등록 : `builder.Services.AddDbContext<UserDb>(...)` 코드는 컨테이너에 "누군가 `UserDb`를 요청하면, `DefaultConnection`에 연결된 SQL Server를 사용하도록 설정된 `DbContextOptions`를 생성하여 제공하라"고 지시합니다.
	- 2. 의존성 해결 : API 엔드포인트나 컨트롤러가 `UserDb`를 생성자 매개변수로 요청하면, 컨테이너는 이 요청을 감지합니다.
	- 3. 객체 생성 및 주입 : 컨테이너는 등록된 설정(`opt.UseSqlServer(...)`)을 사용하여 `DbContextOptions<UserDb>` 객체를 생성합니다. 그리고 이 `options` 객체를 `UserDb`의 생성자(`UserDb(DbContextOptions<UserDb> options)`)에 전달하여 `UserDb` 인스턴스를 만들고, 최종적으로 요청한 클래스에 주입해줍니다.
	- 최종적으로 new 생성자 처리가 필요 없음

### 초과 게시 방지 (Over-posting protection)
> 엔티티 모델에 민감한 정보가 포함되어 있을 경우, 클라이언트가 악의적으로 해당 정보를 업데이트하는 것을 막기 위한 보안 기법
- 간단하게 말하자면
	- 엔티티 그대로 응답하는게 아니라, 불필요한/노출 되면 위험한 정보가 응답되는 걸 방지하기 위해 DTO(Data Transfer Object)로 변환하여 제한한다는 것
- [공식문서 - Tutorial: Create a controller-based web API with ASP.NET Core | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-9.0&tabs=visual-studio#prevent-over-posting)


```c#
namespace TodoApi.Models;

public class TodoItem
{
    public long Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
    public string? Secret { get; set; }
}
```

```c#
namespace TodoApi.Models;

public class TodoItemDTO
{
    public long Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
```

### 닷넷 소스 브라우저 
- [Source Browser](https://source.dot.net/)
- [microsoft/referencesource: Source from the Microsoft .NET Reference Source that represent a subset of the .NET Framework](https://github.com/microsoft/referencesource)


### 닷넷 디버깅 
- [[프로그래밍Tip] C# .net framework 내부까지 디버깅 해 보자](https://www.youtube.com/watch?v=80P0Zmk3KK0)
	- 소스 코드를 다운 받아서 인식 해주는 게 있구나.

**✅ Visual Studio 단축키**
- `F5` : 디버깅 모드로 시작
- `ctrl + F5` : 일반 시작
- `F9` : break point(중단점) 설정/해제 ▶️ 커서 위치 기준
- `F10` : **프로시저(함수 또는 메서드)** 단위로 실행 
- `F11` : **한 줄** 단위로 실행

> 디버깅 = "문제 해결 능력"

[Visual Studio 디버거 사용하기(C# 프로그래밍 학습을 위한)](https://www.youtube.com/watch?v=DIIe6MVKLTg)


### 의존성 주입 확장 메서드

```c#
using TodoApi.Repositories;
using TodoApi.Services;

namespace TodoApi.Extensions
{
    public static class ServiceCollectionExtensions
    {
        public static IServiceCollection AddApplicationServices(this IServiceCollection services)
        {
            services.AddScoped<ITodoItemRepository, TodoItemInmemoryRepository>();
            services.AddScoped<TodoItemService>();
            return services;
        }
    }
}

```
- `static` 선언 : 클래스, 메서드
- `this IServiceCollection services` (핵심✅)
	- 첫 번째 매개변수 앞에 `this` 키워드를 붙여서 정의하면, 해당 매개변수의 타입에 속한 인스턴스 메서드처럼 호출할 수 있습니다.
	- C# 컴파일러는 이 구문을 보고 `AddApplicationServices`가 `IServiceCollection` 타입의 확장 메서드임을 인식합니다.

```c#
// Program.cs 에서 아래와 같이 호출하면
builder.Services.AddApplicationServices();

// C# 컴파일러는 내부적으로 아래와 같이 변환
MyServiceExtensions.AddApplicationServices(builder.Services);
```

이것이 마치 `IServiceCollection` 객체에 `AddApplicationServices`라는 인스턴스 메서드가 있는 것처럼 동작하게 되는 이유입니다. 이 패턴을 통해 개발자는 `Program.cs`의 코드를 간결하고 가독성 높게 유지할 수 있습니다.


### Entity Framework 사용시 DB 동기화 
- powershell에서 커맨드 기반으로 실행 가능
- 또는 `Program.cs`에서 어플리케이션 실행시 직접 실행하는 형태로 동기화 가능 
- 💩 JPA의 ddl-auto와 같은 역할이라서 .. 운영환경에서 사용하면 대재앙을 맞이 할 수 있다
	- 오직 로컬/개발 환경에서만 실행 

```shell
# 설치
> dotnet tool install --global dotnet-ef

# 파일 생성
> dotnet ef migrations add migration1 

# 반영
> dotnet ef database update // MSSQL 주입

# 파일 삭제 
> dotnet ef migrations remove
```
- Migrations 디렉터리 생성되고 `{날짜시간}_migration1.cs`와 `*Snapsho.cs` 파일 생성됨
- `.gitignore` 처리도 필요할 듯 하다
- [ASP.NET Core EF Core - C# 프로그래밍 배우기 (Learn C# Programming)](https://www.csharpstudy.com/web/article/14-ASP-NET-Core-EF-Core)
- [마이그레이션 개요 - EF Core | Microsoft Learn](https://learn.microsoft.com/ko-kr/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli)


> 우선 DbContext 주입이 되지 않는 이슈가 계속 발생한다.. dotnet CLI 실행시 profiles별로 분리되도록 실행하는 방법이 없는지 확인할 필요가 있다. 


### Data Annotation in EF Core
- JPA(ORM 프레임워크)와 비슷한 느낌
- 이걸 기반으로 DB 마이그레이션을 생성 가능하지 않나 싶다. 
- 우선 순위는 명시적인게 높다✅
```c#
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace TodoApi.Models
{
    [Table("TODO_ITEMS")] // UPPER_SNAKE_CASE라서, 실수하기 딱좋다
    public class TodoItem
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Column("ID")]
        public long Id { get; set; }

        [Column("NAME")]
        public string? Name { get; set; }

        [Column("IS_COMPLETE")]
        public bool IsComplete { get; set; }

        [Column("SECRET")]
        public string? Secret { get; set; }
    }
}

```

```c#
namespace TodoApi.Models
{
    public class TodoContext: DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options)
            : base(options)
        {
        }

		//<도메인모델> , TodoItems 는 테이블명 혹은 속성명 
		// 만약 [Table("..")]이 없으면 해당 속성명으로 테이블 쿼리 날림 
        public DbSet<TodoItem> TodoItems { get; set; } = null!;
    }
}

```

Q. 스프링에서 명시적인 선언이 우선순위가 더 높았는데 닷넷도 그런가 봐요 
```text
네, 맞습니다. 스프링과 .NET 모두 명시적인 설정이 컨벤션보다 우선순위가 높습니다.

이는 두 프레임워크가 추구하는 핵심 철학 중 하나입니다. 프레임워크의 규칙(컨벤션)을 따르면 코드를 빠르게 작성할 수 있지만, 복잡한 요구사항이나 기존 시스템과의 통합을 위해서는 개발자가 직접 설정을 정의(명시적 설정)할 수 있는 유연성이 필요하기 때문입니다.

스프링 vs .NET 설정 우선순위
스프링: @Configuration 클래스나 XML 설정 파일에서 정의된 내용이 기본 컨벤션(예: 빈 이름 자동 생성)보다 우선시됩니다. @Override나 @Qualifier 같은 어노테이션은 명시적인 의존성 주입을 가능하게 합니다.

.NET: 엔티티 프레임워크 코어(EF Core)에서 [Table]이나 [Column]과 같은 데이터 어노테이션이나 OnModelCreating 메서드에서 정의된 Fluent API 설정이 기본 컨벤션(예: 테이블명 복수화, 컬럼명 파스칼 케이스)보다 우선적으로 적용됩니다.

두 프레임워크 모두 개발자가 컨벤션에 의존할지, 아니면 명시적으로 재정의할지 선택할 수 있는 구조를 제공합니다.
```


**참고.**
- [Data Annotations in Entity Framework Core - Dot Net Tutorials](https://dotnettutorials.net/lesson/data-annotation-attributes-in-entity-framework-core/#google_vignette)
- [EF Annotation - C# 프로그래밍 배우기 (Learn C# Programming)](https://www.csharpstudy.com/Data/EF-annotation.aspx)


### 라우팅 및 URL 경로 
[자습서: ASP.NET Core를 사용하여 컨트롤러 기반 웹 API 만들기 | Microsoft Learn](https://learn.microsoft.com/ko-kr/aspnet/core/tutorials/first-web-api?view=aspnetcore-8.0&tabs=visual-studio)


### 생성자 초기화 문법

```c#
namespace TodoApi.Repositories
{
    public class TodoItemInmemoryRepository : ITodoItemRepository
    {
        private readonly TodoContext _dbContext;

        public TodoItemInmemoryRepository(TodoContext dbContext)
        {
            _dbContext = dbContext;
        }
 
       
        //..
	}
}
```

리팩터링1. 생성자에 식 본문 사용 
```c#
public TodoItemInmemoryRepository(TodoContext dbContext) => _dbContext = dbContext;
```

리팩터링2. 기본 생성자 사용 
```c#
namespace TodoApi.Repositories
{
    public class TodoItemInmemoryRepository(TodoContext dbContext) : ITodoItemRepository
    {
        private readonly TodoContext _dbContext = dbContext;
	     
		//..   
	}
}
```


### 레코드 타입

```c#
public record TodoItemDto(long Id, string? Name, bool IsComplete);

// TodoItemsController.cs
private static TodoItemDto ItemToDto(TodoItem todoItem) =>
    new(todoItem.Id, todoItem.Name, todoItem.IsComplete);
```
- **위치 기반 초기화 (Positional Initialization)** 또는 **생성자 초기화 (Constructor Initialization)** 라고 부릅니다. 이는 `record`의 주 생성자(Primary Constructor)가 정의된 순서대로 값을 전달하여 초기화합니다.

아래와 같이 선언하는 경우
```c#
public class TodoItemDTO
{
	public long Id { get; set; }
	public string? Name { get; set; }
	public bool IsComplete { get; set; }
}

private static TodoItemDTO ItemToDto(TodoItem todoItem) =>
new TodoItemDTO
{
	Id = todoItem.Id,
	Name = todoItem.Name,
	IsComplete = todoItem.IsComplete
};
```


🔖 [(공식문서)개체 이니셜라이저를 사용하여 개체를 초기화하는 방법 - C# | Microsoft Learn](https://learn.microsoft.com/ko-kr/dotnet/csharp/programming-guide/classes-and-structs/how-to-initialize-objects-by-using-an-object-initializer)

---

### enum to string, string to enum
enum 상수를 사용시 EF Core에서는 기본적으로 int로 값을 저장하게 된다
- 이 경우 enum 상수의 순서가 변경되면 데이터 부정합 발생 (critical)
- JPA에서는 OrderNo로 저장하는 것을 지양하고 `문자열` 그대로 넣는 방식을 권장

[Value Conversions - EF Core | Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/modeling/value-conversions?tabs=data-annotations#the-valueconverter-class)

1. 파스칼 표기법으로 enum 선언
```cs
namespace ShipParticularsApi.Entities
{
    public enum ShipTypes
    {
        Default,
        Roro,
        Ropax,
        CruisePassenger,
        Fishing,
        RefrigeratedCargo,
        GeneralCargo,
        Passenger,
        Vehicle,
        LngCarrier,
        BulkCarrier,
        Combination,
        Chemical,
        Container,
        SpecialCraft,
        Cargo,
        Other,
        GasCarrier,
        OilTanker,
        Tanker
    }
}

```

2. 엔티티 모델 수정
```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using Microsoft.EntityFrameworkCore;

namespace ShipParticularsApi.Entities
{
    [Table("SHIP_INFO")]
    [Index(nameof(ShipKey), IsUnique = true)]
    public class ShipInfo
    {
        [Key]
        [Column("ID")]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public long Id { get; set; }

        [Required]
        [Column("SHIP_KEY")]
        [MaxLength(10)]
        public string ShipKey { get; set; }

        [Column("CALLSIGN")]
        [MaxLength(10)]
        public string Callsign { get; set; }

        [Column("SHIP_NAME")]
        [MaxLength(10)]
        public string ShipName { get; set; }

        [Column("SHIP_TYPE")] // 여기
        public ShipTypes? ShipType { get; set; }

        [Column("SHIP_CODE")]
        [MaxLength(100)]
        public string? ShipCode { get; set; }

        [Column("EXTERNAL_SHIP_ID")]
        [MaxLength(36)]
        public string? ExternalShipId { get; set; }

        [Column("IS_USE_KTSAT")]
        public bool? IsUseKtsat { get; set; }

        [Column("IS_SERVICE")]
        public bool? IsService { get; set; }

        [Column("IS_USE_AIS")]
        public bool? IsUseAis { get; set; }

        public virtual ReplaceShipName? ReplaceShipName { get; set; }
        public virtual ShipModelTest? ShipModelTest { get; set; }
        public virtual ShipSatellite? ShipSatellite { get; set; }
        public virtual ICollection<ShipService>? ShipServices { get; set; }
        public virtual SkTelinkCompanyShip? SkTelinkCompanyShip { get; set; }
    }
}

```

3. convert 정의
```cs
using Microsoft.EntityFrameworkCore.Storage.ValueConversion;

namespace ShipParticularsApi.Entities
{
    public class ShipTypesToStringConverter : ValueConverter<ShipTypes, string>
    {
        public ShipTypesToStringConverter() : base(
            v => ShipTypesToString(v),
            v => StringToShipTypes(v)
        )
        {
        }

        private static string ShipTypesToString(ShipTypes shipTypes)
        {
            return shipTypes switch
            {
                ShipTypes.Default => "-",
                ShipTypes.Roro => "RORO",
                ShipTypes.Ropax => "ROPAX",
                ShipTypes.CruisePassenger => "CRUISE_PASSENGER",
                ShipTypes.Fishing => "FISHING",
                ShipTypes.RefrigeratedCargo => "REFRIGERATED_CARGO",
                ShipTypes.GeneralCargo => "GENERAL_CARGO",
                ShipTypes.Passenger => "PASSENGER",
                ShipTypes.Vehicle => "VEHICLE",
                ShipTypes.LngCarrier => "LNG_CARRIER",
                ShipTypes.BulkCarrier => "BULK_CARRIER",
                ShipTypes.Combination => "COMBINATION",
                ShipTypes.Chemical => "CHEMICAL",
                ShipTypes.Container => "CONTAINER",
                ShipTypes.SpecialCraft => "SPECIAL_CRAFT",
                ShipTypes.Cargo => "Cargo",
                ShipTypes.Other => "OTHER",
                ShipTypes.GasCarrier => "GAS_CARRIER",
                ShipTypes.OilTanker => "OIL_TANKER",
                ShipTypes.Tanker => "Tanker",
                _ => throw new ArgumentException()
            };
        }

        private static ShipTypes StringToShipTypes(string value)
        {
            return value switch
            {
                "-" => ShipTypes.Default,
                "RORO" => ShipTypes.Roro,
                "ROPAX" => ShipTypes.Ropax,
                "CRUISE_PASSENGER" => ShipTypes.CruisePassenger,
                "FISHING" => ShipTypes.Fishing,
                "REFRIGERATED_CARGO" => ShipTypes.RefrigeratedCargo,
                "GENERAL_CARGO" => ShipTypes.GeneralCargo,
                "PASSENGER" => ShipTypes.Passenger,
                "VEHICLE" => ShipTypes.Vehicle,
                "LNG_CARRIER" => ShipTypes.LngCarrier,
                "BULK_CARRIER" => ShipTypes.BulkCarrier,
                "COMBINATION" => ShipTypes.Combination,
                "CHEMICAL" => ShipTypes.Chemical,
                "CONTAINER" => ShipTypes.Container,
                "SPECIAL_CRAFT" => ShipTypes.SpecialCraft,
                "Cargo" => ShipTypes.Cargo,
                "OTHER" => ShipTypes.Other,
                "GAS_CARRIER" => ShipTypes.GasCarrier,
                "OIL_TANKER" => ShipTypes.OilTanker,
                "Tanker" => ShipTypes.Tanker,
                _ => throw new ArgumentException(),
            };
        }
    }
}

```


4. context 설정 
```cs
using Microsoft.EntityFrameworkCore;
using ShipParticularsApi.Entities;

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
	            // 여기
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

                entity.Property(p => p.CreateDateTime)
                    .HasDefaultValueSql("SYSDATETIME()");
            });

            modelBuilder.Entity<ShipService>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithMany(parent => parent.ShipServices)
                .HasForeignKey(child => child.ShipKey)
                .HasPrincipalKey(parent => parent.ShipKey);

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
    }
}

```