# 인수 테스트 작성 가이드  
  
이 문서는 `atdd-camping-tests` 프로젝트의 시스템 레벨 인수 테스트 작성 및 관리를 위한 종합 가이드입니다. `kiosk`, `reservation`, `admin`, `payments` 서비스 간의 복합적인 상호작용을 검증하고, AI를 활용하여 효율적인 테스트 시나리오를 구축하는 데 중점을 둡니다.  
  
## 1. 시스템 개요  
  
`atdd-camping-tests`는 캠핑장 예약 시스템의 통합 테스트를 목표로 합니다. 주요 서비스 간의 상호작용 흐름은 다음과 같습니다.  
  
*   **`kiosk` → `reservation` → `admin`:**  
    사용자는 `kiosk` 서비스를 통해 캠핑장 사이트를 탐색하고 예약을 요청합니다. 이 예약 요청은 `reservation` 서비스로 전달되어 실제 예약이 생성됩니다. 생성된 예약 정보는 `admin` 서비스에서 관리자가 조회하고 상태를 변경할 수 있습니다.  
  
*   **`kiosk` → `payments` (WireMock):**  
    사용자가 `kiosk` 서비스를 통해 예약에 대한 결제를 시도할 때, 결제 요청은 `payments` 서비스로 전달됩니다. 현재 `payments` 서비스는 실제 외부 결제 시스템과의 연동 대신 `WireMock`을 사용하여 모킹(mocking)되어 있습니다. 이를 통해 실제 결제 시스템 없이도 다양한 결제 시나리오를 테스트할 수 있습니다.  
  
**흐름 도식화 (개념적):**  
  
```  
+-------+       +-------------+       +-------+  
| Kiosk | ----> | Reservation | ----> | Admin |  
+-------+       +-------------+       +-------+  
   |   | (Payment Request)   v+----------+  
| Payments | (WireMock)  
+----------+  
```  
  
## 2. Endpoint 요약  
  
각 서비스의 주요 API 엔드포인트는 다음과 같습니다.  
  
### 2-1. Kiosk  
  
| Method | Endpoint | Description | Authentication Required |  
|---|---|---|---|  
| GET | /health | 애플리케이션 상태 확인 | 아니오 |  
| GET | / | 초기 페이지 로드 및 상품 목록 표시 | 아니오 |  
| POST | /api/payments | 결제 생성 | 아니오 |  
| POST | /api/payments/confirm | 결제 승인 | 아니오 |  
| GET | /api/products | 상품 목록 조회 | 아니오 |  
  
### 2-2. Reservation  
  
| Method | Endpoint | (간략)설명 | 인증 필요 여부 |  
|---|---|---|---|  
| POST | `/api/reservations` | 새로운 예약을 생성합니다. | - |  
| GET | `/api/reservations/{id}` | 예약 ID를 사용하여 상세 정보를 조회합니다. | - |  
| GET | `/api/reservations` | 날짜별 또는 고객 이름별 예약을 조회합니다. | - |  
| GET | `/api/reservations/my` | 이름과 전화번호를 사용하여 본인의 예약을 조회합니다. | - |  
| GET | `/api/reservations/calendar` | 특정 사이트의 월별 예약 가능 여부를 조회합니다. | - |  
| PUT | `/api/reservations/{id}` | 확인 코드를 사용하여 예약 정보를 수정합니다. | - |  
| DELETE | `/api/reservations/{id}` | 확인 코드를 사용하여 예약을 취소합니다. | - |  
| GET | `/api/sites` | 모든 캠핑장 사이트의 기본 정보를 조회합니다. | - |  
| GET | `/api/sites/{siteId}` | 특정 사이트의 상세 정보를 조회합니다. | - |  
| GET | `/api/sites/{siteNumber}/availability` | 특정 날짜에 해당 사이트가 예약 가능한지 확인합니다. | - |  
| GET | `/api/sites/available` | 특정 날짜에 예약 가능한 모든 사이트 목록을 조회합니다. | - |  
| GET | `/api/sites/search` | 특정 기간 동안 예약 가능한 사이트를 검색합니다. | - |  
  
### 2-3. Admin  
  
| Method | Endpoint | Description | 인증 필요 여부 (Authentication Required) |  
|---|---|---|---|  
| POST | /auth/login | 로그인 | - |  
| GET | /admin/products | 상품 목록 조회 | 필요 |  
| POST | /admin/products | 상품 등록 | 필요 |  
| GET | /admin/reservations | 예약 목록 조회 | 필요 |  
| PATCH | /admin/reservations/{reservationId}/status | 예약 상태 변경 | 필요 |  
  
### 2-4. Payments (WireMock)  
  
`Payments` 서비스는 `WireMock`을 사용하여 모킹되며, 실제 결제 시스템 없이 다양한 결제 시나리오를 테스트할 수 있습니다.  
  
| Method | Endpoint | Description | Authentication Required |  
|---|---|---|---|  
| POST | `/v1/payments` | 결제 생성 및 콜백 URL 등록 | Yes |  
| POST | `/v1/payments/confirm` | 결제 승인 | Yes |  
| POST | `/v1/payments/{paymentKey}/cancel` | 결제 취소 | Yes |  
  
