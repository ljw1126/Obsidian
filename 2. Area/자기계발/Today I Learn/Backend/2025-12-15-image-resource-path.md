📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 15일`

## 카테고리
`#CSharp`, `#xUnit`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- 이미지 파일 조회/저장에 대한 비즈니스 로직을 검증하기 위해 아래 로직을 사용
```cs
 // ⭐ 컴파일러가 현재 파일의 전체 경로를 주입하도록 하는 헬퍼 메서드입니다.
 private static string GetThisFileDirectory([CallerFilePath] string path = null) => Path.GetDirectoryName(path);
 
 
// 현재 테스트 파일이 있는 디렉터리 (예. C:\...\CCTVAnalysis.Tests\Examples\) 
var currentFileDirectory = GetThisFileDirectory(); 
// C:\...\CCTVAnalysis.Tests\
var projectRootDirectory = Path.GetFullPath(Path.Combine(currentFileDirectory, ".."));  
// C:\...\CCTVAnalysis.Tests\Images\1280x720.jpg
var imageFullPath = Path.Combine(projectRootDirectory, "Images", fileName); 

await using var fileStream = File.OpenRead(imageFullPath);

// do something
```

- 디렉터리 구조가 다르다보니 상대 경로로 parent directory를 찾는 코드가 불편하다 느낌
```text
📁 CCTVAnalysis.Test
|-- 📁Exmaples
|------ImageSharpLocalTest.cs
|-- 📁 Images
|--📁 RepositoryImplements
|------📁Blob
|--📁 ServiceImplements
|------ImageResizeServiceTests.cs
|------ ...
```

### 2. 핵심 내용 / 개념 정리
- 테스트 프로젝트 빌드시 Images 디렉터리도 포함되도록 설정
- 그러고나서 아래와 같이 Images 디렉터리 접근해 사용하도록 리팩터링

csproj 설정 파일에 내용 추가
```xml
 <ItemGroup>
   <Content Include="Images\**\*.*">
     <CopyToOutputDirectory>Always</CopyToOutputDirectory>
     <Link>Images\%(RecursiveDir)%(FileName)%(Extension)</Link>
   </Content>
 </ItemGroup>
```

결과적으로 아래와 같이 이미지 조회가 간결해짐
```cs
 var outputDirectory = AppContext.BaseDirectory;
 var imageFullPath = Path.Combine(outputDirectory, "Images", "1280x720.jpg");

 if (!File.Exists(imageFullPath))
 {
     throw new FileNotFoundException($"Test image file not found at path: {imageFullPath}");
 }

 return File.ReadAllBytes(imageFullPath);
```

**참고. Azurite 테스트 컨테이너 기반 통합 테스트 코드**
```cs
[Collection("Azurite Collection")]
public class CctvImageRepositoryIntegrationTests : IAsyncLifetime
{
    private static TimeSpan immediateRetry(int retryCount) => TimeSpan.Zero;

    private readonly ITestOutputHelper _output;
    private readonly AzuriteFixture _fixture;
    private readonly BlobContainerClient _containerClient;
    private readonly IImageRepository _repository;
    private const string BLOB_CONTAINER_NAME = ..; // blob 컨테이너명
    private const string DUMMY_BLOB_PATH =  ..; // 저장 경로

    public CctvImageRepositoryIntegrationTests(ITestOutputHelper output, AzuriteFixture fixture)
    {
        _output = output;
        _fixture = fixture;
        _containerClient = fixture.BlobServiceClient.GetBlobContainerClient(BLOB_CONTAINER_NAME);
        _repository = new CctvImageRepository(new AzureBlobContainerWrapper(_containerClient), immediateRetry);
    }

    public async Task InitializeAsync()
    {
        _ = await _containerClient.CreateIfNotExistsAsync();

        await UploadTestImageAsync();

        _output.WriteLine($"Created container: {_fixture.ContainerId}");
    }

    private async Task UploadTestImageAsync()
    {
        byte[] testImageBytes = GetValidTestImageBytes();

        var blobClient = _containerClient.GetBlobClient(DUMMY_BLOB_PATH);

        await using var stream = new MemoryStream(testImageBytes);

        _ = await blobClient.UploadAsync(stream, overwrite: true);
        _output.WriteLine($"Uploaded test image to: {BLOB_CONTAINER_NAME}/{DUMMY_BLOB_PATH}");
    }

    private byte[] GetValidTestImageBytes()
    {
        var outputDirectory = AppContext.BaseDirectory;
        var imageFullPath = Path.Combine(outputDirectory, "Images", "1280x720.jpg");

        if (!File.Exists(imageFullPath))
        {
            throw new FileNotFoundException($"Test image file not found at path: {imageFullPath}");
        }

        return File.ReadAllBytes(imageFullPath);
    }

    public async Task DisposeAsync()
    {
        await _containerClient.DeleteIfExistsAsync();
    }

    [Fact]
    public async Task LoadImageAsyncTest()
    {
        // Arrange
        // Act
        var actual = await _repository.LoadImageAsync(DUMMY_BLOB_PATH);

        // Assert
        actual.ShouldNotBeNull();
        actual.Width.ShouldBe(1280);
        actual.Height.ShouldBe(720);
    }
}
```