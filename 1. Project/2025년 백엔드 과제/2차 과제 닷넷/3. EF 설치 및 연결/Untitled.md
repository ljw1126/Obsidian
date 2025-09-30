

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

---
### Migration, Update 명령어 
code-first 방식으로 C# 기반 엔티티 클래스 생성 후 
```shell
// 프로젝트 루트 디렉터리로 이동
> dotnet ef migration add initialCreate

> dotnet ef database update

> dotnet ef migrations remove
```

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



