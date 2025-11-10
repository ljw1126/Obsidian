
> .gitignore에 local.settings.json 있어서 clone시 없음 .. 
### LoggerQueueTrigger
- `[FunctionName("LoggerQueueTrigger")]`
	- `[TimerTrigger("0  0/10 * * * *", RunOnStartup = true)]`

1.blob에 원천 리소스가 저장되어 있고, 10분 단위로 스케쥴러가 동작하는 것으로 보임
- 로그 데이터 취합은 아래와 같은 단위로 동작
	- `10분`
	- `1시간`
	- `하루`
		- 각 시간별 큐에 문자열 `배 키`를 메시지로 해서 발송

2.하위에 `[QueueTrigger("queueName", Connection = "StorageQueue")]`이 실행되는 걸로 보인다.

이때 "StorageQueue" 는 `local.settings.json` 파일에 등록된 속성으로 보이는데, csproj 파일에 `<CopyToPublishDirectory>Never</CopyToPublishDirectory>`있어서 그런가 .. 저장소에도 올라가 있지 않다.

> [!note]
> 스케쥴러 👉 큐 발송 (10분, 60분, 하루) 
> 큐 이벤트 발생 👉 각 시간대별 Function 실행됨


```cs
# 조회
GetShipKeyForDbInsert() // *ManagerDbContext (MSSQL)

# QueueLoggerDataClient
AddQueue10MinAsync(message) // "logger10min" 큐에 메시지 보냄

AddQueue1HourAsync(message) // "logger1hour" 큐에 메시지 보냄

AddQueue1DayAsync(message) // "logger1day" 큐에 메시지 보냄
```

💩 클래스에 static 멤버 변수가 있는데 테스트를 어렵게 한다.. 
- 멀티 스레드로 인한 동시성 이슈 
	- `_functionIsRunningOrNot` 플래그 변수
- 날짜 관련 
	- `DateTime hourStartTime`
	- `DateTime dayStartTime`
	- 로컬 캐시 사용?? 아니면 .. 값 객체 ??

### LoggerDataInsert10Min

ManagerDbContext
- MSSQL로 보임 
	- HELPER에 보면 config 설정으로 문자열을 읽어와 주입하는 Context 주입한다

리팩터링할만한 부분 
- `40번` : filterList 조회와 shipParticulars 조회가 상관없는데 같이 뭉쳐져 있다. 
- `54번` 
	- `LoggerData10MinAnalysis`이 Services 디렉터리에 위치 
	- 절차적으로 처리
	- filterList가 여기서 사용되는데 .. LoggerData10MinAnalaysis 서비스로 이동하면 좋을듯하다 ✅
- `57번` 
	- `UpsertLoggerServerManager(..)`
	- 54번 줄에서 처리한 객체를 전달한다.
	- `*Client` 는 3개다`Cosmos DB(NoSql)` 조회해서 처리하는 로직
	- 최종적으로 V.S Cosmos DB에 저장한다
- `70번`
	- errorList와 frozenList가 둘다 `string[]`인데 `object[]`로 선언되어 있따.
- `89번`
	- 타입 문자열이 공용으로 사용하고 있네 (굳이?)
- `96번`
	- "id" 컨벤션이.... 어쩔수 없나.. 이거 사용하는 곳 다 고쳐야 할듯하다..
- deprecated된 변수가 보인다..
- 문자열로 쿼리를 작성했는데 .. cosmos db 조회하려면 그렇게 할 수 밖에 없는건가 싶음.
	- ✅ 기분 안나쁘게 순수하게 묻기 처음보니..
- `LoggerData10MinAnaysis`
	- `1493번`
		- 문자열로된 날짜를 해당 메서드에서 밖에 사용하지 않는데 위에서 생성해서 전달하고 있다.
		- 고정된 날짜인데 .. 굳이 ? 
			- static한 멤버 변수는 오버인거 같고.. 메서드 내부로 옮긴 다음에 사용하는게 현실적인거 같다. ✅
		- 📌 날짜 유틸 없나 찾아보기
	- 절차
		- 