**WireMock 맵핑 예시 및 확인 주소:**  
  
`payments` 서비스는 `config.properties`에 설정된 `payment.base.url` (기본값: `http://localhost:8083`)을 통해 접근하며, `WireMock` 서버가 이 주소에서 실행됩니다.  
  
**예시 맵핑: `infra/wiremock/mappings/payment-health.json`**  
이 파일은 `/health` 엔드포인트에 대한 간단한 맵핑을 정의합니다.  
  
```json  
{  
  "request": {  
    "method": "GET",  
    "url": "/health"  
  },  "response": {  
    "status": 200  
  }}  
```  
  
*   **WireMock 관리 UI 확인:** WireMock 서버가 실행 중일 때, 일반적으로 `http://localhost:8083/__admin/webapp/` (설정에 따라 포트 변경 가능)에서 WireMock 관리 UI에 접근하여 맵핑 목록, 요청 로그 등을 확인할 수 있습니다.  
*   **맵핑 추가:** `infra/wiremock/mappings/` 디렉토리에 `.json` 파일을 추가하여 새로운 맵핑을 정의할 수 있습니다.  
  
## 3. 사용자 여정 (Happy Path & Sad Path)  
  
인수 테스트는 사용자의 중요한 여정을 `Happy Path` (정상 흐름)와 `Sad Path` (예외/오류 흐름)로 나누어 검증합니다.  
  
### 3.1. Happy Path 예시  
  
*   **정상 예약 성공:** 사용자가 `kiosk`를 통해 캠핑장 사이트를 선택하고, `reservation` 서비스로 예약을 요청합니다. 이 과정에서 `payments` 서비스 (WireMock)를 통한 결제가 성공적으로 이루어집니다. `reservation` 서비스는 예약을 `CONFIRMED` 상태로 처리하고, `admin` 서비스에서 조회 시 `PAID` 상태로 확인되며, 해당 사이트의 재고가 정확히 감소합니다.  
  
### 3.2. Sad Path 예시  
  
*   **결제 실패 보상:** 사용자가 `kiosk`를 통해 예약 후 결제를 시도하지만, `payments` 서비스 (WireMock)에서 4xx/5xx 오류 또는 타임아웃이 발생합니다. 이 경우 `reservation` 서비스는 예약을 미확정 또는 실패 상태로 처리하고, `admin` 서비스에 해당 상태가 반영됩니다. 이전에 예약으로 인해 감소했던 재고는 원래대로 복원됩니다.  
*   **중복/경합 (Idempotency):** 동일한 키(예: 예약 ID)로 결제 또는 예약 요청이 여러 번 시도될 경우, 시스템은 단일 예약만 유지하고 중복 처리를 방지합니다.  
*   **취소/환불:** `reservation` 서비스에서 예약 후 취소를 요청하면, `payments` 서비스 (WireMock)를 통한 환불 처리가 진행되고, `admin` 서비스에 예약 상태가 'CANCELLED' 또는 'REFUNDED'로 반영되며, 재고가 복원됩니다.  
*   **관리자 개입:** `admin` 서비스에서 관리자가 수동으로 예약 상태를 승인 또는 취소할 때, 이 변경 사항이 `reservation` 및 `payments` 서비스에 일관성 있게 전파됩니다.  
  
## 4. 인증 규칙  
  
`Admin` 서비스와 `Payments` 서비스는 **JWT (JSON Web Token) Bearer Token** 방식을 사용하여 인증을 처리합니다.  
  
### 4.1. Admin 서비스 인증 토큰 획득 및 사용  
  
1.  **토큰 획득:**  
    *   `Admin` 서비스의 `POST /auth/login` 엔드포인트에 유효한 사용자 이름과 비밀번호로 로그인 요청을 보냅니다.  
    *   성공적인 응답으로 `Authorization` 헤더 또는 응답 본문에서 JWT 토큰을 획득합니다. (예: `{"token": "eyJ..."}`)  
2.  **토큰 사용:**  
    *   획득한 JWT 토큰은 이후 `Admin` 서비스의 보호된 엔드포인트(예: `/admin/products`, `/admin/reservations`)에 요청을 보낼 때 HTTP `Authorization` 헤더에 `Bearer` 접두사와 함께 포함되어야 합니다.  
    *   **헤더 예시:** `Authorization: Bearer <획득된_JWT_토큰_값>`  
  
### 4.2. Payments 서비스 인증 토큰 획득 및 사용  
  
`Payments` 서비스 또한 JWT Bearer Token을 요구합니다. 현재 `WireMock`으로 모킹되어 있으므로, 실제 토큰 획득 메커니즘은 `Admin` 서비스와 유사하거나 별도의 인증 서비스로부터 발급받는 방식일 수 있습니다. 테스트 시나리오에서는 유효한 JWT 토큰을 시뮬레이션하거나 특정 테스트용 토큰을 사용하여 요청을 보낼 수 있습니다.  
  
### 4.3. 테스트 클라이언트 고려사항  
  
