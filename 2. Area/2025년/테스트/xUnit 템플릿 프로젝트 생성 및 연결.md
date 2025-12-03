
>[!note] 닷넷 진영에서는 xUnit 템플릿 프로젝트를 따로 만들어서 테스트 코드와 패키지를 관리한다. 이를 통해 배포 프로젝트에서 테스트와 관련 내용을 제외할 수 있다함

`xunit 템플릿 프로젝트 생성 (루트 디렉터리)`
```shell
$ dotnet new xunit -n LoggerDataV8.Tests
```

최상위 루트 sln 경로에서 실행
```shell
$ dotnet sln add .\LoggerDataV8.Test\LoggerDataV8.Tests.csproj
```

`테스트 프로젝트에 부모 프로젝트 참조 추가`
```shell
$ dotnet add LoggerDataV8.Tests/LoggerDataV8.Tests.csproj reference LoggerDataV8/LoggerDataV8.csproj
```

프로젝트 계층 구조를 나타내기 위해 최상위 루트 sln을 vim으로 연다
```text
1. LoggerDataV8.Tests가 추가되었는지 확인
Project("{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}") = "LoggerDataV8.Tests", "LoggerDataV8.Tests\LoggerDataV8.Tests.csproj", "{53087D2C-F8BF-44BF-8D5D-9B1A564CD0F9}"
EndProject

2. GlobalSection(NestedProjects) = preSolution 하위에 설정 추가 
	//..
	//LoggerDataV8.Tests 이고 옆에가 Functions 폴더 GUID
	{53087D2C-F8BF-44BF-8D5D-9B1A564CD0F9} = {78DCD72B-5B0A-4249-A990-366F77F74941}
   EndGlobalSection
```


기타. csproj에 수동으로 패키지 변경 후 
```shell
$ dotnet restore
```
