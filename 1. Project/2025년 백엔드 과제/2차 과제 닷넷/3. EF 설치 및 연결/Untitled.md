

### 💣 마이그레이션 생성 및 DB 업데이트 
- 개발/테스트 환경에서는 괜찮지만, 운영 환경에서는 재앙을 일으킬 수 있다. ☠️☠️
	- 마친 JPA의 ddl-auto와 같은 역할인 듯 하다
- `powershell` 로 명령어 실행하는데 우선은 중요하지 않은 듯 하다.


### 패키지 설치
- `[도구 > NuGet 패키지 관리자 > 솔루션용 NuGet 패키지 관리...]` 실행
- 닷넷 코어 버전이 v8.x 라서, 동일한 버전대로 설치.
```text
Microsoft.EntityFrameworkCore.SqlServer
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


