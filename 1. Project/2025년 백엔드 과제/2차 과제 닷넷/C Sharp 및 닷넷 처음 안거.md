
**찾아보기** 
- base 키워드 (상속 관련, 자바로 치면 super)
- ':' 상속 키워드
- namespace
- using 
- 자동 구현 속성(Auto-Implemented Properties) 
- var 예약어

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