현재 `AdminClient.java` 및 `PaymentClient.java`는 인증 토큰을 자동으로 관리하거나 요청에 추가하는 로직을 직접적으로 포함하고 있지 않습니다. 효율적인 테스트를 위해 `com.camping.tests.clients.ApiClient`에 `setAuthToken(String token)`과 같은 메서드를 추가하거나, 각 클라이언트 생성 시 토큰을 주입받아 내부 `RequestSpecification`에 `Authorization` 헤더를 설정하는 방식으로 개선할 것을 권장합니다.  
  
## 5. 시드 데이터 규칙  
  
테스트의 신뢰성과 재현성을 보장하기 위해 시드 데이터 규칙을 준수해야 합니다.  
  
*   **데이터 격리:** 각 테스트 시나리오는 독립적으로 실행되어야 하며, 다른 시나리오의 데이터에 영향을 주거나 받지 않아야 합니다.  
*   **고유 네임스페이스:** 테스트 실행 시마다 무작위 문자열이나 타임스탬프를 포함한 고유한 사용자 ID, 예약 ID, 상품명 등을 생성하여 데이터 충돌을 방지합니다.  
*   **재실행 안전성:** 테스트가 여러 번 실행되어도 항상 동일한 결과를 보장하도록, 테스트 실행 전후로 데이터를 초기화하거나 정리하는 메커니즘(예: `@Before` 또는 `@After` 훅 사용)을 마련해야 합니다.  
  
## 6. 시나리오 예시  
  
인수 테스트 시나리오 작성 시 고려해야 할 좋은 예시와 나쁜 예시입니다.  
  
### 6-1. 좋은 시나리오 예시  
  
```gherkin  
Feature: 예약 및 결제 성공 시나리오  
  
  Scenario: 사용자는 캠핑장 사이트를 예약하고 성공적으로 결제할 수 있다    Given 사용자는 유효한 사용자 정보를 가지고 있다    And 캠핑장 사이트 "사이트A"의 재고가 5개 있다    When 사용자가 "사이트A"를 2박 3일 동안 예약하고 결제를 진행한다    Then 예약이 성공적으로 생성되고, 예약 상태가 'CONFIRMED'이다    And "사이트A"의 재고가 1 감소하여 4개이다    And 관리자 서비스에서 해당 예약의 결제 상태가 'PAID'로 조회된다    And Payments WireMock에 결제 승인 요청이 성공적으로 기록된다```  
**설명:** 이 시나리오는 특정 비즈니스 흐름(예약 및 결제)을 검증하며, `When` 절의 행동이 `Then` 절에서 여러 서비스에 걸친 비즈니스 기대를 명확하게 검증합니다. 또한 `admin` 서비스와 `payments` (WireMock) 서비스의 상태를 관찰하여 통합적인 검증을 수행합니다.  
  
### 6-2. 나쁜 시나리오 예시  
  
```gherkin  
Feature: 잘못된 시나리오 예시  
  
  Scenario: 불필요한 HTTP 상태 코드만 검증하는 시나리오    When 사용자가 예약을 생성하기 위해 POST /api/reservations에 요청을 보낸다    Then 응답 코드가 200 OK이다```  
**문제점:**  
*   **의도-검증 불일치:** 응답 코드 200 OK는 단지 기술적인 성공을 의미하며, 예약이 성공적으로 처리되었는지, 비즈니스 규칙이 제대로 적용되었는지(예: 재고 감소, 예약 상태 변경)에 대한 충분한 비즈니스 검증이 아닙니다.  
*   **환경 의존성:** `/api/reservations`와 같은 하드코딩된 URL 사용은 테스트 환경이 변경될 때마다 시나리오를 수정해야 하는 문제를 발생시킵니다. 베이스 URL은 외부 설정을 통해 관리되어야 합니다.  
  
## 7. 품질 기준 체크 리스트  
  
AI가 생성한 시나리오를 승인하기 전에 다음 체크리스트를 사용하여 검토해야 합니다. 이 체크리스트를 통과한 시나리오만 `@ai-candidate` 태그를 제거하고 적절한 다른 태그를 부여합니다.  
  
*   [ ] **의도-검증 일치:** `When` 절의 행동이 `Then` 절에서 비즈니스 기대를 정확히 검증하는가?  
*   [ ] **환경 독립성:** 호스트, 포트, 인증 토큰 등이 시나리오에 하드코딩되어 있지 않은가? 베이스 URL, 시크릿 등은 외부 설정으로 관리되는가?  
*   [ ] **데이터 격리:** 시나리오가 독립적으로 실행되며, 재실행 시에도 동일한 결과를 보장하는가? 테스트 데이터가 고유하게 생성되고 정리되는가?  
*   [ ] **관찰 가능성:** 테스트 결과가 예약 상태, 관리자 조회, WireMock 호출 기록 등 시스템의 여러 지점에서 명확하게 관찰 및 검증되는가?  
*   [ ] **가독성 및 이해도:** 시나리오가 비즈니스 언어로 명확하고 간결하게 작성되어 모든 이해관계자가 쉽게 이해할 수 있는가?  
*   [ ] **완전성:** 정상, 경계, 예외 시나리오를 적절히 포괄하고 있는가?  
  
## 8. 프로젝트 폴더 구조  
  
