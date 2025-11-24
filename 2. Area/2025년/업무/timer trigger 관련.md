

```cs
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;
using System.Diagnostics;

namespace LoggerDataV8.Functions
{
    public class LoggerTimerTrigger(
        ILogger<LoggerTimerTrigger> logger)
    {
        [Function("LoggerTimerTrigger")]
#if DEBUG
        public async Task RunAsync([TimerTrigger("0  0/10 * * * *", RunOnStartup = true)] TimerInfo myTimer)
#else
        public async Task RunAsync([TimerTrigger("0  0/10 * * * *")] TimerInfo myTimer)
#endif
        {
            Process currentProcess = Process.GetCurrentProcess();

            // 💡 프로세스 ID를 가져와 로그에 출력합니다.
            logger.LogInformation($"[PID Check] 현재 프로세스 ID (PID): {currentProcess.Id}");

            // 💡 락 획득 후, 3분(180초) 동안 프로세스를 멈춥니다.
            // 이 시간 동안 두 번째 인스턴스는 대기합니다.
            Thread.Sleep(180000);
            logger.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");

            return
        }
}
```

timer tirgger는 storage lock을 사용한다네 
- [Timer trigger for Azure Functions | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=python-v2%2Cisolated-process%2Cnodejs-v4&pivots=programming-language-csharp#function-apps-sharing-storage)
- [azure-docs/articles/azure-functions/functions-bindings-timer.md at main · MicrosoftDocs/azure-docs](https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/azure-functions/functions-bindings-timer.md)

```text
The timer trigger uses a storage lock to ensure that there is only one timer instance when a function app scales out to multiple instances. If two function apps share the same identifying configuration and each uses a timer trigger, only one timer runs.
```

---
## Gemini 답변

네, 공식 문서 내용과 사용자님의 해석은 모두 정확합니다. TimerTrigger의 락 메커니즘은 Azure Storage Blob을 사용하며, **식별자를 고정하면 멀티 인스턴스 환경에서 단 하나의 타이머만 실행되는 것을 보장**합니다.

### 1. 🔑 TimerTrigger 락의 원리와 식별자

TimerTrigger의 락은 `azure-webjobs-hosts` 컨테이너 내부에 생성되는 Blob을 통해 구현되며, 이 Blob의 경로는 다음과 같은 구성 요소를 포함하여 **식별자** 역할을 합니다.

$$\text{Lock Path} = \text{locks} / \text{호스트 ID} / \text{함수 이름}$$

#### A. 호스트 ID (Host ID)의 역할

TimerTrigger가 실행되는 **Function Host (프로세스)**는 고유한 식별자(Host ID)를 가집니다.

- **기본 동작 (랜덤 생성):** 식별자를 따로 지정하지 않으면, Function App은 실행될 때마다(특히 콜드 스타트나 스케일 아웃 시) **랜덤한 값**을 호스트 ID로 생성합니다.
    
- **멀티 인스턴스 시 락 동작:**
    
    - **인스턴스 1 (PID 1234):** 락 Blob: `locks/랜덤ID_A/MyTimerFunction` 획득
        
    - **인스턴스 2 (PID 5678):** 락 Blob: `locks/랜덤ID_A/MyTimerFunction` 획득 시도 (실패하고 대기)
        

이 메커니즘 덕분에 **TimerTrigger 락은 기본적으로 멀티 인스턴스 환경에서도 하나의 인스턴스만 실행되도록 보장**합니다.

---

### 2. 🛠️ 호스트 식별자 설정 방법

호스트 ID는 환경 변수(`local.settings.json` 또는 Azure App Settings)를 통해 **명시적으로 고정**할 수 있습니다.

#### A. 설정할 환경 변수

호스트 ID를 고정하려면 **`AzureWebJobsStorage`** 연결 문자열과 함께 **`AzureWebJobsStorage:hostid`**를 사용합니다.

#### B. `local.settings.json` 설정 예시

`local.settings.json` 파일의 `Values` 섹션에 다음 설정을 추가하여 호스트 ID를 고정할 수 있습니다.

JSON

```
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    // 💡 호스트 ID를 명시적으로 고정
    "AzureWebJobsStorage:hostid": "prod-master-host" 
  }
}
```

이 설정을 추가하면, 로컬에서 여러 프로세스를 띄우거나 Azure에서 스케일 아웃이 발생하더라도 **모든 인스턴스가 동일한 호스트 ID를 사용하여 락을 경쟁**하게 됩니다.

---

### 3. 🛡️ Storage를 사용하지 않을 경우 (락 없음)

공식 문서 내용처럼, TimerTrigger의 락 메커니즘은 Azure Storage Blob에 **전적으로 의존**합니다.

**만약 TimerTrigger가 Storage 계정(`AzureWebJobsStorage`)에 접근할 수 없다면, 락 메커니즘은 작동하지 않습니다.**

- **결과:** 락이 없으면 멀티 인스턴스 환경에서 **모든 인스턴스**가 스케줄 시간에 맞춰 TimerTrigger를 동시에 실행하려고 시도하게 됩니다.
    
- **주의:** Function App은 Azure Storage를 내부적으로 사용하므로, `AzureWebJobsStorage` 연결 문자열을 제거하는 것은 **Function App 자체가 작동을 멈추게 하는 원인**이 됩니다. (예: 트리거 작동, 로그 기록 불가 등)
    

따라서 TimerTrigger를 사용할 때는 락을 해제할 방법은 없으며, **항상 Storage 락을 통해 실행의 배타성(Exclusivity)을 보장**받는다고 이해하는 것이 안전합니다.

---

## host id 커스텀 고정
- `AzureFunctionsWebHost__hostid` 고정하면 된다

```text
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "AzureFunctionsWebHost__hostid": "myuniquefunctionappname123456789",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
        //"StorageQueue": "UseDevelopmentStorage=true",
        "spmsstorage": "UseDevelopmentStorage=true",
        "SqlDbConnectionString": "Server=localhost,1433;Database=TestManagerDb;User Id=Tester;Password=qwer789&*(;TrustServerCertificate=True;",
        "CosmosDbEndpoint": "https://localhost:8081",
        "CosmosDbAuthKey": "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==",
        "BlobStorageConnectionString": "UseDevelopmentStorage=true"
    },
    "AppSettings": {
        // ..
    }
}

```

공식 문서 
- [App settings reference for Azure Functions | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-functions/functions-app-settings#azurefunctionswebhost__hostid)
- [Storage considerations for Azure Functions | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-functions/storage-considerations?tabs=azure-cli#override-the-host-id)


**function app 실행 결과**
<img src="./images/함수 hostId 커스텀 설정.png"/>

<img src="./images/함수 hostId 커스텀 설정2.png"/>