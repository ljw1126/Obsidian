# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 11월 27일`

## 카테고리
`#CSharp`, `#Redis`, `#TestContainer`, `#XUnit`

## 주제: TestContainer 기반 Redis 테스트
### 1. 문제 상황 또는 학습 배경
- 접속 정보가 하드코딩 되어 있는 헬퍼 클래스와 강결합되어 있다보니 로컬 테스트가 어려웠음
- Repository / DI 패턴 적용하기 위해 테스트 기반으로 기능 구현 
- Redis 조회/쓰기 두 가지 메서드만 구현하면 되고, 추상화 모델은 2개


### 2. 핵심 내용 / 개념 정리
#### RedisFixture 생성
```cs
public class RedisFixture : IAsyncLifetime
{
    private static readonly RedisContainer _redisContainer = new RedisBuilder()
        .WithCleanUp(true)
        .Build();

    // ConnectionMultiplexer는 Redis 서버와의 연결을 관리하는 핵심 객체입니다.
    private ConnectionMultiplexer _multiplexer;

    // 테스트 클래스에서 접근할 수 있도록 노출합니다.
    public ConnectionMultiplexer Connection => _multiplexer;

    public string ContainerId => $"{_redisContainer.Id}";
    public string ConnectionString => _redisContainer.GetConnectionString();

    public async Task InitializeAsync()
    {
        // 1)
        if (!_redisContainer.State.HasFlag(TestcontainersStates.Running))
        {
            await _redisContainer.StartAsync();
        }

        if (_multiplexer == null || !_multiplexer.IsConnected)
        {
            _multiplexer = await ConnectionMultiplexer.ConnectAsync($"{ConnectionString},allowAdmin=true");
        }
    }

    public async Task DisposeAsync()
    {
        _multiplexer?.Dispose();

        await _redisContainer.StopAsync();
    }
}
```
- 1)  
	- ContainerPerClass로 클래스 실행시 하나의 컨테이너를 공유하도록 함
	- 그런데 경우에 따라 컨테이너가 두번 실행되는 경우를 확인함 (ContainerId 출력해서)
	- 그래서 하나를 강제하기 위해 static하게 `_redisContainer` 선언 후 `InitializeAsync()`에서 조건문 통해 한번만 실행하도록 하여 해결 


#### Redis Repository Test 골격 작성 
```cs
public class LastPositionRedisRepositoryTests : IClassFixture<RedisFixture>, IAsyncLifetime
{
    private readonly RedisFixture _fixture;
    private readonly ITestOutputHelper _output;
    private readonly IServer _redisServer; // 관리
    private readonly IDatabase _redisDb; // 데이터 읽기/쓰기
    private readonly ILastPositionCacheRepository _repository;

    public LastPositionRedisRepositoryTests(RedisFixture redisFixture, ITestOutputHelper output)
    {
        _fixture = redisFixture;
        _output = output;
        _redisServer = redisFixture.Connection.GetServer(endpoint: redisFixture.Connection.GetEndPoints().First());
        _redisDb = redisFixture.Connection.GetDatabase();
        _repository = new LastPositionRedisRepository(redisFixture.Connection);
    }

    public async Task InitializeAsync()
    {
        _output.WriteLine($"ContainerId : {_fixture.ContainerId}");
        await _redisServer.FlushDatabaseAsync();
        await Task.Delay(100); // StackExchange.Redis.RedisConnectionException 때문에 짧은 지연 시간 주어 연결 안정화 대기
    }
    public Task DisposeAsync() => Task.CompletedTask;
    
	// 테스트 작성
	// IDatabase로 데이터 초기화
}
```
- Delay(100) 준 이유가 timeout exception이 발생하여 추가


#### 그 외.

**Redis 컨테이너 생성**
```shell
$ docker run --name local-redis -p 6379:6379 -d redis
```


**local.settings.json 설정**
```text
"Redis:ConnectionString": "localhost:6379,syncTimeout=5000,abortConnect=false,allowAdmin=true"
```
- 테스트시 데이터 초기화를 위해 flush가 필요하여 옵션 추가


**Scope 설정**
```cs
_ = services.AddSingleton<IConnectionMultiplexer>(_ => ConnectionMultiplexer.Connect(config["Redis:ConnectionString"]));
```
- Repository에서 ConnectionMutilpexer를 주입받아 사용한다 

```cs
private readonly IDatabase _redis;

public LastPositionRedisRepository(IConnectionMultiplexer multiplexer)
{
    _redis = multiplexer.GetDatabase(0);
}
```

또는 IDatabase 자체를 Singleton으로 등록해서 사용해도 된다
```cs
_ = services.AddSingleton<IDatabase>(sp => 
{
    var multiplexer = sp.GetRequiredService<IConnectionMultiplexer>();
    return multiplexer.GetDatabase(0);
});


// 이후 Repository 생성자에서는 IDatabase를 직접 주입받습니다. // public LastPositionRedisRepository(IDatabase db) { ... }
```