`atdd-camping-tests` 프로젝트의 주요 폴더 구조는 다음과 같습니다.  
  
```  
/Users/leejinwoo1126/Home/gitRepository/atdd-camping-tests/  
├───.git/  
├───.gradle/  
├───.idea/  
├───build/  
├───docs/  
│   ├───acceptance-test-guide.md (현재 문서)  
│   ├───auth-setup.md (인증 및 실행 상세 가이드, 통합됨)  
│   └───api/  
│       ├───admin-spec.md  
│       ├───kiosk-spec.md  
│       ├───payments-spec.md  
│       └───reservation-spec.md  
├───gradle/  
├───infra/  
│   ├───docker-compose-infra.yml  
│   ├───docker-compose.yml  
│   ├───db/  
│   │   └───init.sql  
│   ├───dockerfiles/  
│   └───wiremock/  
│       └───mappings/  
│           └───payment-health.json (WireMock 맵핑 파일)  
├───out/  
├───repos/ (각 서비스의 코드베이스)  
│   ├───atdd-camping-admin/  
│   ├───atdd-camping-kiosk/  
│   ├───atdd-camping-payments/  
│   └───atdd-camping-reservation/  
└───src/  
    └───test/        ├───java/        │   └───com/        │       └───camping/        │           └───tests/        │               ├───RunCucumberTest.java (테스트 러너)        │               ├───clients/ (API 클라이언트)        │               ├───common/        │               ├───config/ (테스트 설정, 예: TestConfig.java)        │               ├───context/        │               ├───dto/        │               └───steps/ (Cucumber Step Definitions)        └───resources/            ├───config.properties (환경 설정 파일)            └───features/ (Gherkin Feature 파일)                ├───e2e/                └───smoke/```  
  
## 9. 터미널 실행 명령어  
  
`atdd-camping-tests` 프로젝트는 Gradle을 사용하여 테스트를 실행합니다.  
  
### 9.1. 기본 테스트 실행  
  
```bash  
# 모든 인수 테스트 실행 (특별한 태그 필터링 없이)  
./gradlew :atdd-tests:test```  
  
### 9.2. 태그를 이용한 선택적 테스트 실행  
  
*   **`@ai-candidate`:** AI가 새로 생성한 시나리오에 기본적으로 부여되는 태그입니다. 이 태그가 붙은 시나리오는 아직 검토 및 승인 대기 중임을 의미합니다.  
    *   **실행:** `./gradlew :atdd-tests:test -Dcucumber.filter.tags="@ai-candidate"`  
*   **`@e2e`:** 검토를 통과하고 시스템 레벨 인수 테스트로 승인된 시나리오에 부여되는 태그입니다.  
    *   **실행:** `./gradlew :atdd-tests:test -Dcucumber.filter.tags="@e2e"`  
*   **`@smoke`:** 가장 핵심적인 기능에 대한 빠른 검증을 위한 시나리오에 부여합니다.  
*   **`@payment-failure`:** 결제 실패와 관련된 시나리오에 부여합니다.  
  
**다중 태그 조건:**  
  
```bash  
# 여러 태그를 조합하여 테스트 실행 (AND 조건)  
./gradlew :atdd-tests:test -Dcucumber.filter.tags="@e2e and @smoke"  
  
# 여러 태그를 조합하여 테스트 실행 (OR 조건)  
./gradlew :atdd-tests:test -Dcucumber.filter.tags="@e2e or @smoke"  
  
# 특정 태그를 제외하고 테스트 실행  
./gradlew :atdd-tests:test -Dcucumber.filter.tags="not @deprecated"  
```  
  
### 9.3. 로그 디버깅 방법  
  
(이 섹션은 추가 정보가 제공될 예정입니다. 일반적으로 테스트 실패 시 Gradle 빌드 로그 또는 각 서비스의 애플리케이션 로그를 통해 문제를 진단합니다. 서비스별 로그 파일 위치나 디버깅 설정에 대한 자세한 정보가 필요합니다.)  
  
## 10. WireMock 설정  
  
`Payments` 서비스의 모킹을 위해 `WireMock`이 사용됩니다.  
  
### 10.1. `config.properties` 설정  
  
`src/test/resources/config.properties` 파일에서 `payment.base.url`을 `WireMock` 서버의 주소로 설정합니다.  
  
```properties  
payment.base.url=http://localhost:8083 # WireMock 서버 주소  
```  
  
### 10.2. 맵핑 파일 관리  
  
`WireMock`의 응답 맵핑은 `infra/wiremock/mappings/` 디렉토리에 `.json` 파일 형태로 저장됩니다. 각 `.json` 파일은 특정 요청(URL, HTTP 메서드 등)에 대한 응답을 정의합니다.  
  
**예시: `payment-health.json`**  
  
```json  
{  
  "request": {  
    "method": "GET",  
    "url": "/health"  
  },  "response": {  
    "status": 200  
  }}  
```  
  
### 10.3. Docker Compose를 통한 WireMock 실행  
  
`WireMock` 서버는 `infra/docker-compose-infra.yml` 파일을 통해 다른 인프라 서비스와 함께 실행됩니다.  
  
**WireMock 포함 서비스 실행:**  
  
