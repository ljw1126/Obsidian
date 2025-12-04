

### C# 

**✅ 함수, 클래스, 멤버 변수 선언시**
- `UpperCamelCase` 사용

//TODO. 하이라이트 처리 해주기 
```c#
 [Route("api/[controller]")] // api/TodoItems , 대소문자 구분 x
 [ApiController]
 public class TodoItemsController : ControllerBase { 
	// .. 
	
	[HttpGet]
	public async Task<ActionResult<IEnumerable<TodoItemDTO>>> GetTodoItems()
	{
	    return await _context.TodoItems
	        .Select(x => ItemToDto(x))
	        .ToListAsync();
	}
 }
 
 namespace TodoApi.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string? Name { get; set; }
        public bool IsComplete { get; set; }
        public string? Secret { get; set; }
    }
}
 
```


**✅ DBO(▶️ Entity)**
- JPA 같은 느낌으로 어노테이션 기반 컬럼명 명시/맵핑 가능
- 이때 클래스내 변수명은 **UpperCamelCase**

**✅ 레파지토리 패턴**
- [F-lab - 레포지토리 패턴의 이해와 활용](https://f-lab.ai/en/insight/understanding-repository-pattern-20250305)

✅ 스택 오버플로우 C# 컨벤션
- [Code style for private methods in C# - Stack Overflow](https://stackoverflow.com/questions/2758684/code-style-for-private-methods-in-c-sharp#:~:text=All%20method%20names%20in%20C%23%20start%20with%20an,asking%20for%20a%20%C2%BBwhy%C2%AB%20is%20a%20little%20misplaced.)
	- private `_멤버`

컨벤션 가이드 (MS AI)

```text
1. General Rules
- Camel Case: Use camel case for local variables and private fields (e.g., myVariable).
- Descriptive Names: Choose meaningful and descriptive names that clearly indicate the purpose of the variable.
- Avoid Abbreviations: Avoid using short or unclear abbreviations unless they are widely understood (e.g., use customerId instead of custId).
- No Special Characters: Variable names should not include special characters except underscores (_) in specific cases (e.g., private fields).
Avoid Reserved Keywords: Do not use C# reserved keywords as variable names unless prefixed with @ (e.g., @class).

2. Specific Guidelines
- Local Variables: Use camel case (e.g., totalAmount, isAvailable).
- Private Fields: Use camel case prefixed with an underscore (_) (e.g., _userName, _isActive).
- Constants: Use Pascal case with all uppercase letters and underscores for word separation (e.g., MAX_VALUE, DEFAULT_TIMEOUT).
- Static Fields: Use Pascal case (e.g., GlobalCounter, DefaultSettings).


// 예시
// Local variable
int totalAmount = 100;

// Private field
private string _userName;

// Constant
const int MAX_RETRIES = 5;

// Static field, 정적 필드
public static int GlobalCounter = 0;
```
### xUnit 


### DB
- 테이블명, 컬럼명은 **UPPER_SNAKE_CASE** 
	- ex. `TEST_TABLE`, `TEST_COLUM`
