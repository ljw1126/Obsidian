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

> [!note] 서킷 브레이커의 fallback을 설정하지 않으면 기본적으로 어떻게 동작하지? 예외처리는?


**circuit breaker 어노테이션 방식 적용**
```java
@CircuitBreaker(name = "특정할이름", fallbackMethod = "실패시수행할메소드이름")
```
- configuration 클래스를 주입해서, 인터셉터 해서 헤드에 값을 넣거나 예외 처리 커스텀 가능

```java
@CircuitBreaker(name = "특정할이름", fallbackMethod = "실패시수행할메소드이름") @GetMapping("/") 
public String mainP() { 
	return restTemplate.getForObject("/data", String.class); 
}
```

**circuit breaker 설정**
- circuit breaker 적용할 특정 name 메서드에 대해 설정을 커스텀 가능
- 이때 특정한 이름에 대해서 변수 설정을 진행해 인스턴스 별로 다른 circuit breaker 패턴을 지정할 수 있다

```text
// application.properties
resilience4j.circuitbreaker.instances.특정할이름.base-config=설정셋
```


변수에 대한 특정한 이름 설정
```text
resilience4j.circuitbreaker.configs.설정셋.메소드들=값 

#예시 
resilience4j.circuitbreaker.configs.default.failure-rate-threshold=10
```

슬라이딩 윈도우 공통 사항
```text
#실패 및 지연을 체크할 슬라이딩 윈도우 타입 
resilience4j.circuitbreaker.configs.default.sliding-window-type=count_based 

#슬라이딩 윈도우 크기 
resilience4j.circuitbreaker.configs.default.sliding-window-size=5
```

실패에 대한 설정 
```text
#서킷을 오픈할 실패 비율 (실패 수 / 슬라이딩 윈도우 크기) , 현재 10퍼
resilience4j.circuitbreaker.configs.default.failure-rate-threshold=10

#서킷을 오픈하기 위해 최소 실패 수 (슬라이딩 윈도우를 다 채우지 못했지만 최소값을 설정 가능) resilience4j.circuitbreaker.configs.default.minimum-number-of-calls=5
```

지연에 대한 설정
```text
#서킷을 오픈할 지연 비율 (지연 수 / 슬라이딩 윈도우 크기) 
resilience4j.circuitbreaker.configs.default.slow-call-rate-threshold=10 

#지연으로 판단할 시간 
resilience4j.circuitbreaker.configs.default.slow-call-duration-threshold=3000ms
```

half-open 상태 설정
```text
#half open 상태에서 다른 상태로 전환하기 위한 판단 수 
resilience4j.circuitbreaker.configs.default.permitted-number-of-calls-in-half-open-state=10 

#half open 상태 유지 시간 (만약 0이면 위에서 설정한 값 만큼 수행 후 다음 상태로 전환) 
resilience4j.circuitbreaker.configs.default.max-wait-duration-in-half-open-state=0 

#open 상태에서 half open으로 전환까지 기다리는 시간 
resilience4j.circuitbreaker.configs.default.wait-duration-in-open-state=600000ms 

#open 상태에서 half open 으로 자동 전환 (true시 일정 시간이 지난 후 자동 전환) 
resilience4j.circuitbreaker.configs.default.automatic-transition-from-open-to-half-open-enabled=true 

#상태 체크 표시 (actuator용) 
resilience4j.circuitbreaker.configs.default.register-health-indicator=true
```

예외 처리 (아래 예외 또한 실패로 받아드리기 때문에 처리하지 않으면 실패수에 포함 됨)
```text
resilience4j.circuitbreaker.configs.default.ignore-exceptions[0]=java.io.IOException 
resilience4j.circuitbreaker.configs.default.ignore-exceptions[1]=java.util.concurrent.TimeoutException 
resilience4j.circuitbreaker.configs.default.ignore-exceptions[2]=org.springframework.web.client.HttpServerErrorException
```