```bash  
docker-compose -f infra/docker-compose-infra.yml up -d```  
  
이 명령어를 통해 `WireMock` 서버가 백그라운드에서 실행되며, `payment.base.url`에 설정된 주소로 들어오는 요청에 대해 정의된 맵핑에 따라 응답합니다.  
  
### 10.4. WireMock 관리 UI  
  
실행 중인 `WireMock` 서버의 관리 UI는 웹 브라우저를 통해 접근할 수 있습니다.  
*   **주소:** `http://localhost:8083/__admin/webapp/` (포트는 `config.properties`의 `payment.base.url`에 따라 변경될 수 있습니다.)  
관리 UI에서는 현재 로드된 맵핑 목록을 확인하고, `WireMock`이 수신한 요청의 로그를 분석하여 테스트 중인 서비스가 `WireMock`과 올바르게 상호작용하는지 검증할 수 있습니다. 이는 특히 결제 실패, 타임아웃 등 `Sad Path` 시나리오를 테스트할 때 유용합니다
```


---

### 일부 수정

# 인수 테스트 작성 가이드

- 이 문서는 `atdd-camping-tests` 프로젝트의 시스템 레벨 인수 테스트 작성 및 관리를 위한 종합 가이드입니다. 
- `kiosk`, `reservation`, `admin`, `payments` 서비스 간의 복합적인 상호작용을 검증하고, AI를 활용하여 효율적인 테스트 시나리오를 구축하는 데 중점을 둡니다.

## 1. 시스템 개요

`atdd-camping-tests`는 캠핑장 예약/관리 시스템의 인수 테스트를 목표로 합니다. 주요 서비스 간의 상호작용 흐름은 다음과 같습니다.

```
+-------+       +-------------+       +-------+
| Kiosk | ----> | Reservation | ----> | Admin |
+-------+       +-------------+       +-------+
   |
   | (Payment Request)
   v
+----------+
| Payments | (WireMock)
+----------+
```

## 2. Endpoint 요약

각 서비스의 주요 API 엔드포인트는 다음과 같습니다.

### 2-1. Kiosk

| Method | Endpoint             | Description         | Authentication Required |
|---|----------------------|---------------------|-------------------------|
| GET | `/`                    | 초기 페이지 로드 및 상품 목록 표시 | ❌                     |
| GET | `/health`              | 헬스 체크               | ❌                       |
| POST | `/api/payments`        | 결제 생성               | ❌                     |
| POST | `/api/payments/confirm` | 결제 승인               | ❌                     |
| GET | `/api/products`        | 상품 목록 조회            | ❌                     |

### 2-2. Reservation

| Method | Endpoint | Description | Authentication Required |
|---|---|---------------------|-------------------------|
| POST | `/api/reservations` | 신규 예약 생성 | ❌ |
| GET | `/api/reservations/{id}` | 예약 상세 정보 조회 | ❌ |
| GET | `/api/reservations` | 예약 조회 (날짜별 또는 고객 이름별) | ❌ |
| GET | `/api/reservations/my` | 본인 예약 조회 | ❌ |
| GET | `/api/reservations/calendar` | 특정 사이트의 월별 예약 가능 여부 조회 | ❌ |
| PUT | `/api/reservations/{id}` | 예약 정보 수정 (예약 확인 코드 사용) | ❌ |
| DELETE | `/api/reservations/{id}` | 예약 취소 (예약 확인 코드 사용) | ❌ |
| GET | `/api/sites` | 캠핑장 사이트 목록 조회 | ❌ |
| GET | `/api/sites/{siteId}` | 사이트 상세 정보 조회 | ❌ |
| GET | `/api/sites/{siteNumber}/availability` | 특정 날짜에 해당 사이트가 예약 가능 여부 확인 | ❌ |
| GET | `/api/sites/available` | 특정 날짜에 예약 가능한 사이트 목록 조회 | ❌ |
| GET | `/api/sites/search` | 특정 기간 동안 예약 가능한 사이트 검색 | ❌ |

### 2-3. Admin

| Method | Endpoint | Description | Authentication Required |
|---|---|---------------------|-------------------------|
| POST | /auth/login | 로그인 | ❌ |
| GET | /admin/products | 상품 목록 조회 | ✅ |
| POST | /admin/products | 상품 등록 | ✅ |
| GET | /admin/reservations | 예약 목록 조회 | ✅ |
| PATCH | /admin/reservations/{reservationId}/status | 예약 상태 변경 | ✅ |

### 2-4. Payments (WireMock)

`Payments` 서비스는 `WireMock`을 사용하여 모킹되며, 실제 결제 시스템 없이 다양한 결제 시나리오를 테스트할 수 있습니다.

| Method | Endpoint | Description | Authentication Required |
|---|---|---------------------|-------------------------|
| POST | `/v1/payments` | 결제 생성 및 콜백 URL 등록 | Yes |
| POST | `/v1/payments/confirm` | 결제 승인 | Yes |
| POST | `/v1/payments/{paymentKey}/cancel` | 결제 취소 | Yes |

**WireMock 맵핑 예시 및 확인 주소:**

`payments` 서비스는 `config.properties`에 설정된 `payment.base.url` (기본값: `http://localhost:8083`)을 통해 접근하며, `WireMock` 서버가 이 주소에서 실행됩니다.

