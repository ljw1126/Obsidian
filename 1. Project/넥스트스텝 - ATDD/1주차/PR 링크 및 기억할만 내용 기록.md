https://github.com/jung0930/atdd-camping-reservation/blob/eb1e5f5b19d6d97cead3270b3834121505a4abf0/src/test/java/com/camping/legacy/acceptance/reservation/ReservationAcceptanceTest.java#L341

https://github.com/heeun98/atdd-camping-reservation/tree/step2

https://github.com/h2jinee/atdd-camping-reservation/blob/step2/src/test/java/com/camping/acceptance/ConcurrencyAcceptanceTest.java

- `API 엔드 포인트는 별도 상수로 관리해도 좋을 것 같아요!`
	- 중복되기 때문에 

`LocalDate.of(..)` 로 예약 하게 될 경우 
- 서비스 로직에서 과거 날짜로 예약 할 수 없다는 예외 터뜨림 
- 에러 코드는 409
	- RuntimeException은 다 그런건가 ?

테스트 데이터가 중복이 된다. 
```java
ReservationRequest request = new ReservationRequest(  
        "홍길동",  
        start,  
        end, // 31박  
        "A-1",  
        "01012345678",  
        4,  
        "12가3456",  
        ""  
);
```


assert도 중복 

인수 테스트에서 메시지를 알아야 하는것이가??
- 예외 메시지 관리하는 방법으로 통일 시킬 수 없을까 ?


```shell
✦ 요청하신 두 가지 케이스에 대한 테스트 가이드를
  ReservationAcceptanceTest.java에 작성해 두었습니다.

  1. 동시성 이슈 테스트 (`createReservation_concurrency`)
   - ExecutorService를 사용하여 여러 스레드에서 동시에 예약 요청을 보내는 구조로
     작성했습니다.
   - CountDownLatch를 통해 스레드들이 동시에 시작하도록 조정했습니다.
   - 예상 결과: 현재 서비스 코드(ReservationService)에는 DB 락(Lock)이나
     synchronized 처리가 되어 있지 않아, 이 테스트는 실패할 가능성이 높습니다.
     (두 요청 모두 성공해버리는 문제 발생 예상)

  2. 가격 정책 검증 테스트 (`createReservation_weekend_price_policy`)
   - LocalDate.now() 문제를 해결하기 위해 TemporalAdjusters를 사용하여 테스트
     실행 시점을 기준으로 "가장 가까운 성수기 주말"을 동적으로 계산하도록
     작성했습니다.
   - 주의사항: 시나리오상 "총 결제 금액이 136,000원이어야 한다"고 되어 있으나,
     현재 ReservationResponse 응답 객체에 가격 정보(totalPrice)가 포함되어 있지
     않습니다.
       - 이에 따라 가격 검증 코드는 주석 처리(TODO)해 두었습니다.
       - 제대로 검증하려면 ReservationResponse DTO에 필드를 추가하고,
         Service에서 이를 반환하도록 수정해야 합니다.
```


현재 가격 정책이 서비스 코드에 하드코딩되어 있다. 그리고 응답에 얼마인지 요금이 나와있지 않다.
```java
@DisplayName("성수기 주말 할증 요금 자동 계산")  
@Test  
void createReservation_weekend_price_policy() {  
    // given  
    // 동적으로 돌아오는 토요일 계산 (테스트 시점에 따라 가변적)    LocalDate nextSaturday = LocalDate.now().with(TemporalAdjusters.next(DayOfWeek.SATURDAY));  
    LocalDate nextSunday = nextSaturday.plusDays(1);  
  
    // 성수기(7,8월)가 아닌 경우, 내년 7월의 토요일로 설정하여 성수기 테스트 가능  
    int currentYear = LocalDate.now().getYear();  
    LocalDate peakDate = LocalDate.of(currentYear, 7, 10);  
    if (peakDate.isBefore(LocalDate.now())) {  
        peakDate = LocalDate.of(currentYear + 1, 7, 10);  
    }  
    LocalDate peakWeekendStart = peakDate.with(TemporalAdjusters.next(DayOfWeek.SATURDAY));  
    LocalDate peakWeekendEnd = peakWeekendStart.plusDays(1);  
  
    ReservationRequest request = new ReservationRequest(  
            "성수기주말고객",  
            peakWeekendStart,  
            peakWeekendEnd,  
            "A-1", // 대형 80,000원  
            "01012345678",  
            4,  
            "12가3456",  
            ""  
    );  
  
    // when & then  
    RestAssured.given().log().all()  
            .contentType(ContentType.JSON)  
            .body(request)  
            .when().post(END_POINT)  
            .then().log().all()  
            .statusCode(HttpStatus.CREATED.value());  
            // .body("totalPrice", equalTo(136000));   
// TODO: 현재 ReservationResponse에 가격 정보(totalPrice)가 포함되어 있지 않아 검증 불가.  
            // DTO 및 Service 수정 필요.  
}
```