**fallback 메소드 설정 예시**
```java
@CircuitBreaker(name = "특정할이름", fallbackMethod = "실패시수행할메소드이름") 
@GetMapping("/") public String mainP() { 
	return restTemplate.getForObject("/data", String.class); 
} 

private String 실패시수행할메소드이름(Throwable throwable) { 
	return throwable.getMessage(); 
}
```
- 이때 fallback 메소드는 circuit breaker가 적용된 메소드가 가지는 인자를 필수적으로 가져야 한다
	- 파라미터에 추가하면 된다 (기본 Throwable)


**상황1.** 
- server1 (실행 중), server2 (실행하지 않은 상태)
- 호출 실패에 대한 설정 비율이 10퍼센트이고, 10개가 덜 채워진 상태에서 최소 5개가 실패하면 fallbackMethod를 실행한다
- 즉 server1에서 server2를 5번 호출 실패하면 `OPEN` 상태가 된다 

```text
# 6번째 호출시
CircuitBreaker 'requestData' is OPEN and does not permit further calls
```

**상황2.**
- server 1,2 둘다 실행중 상태
- server2 의 지연시간을 `5초`로 설정
- server1에서 server2 요청시 지연이 10% 또는 처음 최소 5번 발생하면 `OPEN` 상태가 된다

```text
# 6번째 호출시
CircuitBreaker 'requestData' is OPEN and does not permit further calls
```

---

## 7. retry 
- retry 모듈은 실패한 요청을 재시도하는 기능을 가짐
- 자동 재시도 후 fallback method 호출
- retry의 기본 우선 순위는 circucit breaker가 실행된 이후 실행됨
	- 그래서 circuit breaker에 fallback method 설정한 경우 retry 처리하지 않아야 함 (`기본`)
	- fallback method 실행 전에 retry 실행시키고 싶다면 내부 config 설정을 통해 우선 순위 (Order)를 변경하면 된다

실패가 발생해서 필수적으로 재시도해야 하는 Controller, Service 단의 메소드 상단에 retry 어노테이션 선언을 통해 실패 상황을 재시도 할 수 있다.

**어노테이션 방식 적용**
```java
@Retry(name = "특정할이름", fallbackMethod = "실패시수행할메소드이름")
```

**예시**
```java
@Retry(name = "MainControllerMethod1", fallbackMethod = "실패시수행할메소드이름") 
@GetMapping("/") 
public String mainP() { 
	return restTemplate.getForObject("/data", String.class); 
}
```

**name에 대한 retry 설정**
- retry 적용시 특정 name 메소드에 대해 retry 설정을 적용하는 방법 

```text
# 형식
resilience4j.retry.instances.특정할이름.base-config=설정셋

# 예시
resilience4j.retry.instances.MainControllerMethod1.base-config=default
```

**retry 설정**
- application.properties에 retry 모듈이 제공하는 메소드 설정을 진행
- 이때 특정한 이름에 대해서 변수 설정을 진행해 인스턴스 별로 다른 retry 패턴을 지정할 수 있다

```text
resilience4j.retry.configs.설정셋.메소드들=값 #예시

resilience4j.retry.configs.default.max-attempts=10
```

재요청 설정
```text
#재요청 시도 횟수 
resilience4j.retry.configs.default.max-attempts=3 

#재요청 간격 
resilience4j.retry.configs.default.wait-duration=3000ms
```

예외 처리 (retry에 포함)
- 만약 ignore에 포함되어 있는 경우 ignore 우선
```text
resilience4j.retry.configs.default.retry-exceptions[0]=java.io.IOException
```

예외 처리 무시
```text
resilience4j.retry.configs.default.ignore-exceptions[0]=java.io.IOException
```


**fallback method 설정**
- 재시도 최대 횟수가 초과된 이후 실행한 메소드로 어노테이션 인자를 통해 설정
- `@CircuitBreaker` 와 동일하여 생략

---

## 8. bulk head
- 직접 실무에 사용할 경우 자체 검증 필요
- 간단 테스트만 진행
- `bulk head` 모듈은 동시 요청을 제한하는 기능을 가짐
- `bulk head` 모듈은 두 가지 타입을 제공
	- `bulkhead (semaphore, 세마포어)`
		- 세마포어 알고리즘을 통해 공유 자원 접근을 제한하는 방식
	- `thread-pool-bulkhead (fixed thread pool)`
		- 고정된 사이즈의 스레드를 지정하는 방식
	- 하나의 메소드에 대해 두 bulkhead 타입 중 하나를 선택해 구현해야 함

