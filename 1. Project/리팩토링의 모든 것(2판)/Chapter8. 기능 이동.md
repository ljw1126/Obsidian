
### 8.1 함수 옮기기 
예시1
```text
trackSummary(..)
- calculateDistance()
- distance(..)
- radians(..)
- calculateTime()

# 리팩터링1.
trackSummary(..)
- calculateDistance()
- distance(..)
- radians(..)
- calculateTime()

top_calculateDistance(..) 최상위로 분리


# 리팩터링2. distacne와 radians 옮기기
trackSummary(..)
- calculateDistance()
	- distance(..)
	- radians(..)
- calculateTime()

top_calculateDistance(..) 최상위로 분리
	- distance(..)
	- radians(..)

# 리팩터링3. 
- calculateDistance()에서 top_calculateDistance() 호출
- 이상 없으면 calculateDistance() 제거
- top_calculateDistance() 직접 호출하도록 변경
- top_calculateDistance() > totalDistance() renaming
- 
```

예시1의 경우 자바스크립트의 함수 중첩 사용하나 자바는 지원하지 않는다.
그래서 **메서드 분리**로 선언하여 구현하였다.

포인트는 함수 중첩을 사용해 함수 옮기기를 하는데 있어 의존하는게 없다면 최상위로 뽑는것으로 보인다

> 중첩 함수를 사용하다보면 숨겨진 데이터끼리 상호 의존하기가아주 쉬우니 중첩함수는 되도록 만들지 말자. p285