예약 취소 후 예약 사이트를 다시 검색한다는 시나리오가 있는데 .. Reservation 인수 테스트에서 사이트 검색 요청을 보내도 되는건가??
```java
@DisplayName("올바른 예약 확인 코드로 취소 시 예약이 취소되어야 한다")  
@Test  
void cancelReservation_success() {  
    // given: 예약 생성  
    LocalDate start = LocalDate.now();  
    LocalDate end = start.plusDays(3);  
  
    ReservationRequest request = new ReservationRequest(  
            "홍길동",  
            start,  
            end,  
            "A-1",  
            "01012345678",  
            4,  
            "12가3456",  
            ""  
    );  
  
    String confirmationCode = RestAssured.given()  
            .contentType(ContentType.JSON)  
            .body(request)  
            .when().post("/api/reservations")  
            .then()  
            .statusCode(HttpStatus.CREATED.value())  
            .extract().path("confirmationCode");  
  
    Long reservationId = RestAssured.given()  
            .contentType(ContentType.JSON)  
            .body(request)  
            .when().get("/api/reservations?customerName=홍길동") // 이름으로 조회해서 ID 찾기 (생성 응답에 ID가 없을 수도 있어서 확인 필요하지만, 보통 응답에 ID 포함됨. 여기선 생성 응답 확인 안했으니 조회로 찾음. 혹은 생성 응답에서 ID 추출 시도)  
            .then()  
            .extract().jsonPath().getLong("[0].id");  
  
    // 생성 응답에 id가 있는지 확인해볼 필요가 있으나, 안전하게 목록 조회로 ID 획득  
  
    // when    RestAssured.given().log().all()  
            .queryParam("confirmationCode", confirmationCode)  
            .when().delete("/api/reservations/" + reservationId)  
            .then().log().all()  
            .statusCode(HttpStatus.OK.value())  
            .body("message", containsString("취소"));  
}
```


시나리오에 있는 날짜 정보가 오히려 혼란을 야기하고 있다. 

예약 취소 후 상태 코드를 확인하기 위해 다시 요청해야 하는건가??
```text
Scenario: 올바른 예약 확인 코드로 취소 시 재고가 즉시 복구됨  
  Given "A-1" 사이트가 해당 예약(ID 100)으로 점유되어 있다  
  When 사용자가 ID 100번과 코드 "SAFE12"로 취소를 요청한다  
  Then 취소가 성공적으로 처리되어야 한다 (Status 200)  
  And 예약 상태가 "CANCELLED"로 변경되어야 한다  
  And 해당 날짜에 "A-1" 사이트를 다시 검색하면 결과에 노출되어야 한다 (재예약 가능)
```
- 최근에 단위/통합 테스트를 작성할때는 해당 도메인, 서비스 등에 대한 기능 검증을 목적으로 하다보니 .. 
	- 제가 검증하려는 도메인 내에 객채와 협력 관계를 검증하는 경우가 많았던거 같은데 .. 

> 인수 테스트를 할때는 도메인의 경계를 허물어야 하는가 보다.. 그렇지 않으면 검증을 할 수 없더라 .. 

아주 훌륭한 고민입니다! 이 지점이 바로 **'단위 테스트(Unit Test)'**와 **'인수 테스트(Acceptance Test)'**의 가장 큰 차이점이 드러나는 부분입니다.

결론부터 말씀드리면, **인수 테스트에서는 경계를 허물고 시나리오 기반으로 사이트 검색을 호출하는 것이 맞습니다.**

그 이유를 세 가지 관점에서 설명해 드릴게요.

---

### 1. 사용자의 "검증 목적"에 집중하세요

사용자(고객) 입장에서 "예약 취소가 성공했다"는 것은 단순히 데이터베이스의 상태값이 `CANCELLED`가 된 것을 의미하지 않습니다. 사용자가 진짜 원하는 결과는 **"내가 취소했으니, 이제 다른 사람(혹은 나)이 그 자리를 다시 예약할 수 있는 상태가 되었는가?"**입니다.

인수 테스트는 **'사용자 스토리'**를 검증하는 것이 목적이므로, 시스템 전체가 유기적으로 돌아가서 최종 비즈니스 가치가 실현되었는지 확인해야 합니다.

### 2. 도메인 경계 vs 비즈니스 흐름

- **단위/통합 테스트**: `ReservationService`가 취소 로직을 잘 수행하는지, `SiteService`의 가용 로직이 맞는지 각자의 경계 내에서 검증합니다.
    
- **인수 테스트**: 도메인 간의 **협력**을 검증합니다. 예약 도메인에서 취소가 일어났을 때, 검색 도메인에 그 영향이 올바르게 전파되었는지를 확인하는 것이 인수 테스트의 핵심 역할입니다.
    

### 3. "Black Box" 테스트의 원칙

