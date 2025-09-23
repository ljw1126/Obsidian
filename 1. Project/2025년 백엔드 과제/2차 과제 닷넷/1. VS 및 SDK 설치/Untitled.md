- [raw.githubusercontent.com/github/gitignore/master/VisualStudio.gitignore](https://raw.githubusercontent.com/github/gitignore/master/VisualStudio.gitignore)
	- [https://github.com/github/gitignore/blob/main/VisualStudio.gitignore - Google 검색](https://www.google.com/search?q=https://github.com/github/gitignore/blob/main/VisualStudio.gitignore) 
- [C# Clean Code: Coding Conventions - DEV Community](https://dev.to/moh_moh701/c-clean-code-coding-conventions-2c41#:~:text=Learn%20the%20essential%20coding%20conventions%20in%20C%23%20to,with%20practical%20examples%20and%20Visual%20Studio%E2%80%99s%20refactoring%20tools.)
- [.NET Coding Conventions - C# | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [C# docs - get started, tutorials, reference. | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/)
- [visual studio shortcut cheat sheet](https://visualstudio.microsoft.com/keyboard-shortcuts.pdf)
- [공식 tip 문서](https://learn.microsoft.com/en-us/visualstudio/ide/default-keyboard-shortcuts-in-visual-studio?view=vs-2022)
- [공식 유튜브](https://www.youtube.com/watch?v=r3ys7oL9izQ&list=PLdo4fOcmZ0oWunQnm3WnZxJrseIw2zSAk&index=3)


**설치**
- [Visual Studio: 소프트웨어 개발을 위한 IDE 및 코드 편집기](https://visualstudio.microsoft.com/ko/)
- 실행 후 워크로드 설치
	- ASP.NET 및 웹 개발 (ASP.NET and web development)
		- ✅ 핵심
		- C#으로 웹 사이트 , API 서버, 웹 애플리케이션을 만드는데 필요한 모든 것이 포함
		- 포함된 주요 기능
			- ASP.NET Core : 최신 크로스플랫폼 .NET 웹 프레임워크 (✅ 가장 중요)
			- .NET Framework 개발 도구 
			- IIS Express (개발용 경량 웹 서버)
			- API 및 웹 애플리케이션 개발 도구
	- 데이터 스토리지 및 처리 (Data storage and processing)
		- 해당 워크로드를 통해 DB 관련 작업을 Visual Studio 내에서 편리하게 할 수 있음
		- 포함된 주요 기능
			- SQL Server Data Tools (SSDT) : DB 프로젝트 관리 
			- Entity Framework Core 도구 : C# 코드로 DB를 다루는 기술 (ORM)


>[!info] SDK (Software Development Kit)
>소프트웨어 개발을 지원하기 위해 제공되는 도구와 리소스의 집합


> [!note] xUnit, JUnit
>  - xUnit, NUnit, MSTest: C# (.NET) 진영의 대표적인 3대 유닛 테스트 프레임워크
>  - 왜 "JUnit을 사용한다"라고 말했을까?
>    -  가장 유력한 추측은, 설명해주신 분이 Java 개발 경험이 많으시거나, 'JUnit'이라는 단어를 '단위 테스트 프레임워크'를 지칭하는 일반 명사처럼 사용하셨을 가능성이 매우 높습니다
>    - 특히 NUnit은 Java의 JUnit을 .NET으로 포팅하면서 시작된 프로젝트라, 두 프레임워크는 이름도 비슷하고 구조나 사용법도 매우 유사합니다. 그래서 JUnit에 익숙한 개발자들은 .NET의 테스트 프레임워크를 그냥 통칭해서 JUnit이라고 부르는 경우가 종종 있습니다

---

### SDK 설치 확인 방법

window powershell 사용
```shell
> dotnet --version
9.0.305   // 8버전을 설치해야 한다 😂
```


> 속상하다.. wsl2는 우분투 환경이라서 .. 완전 독립된 세상🥹


**윈도우 x64, x86 확인 방법**
- 윈도우 + i 눌러서 설정 창을 연다
- `시스템`을 클릭한다 
- 정보 확인
	- 장치 사양에 **시스템 종류** 확인
		
```text
장치 이름	BOOK-J8DV05RKE8
프로세서	Intel(R) Core(TM) Ultra 7 155H(3.80 GHz)
설치된 RAM	32.0GB(31.6GB 사용 가능)
장치 ID	1DE377C1-8815-4917-80DF-F860129B04E8
제품 ID	00342-21175-79369-AAOEM
시스템 종류	64비트 운영 체제, x64 기반 프로세서 ✅
펜 및 터치	10개의 터치 포인트를 사용할 수 있는 터치 지원
```

닷넷 8 버전 다운로드
- [.NET 8.0 다운로드(Linux, macOS 및 Windows) | .NET](https://dotnet.microsoft.com/ko-kr/download/dotnet/8.0)
- `앱 빌드 - SDK`와 `앱 실행 - 런타임`으로 나눠지는데 JDK와 JRE 구분 같은 걸로 보인다
	- `앱 빌드 - SDK` 설치 : v8.0.414
		- 포함된 런타임 
			- .NET 런타임 8.0.20
			- ASP.NET 코어 런타임 8.0.20
			- .NET 테스크톱 런타임 8.0.20
		- 언어 지원 
			- C# 12.0, F# 8.0, Visual Basic 16.9

<img src="sdk8 설치 페이지.png">


<img src="dotnet8 install.png">

---

### 최소 API (공식 예제)
- [Kestrel ASP.NET Core의 웹 서버 | Microsoft Learn](https://learn.microsoft.com/ko-kr/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-8.0)


**NuGet ?** 
- 패키지 관리자 
- `[도구 > NuGet 패키지 관리자 > 솔루션용 NuGet 패키지 관리]`
- .NET 프로젝트에 필요한 라이브러리, 프레임워크, 도구 등을 쉽게 추가하고 관리할 수 있도록 도와주는 시스템입니다.
	- maven, gradle 처럼 NuGet은 `.csproj` 파일에 패키지 정보를 기록하고, 필요할 때 자동으로 패키지를 다운로드하고 프로젝트에 통합합니다. 
	- ASP.NET Core 에서는 `NuGet`이 패키지 관리를, `MSBuild`가 빌드 자동화 역할을 담당하여 두 역할이 분리되어 있음
	- 따라서 MS에서는 Java 진영의 Maven, Gradle과 같이 패키지 관리와 빌드 자동화를 통합하는 단일 빌드 도구는 제공 🙅‍♂️

**namespace**
```c#
namespace TodoApi
{
    public class Todo
    {
    }
}
```
- `TodoApi.Todo`로 접근해서 사용해야 한다.
- 상단에 `using TodoApi;` 선언하게 되면 .. 네임 스페이스를 붙이지 않아도 된다.



```c#
class TodoDb : DbContext
{
	// 생성자
    public TodoDb(DbContextOptions<TodoDb> options) : base(options) {  }

	// DB 테이블에 매핑되는 속성
	// Todo 클래스가 DB의 Todos라는 이름의 테이블과 연결된다는 것을 의미
    public DbSet<Todo> Todos => Set<Todo>();
}

```

`public TodoDb(DbContextOptions<TodoDb> options) : base(options) {  }`
- 이것은 **생성자**입니다. `TodoDb` 클래스가 초기화될 때 호출됩니다.
- `DbContextOptions<TodoDb> options`는 **데이터베이스 연결 설정 정보**를 담고 있습니다. 예를 들어, 어떤 데이터베이스(SQL Server, SQLite, InMemory)에 연결할지, 연결 문자열은 무엇인지 등의 정보가 이 `options` 객체를 통해 전달됩니다.
- `: base(options)`는 부모 클래스인 `DbContext`의 생성자를 호출하여, 전달받은 연결 설정을 부모 클래스에 넘겨주는 역할을 합니다.

`public DbSet<Todo> Todos => Set<Todo>();`
- `DbSet<T>`은 **데이터베이스의 테이블**에 매핑되는 속성입니다.
- `DbSet<Todo>`는 `Todo` 클래스가 데이터베이스의 **`Todos`** 라는 이름의 테이블과 연결된다는 것을 의미합니다.
- **`=> Set<Todo>()`** 는 `Todos` 속성이 `Todo` 엔터티에 대한 `DbSet`을 반환하도록 설정하는 람다식(Lambda Expression)입니다. 이를 통해 코드에서 `db.Todos`와 같이 접근하여 `Todo` 테이블에 대한 **CRUD(생성, 읽기, 업데이트, 삭제)** 작업을 수행할 수 있습니다.

요약. Gemini
```text
TodoDb 클래스는 데이터베이스에 연결하고, Todo 모델과 Todos 테이블을 매핑하여 데이터베이스 작업을 처리하는 역할을 합니다. 스프링 개발자에게 익숙한 개념으로 비유하자면, @Entity가 붙은 Todo 클래스를 관리하는 @Repository 인터페이스와 EntityManager의 역할을 동시에 수행한다고 생각하시면 이해가 빠를 것입니다.
```


```c#
app.MapPost("/todoitems", async (Todo todo, TodoDb db) => {
    db.Todos.Add(todo);
    await db.SaveChangesAsync();

    return Results.Created($"/todoitems/{todo.Id}", todo);
});
```
- JPA 영속성 컨텍스트처럼 
	- Add() 통해 엔티티를 컨텍스트에 추가후 SaveChangesAsync() 호출하여 DB 변경사항을 flush 하는 방식
- `$` 문법 
	- 문자열 보간 (Interpolation)
	- `$"/todoitems/{todo.Id}"`에서 `$`는 **C#의 문자열 보간(String Interpolation)** 문법
	- 이는 자바스크립트의 템플릿 


```c#
app.MapPut("/todoitems/{id}", async (int id, Todo inputTodo, TodoDb db) =>
{
    var todo = await db.Todos.FindAsync(id);

    if (todo is null) return Results.NotFound();
	
	// 영속성 컨텍스트의 더티 체킹 같은 느낌
    todo.Name = inputTodo.Name;
    todo.IsComplete = inputTodo.IsComplete;

    await db.SaveChangesAsync();

    return Results.NoContent();
});
```
- `SaveChangesAsync()` 호출하기 전까지 변경 사항은 DB에 반영되지 않음
- 1. 더티체킹과 `SaveChangesAsync()`
	- Entity Framework Core는 `DbContext`에 의해 추적되는 엔티티(여기서는 `todo` 객체)의 **변경 상태를 자동으로 감지**합니다.
		- 1. **`db.Todos.FindAsync(id)`**: 이 코드가 실행되면, `Todo` 엔티티는 `DbContext`에 의해 **"추적(Tracking)"**되기 시작합니다.
		- 2. **`todo.Name = inputTodo.Name;`**: 이 코드는 추적 중인 `todo` 객체의 속성 값을 변경합니다. Entity Framework Core는 이 변경을 내부적으로 감지하고, 해당 엔티티의 상태를 **`Modified`**로 표시합니다.
		- 3. **`await db.SaveChangesAsync()`**: 이 메서드를 호출하는 순간, Entity Framework Core는 **`Modified` 상태로 표시된 모든 엔티티들을 찾아내고**, 그 변경 사항을 바탕으로 `UPDATE` 쿼리를 생성한 후 **데이터베이스에 전송**합니다.
	- `SaveChangesAsync()`를 호출해야만 변경 사항이 실제로 데이터베이스에 반영되는 구조입니다.
- 2. `NoContent()` 의미
	- `Results.NoContent()`는 **HTTP 204 No Content** 상태 코드를 반환합니다. 이 코드는 **"요청이 성공적으로 처리되었지만, 응답 본문에 보낼 내용이 없습니다"** 라는 의미입니다.
	- `PUT` 요청은 리소스를 **수정**하는 작업이므로, 클라이언트에게 수정된 객체 자체를 다시 반환하기보다 성공 상태만 알려주는 것이 일반적입니다

**✅요약하자면**
`SaveChangesAsync()`는 더티 체킹으로 감지된 변경 사항을 데이터베이스에 실제로 반영하는 **커밋(commit)** 역할을 합니다. `MapPut`은 이러한 변경 로직을 처리하는 API 엔드포인트를 정의하는 메서드입니다.


**http 요청** 
<img src="http 생성 및 POST 요청.png">

`GroupMap`이라고 prefix를 선언해서 처리하는게 가능
```c#
var todoItems = app.MapGroup("/todoitems");
```

```c#
var todoItems = app.MapGroup("/todoitems");

todoItems.MapGet("/", GetAllTodos);
// ..

static async Task<IResult> GetAllTodos(TodoDb db)
{
    return TypedResults.Ok(await db.Todos.ToListAsync()));
}
```

Q. TypedResults는 스프링의 ResponseEntity와 같은 걸까요? `Task<IResult>`는 반환타입을 명시한거구요. ▶️ Task는 Webflux의 Mono, Flux 같은 느낌

```text
네, 맞습니다. **`TypedResults`는 스프링의 `ResponseEntity` 객체와 동일한 역할을 한다고 볼 수 있습니다.**

`TypedResults`는 HTTP 응답 상태 코드(예: 200 OK, 201 Created, 404 Not Found)와 함께 응답 본문 데이터를 생성하는 헬퍼 클래스입니다. `ResponseEntity`가 상태 코드, 헤더, 본문을 한 번에 담아 반환하는 것처럼, `TypedResults`는 **`IResult`**라는 인터페이스를 구현하는 응답 객체를 만들어줍니다.

### `Task<IResult>`의 의미

`Task<IResult>`는 **메서드의 반환 타입을 명시**하는 것입니다.

- **`Task`**: 메서드가 **비동기적으로 실행**된다는 것을 나타냅니다. `await db.Todos.ToListAsync()`처럼 비동기 작업을 포함하므로 `Task`를 반환 타입으로 사용합니다.
    
- **`IResult`**: HTTP 응답을 나타내는 **인터페이스**입니다. `TypedResults.Ok()`, `Results.NotFound()`와 같은 모든 응답 헬퍼 메서드는 이 `IResult` 인터페이스를 반환합니다.
    

따라서 `Task<IResult>`는 **"이 메서드는 비동기적으로 실행되며, 최종적으로 HTTP 응답을 나타내는 `IResult` 객체를 반환한다"**는 의미입니다.
```

---

### SQL Server Client 설치 
- [SQL Server 다운로드 | Microsoft](https://www.microsoft.com/ko-kr/sql-server/sql-server-downloads?msockid=3524f9483223634a38d5ef123336626b)
	- 우선 SQL Server 2022 개발자 설치
	- [공식 sql-server-samples/samples/tutorials/c#/Windows at master · microsoft/sql-server-samples](https://github.com/microsoft/sql-server-samples/tree/master/samples/tutorials/c%23/Windows) // 안봄
- [SQL Server Management Studio 설치 | Microsoft Learn](https://learn.microsoft.com/ko-kr/ssms/install/install?redirectedfrom=MSDN)
	- 이것도 설치해야 하는 듯함
- Visual Studio용 SSDT(SQL Server Data Tools) 
	- [SSDT (SQL Server Data Tools)설치 - SQL Server Data Tools (SSDT) | Microsoft Learn](https://learn.microsoft.com/ko-kr/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-ver17&tabs=vs2022)
	- `선택사항`인듯


> SQL Server는 MSSQL을 의미한다 
> MSSQL은 Microsoft SQL Server의 줄임말

> SSMS가 클라이언트 도구이고, SQL Server (MSSQL)은 DBMS에 해당한다




---

TODO. [자습서: ASP.NET Core를 사용하여 컨트롤러 기반 웹 API 만들기 | Microsoft Learn](https://learn.microsoft.com/ko-kr/aspnet/core/tutorials/first-web-api?view=aspnetcore-8.0&tabs=visual-studio)


1. 닷넷 Marketplace - Theme // 설치후 재시작 필요
- [Search results - theme | Visual Studio , Visual Studio Marketplace](https://marketplace.visualstudio.com/search?term=theme&target=VS&category=All%20categories&cost=free&vsVersion=vs2022&sortBy=Installs)
- `[도구 > 테마]`

2. V.S 코파일럿 무료 사용 가이드 
- [Visual Studio에서 GitHub Copilot 무료 제공 - Visual Studio (Windows) | Microsoft Learn](https://learn.microsoft.com/ko-kr/visualstudio/ide/copilot-free-plan?view=vs-2022)
- **월별 사용량 제한**
	- **코드 자동완성(Completions): 월 2,000회**
	- **Copilot Chat(채팅): 월 50회**
	- 프리미엄 모델 요청: 월 50회

3. 유용한 플러그인 모음 
- [Visual studio 2022 확장 추천](https://nanagame2.tistory.com/213)
- [Reddit - Visual Studio 2022에 제일 좋고 유용한 확장 프로그램 뭐 있어? : r/csharp](https://www.reddit.com/r/csharp/comments/r3b082/best_most_useful_extensions_for_visual_studio_2022/?tl=ko)

 4. 폰트 변경
- 네이버 D2 폰트 설치 
- `[도구 > 옵션 > 환경 > 글꼴 및 색]`에서 글꼴 변경, 크기(`15`)도 조절

 5. 에디터 줄번호표시
- `[도구 > 옵션 > 텍스트 편집기 > 모든 언어]` 에서 **줄 번호**✅

1. 확장 도구 
- `[확장 > 확장 관리]`
```text
file icon
https://marketplace.visualstudio.com/items?itemName=MadsKristensen.FileIcons&ssr=false#overview

sonarqube
https://marketplace.visualstudio.com/items?itemName=SonarSource.SonarLintforVisualStudio2022

Roslynator
https://marketplace.visualstudio.com/items?itemName=josefpihrt.Roslynator2022

CodeMaid
https://marketplace.visualstudio.com/items?itemName=SteveCadwallader.CodeMaidVS2022

Viasfora
https://marketplace.visualstudio.com/items?itemName=TomasRestrepo.Viasfora

CommentsPlus
https://marketplace.visualstudio.com/search?term=CommentsPlus&target=VS&category=All%20categories&cost=free&vsVersion=vs2022&sortBy=Relevance

[추가 설치]
- IndentRainbow 
- CodeBlockEndTag
- File Path On Footer
- comment remover
- RunAsAdmin : 1. Go to Tools -> Options -> RunAsAdmin enable
```