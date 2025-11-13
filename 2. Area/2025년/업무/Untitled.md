
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