**예시 맵핑: `infra/wiremock/mappings/payment-health.json`**
이 파일은 `/health` 엔드포인트에 대한 간단한 맵핑을 정의합니다.

```json
{
  "request": {
    "method": "GET",
    "url": "/health"
  },
  "response": {
    "status": 200
  }
}
```

*   **WireMock 관리 UI 확인:** WireMock 서버가 실행 중일 때, 일반적으로 `http://localhost:8083/__admin/webapp/` (설정에 따라 포트 변경 가능)에서 WireMock 관리 UI에 접근하여 맵핑 목록, 요청 로그 등을 확인할 수 있습니다.
*   **맵핑 추가:** `infra/wiremock/mappings/` 디렉토리에 `.json` 파일을 추가하여 새로운 맵핑을 정의할 수 있습니다.

## 3. 사용자 여정 (Happy Path & Sad Path)

인수 테스트는 사용자의 중요한 여정을 `Happy Path` (정상 흐름)와 `Sad Path` (예외/오류 흐름)로 나누어 검증합니다.

### 3.1. Happy Path 예시

*   **정상 예약 성공:** 사용자가 `kiosk`를 통해 캠핑장 사이트를 선택하고, `reservation` 서비스로 예약을 요청합니다. 이 과정에서 `payments` 서비스 (WireMock)를 통한 결제가 성공적으로 이루어집니다. `reservation` 서비스는 예약을 `CONFIRMED` 상태로 처리하고, `admin` 서비스에서 조회 시 `PAID` 상태로 확인되며, 해당 사이트의 재고가 정확히 감소합니다.

### 3.2. Sad Path 예시

*   **결제 실패 보상:** 사용자가 `kiosk`를 통해 예약 후 결제를 시도하지만, `payments` 서비스 (WireMock)에서 4xx/5xx 오류 또는 타임아웃이 발생합니다. 이 경우 `reservation` 서비스는 예약을 미확정 또는 실패 상태로 처리하고, `admin` 서비스에 해당 상태가 반영됩니다. 이전에 예약으로 인해 감소했던 재고는 원래대로 복원됩니다.
*   **중복/경합 (Idempotency):** 동일한 키(예: 예약 ID)로 결제 또는 예약 요청이 여러 번 시도될 경우, 시스템은 단일 예약만 유지하고 중복 처리를 방지합니다.
*   **취소/환불:** `reservation` 서비스에서 예약 후 취소를 요청하면, `payments` 서비스 (WireMock)를 통한 환불 처리가 진행되고, `admin` 서비스에 예약 상태가 'CANCELLED' 또는 'REFUNDED'로 반영되며, 재고가 복원됩니다.
*   **관리자 개입:** `admin` 서비스에서 관리자가 수동으로 예약 상태를 승인 또는 취소할 때, 이 변경 사항이 `reservation` 및 `payments` 서비스에 일관성 있게 전파됩니다.

## 4. 인증 규칙

`Admin` 서비스와 `Payments` 서비스는 **JWT (JSON Web Token) Bearer Token** 방식을 사용하여 인증을 처리합니다.

### 4.1. Admin 서비스 인증 토큰 획득 및 사용

1.  **토큰 획득:**
    *   `Admin` 서비스의 `POST /auth/login` 엔드포인트에 유효한 사용자 이름과 비밀번호로 로그인 요청을 보냅니다.
    *   성공적인 응답으로 `Authorization` 헤더 또는 응답 본문에서 JWT 토큰을 획득합니다. (예: `{"token": "eyJ..."}`)
2.  **토큰 사용:**
    *   획득한 JWT 토큰은 이후 `Admin` 서비스의 보호된 엔드포인트(예: `/admin/products`, `/admin/reservations`)에 요청을 보낼 때 HTTP `Authorization` 헤더에 `Bearer` 접두사와 함께 포함되어야 합니다.
    *   **헤더 예시:** `Authorization: Bearer <획득된_JWT_토큰_값>`

### 4.2. Payments 서비스 인증 토큰 획득 및 사용

`Payments` 서비스 또한 JWT Bearer Token을 요구합니다. 현재 `WireMock`으로 모킹되어 있으므로, 실제 토큰 획득 메커니즘은 `Admin` 서비스와 유사하거나 별도의 인증 서비스로부터 발급받는 방식일 수 있습니다. 테스트 시나리오에서는 유효한 JWT 토큰을 시뮬레이션하거나 특정 테스트용 토큰을 사용하여 요청을 보낼 수 있습니다.

### 4.3. 테스트 클라이언트 고려사항

현재 `AdminClient.java` 및 `PaymentClient.java`는 인증 토큰을 자동으로 관리하거나 요청에 추가하는 로직을 직접적으로 포함하고 있지 않습니다. 효율적인 테스트를 위해 `com.camping.tests.clients.ApiClient`에 `setAuthToken(String token)`과 같은 메서드를 추가하거나, 각 클라이언트 생성 시 토큰을 주입받아 내부 `RequestSpecification`에 `Authorization` 헤더를 설정하는 방식으로 개선할 것을 권장합니다.

