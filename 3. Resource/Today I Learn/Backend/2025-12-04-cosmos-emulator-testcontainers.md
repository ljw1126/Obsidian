# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 04일`

## 카테고리
`#CSharp`, `#Testcontainers`, `#ContainerPerCollection`, `#Cosmos`, `#xUnit`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- 최종적으로 저장되어야 하는 cosmos db에 대한 테스트를 진행하기 위해 Testcontainers 도입
- CosmosFixture 설정부터, 초기화 이슈가 많이 발생하여 트러블 슈팅

### 2. 핵심 내용 / 개념 정리

**패키지 설치**
```shell
$ dotnet add package Testcontainers.Xunit  // 설치한 한 경우
$ dotnet add package Testcontainers.CosmosDb
```

**CosmosFixture 정의**
```cs
using DotNet.Testcontainers.Builders;
using DotNet.Testcontainers.Containers;
using Microsoft.Azure.Cosmos;
using Testcontainers.CosmosDb;

namespace CCTVAnalysis.Tests.Fixtures;

// https://devblogs.microsoft.com/ise/testing-with-testcontainers/
// https://github.com/Azure/azure-cosmos-db-emulator-docker?tab=readme-ov-file#linux-based-emulator-preview
public class CosmosFixture : IAsyncLifetime
{
    public string DEFAULT_ACCOUNT_KEY = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";

    private readonly CosmosDbContainer _cosmosContainer = new CosmosDbBuilder()
        .WithImage("mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:vnext-preview")
        .WithCleanUp(true)
        .WithPortBinding(8081, 8081)
        .WithEnvironment("AZURE_COSMOS_EMULATOR_PARTITION_COUNT", "1")
        .WithWaitStrategy(Wait.ForUnixContainer().UntilExternalTcpPortIsAvailable(8081))
        .WithCreateParameterModifier(parameters =>
        {
            parameters.Tty = false; // Disable TTY allocation
        })
        .Build();

    public string ContainerId => $"{_cosmosContainer.Id}";
    public string ConnectionString => _cosmosContainer.GetConnectionString();

    private CosmosClient _cosmosClient;
    private ushort Port => _cosmosContainer.GetMappedPublicPort(8081);
    private string Hostname => _cosmosContainer.Hostname;
    public CosmosClient CosmosClient => _cosmosClient;

    // 1. 스레드 동기화를 위한 SemaphoreSlim 객체 생성 (동시 접근을 1개로 제한)
    private static readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);

    public async Task InitializeAsync()
    {
        // 2. 세마포어를 획득하여 진입 (첫 번째 스레드만 즉시 통과)
        await _semaphore.WaitAsync();
        try
        {
            // 여기서 Race Condition이 발생해서 Docker.DotNet.DockerApiException 발생했다고 유추..
            if (!_cosmosContainer.State.HasFlag(TestcontainersStates.Running))
            {
                await _cosmosContainer.StartAsync();
                await Task.Delay(TimeSpan.FromSeconds(15));
            }

            var clientOptions = new CosmosClientOptions
            {
                ConnectionMode = ConnectionMode.Gateway,
                HttpClientFactory = () => new HttpClient(new HttpClientHandler
                {
                    ServerCertificateCustomValidationCallback = HttpClientHandler.DangerousAcceptAnyServerCertificateValidator,
                    SslProtocols = System.Security.Authentication.SslProtocols.Tls12,
                })
            };

            // NOTE. https 사용시 ssl 관련 예외 발생

            _cosmosClient = new CosmosClient($"http://{Hostname}:{Port}", DEFAULT_ACCOUNT_KEY, clientOptions);
        }
        finally
        {
            _ = _semaphore.Release();
        }
    }

    public async Task DisposeAsync()
    {
        if (_cosmosContainer != null)
        {
            try
            {
                await _cosmosContainer.StopAsync();
                await _cosmosContainer.DisposeAsync();
            }
            catch
            {
                // Ignore exceptions during clean up
            }
        }
    }

    public static async Task<Container> CreateContainerAsync(CosmosClient cosmosClient, string dbName, string containerName, string partitionKey)
    {
        // 1. DB 생성 
        var databaseResponse = await cosmosClient.CreateDatabaseIfNotExistsAsync(dbName);

        // 2. 컨테이너 생성
        var containerResponse = await databaseResponse.Database.CreateContainerIfNotExistsAsync(
            containerName,
            partitionKeyPath: partitionKey,
            throughput: 400
        );

        return containerResponse.Container;
    }
}

```


**💩 트러블슈팅1**
cosmos emulator 이미지의 태그를 latest로 했을 때 SSL 이슈 발생
```shell
System.Net.Http.HttpRequestException : The SSL connection could not be established, see inner exception.
---- System.IO.IOException : Received an unexpected EOF or 0 bytes from the transport stream.
```

**✅해결**
- 태그를 `vnext-preview` 사용하니 해결
- ms 게시물 참고해서 설정
	- https://devblogs.microsoft.com/ise/testing-with-testcontainers/
