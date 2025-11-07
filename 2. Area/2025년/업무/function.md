
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


스케쥴러 👉 큐 발송 (10분, 60분, 하루) 
큐 이벤트 발생 👉 각 시간대별 Function 실행됨


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


### LoggerDataInsert1Day




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


### 프로세스 종료 

```shell
$ Get-Process | Where-Object { $_.ProcessName -eq 'func' -or $_.ProcessName -eq 'dotnet' } | Stop-Process -Force

$ Stop-Process -Id {Id} -Force
```