## 5. 시드 데이터 규칙

테스트의 신뢰성과 재현성을 보장하기 위해 시드 데이터 규칙을 준수해야 합니다.

*   **데이터 격리:** 각 테스트 시나리오는 독립적으로 실행되어야 하며, 다른 시나리오의 데이터에 영향을 주거나 받지 않아야 합니다.
*   **고유 네임스페이스:** 테스트 실행 시마다 무작위 문자열이나 타임스탬프를 포함한 고유한 사용자 ID, 예약 ID, 상품명 등을 생성하여 데이터 충돌을 방지합니다.
*   **재실행 안전성:** 테스트가 여러 번 실행되어도 항상 동일한 결과를 보장하도록, 테스트 실행 전후로 데이터를 초기화하거나 정리하는 메커니즘(예: `@Before` 또는 `@After` 훅 사용)을 마련해야 합니다.

## 6. 시나리오 예시

인수 테스트 시나리오 작성 시 고려해야 할 좋은 예시와 나쁜 예시입니다.

### 6-1. 좋은 시나리오 예시

```gherkin
Feature: 예약 및 결제 성공 시나리오

  Scenario: 사용자는 캠핑장 사이트를 예약하고 성공적으로 결제할 수 있다
    Given 사용자는 유효한 사용자 정보를 가지고 있다
    And 캠핑장 사이트 "사이트A"의 재고가 5개 있다
    When 사용자가 "사이트A"를 2박 3일 동안 예약하고 결제를 진행한다
    Then 예약이 성공적으로 생성되고, 예약 상태가 'CONFIRMED'이다
    And "사이트A"의 재고가 1 감소하여 4개이다
    And 관리자 서비스에서 해당 예약의 결제 상태가 'PAID'로 조회된다
    And Payments WireMock에 결제 승인 요청이 성공적으로 기록된다
```
**설명:** 이 시나리오는 특정 비즈니스 흐름(예약 및 결제)을 검증하며, `When` 절의 행동이 `Then` 절에서 여러 서비스에 걸친 비즈니스 기대를 명확하게 검증합니다. 또한 `admin` 서비스와 `payments` (WireMock) 서비스의 상태를 관찰하여 통합적인 검증을 수행합니다.

### 6-2. 나쁜 시나리오 예시

```gherkin
Feature: 잘못된 시나리오 예시

  Scenario: 불필요한 HTTP 상태 코드만 검증하는 시나리오
    When 사용자가 예약을 생성하기 위해 POST /api/reservations에 요청을 보낸다
    Then 응답 코드가 200 OK이다
```
**문제점:**
*   **의도-검증 불일치:** 응답 코드 200 OK는 단지 기술적인 성공을 의미하며, 예약이 성공적으로 처리되었는지, 비즈니스 규칙이 제대로 적용되었는지(예: 재고 감소, 예약 상태 변경)에 대한 충분한 비즈니스 검증이 아닙니다.
*   **환경 의존성:** `/api/reservations`와 같은 하드코딩된 URL 사용은 테스트 환경이 변경될 때마다 시나리오를 수정해야 하는 문제를 발생시킵니다. 베이스 URL은 외부 설정을 통해 관리되어야 합니다.

## 7. 품질 기준 체크 리스트

AI가 생성한 시나리오를 승인하기 전에 다음 체크리스트를 사용하여 검토해야 합니다. 이 체크리스트를 통과한 시나리오만 `@ai-candidate` 태그를 제거하고 적절한 다른 태그를 부여합니다.

*   [ ] **의도-검증 일치:** `When` 절의 행동이 `Then` 절에서 비즈니스 기대를 정확히 검증하는가?
*   [ ] **환경 독립성:** 호스트, 포트, 인증 토큰 등이 시나리오에 하드코딩되어 있지 않은가? 베이스 URL, 시크릿 등은 외부 설정으로 관리되는가?
*   [ ] **데이터 격리:** 시나리오가 독립적으로 실행되며, 재실행 시에도 동일한 결과를 보장하는가? 테스트 데이터가 고유하게 생성되고 정리되는가?
*   [ ] **관찰 가능성:** 테스트 결과가 예약 상태, 관리자 조회, WireMock 호출 기록 등 시스템의 여러 지점에서 명확하게 관찰 및 검증되는가?
*   [ ] **가독성 및 이해도:** 시나리오가 비즈니스 언어로 명확하고 간결하게 작성되어 모든 이해관계자가 쉽게 이해할 수 있는가?
*   [ ] **완전성:** 정상, 경계, 예외 시나리오를 적절히 포괄하고 있는가?

## 8. 프로젝트 폴더 구조

`atdd-camping-tests` 프로젝트의 주요 폴더 구조는 다음과 같습니다.

