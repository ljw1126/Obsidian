- **참고.** 
	- [유튜브 - 개발자 유미](https://youtu.be/UpwJOImeYcA?si=XYcVxNXeVQNNlCLI)
	- [블로그 - 개발자 유미](https://www.devyummi.com/)

## 1. MSA 장애 대응
**MSA 장애 상황과 대응**
- MSA는 여러 개의 마이크로 서비스가 서로 호출을 하며 하나의 시스템을 이룬다
- 각각의 서비스는 무응답, 지연, 실패와 같은 장애 상황이 발생가능하다
- 이때 해당 서비스의 circuit를 open하여 일시적으로 다른 작업을 처리하도록 하는 패턴이 `서킷 브레이커 패턴` 이다
- 서킷 기반의 장애 대응 솔루션을 제공하는 프레임워크는 다음 두 가지가 있다 
	- `Netflix Hystrix` : 지원 중단
	- `Resilience4J` : Spring Cloud에서 Hystrix를 계승, 자바용 내결함성 라이브러리로 장애 상황에 대한 여러 솔루션을 제공

**장애 상황이란?**
- 실패: 요청시 응답이 안옴 (health check로 확인 가능)
- 지연: 요청 했는데 평소보다 늦게 응답이 옴 (응답이 왔기 때문에 파악하기 어려움)

---

## 2. 모듈별 역할 및 동작
- [Spring Cloud Circuit Breaker](https://docs.spring.io/spring-cloud-circuitbreaker/reference/index.html)
- [resilience4j 공식 문서](https://resilience4j.readme.io/docs/getting-started)

**Resilience4J 제공 모듈**
- `circuit-breaker`
	- 회로 차단기 
	- 실패, 지연
- `fall-back`
	- circuit-breaker가 실패 또는 지연 상황 발생으로 circuit을 open할 경우 대비책으로 동작할 메서드 설정
- `retry`
	- 실패한 요청을 일정 시간 이후에 자동 재시도하는 모듈
- `bulk-head`
	- 동시(병렬) 실행 제한을 위한 모듈
- `rate-limiter`
	- 속도(성능) 제한
	- 마이크로 서비스 내부 실행 중 일정 비율 이상의 부하를 막기 위한 속도 제한 모듈
- `time-limiter`
	- 시간 초과 제한
	- 마이크로 서비스 내부 실행 중 일정 시간 이상의 지연을 막기 위한 시간 제한 모듈
- `cache`
	- 요청에 대한 결과 저장(캐싱)을 위한 모듈


> [!info] 서킷 브레이커는 외부 모듈과의 통신 지연을 다루고, time-limiter 모듈은 해당 서버의 스레드의 실행 시간 지연에 대한 정책을 다룸


`circuit-breaker`
- 핵심 모듈로 장애 또는 지연 상황에서 circuit을 일시적으로 오픈하는 기능을 제공
- 서킷을 오픈하는 조건은 아래와 같다
	- `실패`: 호출 후 특정 비율 이상 실패
	- `지연` : 호출 후 특정 비율 이상 지연

![[스크린샷 2025-03-22 오후 9.30.15.png]]
- `CLOSED` : 평상시 정상 상태 
- `OPEN` : 평상시 상태에서 사용자가 설정한 임계치 이상의 지연율 및 실패율이 달성되면 회로를 끊어버리는 상태
- `HALF_OPEN` : `OPEN` 상태에서 설정한 시간이 지난 후 `HALF_OPEN` 상태로 전환, 이때 `CLOSED`, `OPEN` 전환 여부를 판단한다


**함께 읽어보면 좋을 자료**
- [G마켓 - API 메시업과 Fault Tolerance 문제 해결 전략](https://dev.gmarket.com/86)
- [올리브영-CircuitBreaker를 사용한 장애 전파 방지](https://oliveyoung.tech/2023-08-31/circuitbreaker-inventory-squad/)

---


---
## 5. 슬라이딩 윈도우 기법
- 네트워크간 패킷 교환시에도 사용되고 Resilience4J 안에서도 사용된다
- 슬라이딩 윈도우 기법을 통해 지정해 둔 사이즈의 슬라이딩 윈도우 터널을 생성하고 내부에서 실패 및 지연 개수를 측정한다

![[스크린샷 2025-03-22 오후 10.09.14.png]]

---

## 6. 서킷 브레이커 구현
- `server1(8080) -> server2(9000)` 호출하는 형태의 멀티 모듈 프로젝트 생성

서킷 브레이커의 fallback을 설정하지 않으면 기본적으로 어떻게 동작하지? 예외처리는?


