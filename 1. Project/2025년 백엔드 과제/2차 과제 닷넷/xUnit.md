- [공식 문서 - What NuGet Packages Should I Use? [xUnit.net v2] | xUnit.net](https://xunit.net/docs/nuget-packages-v2)

### Visual Studio 패키지 설치

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

### Assert 축약
```c#
using Xunit;
using static Xunit.Assert; // ✅

namespace BasicEntityFrameworkExample.Tests
{
    public class UnitTest1
    {
        [Fact]
        public void PassingTest()
        {
            int expected = 4;
            int actual = Add(2, 2);
			
			// static 선언을 통해 축약 가능
            Equal(expected, actual);
        }
        
        int Add(int x, int y)
		{
		    return x + y;
		}
	}
	
	
}
```

### Assert 대안 
- xUnit Assert (기본)외에 FluentAssertions 가 대안이 있다 
	- [Introduction - Fluent Assertions](https://fluentassertions.com/introduction)
		- 사용하는지는 모르겠다 .. 🤓🧐

### Assert 공식 예제 깃허브 
[samples.xunit/v2 at main · xunit/samples.xunit](https://github.com/xunit/samples.xunit/tree/main/v2)
