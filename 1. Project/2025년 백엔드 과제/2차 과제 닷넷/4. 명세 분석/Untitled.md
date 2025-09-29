

**SHIP PARTICULAR**
- **Vessel Particulars** 라고도 함
- **선박 제원 정보**를 의미함 🚢
- 이 정보는 선박의 물리적, 설계, 기능적 특성을 정의하는 모든 정략적이고 확정적인 사항들을 포함합니다. 즉, 선박의 명세서 라고 할 수 있습니다.
- SHIP PARTICULAR에는 일반적으로 다음과 같은 중요한 정보들이 포함됩니다 
	- 1. 선박의 크기 및 용적 관련 정보
	- 2. 톤수 
	- 3. 엔진 및 속도
	- 4. 선박 종류 및 용도
	- 5. 선적 능력(Capacity)
	- 6. 등록 및 식별 정보
- 📌사용 목적 
	- SHIP PARTICULAR는 해운, 물류, 보험, 조선 등 다양한 해사 산업 분야에서 **선박의 성능, 능력, 특성을 파악**하고 **각종 계약, 운항 계획, 세금 및 수수료 산정** 등의 목적으로 매우 중요하게 사용됩니다.

---

> SHIP PARTICULAR 모듈에 포함되는 일부 테이블에 대한 CRUD 구현

**테이블 목록**
- 1. SHIP_INFO
- 2. SHIP_SERVICE
- 3. REPLACE_SHIP_NAME
- 4. SHIP_SATELLITE
- 5. SHIP_MODEL_TEST
- 6. SK_TELINK_COMPANY_SHIP

> 🧐 플로우 차트 기준으로는 한꺼번에 처리되는 느낌인데 .. 

---

**프로젝트 생성** 
- 프로젝트명 : `ShipParticularsApi`
- 닷넷 `v8.0` 

**패키지 설치**
- `[도구 > NuGet 패키지 관리자 > 솔루션용 NuGet 패키지 관리...]` 실행
- 닷넷 코어 버전이 v8.x 라서, 동일한 버전대로 설치.

```text
Microsoft.EntityFrameworkCore.SqlServer  // 8.0.20
Microsoft.EntityFrameworkCore.Tools
Microsoft.EntityFrameworkCore.Design
Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore  //추가 설치
```

---
### Data Annotation

✅ `[Key]`의 경우 
- 해당 속성이 Primay Key임을 나타냄
- `[Key]` 애노테이션이 붙어 있으면 EF Core는 기본적으로 auto-increment (Identity) 속성을 부여함
- 기본적으로 `Clustered Index` 를 생성