`로직 분석`
- 큐 이벤트 트리거 실행 : "logger10min"
- DB 조회 (2회)
	- `ManagerDbContext()`
	- 필터 목록이랑 선박 정보   // ✅ 데이터 2가지가 필요..
- 비즈니스 로직 2개 실행 
	- 1. 서비스 인스턴스 메서드 실행 
		- cosmos db 조회 
		- blob 조회 (1)
			- 디렉터리 > 년 > 월 > 일 > 파일 순으로 탐색 
			- 최종 `파일` 리스트를 반환
		- `파일` 리스트가 0인 경우 
			- 로깅 후 false 반환 
		- 파싱 로직 수행 (파일 리스트 순차 조회하면서 실행)
			- `ManagerDbContext` 조회
			- `DownloadBlobToCsvMemoryAsync()` : blob 파일을 csv 형태로 다운로드 함 (`Blob`)
			- `Upsert10MinControlLogAsync()` : 조건에 맞으면 cosmos 저장 : `10분 history 로그 저장`
			- `MakeAverage()` : 평균 계산 + 에러 정보 
			- `CalculateShipData(..)` : slip, 연료 마력 등 2차 검사 
			- `ParseWeather(..)` : 날짜 파싱
				- averageData 순회하면서 조건별로 내용을 갱신하네 📌 이게 과연 맞는건가?? 최종적으로 참조를 리턴하긴한다..
			- `ParseDraftAsync(..)`
				- 검증과 함께 Cosmos DB에 Noon 보고서 조회도 한다. 
				- 💩 `SpeedLossComputer` 클래스에 플래그 비트 사용
			- `CheckPrimaryData(..)`
			- `Insert10MinDataToDocumentDbAsync(..)` : cosmos 저장 
			- `ConvertDynamicByDistionary(..)`
			- `SetLastPositionControllerAsync(..)`
	- 2. 정적 메서드 (`Cosmos db` 3개 사용)
		- 조회 3번, 추가 1번

> HELPERS.RedisTemplate.LastPositionClient 
> - 깨진 유리창의 법칙 .. switch문 .. waterfall로 작성되어 있다..💩

> blob, cosmos(조회, 히스토리 로그 저장), redis, mssql 다 호출하네 .. 
- mssql은 매니저 디비로 설정 읽기 
- blob 리소스 읽음 
- cosmos는 읽거나, 쓰거나 
- redis 쓰거나 
### LoggerDataInsert1Hour

`첫번째`
- 마찬가지로 상단에서 filterList와 ShipParticulars를 한번에 조회한다.
- 근데 filterList를 사용하는 메서드가 따로있고, 그 안에서도 3군데에서 사용하는데 엄청 멀다💩

`try catch문`
- 내부 함수
	- `GetLastFirstInsertDate1DayAsync(..)`
	- `Query1HourDataAsync(..)`
	- `MakeAveraging1Day(..)`
- `InsertDocumentDbAsync1DayAsync(..)` :  Cosmos DB 저장 (NoSQL)

`절차`
- ManagerDbContext 조회 (2건)
- `GetLastFirstInsertDate1HourAsync(skey)` 
	- 마지막 1시간 데이터 입력 시간 가져오기
	- cosmos 디비 조회하거나 초기화하거나 
- `Query10MinDataAsync()`
	- 10분 db에서 데이터 부르기
	- cosmos 디비 조회만
- `MakeAveraging1Hour(..)`
	- 절차적인 로직
	- 리스트 안에 dictionary 자료구조를 담아서 리턴
- `InsertDocumentDbAsync1HourAsync(..)`
	- cosmos 디비에 저장 
		- 에러 터지면 로그를 `Sentry`라는 곳에 보낸듯함

`개선할만한 부분`
- `MakeAveraging1Hour(..)`
	- 절차적 
	- LoggerDataSettings가 static helper method로 보이는데 굳이 따로 뽑을 필요가 있었을가?
		- 필터 리스트를 일급 컬렉션으로 감싸는건 어떠할까?


### LoggerDataInsert1Day

`절차`
- 마찬가지로 ManagerDb(mssql) 두번 조회 
- `GetLastFirstInsertDate1DayAsync(skey)`
	- cosmos 디비 조회해서 없으면 초기화하고 날짜 반환 