- 그외 깃허브 이슈 참고 
	- https://github.com/testcontainers/testcontainers-dotnet/issues/421


**💩 트러블슈팅2**
- `IClassFixture<CosmosFixture>` 활용해 클래스당 하나의 컨테이너를 공유하도록 함 
- 테스트 메서드가 몇개 없는데 간간이 아래 예외가 발생함

```shell
Docker.DotNet.DockerApiException : Docker API responded with status code=InternalServerError, response={"message":"failed to set up container networking: driver failed programming external connectivity on endpoint cool_gauss (fa4cbefa5f0436dcebfb08b083c67ecfeb65214ffa8e936b2d9de9bd4c9a6ede): Bind for 0.0.0.0:8081 failed: port is already allocated"}
```

**✅해결**
CosmosFixture 초기화 부분에서 병렬 실행시 Race Condition 이슈가 발생한게 아닌가 추측
(이에 대한 테스트를 확인하지 않음)
```cs
if (!_cosmosContainer.State.HasFlag(TestcontainersStates.Running))
{
    await _cosmosContainer.StartAsync();
    await Task.Delay(TimeSpan.FromSeconds(5));
}
```

gemini한테 질문시 아래의 방법을 가이드하여 해결

```cs
// 1. 스레드 동기화를 위한 SemaphoreSlim 객체 생성 (동시 접근을 1개로 제한)
private static readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);

public async Task InitializeAsync()
{
    // 2. 세마포어를 획득하여 진입 (첫 번째 스레드만 즉시 통과)
    await _semaphore.WaitAsync();
    try
    {
        // 여기서 Race Condition이 발생해서 Docker.DotNet.DockerApiException 발생했다고 유추..
        if (!_cosmosContainer.State.HasFlag(TestcontainersStates.Running))
        {
            await _cosmosContainer.StartAsync();
            await Task.Delay(TimeSpan.FromSeconds(10));
        }

        var clientOptions = new CosmosClientOptions
        {
            ConnectionMode = ConnectionMode.Gateway,
            HttpClientFactory = () => new HttpClient(new HttpClientHandler
            {
                ServerCertificateCustomValidationCallback = HttpClientHandler.DangerousAcceptAnyServerCertificateValidator,
                SslProtocols = System.Security.Authentication.SslProtocols.Tls12,
            })
        };

        // NOTE. https 사용시 ssl 관련 예외 발생

        _cosmosClient = new CosmosClient($"http://{Hostname}:{Port}", DEFAULT_ACCOUNT_KEY, clientOptions);
    }
    finally
    {
        _ = _semaphore.Release();
    }
}
```



**💩 트러블슈팅3**
- 전체 테스트 실행시 cosmos emulator 쪽에서 두 가지 예외 발생 
- 컨테이너가 초기화되기까지 충분히 대기가 되지 않아 발생한 이슈로 추측

```shell
# 실패1
Docker.DotNet.DockerApiException : Docker API responded with status code=InternalServerError, response={"message":"failed to set up container networking: driver failed programming external connectivity on endpoint competent_raman (94eaa466dbdbeaf9df2297dec93700f1241bdfae0f6730a4d6b841883490753f): Bind for 0.0.0.0:8081 failed: port is already allocated"}

# 실패2
System.TimeoutException : The operation has timed out.
```


**✅해결**
- 1. CollectionDefinition과 Collection으로 컨테이너 사용하는 클래스를 묶음
- 2. CosmosFixture에서 `InitializeAsync` 초기화시 딜레이를 20초로 증가 
	- `await Task.Delay(TimeSpan.FromSeconds(20));`
```cs
using CCTVAnalysis.Tests.Fixtures;

[CollectionDefinition("CosmosDB Collection")]
public class CosmosTestCollection : ICollectionFixture<CosmosFixture>
{
}
```

```cs
[Collection("CosmosDB Collection")]
public class CctvAnalysisRepositoryTests : IAsyncLifetime
{
    private const string DATABASE_NAME = "..";
    private const string CONTAINER_NAME = "..";
    private const string PARTITION_KEY = "/..";

    private readonly ITestOutputHelper _output;
    private readonly CosmosFixture _fixture;
    private Container _container;
    private ICctvAnalysisRepository _repository;

    public CctvAnalysisRepositoryTests(ITestOutputHelper output, CosmosFixture fixture)
    {
        _output = output;
        _fixture = fixture;
    }

    public async Task InitializeAsync()
    {
        _container = await CosmosFixture.CreateContainerAsync(_fixture.CosmosClient, DATABASE_NAME, CONTAINER_NAME, PARTITION_KEY);
        await Task.Delay(200);
        _repository = new CctvAnalysisRepository(new CosmosDbClient(_fixture.CosmosClient, DATABASE_NAME, CONTAINER_NAME));
        _output.WriteLine($"Container Id : {_fixture.ContainerId}");
    }

    public async Task DisposeAsync()
    {
        if (_container != null)
        {
            _ = await _fixture.CosmosClient.GetDatabase(DATABASE_NAME).DeleteAsync();
        }
    }
	

	// .. 테스트 케이스 작성    
}
```