> [!note] `[Key]`에 명시적으로 `[DatabaseGenerated(DatabaseGeneratedOption.Identity)]` 표기하면 안되는가에 대해서
- `[Key]`에서 <u>자동적(Convention)으로 가정하여 관리</u>해주기 떄문에 굳이 붙일 필요는 없다
	- 해당 `[DatabaseGenerated(DatabaseGeneratedOption.Identity)]`의 의미는 insert 시에만 데이터베이스가 식별자를 생성자를 관리한다는 의미
	- `[Key]`만 있어도 auto-icrement가 정상적으로 동작함

 **[DatabaseGenerated] 옵션의 종류와 의미**
  질문하신 대로 이 속성은 데이터베이스가 값을 어떻게 생성하는지를 EF Core에게 알려주는 역할을 합니다. 세 가지 주요 옵션이 있습니다.

   1. `DatabaseGeneratedOption.Identity` (가장 흔함)
       * 의미: 값은 레코드가 INSERT될 때 데이터베이스에 의해 단 한 번 생성됩니다. UPDATE 시에는 변경되지 않습니다.
       * 동작: EF Core는 INSERT SQL 문을 생성할 때 이 컬럼을 포함시키지 않습니다. 대신, INSERT가 성공한 후 데이터베이스가 생성한 새 ID 값을 반환받아 C# 엔티티
         객체의 Id 속성에 채워 넣습니다.
       * 사용 예: 일반적인 숫자 자동 증가 기본 키 (IDENTITY(1,1)).

   2. `DatabaseGeneratedOption.Computed`
       * 의미: 값은 레코드가 INSERT 또는 UPDATE될 때마다 데이터베이스에 의해 계산됩니다.
       * 동작: EF Core는 INSERT나 UPDATE 시에 이 컬럼의 값을 보내려고 시도하지 않습니다. 대신, 작업이 끝난 후 항상 데이터베이스에서 최신 값을 다시 읽어와 엔티티
         객체를 업데이트합니다.
       * 사용 예: FullName AS (FirstName + ' ' + LastName) 같은 SQL의 계산 열, 또는 데이터베이스 트리거(trigger)에 의해 자동으로 갱신되는 LastModifiedDate 컬럼.

   3. `DatabaseGeneratedOption.None`
       * 의미: 데이터베이스가 값을 전혀 생성하지 않습니다. 값은 항상 애플리케이션(C# 코드)에서 제공해야 합니다.
       * 동작: EF Core는 INSERT 시에 C# 엔티티 객체에 할당된 값을 DB에 그대로 전달합니다.
       * 사용 예:
           * SHIP_KEY 처럼 애플리케이션이 직접 생성하고 할당해야 하는 비즈니스 키.
           * Guid 타입의 PK를 코드에서 Guid.NewGuid()로 직접 생성하여 할당하는 경우.

**결론 및 권장 사항**
-  "명시적인 것이 암시적인 것보다 낫다 (Explicit is better than implicit)." 라는 프로그래밍 격언이 있습니다.
- EF Core의 규칙을 알고 있다면 [DatabaseGenerated] 속성을 생략해도 괜찮습니다. 하지만 코드의 명확성을 위해, 그리고 이 코드를 처음 보는 다른 개발자(또는 미래의 나
  자신)를 위해 명시적으로 적어주는 것이 매우 좋은 습관입니다.


✅  `[Required]`의 경우 
- NOT NULL에 해당

✅  `[MaxLength(100)]`
- NVARCHAR(100)으로 컬럼 속성 설정


**일대다 관계의 자식 테이블에 외래키 설정하는 방법 (2가지)**
📌방법1. 권장되는 표준 방법
```c#
[Key]
[Column("ID")]
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
public long Id { get; set; }

[Required]
[Column("SHIP_KEY")]
[MaxLength(10)]
public string ShipKey { get; set; }

[ForeignKey(nameof(ShipKey))]
public ShipInfo ShipInfo { get; set; }
```
- 장점 
	- nameof(ShipKey)를 사용했기 때문에, 만약 ShipKey 속성의 이름을 바꾸면 컴파일러가 즉시 오류를 알려줌. 런타임 에러를 방지할 수 있음
	- 가독성 : ShipInfo 속성에 `[ForeignKey]`가 붙어 있어 이 관계는 해당 키를 통해 맺어진다는 의도가 명확해짐

방법2. 
```c#
[Key]
[Column("ID")]
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
public long Id { get; set; }

[ForeignKey("ShipInfo")] // 이 FK는 ShipInfo 속성과 연결됩니다 (navigation)
[Required]
[Column("SHIP_KEY")]
[MaxLength(10)]
public string ShipKey { get; set; }

public ShipInfo ShipInfo { get; set; }
```
- 단점 : 
	- ShipInfo의 속성 이름이 VesselInfo로 바뀌면 컴파일 오류없이 런타임 에러가 발생 가능
		- 문자열은 리팩토링에 취약
	- 가독성: 관계의 핵심은 ShipInfo라는 '탐색 속성'인데, 설정은 ShipKey라는 '데이터 속성'에 붙어 있어 코드의 의도를 파악하기 위해 두 속성을 번갈아 봐야 함

---

외래키 양방향 설정시 migration 생성이 되지 않는다.. (삽질 2시간)
- 양쪽 다 명시했는데 기본적으로 `[InverseProperty]`, `[ForeignKey]` 둘 다 PK (ex. long Id) 를 바라보기 때문에 지금과 같은 `string ShipKey`를 바라보지 않아 빌드 에러가 발생 ..
- 해결 방안 
	- DbContext 설정하는 곳에서 Fluent API 활용하여 명시적으로 대체키에 대한 관계 설정을 할 수 있다.
	- 엔티티 프레임워크 관련 기본 패키지 여러개를 설치 다 했다면 Fluent API가 포함되어 있을 것이다. !
	
```c#
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
	            // HasOne: ReplaceShipName은 하나의 ShipInfo를 가짐
                entity.HasOne(child => child.ShipInfo)
	            // WithMany: 그 ShipInfo는 여러 개의 ReplaceShipNames를 가짐
                .WithMany(parent => parent.ReplaceShipNames)
                // HasForeginKey : 관계의 외래키는 ReplaceShipName의 ShipKey 속성입니다.
                .HasForeignKey(child => child.ShipKey)
                // HasPrinipalKey : 관계의 대상 키는 ShipInfo의 ShipKey 속성입니다. (PK가 아니다!)
                .HasPrincipalKey(parent => parent.ShipKey);
            });

            modelBuilder.Entity<ShipModelTest>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithMany(parent => parent.ShipModelTests)
                .HasForeignKey(child => child.ShipKey)
                .HasPrincipalKey(parent => parent.ShipKey);
            });

            modelBuilder.Entity<ShipSatellite>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithMany(parent => parent.ShipSatellites)
                .HasForeignKey(child => child.ShipKey)
                .HasPrincipalKey(parent => parent.ShipKey);
            });

            modelBuilder.Entity<ShipService>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithMany(parent => parent.ShipServices)
                .HasForeignKey(child => child.ShipKey)
                .HasPrincipalKey(parent => parent.ShipKey);
            });

            modelBuilder.Entity<SkTelinkCompanyShip>(entity =>
            {
                entity.HasOne(child => child.ShipInfo)
                .WithMany(parent => parent.SkTelinkCompanyShips)
                .HasForeignKey(child => child.ShipKey)
                .HasPrincipalKey(parent => parent.ShipKey);
            });
        }
    }
}

```

---
### Migration, Update 명령어 

```shell
// 프로젝트 루트 디렉터리로 이동
> dotnet ef migration add initialCreate

> dotnet ef database update

> dotnet ef migrations remove
```



---

[인덱스 - EF Core | Microsoft Learn](https://learn.microsoft.com/ko-kr/ef/core/modeling/indexes?tabs=data-annotations)

[DbContext 수명, 구성 및 초기화 - EF Core | Microsoft Learn](https://learn.microsoft.com/ko-kr/ef/core/dbcontext-configuration/#configuring-the-database-provider)
- 데이터베이스별로 필요한 NuGet 패키지 표있음 

[값 변환 - EF Core | Microsoft Learn](https://learn.microsoft.com/ko-kr/ef/core/modeling/value-conversions?tabs=data-annotations)
- enum 문자열 관련된 내용인데 어렵..

[관계 소개 - EF Core | Microsoft Learn](https://learn.microsoft.com/ko-kr/ef/core/modeling/relationships)
- 연관 관계에 대한 내용 
	- 일대다, 일대일, 다대일, 다대다

