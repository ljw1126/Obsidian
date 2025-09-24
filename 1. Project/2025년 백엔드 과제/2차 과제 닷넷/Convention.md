

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
### xUnit 


### DB
- 테이블명, 컬럼명은 **UPPER_SNAKE_CASE** 
	- ex. `TEST_TABLE`, `TEST_COLUM`
