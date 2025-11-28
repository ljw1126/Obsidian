
### ref
- 비동기 키워드가 붙은 함수는 ref, in/out 키워드를 사용 못한다.

logger10min에서 호출하는 함수 로직 중에
```cs
private async Task ShipDataParsing(string shipKey, List<string> fileListForParse)
{
	// .. 

	// 변수 초기화
	var dataCount = 0; 
	foreach (var fileItem in fileListForParse)
	{
		try 
		{
			// ..

			//해당 blob파일을 메모리에 csv 형태로 다운로드함.
			var shipDataAfterFilter = new List<Dictionary<string, dynamic>>();
			using (var resultShipDataCsv = await blobLoggerDataService.DownloadBlobToCsvMemoryAsync(fileItem))
			{
    				shipDataAfterFilter = await DaqDataFilter(shipKey, fileItem, resultShipDataCsv, filterList, ref dataCount); // 여기..
			}

			
		}		
	}	
}

 // 컴파일 에러. 비동기 함수의 경우 ref , in/out 키워드 사용 X
 private async Task<List<Dictionary<string, dynamic>>> DaqDataFilter(/*생략*/ ref int dataCount)
 {
	// ..

	while(resultShipDataCsv.Read())
	{
		dataCount++;
		//
	}

	return resultShipData;
 }
```

<img src="./images/ref 키워드.png"/>


테스트 해본 결과 누적합에 해당한 것으로 판단 

```cs
public class RefKeywordTests
{

    [Fact]
    public void AccTest()
    {
        int a = 9;
        AddOne(ref a);

        Assert.Equal(10, a);
    }

    private void AddOne(ref int number)
    {
        number += 1;
    }

    [Fact]
    public void RepeatAddTest()
    {
        int a = 0;
        for (int i = 1; i <= 10; i++)
        {
            AddOne(ref a);
        }

        Assert.Equal(10, a);
    }
}
```


### 테스트 프로젝트 추가 

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
