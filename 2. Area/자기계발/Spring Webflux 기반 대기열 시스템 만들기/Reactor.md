
- `emitt` : 방출하다


**publisher**
1. `Flux`
	1. 0~N개의 아이템을 가질 수 있는 데이터 스트림
	2. onNext(0~N), onComplete, onError
2. `Mono`
	1. 0개 또는 1개의 아이템을 가지는 데이터 스트림 
	2. onNext(0 ~ 1), onComplete, onError

```java

public class Publisher {

	public Flux<Integer> startFlow() {
		return Flux.range(1, 10).log(); // 이터러블이나 spread opration으로 처리가능
	}
	
	public Flux<Integer> startFlow2() {
		return Flux.fromIterable(List.of("a", "b", "c")).log();
	}

	public Mono<Integer> startMono() {
		return Mono.just(1).log();
	}

	public Mono<Integer> startEmptyMono() {
		return Mono.empty().log();
	}

	public Mono<?> startErrorMono() {
		return Mono.error(new RuntimeException("hello reactor")).log();
	}
	
}
```

**testing(StepVerifier)**
- [reference](https://projectreactor.io/docs/core/release/reference/testing.html#testing-a-scenario-with-stepverifier)

의존성 추가 (`build.gradle`)
```text
dependencies { testCompile 'io.projectreactor:reactor-test' }
```

```java
class PublisherTest {
	private Publisher publisher = new Publisher();

	@Test
	void startFlow() {
		StepVerifier.create(publisher.startFlux())
			.expectNext(1,2,3,4,5,6,7,8,9,10)
			.verifyComplete();
	}

	@Test
	void startFlow2() {
		StepVerifier.create(publisher.startFlow2())
			.expectNext("a", "b", "c")
			.verifyComplete();
	}
	
	@Test
	void startMono() {
		StepVerifier.create(publisher.startMono())
			.expectNext(1)
			.verifyComplete();
	}

    @Test
	void startEmptyMono() {
		StepVerifier.create(publisher.startEmptyMono())
			.verifyComplete();
	}
	
	@Test
	void startErrorMono() {
		StepVerifier.create(publisher.startErrorMono())
			.expectError(RuntimeException.class)
			.verify();
	}
}

```


**Operator 메서드**
- 스트림 데이터를 처리 
	- `map`
	- `filter`
	- `take`
	- `flatMap`

```java
public class OperatorExample {

	public Flux<Integer> fluxMap() {
		return Flux.range(1, 5)
					.map(i -> i * 2)
					.log();
	}
	
	public Flux<Integer> fluxFilter() {
		return Flux.range(1, 10)
					.filter(i -> i > 5)
					.log();
	}

	public Flux<Integer> fluxTake() {
		return Flux.range(1, 10)
					.filter(i -> i > 5)
					.take(3) // limit 같은 느낌
					.log();
	}
	
	public Flux<Integer> fluxFlatMap() {
		return Flux.range(1, 10)
					.flatMap(i -> Flux.range(i * 10, 10))
					.log();
	}

	public Flux<Integer> fluxFlatMap2() {
		return Flux.range(1, 9)
					.flatMap(i -> Flux.range(1, 9)
						.map(j -> {
							System.out.println("%d * %d = %d", i, j, i * j)
							return i * j;
						})
					)
					.log();
	}
	
}
```


```java
class OperatorExampleTest {
	private OperatorExample operator = new OperatorExample();

	@Test
	void fluxMap() {
		StepVerifier.create(operator.fluxMap())
			.expectNext(2, 4, 6, 8, 10)
			.verifyComplete();
	}
	
	@Test
	void fluxFilter() {
		StepVerifier.create(operator.fluxFilter())
			.expectNext(6, 7, 8, 9, 10)
			.verifyComplete();
	}
	
	@Test
	void fluxTake() {
		StepVerifier.create(operator.fluxTake())
			.expectNext(6, 7, 8)
			.verifyComplete();
	}
	
	@Test
	void fluxFlatMap() {
		StepVerifier.create(operator.fluxFlatMap())
			.expectNextCount(100) // 카운팅
			.verifyComplete();
	}


	@Test
	void fluxFlatMap2() {
		StepVerifier.create(operator.fluxFlatMap2())
			.expectNextCount(81) // 카운팅
			.verifyComplete();
	}
}
```
- flatMap은 **비동기**이고 순서를 보장하지 않는다
	- 1:N 관계로 mapping 처리 된다
	- 동기로 본다면 예제 코드가 2중 for문으로 생각해도 무방


**메서드2**
> javadoc을 보니 이미지가 있다
- `concatMap` 
	- 다음 요소 처리할 때까지 대기 (동기식), 순서가 중요한 작업에서 사용
	- concat이기 때문에 map에 대한 내용을 추가해주는 듯
	- 비동기 처리도 가능하다는 듯(?, 필요시 찾아보니)
- `flatMapMany`
	- mono to flux
- ✨ `switchIfEmpty` / `defaultIfEmpty` : 전달받은 값이 없을 때 사용
- `merge` / `zip` : Flux 여러 개를 하나로 합칠 때

```java
public class OperatorExample2 {
	public Flux<Integer> fluxConcatMap() {
		return Flux.range(1, 10)
					.concatMap(i -> Flux.range(i * 10, 10).delayElements(Duration.ofMillis(100)))
					.log();
	}

	public Flux<Integer> monoFlatMapMany() {
		return Mono.just(10)
				.flatMapMany(i -> Flux.range(1, i))
				.log();
	}

	public Mono<Integer> defaultIfEmpty() {
		return Mono.just(100)
				.filter(i -> i > 100)  // 여기서 끝나면 verifyComplete()
				.defaultIfEmpty(30);
	}

	public Mono<Integer> switchIfEmpty() {
		return Mono.just(100)
				.filter(i -> i > 100) 
				.switchIfEmpty(Mono.just(30).map(i -> i * 2)); // 예외를 반환할 수도 있다
	}

	public Mono<Integer> switchIfEmptyThrowException() {
		return Mono.just(100)
				.filter(i -> i > 100) 
				.switchIfEmpty(Mono.error(new RuntimeException("Not exist value"))).log(); 
	}

	public Flux<String> fluxMerge() {
		return Flux.merge(Flux.fromIterable("a", "b", "c"), Flux.just("d")).log();
	}

	public Flux<String> monoMerge() {
		return Mono.just("1").mergeWith(Mono.just("2")).mergeWith(Mono.just("3"));
	}

	public Flux<String> fluxZip() {
		return Flux.zip(Flux.just("a", "b", "c"), Flux.just("d", "e", "f"))
		.map(i --> i.getT1() + i.getT2())
		.log();
	}

	public Mono<Integer> monoZip() {
		return Mono.zip(Mono.just(1), Mono.just(2), Mono.just(3))
					.map(i-> i.getT1() + i.getT2() + i.getT3())
					.log();
	}
}
```


```java
class OperatorExample2Test {
	private OperatorExample2 operator = new OperatorExample2();

	@Test
	void fluxConcatMap() {
		StepVerifier.create(operator.fluxConcatMap())
			.expectNextCount(100)
			.verifyComplete();
	}

	@Test
	void monoFlatMapMany() {
		StepVerifier.create(operator.monoFlatMapMany())
			.expectNextCount(10)
			.verifyComplete();
	}

	@Test
	void defaultIfEmpty() {
		StepVerifier.create(operator.defaultIfEmpty())
			.expectNext(30)
			.verifyComplete();
	}

	@Test
	void switchIfEmpty() {
		StepVerifier.create(operator.switchIfEmpty())
			.expectNext(60)
			.verifyComplete();
	}

	@Test
	void switchIfEmptyThrowException() {
		StepVerifier.create(operator.switchIfEmptyThrowException())
			.expectError()
			.verify();
	}

	@Test
	void fluxMerge() {
		StepVerifier.create(operator.fluxMerge())
			.expectNext("a", "b", "c", "d")
			.verifyComplete();
	}

	@Test
	void monoMerge() {
		StepVerifier.create(operator.monoMerge())
			.expectNext("1", "2", "3")
			.verifyComplete();
	}

	@Test
	void fluxZip() {
		StepVerifier.create(operator.fluxZip())
			.expectNext("ad", "be", "cf")
			.verifyComplete();
	}

	@Test
	void monoZip() {
		StepVerifier.create(operator.monoZip())
			.expectNext(6)
			.verifyComplete();
	}
}
```


**집계 메서드**
- `count`
- `distinct`
- `reduce`
- `groupby`

```java
public class OperatorExample3 {
	public Mono<Long> fluxCount() {
		return Flux.range(1, 10).count().log();
	}

	public Flux<String> fluxDistinct() {
		return Flux.fromIterable(List.of("a", "b", "c", "a", "b")).distinct().log();
	}

	public Mono<Integer> fluxReduce() {
		return Flux.range(1, 10)
					.reduce((i, j) -> i + j)
					.log();
	}

	public Flux<Integer> fluxGroupBy() {
		return Flux.range(1, 10)
					.grouopBy(i -> i % 2 == 0 ? "even" : "odd")
					.flatMap(group -> group.reduce((i, j) -> i + j))
					.log();
	}
}
```



```java
class OperatorExample3Test {
	private OperatorExample3 operator = new OperatorExample3();

	@Test
	void fluxCount() {
		StepVerifier.create(operator.fluxCount())
			.expectNext(10L)
			.verifyComplete();
	}

	@Test
	void fluxDistinct() {
		StepVerifier.create(operator.fluxDistinct())
			.expectNext("a", "b", "c")
			.verifyComplete();
	}

	@Test
	void fluxReduce() {
		StepVerifier.create(operator.fluxReduce())
			.expectNext(55)
			.verifyComplete();
	}

	@Test
	void fluxGroupBy() {
		StepVerifier.create(operator.fluxGroupBy())
			.expectNext(30)
			.expectNext(25)
			.verifyComplete();
	}
}
```


**백 프레셔 관련 메서드**
- `delaySequence` / `limitRate`
- `sample` : 샘플링

```java
public class OperatorExample4 {
	public Flux<Integer> fluxDelayAndLimit() {
		return Flux.range(1, 10)
					.delaySequence(Duration.ofSeconds(1))
					.log()
					.limitRate(2); // 2개씩 퍼블리셔에서 전달
	}

	public Flux<Integer> fluxSample() {
		return Flux.range(1, 100)
				.delaySequence(Duration.ofSecords(100))
				.sample(Duration.ofMillis(300))
				.log();
	}
}
```


```java
class OperatorExample4Test {
	private OperatorExample4 operator = new OperatorExample4();

	@Test
	void fluxDelayAndLimit() {
		StepVerifier.create(operator.fluxDelayAndLimit())
			.expectNext(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
			.verifyComplete();
	}

	@Test
	void fluxSample() {
		StepVerifier.create(operator.fluxSample())
			.expectNextCount(1000) // ?
			.verifyComplete();
	}
}
```


### Schedulers
- 스레드 형식 지정
	- `immediate()`
	- `single()`
	- `parallel()`
	- `boundedElastic()`

```java
public class Scheduler1 {
	public Flux<Integer> fluxMapWithSubscribeOn() {
		return Flux.range(1, 10)
					.map(i -> i * 2)
					.subscribeOn(Schedulers.boundedElastic())
					.log();
	}
	
	public Flux<Integer> fluxMapWithPublishOn() {
		return Flux.range(1, 10)
					.map(i -> i + 1)
					.publishOn(Schedulers.boundedElastic()) // boundedElastic 스레드 사용
					.log()
					.publishOn(Schedulers.parallel()) // parallel 스레드 사용
					.log()
					.map(i -> i * 2);
	}

}
```

```java
class Scheduler1Test {
	private Scheduler1 scheduler = new Scheduler1();
	
	@Test
	void fluxDelayAndLimit() {
		StepVerifier.create(scheduler.fluxMapWithSubscribeOn())
			.expectNext(10)
			.verifyComplete();
	}

	@Test
	void fluxMapWithPublishOn() {
		StepVerifier.create(scheduler.fluxMapWithPublishOn())
			.expectNext(10)
			.verifyComplete();
	}
}
```