- `Query1HourDataAsync(..)`
	- cosmos 디비 조회 
- `MakeAveraging1Day(..)`
	- 하루치 평균 만드는거.. 절차적 
- `InsertDocumentDbAsync1DayAsync(..)`
	- cosmos 디비 저장
		- 예외 발생시  `Sentry`에 로그 발송


`개선할만한 부분`
- `67번 줄` : 세미콜론 두 개 찍힘
- 1Hour, 1Day의 로직이 거의 동일
	- 조건식 안에 다른 부분이 부분적으로 존재한다. 
	- 둘다 `List<Dictionary<string, dynamic>>` 타입을 리턴한다
		- 반복문 돌면서 Dictionary를 생성해 리스트 추가

### suffix Http.cs에 대해

```cs
// [FunctionName("Logger10MinHttp")]
public static async Task<IActionResult> RunAsync(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]
    HttpRequest req, ILogger log)    
{
	// 내부적으로 위해 Logger* 클래스의 public 메서드를 호출하고 있다.
}
```
- 추측으로는 테스트 용도로 azure function에 등록해서 http 요청을 보내 확인하려고 한거 같다.


----


**Azurite 설치(공식 문서)** 
[Use the Azurite emulator for local Azure Storage development | Azure Docs](https://docs.azure.cn/en-us/storage/common/storage-use-azurite?toc=%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json)
[Install and run the Azurite emulator for Azure Storage | Azure Docs](https://docs.azure.cn/en-us/storage/common/storage-install-azurite?tabs=github%2Cblob-storage)
```shell
git clone https://github.com/Azure/Azurite.git

npm install 
npm run build 
npm install -g
```

```shell
// 실행
azurite --silent --location c:\azurite --debug c:\azurite\debug.log

Azurite Blob service is starting at http://127.0.0.1:10000
Azurite Blob service is successfully listening at http://127.0.0.1:10000
Azurite Queue service is starting at http://127.0.0.1:10001
Azurite Queue service is successfully listening at http://127.0.0.1:10001
Azurite Table service is starting at http://127.0.0.1:10002
Azurite Table service is successfully listening at http://127.0.0.1:10002
```

> GUI를 제공하지 않는다.
> 마찬가지로 wsl2에서 npm으로 설치하는 바람에 .. powershell에서 못찾는다💩


```shell
docker run -d -p 10000:10000 -p 10001:10001 -p 10002:10002 --name azurite-local mcr.microsoft.com/azure-storage/azurite
```

```text
// local.settings.json 의 Values에 아래 항목 추가하면 로컬에 설치된 Azurite를 찾는다.. 그러다보니 Azurite 꺼져 있으면 .. 실패함
"StorageQueue": "UseDevelopmentStorage=true"
```


**Azure Storage Explorer**
- [Azure Storage Explorer – 클라우드 저장소 관리 | Microsoft Azure](https://azure.microsoft.com/ko-kr/products/storage/storage-explorer/?msockid=3524f9483223634a38d5ef123336626b)
- [Azurite를 사용하여 자동화된 테스트 실행 - Azure Storage | Microsoft Learn](https://learn.microsoft.com/ko-kr/azure/storage/blobs/use-azurite-to-run-automated-tests?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fazure%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json)

표시 이름 : `local-1`
계정 이름 : `devstoreaccount1`
계정키 : `Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==`
Blob 포트 : 10000
큐 포트 : 10001
테이블 포트 : 10002


**Function App 기본 생성** 
- `*V8` 형태로 프로젝트 생성

```shell
// 프로젝트 디렉터리로 이동 
dotnet run

unhandled exception: An error occurred trying to start process 'func' ... 지정된 파일을 찾을 수 없습니다.
```
- 에러: core tool 설치해야 `func` 명령을 실행할 수 있다함


**azure-functions-core-tools 설치**

💩 실패
```shell
// npm 설치시 powershell에서 경로를 찾지 못함.. 그래서 삭제함
$ npm i -g azure-functions-core-tools@4
$ func -v
4.4.0
```
[Azure/azure-functions-core-tools: Command line tools for Azure Functions](https://github.com/Azure/azure-functions-core-tools)

✅ exe 64bit로 설치
[Develop Azure Functions locally using Core Tools | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-csharp)

```text
윈도우 환경 변수 Path에 func.exe 있는 디렉터리 경로 등록 
C:\Program Files\Microsoft\Azure Functions Core Tools

그리고 V.S 재시작하기 
```

```shell
// 해당 function 프로젝트 경로에서 
$ dotnet run
```

### HttpExxample

```cs
 public class HttpExample
 {
     private readonly ILogger<HttpExample> _logger;

     public HttpExample(ILogger<HttpExample> logger)
     {
         _logger = logger;
     }

     [Function("HttpExample")]
     public IActionResult Run([HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequest req)
     {
         _logger.LogInformation("C# HTTP trigger function processed a request.");
         return new OkObjectResult("Welcome to Azure Functions!");
     }
 }
```

<img src="./images/최초 http function 실행.png">
<img src="./images/HttpExample Get 요청.png">

### TimerExample

[in-process 및 격리된 작업자 프로세스 .NET Azure Functions 간 차이점 | Microsoft Learn](https://learn.microsoft.com/ko-kr/azure/azure-functions/dotnet-isolated-in-process-differences)

> [!info]
> [In Process 모델에 대한 지원은 2026년 11월 10일에 종료됩니다](https://aka.ms/azure-functions-retirements/in-process-model). 전체 지원을 위해 [앱을 격리된 작업자 모델로 마이그레이션](https://learn.microsoft.com/ko-kr/azure/azure-functions/migrate-dotnet-to-isolated-model)하는 것이 좋습니다.

실행모드 비교 테이블에 보면 
- 이전 v6의 경우 `Microsoft.Azure.WebJobs.Extensions.*` 패키지로 시작
- 지금 격리된 작업자 모델 `Microsoft.Azure.Functions.Worker.EXtensions.*`
- 이외에 


`Microsoft.Azure.Functions.Worker.EXtensions.Storage v6.8.0 설치
- TimerTrigger와 TimerInfo 첨부하기 위해서 설치


```cs
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;

namespace LoggerDataV8
{
    public class TimerExample
    {
        private readonly ILogger _logger;

        public TimerExample(ILoggerFactory loggerFactory)
        {
            _logger = loggerFactory.CreateLogger<TimerExample>();
        }

        [Function("TimerExample")]
        public void Run([TimerTrigger("0 */1 * * * *")] TimerInfo myTimer)
        {
            _logger.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");

            if (myTimer.ScheduleStatus is not null)
            {
                _logger.LogInformation($"Next timer schedule at: {myTimer.ScheduleStatus.Next}");
            }
        }
    }
}

```

```shell
> dotnet run
```

<img src="./images/최초 TimerTrigger 동작 확인.png">

### QueueExample 

`local.settings.json`
```json
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
        "StorageQueue": "UseDevelopmentStorage=true" // 요거 추가
    }
}

```

`docker container` 실행
```shell
docker run -d -p 10000:10000 -p 10001:10001 -p 10002:10002 --name azurite-local mcr.microsoft.com/azure-storage/azurite
```

`azure storage explorer` 실행
- 착하게도 예물레이터 찾아주네 
- 큐에 `[큐 만들기]` 후 **이름** 기재
- `+추가` 버튼 눌러서 메시지 삽입 
	- `{"id": 1, "status": "new"}`
- 현재 function app 실행하고 있지 않아 메시지 유지되고 있음


`function 실행 후 출력 확인`
```cs
public class QueueExample
{
    private readonly ILogger<QueueExample> _logger;

    public QueueExample(ILogger<QueueExample> logger)
    {
        _logger = logger;
    }

    [Function(nameof(QueueExample))]
    public async Task RunAsync([QueueTrigger("logger10min", Connection = "StorageQueue")] QueueMessage message)
    {
        _logger.LogInformation("C# Queue trigger function processed: {messageText}", message.MessageText);
    }
}
```


```shell
// 해당 function 프로젝트 디렉터리에서
$ dotnet run 
```

<img src="./images/QueueExample 메시지 소비.png">


### 프로세스 시작 
```cs
// function app 의 root 디렉터리에서
dotnet run

또는

func start
```
[Develop Azure Functions locally using Core Tools | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-csharp#run-a-local-function)

### 프로세스 종료 

```shell
$ Get-Process | Where-Object { $_.ProcessName -eq 'func' -or $_.ProcessName -eq 'dotnet' } | Stop-Process -Force

$ Stop-Process -Id {Id} -Force
```


---

### 신기한거

C#에서는 DateTime을 빼기 연산하면 시간 차가 구해지나 보다..

🤔솔루션 탐색기에는 폴더 안에 프로젝트가 나열되어 있는데, 터미널에서 `ls -al`로 확인시 루트 디렉터리 하위에 프로젝트 배치되어 있음 !! 

✅ `*.sln`에 보면 GUID 형태로 폴더가 선언되어있고, 그 폴더의 GUID를 하위 프로젝트들이 참여하고 있는 형태라 Visual Studio에서 계층형으로 표현되는 거였음!! 👉 설정 파일에서 reference가 `../HELPER/etc..`되어있어서 이상했는데 이런 설정이!


🤔 DbContext 초기화의 비밀 .. 

```cs
protected override OnConfiguring(..) {
	// ✅ optionBuilder 통해서 connectionString 하드코딩된 것을 넣어서 Db 연결되는 거였구나.
}
```
[DbContext.OnConfiguring(DbContextOptionsBuilder) Method (Microsoft.EntityFrameworkCore) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext.onconfiguring?view=efcore-9.0)

`new XXXDbContext()`호출하면 `OnConfiguring()` 메서드가 같이 실행된다. 이때 문서에 보면 base에는 nothing이라고 되어 있음. 

---

### 외부 인프라 연결 

`MSSQL` , `Redis`
- DataGrip 설치해 연결 구성 
	- 30일 후 dbeaver로 대체

`spmsstorage`
- Queue, Blob 사용
- MS Azure Storage Explorer로 Azure 로그인 후 바로 확인 가능

`Cosmos DB`
- 





---

### 찾아보기 


```cs
public static class LoggerQueueTrigger
{
	private static bool _functionIsRunningOrNot = false;
	private static DateTime hourStartTime = DateTime.Now;
	private static DateTime dayStartTime = DateTime.Now;

	[FunctionName("LoggerQueueTrigger")]
#if DEBUG
	public static async Task RunAsync([TimerTrigger("0  0/10 * * * *", RunOnStartup = true)] TimerInfo myTimer, ILogger log)
#else
	public static async Task RunAsync([TimerTrigger("0  0/10 * * * *")] TimerInfo myTimer, ILogger log)
#endif
	{
		log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
		if (_functionIsRunningOrNot == true)
		{
			log.LogInformation($"Other Instance is Running at: {DateTime.Now}");
			return;
		}
		_functionIsRunningOrNot = true;
		
		// .. do something
	}
}	
```

김영한님 스레드 강의에서 .. 이러한 static 변수는 언제 반영될지 알 수 없기 때문에 .. 안티패턴이라고 했던게 기억이남.. 

아래 내용이 하나의 프로세스에 멀티 스레드로 동작하다보니 요청 올때마다 실행한다는 건가 싶은데 .. (강의 내용 재확인.. C# 진영도 비슷한지 확인)

---

### 도커로 환경 구성을 한다면 
azurite
- blob의 특정 경로에 파일 필요
- queue에 3개 필요

mssql
- manager db 조회
	- 필요한 테이블만 추가하면 되지 않을까 싶다. 
	- 데이터랑 

cosmos db 
- logger collection
- .. 그외 여러개 

redis 


---

### Reference 
[Introduction - Training | Microsoft Learn](https://learn.microsoft.com/en-us/training/modules/develop-test-deploy-azure-functions-with-visual-studio/1-introduction)

[🚀 Getting Started with Azure Functions in .NET 8: A Modern Approach to Serverless | by santosh santosh | Medium](https://medium.com/@santoshg.santosh/getting-started-with-azure-functions-in-net-8-a-modern-approach-to-serverless-f230f1987193)

