

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
### xUnit 


### DB
- 테이블명, 컬럼명은 **UPPER_SNAKE_CASE** 
	- ex. `TEST_TABLE`, `TEST_COLUM`
