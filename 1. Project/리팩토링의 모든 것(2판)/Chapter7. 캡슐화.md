
### 7.1
- 자바의 깊은 복사는 어떻게?
	- `rawData()` getter 호출시 javascript는 loadash를 사용하여 깊은 복사 수행 
		- **모든 중첩 구조 포함됨**
		- 자바에서 사용 가능한 deep copy 종류 : https://myborn.tistory.com/23
- 그리고 setUsage(..) 호출시 3 depth는 들어가는데 .. 위임하는게 맞나 싶다

처음부터 혼란 스럽다.. depth가 깊어지다보니
메서드 체인 방식으로 자바스크립트에서 호출했는데
실무에서도 depth가 1이상 넘어본적 없는데, 
내부적으로 setter를 호출하여 파라미터를 넘기는게 맞는건가??


### 