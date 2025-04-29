
## Redis
> [!note] 
> 변경이 적고, 접근 빈도가 높은 데이터의 경우 캐싱

**cache aside pattern**
- 대량의 읽기 요청을 고려한 패턴
- 1차적으로 분산 캐시(Redis)에 데이터를 저장
- 그리고 각 서버는 local cache를 사용해 성능 향상하는 전략
- 캐시 성능과 높은 응답성 보장 
- 데이터 수정이 발생하면 1차 분산 캐시(Redis)의 수정이 먼저 일어나야 함

> 하나의 서비스/데이터 특성에 대해 여러 캐시를 나눠 사용하기도 한다

**Write back pattern**
- 대량의 쓰기 요청을 고려한 패턴
- 데이터를 저장할 때 DB에 바로 쿼리하지 않고, 캐시에 모아서 일정 주기 배치 작업을 통해 DB에 반영
- 캐시에 모아놨다가 DB에 쓰기 때문에 쓰기 쿼리 횟수 비용과 부하를 줄일 수 있다
- 단, 캐시에서 오류가 발생하면 데이터를 영구 소실 할 수 있다 -> 디스크에 저장하는 방식 있는 걸로 암
	- 그리고 자주 사용되지 않은 불필요한 리소스를 Redis 저장

🛜 \[REDIS\] 캐시(Cache) 설계 전략 지침 100 % 총 정리 - 인파 기술블로그 ([바로가기](https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-%EC%BA%90%EC%8B%9CCache-%EC%84%A4%EA%B3%84-%EC%A0%84%EB%9E%B5-%EC%A7%80%EC%B9%A8-%EC%B4%9D%EC%A0%95%EB%A6%AC#write_back_%ED%8C%A8%ED%84%B4))
## Spring Webflux
- Reactive Stream API
- non-block
- aysnchronous

### CPU Bound vs I/O Bound
> Spring Webflux는 I/O Bound에 가깝다

**CPU Bound**
- 컨텍스트 스위칭 발생
- 최소화하려면 코어 수 증가 필요 (1 Core가 번갈아 가며 두 앱을 실행)

**I/O Bound**
- Web Application 관점에서 I/O Bound가 발생하는 것은
	- 클라이언트 요청/응답
	- DB
	- 외부 API Server
- I/O 요청 처리량을 늘릴려면
	- 전통적인 방법으로는 **스레드** 늘림 (= 멀티 스레드 방식)
		- 하지만 한계있어서 OOM이 발생가능
		- 스레드를 무리하게 만들거나, 생성/소멸에 대한 리소스 낭비
		- 그래서 `스레드 풀 전략`을 사용함 (물론 컨텍스트 스위칭 비용은 있음)
	- 그리고 서버를 scale-out하고 로드밸런싱 하는 등의 전략을 추가로 생각가능

### Sync / Async

**Async**
- 요청 작업을 기다리지 않고 다음 작업을 실행 가능하다
- Async 작업은 완료 순서를 보장하지 않는다
- ComputableFuture 사용하면 메인 스레드 대신 다른 스레드가 생성되서 처리된다

**I/O Multiplexing**
- i/o 요청에 대한 이벤트를 보내고, 대기 중에 있다가 응답 이벤트를 받으면 다음 i/o 요청 발생

**Thread per request**
- 스레드 풀 전략 중에 캐시 전략이 있었다
- 요청만큼 스레드 바로 생성해서 처리하는 것 (결국 비즈니스 상황에 맞는 전략을 선택하는게 중요)
### Spring MVC vs Webflux

Spring MVC의 경우 
- Servlet 컨테이너와 Spring 컨테이너를 거쳐 하나의 요청 당 Thread 하나씩 할당되어 처리하기 때문에 문제 파악이나 흐름을 이해하는데 직관적
- blocking i/o로 이뤄지기 때문에 대량의 동시성 처리에 한계가 있다

### Reactive Stream
참고
- https://www.reactive-streams.org/
- https://projectreactor.io/

**1. Reatice Stream**
- 비동기 스트림 처리 
- 논블로킹 백프레셔 제공
- Publisher, Subscriber, Subscription, Processor 로 구성
- 이를 구현한 프로젝트 Reactor를 Spring Webflux 에서 활용


### R2DBC
> Reactive Relational DataBase Connectivity

참고. 
- [spring blog - Reactive Programming and Relational Databases](https://spring.io/blog/2018/12/07/reactive-programming-and-relational-databases)
- [Spring Data R2DBC](https://spring.io/projects/spring-data-r2dbc#overview)
- [Spring Data R2DBC Reference](https://docs.spring.io/spring-data/r2dbc/docs/current-SNAPSHOT/reference/html/#preface)
- https://github.com/asyncer-io/r2dbc-mysql


앞에 처리 로직이 비동기이더라도 후반 데이터베이스에 저장하는 로직이 동기이면 
결국 전체가 동기로 동작하는거랑 같음

R2DBC는 JDBC와 달리 
- Asynchoronous Database Access
- Reactive Stream
- Nonblocking I/O
- Open specification
	- H2, MariaDB, MySQL, Oracle, PostgreSQL, ...

r2dbc-api 기준 스펙이 있는듯함

Spring MVC
- RestTemplate (그 외 RestClient, FeignClient을 사용 가능, `library 프로젝트 참고`)
- JDBC, JPA

Spring Webflux
- WebClient
- R2DBC

[R2DBC 공식 문서를 읽어보자](https://docs.spring.io/spring-data/r2dbc/docs/current-SNAPSHOT/reference/html/#preface)


### Reactive Redis
- 일반 Redis 서버와의 통신은 동기적으로 실행됨
- Reactive Redis를 사용해서 비동기 통신 가능하다 함
	- Reactive Stream
	- Nonblocking I/O
	- Spring Data Reactive Redis
		- lettuce - Advanced Java Redis client (깃 저장소)
- spring webflux가 redis 통신할 때 reactive redis 사용
	- spring data reactive redis
	- [스프링 공식 예제](https://spring.io/guides/gs/spring-data-reactive-redis)


### Spring MVC vs Spring Webflux 성능 비교✨
- Spring MVC
	- JPA, Spring Data Redis 사용 
- Spring Webflux
	- R2DBC, Spring Data Reactive Redis 사용 
- 도커 컨테이너로 Redis, MySQL 올리고 성능 비교
- Apache JMeter 사용해 성능 테스트 **3가지** 수행
	- `/health` 체크에 대한 성능 테스트 
	- MySQL 요청에 대한 성능 테스트 
	- Redis 요청에 대한 성능 테스트

### Blockhound
- 비동기 어플리케이션에서 블로킹을 검출하는 기능 제공
- 의존성 추가 필요 