인수 테스트는 내부 코드가 어떻게 짜여 있는지 모르는 **'블랙박스 테스트'**여야 합니다. 만약 취소 테스트에서 DB를 직접 뒤져서 `status`만 확인한다면, 그것은 내부 구현에 의존하는 테스트가 됩니다. 반면, **"실제 검색 API를 던져서 확인"**하는 방식은 내부 구현이 바뀌어도(예: 검색 엔진 도입 등) 변하지 않는 가장 견고한 검증 방법입니다


### 💡 테스트 작성 팁: `Step`의 재사용

도메인별로 테스트 클래스를 나누다 보면 중복 코드가 걱정될 수 있습니다. 이때는 **'Step 클래스'**를 활용해 보세요.

- `ReservationSteps`: 예약 생성, 취소 요청 API 호출 로직 관리
    
- `SiteSteps`: 사이트 검색, 가용성 확인 API 호출 로직 관리
    

이렇게 분리해두면 `ReservationAcceptanceTest` 안에서도 `SiteSteps.사이트_검색_요청(...)`을 호출하여 아주 깔끔하게 시나리오를 완성할 수 있습니다.

---

### 🚀 PR에 담을 수 있는 논리

리뷰어에게 본인의 고민과 결론을 이렇게 전달해 보세요.

> "예약 취소 시나리오에서 사이트 재노출 여부를 검증하기 위해 사이트 검색 API를 호출했습니다. 이는 인수 테스트가 **단일 도메인의 기능을 넘어 '사용자가 체감하는 최종 비즈니스 가치'를 검증해야 한다**고 판단했기 때문입니다. 도메인 간의 물리적 경계보다는 **사용자의 시나리오 흐름**을 우선순위에 두고 테스트를 작성했습니다."

---

이렇게 작성하시면 리뷰어님께서 "인수 테스트의 본질을 정확히 꿰뚫고 계시네요!"라는 찬사를 보내실 확률이 높습니다.

**이제 `SiteSteps`와 `ReservationSteps`를 활용해서 이 시나리오를 코드로 옮겨보시겠어요? 아니면 Steps 클래스를 어떻게 구조화하면 좋을지 예시가 필요하신가요?**


---

260117 메모

```text
c + a + p // 매개변수 추출

생성 팩토리 메서드 
- static
- var

 
*통합 테스트를 하게 될 경우 application 을 찾고, resource를 오버라이딩 하는 방식이려나??
 
@Sql
@DirectContext
JPA, JdbcTemplate 


컨트롤러에서 예외 처리 일관성이 없네 => 글로벌 예외 처리 필요
- CONFLICT : 예약 요청 실패시
- BAD_REQUEST : 사이트 관련

예외를 검증할때 메시지가 변할 수 있다. 
- 예외 메시지도 관리하는게 낫지 않나 싶다.

예약 
예약을 요청한다

END_POINT 분리


예약을 요청한다 >> 반복되네 

ReservationRequest >> 빌더 적용하기 (코드 레벨, step3에서)


   .body("$", hasSize(0));

남은 테스트
- 동시성 제어
- 할증 계산
- 가용성 


기간내_중간_날짜가_예약된_사이트는_검색에서_제외된다()

서비스 로직 분석 결과, 현재 가용성 체크가 시작일과 종료일 두 지점(Point)에 대해서만 수행되고 있음을 발견했습니다. 이로 인해 연박 예약 시 중간 날짜에 점유된 예약을 걸러내지 못하는 비즈니스 결함이 발생합니다. Step 3에서는 이를 특정 시점이 아닌 '기간(Period)' 단위의 겹침 로직으로 개선하여 정합성을 확보할 예정입니다
예약 취소 케이스 검증 완료 

LocalDate로 보내면 안되고 날짜를 문자열로 치환해서 보내기 
size = null 입력하면 맵핑시 ""가 들어가서 조회가 안됨 >> 로직 버그 있다


올바른_예약_확인코드로_취소시_재고가_즉시_복구된다
>> 다른 이유에서 실패했던거 같다 
>> 보니깐 당일 예약이면 상태코드가 다른데, CANCELLED 상태가 되니 검색이 된다

성수기 요금 계산을 할때**
- 서비스 내부에 LocalDate.now()가 고정되어 있어서 조작이 어렵다 
- 테스트의 날짜가 지나면 실패하는 테스트가 된다.
- TestConfiguration이라도 사용해서 생성자 주입으로 시간 전략을 고르도록 해야 하나 싶기도한데 .. 인수 테스트가 세부적인 비즈니스 로직에 의존하면 안된다는거 같은데 .. 이걸 구현하는게 아니지 않나 싶다. 

인수 테스트(Step 2): 블랙박스 원칙에 따라 서버 내부를 조작하지 마세요. 대신 테스트 코드에서 서버가 받아들일 수 있는 미래의 성수기 주말 날짜를 동적으로 계산해서 요청을 보내세요.

*step 3단계에서는 단위/통합 테스트 작성하는가?

*동시성 요청에 뭐라뭐라 하는거 ? ..
 



```