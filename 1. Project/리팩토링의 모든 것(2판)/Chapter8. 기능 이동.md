
### 8.1 함수 옮기기 
예시1의 경우 자바스크립트의 중첩 함수를 사용하나 자바는 중첩 함수를 지원하지 않는다.
고로 메서드를 개별로 선언하여 사용할 수 밖에 없었다.


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