**각 타입이 존재하는 이유**
`@Async` 어노테이션이 선언된 비동기 메소드 + `bulkhead(semaphore)` 방식의 자원 제한을 사용할 경우 요청에 의해 생성된 스레드는 재사용되지 않고 게속적으로 증가하는 결과를 가진다고 한다. 
(세마포어 카운트가 0이 되어도 새로운 스레드를 생성하기 때문에 bulk head 기능을 못함)

>[!note] @Async 방식을 사용하여 다른 스레드 생성해서 이벤트 처리하거나 한다

따라서 비동기 메소드의 경우 해당 메소드에 대해 전체 스레드 개수를 제한하는 `thread-poll-bulkhead` 방식을 사용하는 것이 좋다고 한다

[Bulkhead pattern: semaphore vs threadPool](https://dev.to/gabrielaramburu/bulkhead-pattern-semaphore-vs-threadpool-226p)


**특정 메서드에 bulk head 패턴 적용**
- 동시 요청 제한해야 하는 메서드에 추가 

```java
// 세마포어 방식
@Bulkhead(name = "MainControllerMethod1", type = Bulkhead.Type.SEMAPHORE, fallbackMethod = "실패시수행할메소드이름") 
@GetMapping("/") 
public String mainP() { 
	return restTemplate.getForObject("/data", String.class); 
}

// 스레드 풀 벌크헤드 방식
@Bulkhead(name = "MainControllerMethod1", type = Bulkhead.Type.THREADPOOL, fallbackMethod = "실패시수행할메소드이름") 
@GetMapping("/") 
public String mainP() { 
	return restTemplate.getForObject("/data", String.class); 
}
```


**name에 대한 bulk head 설정**
- 마찬가지로 `application.properties` 설정


bulkhead (semaphore) 방식
```text
resilience4j.bulkhead.instances.특정할이름.base-config=설정셋
```

thread-pool-bulkhead (fixed thread pool) 방식
```text
resilience4j.thread-pool-bulkhead.instances.특정할이름.base-config=설정셋
```


**bulk head 설정**
- 마찬가지로 `application.properties` 설정

bulkhead (semaphore) 방식
```text
#동시 요청 세마포어 카운터 수 
resilience4j.bulkhead.configs.default.max-concurrent-calls=1

#동시 요청 초과시 웨이팅 시간 (초과되면 exception 발생) 
resilience4j.bulkhead.configs.deafult.max-wait-duration=1000ms
```

thread-pool-bulkhead (fixed thread pool) 방식
```text
#최대 스레드 풀 
resilience4j.thread-pool-bulkhead.configs.defatul.max-thread-pool-size=10

#기본 스레드 풀 
resilience4j.thread-pool-bulkhead.configs.default.core-thread-pool-size=5

#스레드풀 초과시 요청이 기다릴 대기큐 크기
resilience4j.thread-pool-bulkhead.configs.default.queue-capacity=50
```


**참고. thread-pool-bulkhead 오류에 대한 GitHub Issue**
- `errorThreadPool bulkhead is only applicable for completable futures`
- [링크](https://github.com/resilience4j/resilience4j/issues/826)

**fallback method 설정**
- `@CircuitBreaker` 와 동일하여 생략

---

## 9. actuator 서킷 상태 확인 

**actuator 의존성 추가** 
- `build.gradle` 설정
```text
dependencies { 
	implementation 'org.springframework.boot:spring-boot-starter-actuator' 
}
```

**actuator 엔드포인트 활성화**
- `application.properties` 설정 
```text
# actuator 
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
management.health.circuitbreakers.enabled=true

# resilience4j
resilience4j.circuitbreaker.configs.default.register-health-indicator=true
```


>[!info] 
>- 엔드 포인트(요청 경로) : `/actuator/health`
>- 모니터링 도구 연결해서 시각화 가능



