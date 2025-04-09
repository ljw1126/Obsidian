

## @Transactional 사용에 대해
- [YouTube - 토비의 스프링](https://www.youtube.com/watch?v=-961J2c1YsM&list=PLv-xDnFD-nnnh1E86UATCcZm2TUePEyAu)
- [인프런 - 테스트에서의 Transactional 사용에 대해 질문이 있습니다](https://www.inflearn.com/community/questions/792383/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%97%90%EC%84%9C%EC%9D%98-transactional-%EC%82%AC%EC%9A%A9%EC%97%90-%EB%8C%80%ED%95%B4-%EC%A7%88%EB%AC%B8%EC%9D%B4-%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4)
- [기억보단 기록을 - 테스트 데이터 초기화에 @Transactional 사용하는 것에 대한 생각(반대파 입장ㅋㅋ)](https://jojoldu.tistory.com/761)

**테스트 실행 전**에 트랜잭션을 실행하고, 테스트 실행 과정 중에 애플리케이션 레벨에 있는 트랜잭션을 참여시켜 버린다.
- 물리적인건 테스트의 트랜잭션이고, 그 하위에 논리적인 트랜잭션은 테스트 실행시 참여되는 애플리케이션 레벨의 트랜잭션을 뜻하는 듯하다

**롤백 테스트**
- DB를 하나 쓰는데 각 테스트가 서로 영향을 주지 않도록 하기 위해 테스트 하는 동안 건드린 작업을 초기 상태로 돌린다.
	- 외부 시스템에 의존하는 경우 테스트가 서로 영향을 받아 실패할 수 있다
	- 그래서 `isolation`이 중요
- `Spring v1.1.2` 에 등장
	- 애노테이션 지원하지 않던 시절인데 애노테이션 없이도 동작 가능하도록 구현할 수 있다
	- `AbstractTransactionalSpringContextTests` - Deprecated Spring v3.0

**문제**
- DB 사용하는 테스트를 사용했는데, 테스트 코드에서는 더티 체킹이 되서 반영됨
	- 테스트 시작 전에 트랜잭션이 시작되어 애플리케이션 레벨의 트랜잭션이 참여되어 동작 (테스트 종료 전까지 트랜잭션 유지)
- 그러나 실제 서버 실행시 
	- 서비스 레이어에서 트랜잭션이 끝나고 detech 상태가 되어 더티 채킹이 되지 않아 반영 안되는 이슈가 발생
	- 특히 JPA에서 문제를 일으킴

> 테스트 케이스가 잘 동작한다는 것이 실제 운영 환경에서 그대로 잘 동작할 것이라는 보장이 없다. 


**클린업**
- 테스트 사이에 데이터베이스를 초기화 돌릴 수 있다.
	- 하지만 매번 작성하자니 테스트 코드가 복잡해질 수 있다
- 테스트에 @Transactional을 사용하는 것이 항상 좋은 것이 아니다
	- 어떤 경우에는 다른 방식이 더 적합할 수 있다

> 물론 이에 가장 좋은 답은 '테스트 코드가 실제 시스템의 동작을 정확히 반영하도록 작성되어야 한다'는 것입니다. 이를 위해 테스트 코드와 실제 코드 사이의 동작 차이를 최소화해야 합니다.

JPA - flush 해서 rollback 

JPA 사용하고 롤백 테스트를 할 경우 repository에서 save()만 호출할 경우 <u>메모리에만 존재하고 실제 DB에는 날라가지 않음</u> -> flush 강제 필요한데, 성공했다 착각하고 넘어가는데 심각한 문제

> 단점에도 불구하고 `@Transactional` 테스트를 적극적으로 권장합니다. (Toby) 
> (..) 대신 `@Transactional` 테스트에서 제대로 검증이 되지 않는 문제를 잘 인식하고 작성을 해야 합니다. 

> 테스트를 웬만큼 잘 작성해도 애플리케이션 코드를 완벽하게 검증할 수 없다는 사실을 인색해야 합니다. (Toby)


**📚도서 추천**
- [단위 테스트 (Unit Testing)](https://www.yes24.com/Product/Goods/104084175)
	- 롤백 테스트, 데이터 베이스 클린업에 대한 내용이 나옴
		- 끝나고 전체 롤백
		- 매 테스트 종료시 매번 롤백 (AfterEach)
		- 테스트 시작전 초기화 (BeforeEach)


---
### Reference
- [Spring 트랜잭션은 언제 어떻게 롤백 될까?](https://dkswnkk.tistory.com/760)
- [트랜잭션의 동시성 문제를 알아보자](https://velog.io/@jeong_hun_hui/트랜잭션의-동시성-문제를-알아보자)
- [반려생활 - @Transactional 없애려다 오픈소스 라이브러리까지 만든 이야기](https://blog.ban-life.com/transactional-%EC%97%86%EC%95%A0%EB%A0%A4%EB%8B%A4-%EC%98%A4%ED%94%88%EC%86%8C%EC%8A%A4-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC%EA%B9%8C%EC%A7%80-%EB%A7%8C%EB%93%A0-%EC%9D%B4%EC%95%BC%EA%B8%B0-5426116036bb?gi=2012e27261d9)
- [기억보단 기록을 - 테스트 데이터 초기화에 @Transactional 사용하는 것에 대한 생각(반대파 입장ㅋㅋ)](https://jojoldu.tistory.com/761)