```
atdd-camping-tests/
├───docs/
│   ├───acceptance-test-guide.md (현재 문서)
│   ├───auth-setup.md (인증 및 실행 상세 가이드, 통합됨)
├───infra/
│   ├───docker-compose-infra.yml
│   ├───docker-compose.yml
│   ├───db/
│   │   └───init.sql
│   ├───dockerfiles/
│   └───wiremock/
│       └───mappings/
│           └───payment-health.json (WireMock 맵핑 파일)
├───out/
├───repos/ (각 서비스의 코드베이스)
│   ├───atdd-camping-admin/
│   ├───atdd-camping-kiosk/
│   ├───atdd-camping-payments/
│   └───atdd-camping-reservation/
└───src/
    └───test/
        ├───java/
        │   └───com/
        │       └───camping/
        │           └───tests/
        │               ├───RunCucumberTest.java (테스트 러너)
        │               ├───clients/ (API 클라이언트)
        │               ├───common/
        │               ├───config/ (테스트 설정, 예: TestConfig.java)
        │               ├───context/
        │               ├───dto/
        │               └───steps/ (Cucumber Step Definitions)
        └───resources/
            ├───config.properties (환경 설정 파일)
            └───features/ (Gherkin Feature 파일)
                ├───e2e/
                └───smoke/
```

## 9. 터미널 실행 명령어

`atdd-camping-tests` 프로젝트는 Gradle을 사용하여 테스트를 실행합니다.

### 9.1. 기본 테스트 실행

```bash
# 모든 인수 테스트 실행 (특별한 태그 필터링 없이)
./gradlew :atdd-tests:test
```

### 9.2. 태그를 이용한 선택적 테스트 실행

*   **`@ai-candidate`:** AI가 새로 생성한 시나리오에 기본적으로 부여되는 태그입니다. 이 태그가 붙은 시나리오는 아직 검토 및 승인 대기 중임을 의미합니다.
    *   **실행:** `./gradlew :atdd-tests:test -Dcucumber.filter.tags="@ai-candidate"`
*   **`@e2e`:** 검토를 통과하고 시스템 레벨 인수 테스트로 승인된 시나리오에 부여되는 태그입니다.
    *   **실행:** `./gradlew :atdd-tests:test -Dcucumber.filter.tags="@e2e"`
*   **`@smoke`:** 가장 핵심적인 기능에 대한 빠른 검증을 위한 시나리오에 부여합니다.
*   **`@payment-failure`:** 결제 실패와 관련된 시나리오에 부여합니다.

**다중 태그 조건:**

```bash
# 여러 태그를 조합하여 테스트 실행 (AND 조건)
./gradlew :atdd-tests:test -Dcucumber.filter.tags="@e2e and @smoke"

# 여러 태그를 조합하여 테스트 실행 (OR 조건)
./gradlew :atdd-tests:test -Dcucumber.filter.tags="@e2e or @smoke"

# 특정 태그를 제외하고 테스트 실행
./gradlew :atdd-tests:test -Dcucumber.filter.tags="not @deprecated"
```

### 9.3. 로그 디버깅 방법

(이 섹션은 추가 정보가 제공될 예정입니다. 일반적으로 테스트 실패 시 Gradle 빌드 로그 또는 각 서비스의 애플리케이션 로그를 통해 문제를 진단합니다. 서비스별 로그 파일 위치나 디버깅 설정에 대한 자세한 정보가 필요합니다.)

## 10. WireMock 설정

`Payments` 서비스의 모킹을 위해 `WireMock`이 사용됩니다.

### 10.1. `config.properties` 설정

`src/test/resources/config.properties` 파일에서 `payment.base.url`을 `WireMock` 서버의 주소로 설정합니다.

```properties
payment.base.url=http://localhost:8083 # WireMock 서버 주소
```

### 10.2. 맵핑 파일 관리

`WireMock`의 응답 맵핑은 `infra/wiremock/mappings/` 디렉토리에 `.json` 파일 형태로 저장됩니다. 각 `.json` 파일은 특정 요청(URL, HTTP 메서드 등)에 대한 응답을 정의합니다.

**예시: `payment-health.json`**

```json
{
  "request": {
    "method": "GET",
    "url": "/health"
  },
  "response": {
    "status": 200
  }
}
```

### 10.3. Docker Compose를 통한 WireMock 실행

`WireMock` 서버는 `infra/docker-compose-infra.yml` 파일을 통해 다른 인프라 서비스와 함께 실행됩니다.

**WireMock 포함 서비스 실행:**

```bash
docker-compose -f infra/docker-compose-infra.yml up -d
```

이 명령어를 통해 `WireMock` 서버가 백그라운드에서 실행되며, `payment.base.url`에 설정된 주소로 들어오는 요청에 대해 정의된 맵핑에 따라 응답합니다.

### 10.4. WireMock 관리 UI

실행 중인 `WireMock` 서버의 관리 UI는 웹 브라우저를 통해 접근할 수 있습니다.
*   **주소:** `http://localhost:8083/__admin/webapp/` (포트는 `config.properties`의 `payment.base.url`에 따라 변경될 수 있습니다.)
관리 UI에서는 현재 로드된 맵핑 목록을 확인하고, `WireMock`이 수신한 요청의 로그를 분석하여 테스트 중인 서비스가 `WireMock`과 올바르게 상호작용하는지 검증할 수 있습니다. 이는 특히 결제 실패, 타임아웃 등 `Sad Path` 시나리오를 테스트할 때 유용합니다.