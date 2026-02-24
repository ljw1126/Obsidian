# 인증 및 실행 가이드 문서  
  
## 1. 개요 및 목적  
  
이 문서는 `atdd-camping-tests` 프로젝트의 인수 테스트를 실행하기 위한 인증 설정 및 전반적인 실행 환경 구성에 대한 가이드라인을 제공합니다. 특히 `Admin` 서비스와 `Payments` 서비스의 인증 메커니즘을 이해하고, 테스트에 필요한 자격 증명을 관리하는 방법을 설명합니다.  
  
## 2. 인증 메커니즘 및 설정  
  
현재 시스템은 `Admin` 서비스와 `Payments` 서비스에서 인증을 요구합니다. `Kiosk` 및 `Reservation` 서비스의 공개된 엔드포인트는 일반적으로 인증이 필요 없습니다.  
  
### 2.1. Admin 서비스 인증  
  
`Admin` 서비스에 대한 인증은 `POST /auth/login` 엔드포인트를 통해 이루어집니다. 이 요청의 성공적인 응답으로 인증 토큰(예: JWT)을 획득할 수 있습니다. 획득된 토큰은 이후 `Admin` 서비스의 보호된 엔드포인트에 요청을 보낼 때 `Authorization` 헤더에 `Bearer` 토큰 형태로 포함되어야 합니다.  
  
**개념적인 로그인 및 토큰 사용 흐름:**  
  
1.  **로그인 요청:**  
    *   `POST /auth/login` 엔드포인트에 유효한 사용자 이름과 비밀번호로 요청을 보냅니다.  
    *   예상 응답: `{ "token": "획득된_인증_토큰_값" }` (또는 유사한 구조)  
  
2.  **토큰 저장:**  
    *   응답에서 `token` 값을 추출하여 변수에 저장합니다.  
  
3.  **보호된 리소스 접근:**  
    *   이후 `Admin` 서비스의 `GET /admin/products`와 같은 보호된 엔드포인트에 요청을 보낼 때, `Authorization: Bearer 획득된_인증_토큰_값` 헤더를 포함해야 합니다.  
  
**현재 테스트 클라이언트 (`AdminClient.java`) 고려사항:**  
  
현재 `AdminClient.java`는 인증 토큰을 자동으로 관리하거나 요청에 추가하는 로직을 직접적으로 포함하고 있지 않습니다. 따라서 `Admin` 서비스의 보호된 엔드포인트를 테스트하기 위해서는 다음 중 한 가지 방법으로 클라이언트를 확장해야 합니다.  
  
*   `ApiClient`에 `setAuthToken(String token)`과 같은 메서드를 추가하여 `RequestSpecification`에 `Authorization` 헤더를 설정하도록 합니다.  
*   각 Step Definition에서 토큰을 획득하고 `AdminClient` 호출 시 헤더를 명시적으로 추가하도록 합니다. (이는 코드 중복을 야기할 수 있으므로 권장하지 않습니다.)  
*   `AdminClient` 생성 시 토큰을 주입받아 내부 `ApiClient`에 설정하도록 합니다.  
  
```java  
// 예시: AdminClient에 토큰 설정 로직 추가 (개념적 코드)  
public class AdminClient {  
    private final ApiClient api;    private String authToken; // 인증 토큰을 저장할 필드  
    public AdminClient() {        this.api = new ApiClient(TestConfig.getAdminBaseUrl());    }  
    public void setAuthToken(String token) {        this.authToken = token;        // ApiClient의 RequestSpecification에 Authorization 헤더를 설정하는 메서드가 필요        // 또는 ApiClient의 RequestSpecification을 직접 수정할 수 있는 방법이 필요        // 예: api.addHeader("Authorization", "Bearer " + token);    }  
    public ExtractableResponse<Response> getFromAdmin(String path) {        // 토큰이 있을 경우, 요청에 헤더 추가        if (authToken != null) {            // 이 부분은 ApiClient 내부에서 처리되거나,            // RequestSpecification을 업데이트하는 방식으로 구현될 수 있습니다.            // 예: api.getWithAuth(path, authToken);        }        return api.get(path);    }  
    // 로그인 메소드 추가 고려    public ExtractableResponse<Response> login(String username, String password) {        // ... /auth/login 엔드포인트 호출 로직 ...        // 성공 시 응답에서 토큰 추출 후 setAuthToken 호출        return null;    }}  
```  
  
### 2.2. Payments 서비스 인증  
  
`Payments` 서비스(WireMock으로 모킹됨) 또한 `POST /v1/payments`, `POST /v1/payments/confirm`, `POST /v1/payments/{paymentKey}/cancel`과 같은 엔드포인트에 대해 인증을 요구합니다. `payments-spec.md`에 따르면 `Authentication Required`로 명시되어 있습니다.  
  
`Admin` 서비스와 유사하게, `Payments` 서비스에 접근하기 위한 인증 토큰을 획득하고 이를 요청 헤더에 포함하는 과정이 필요합니다. 토큰 획득 메커니즘은 별도의 인증 서버를 통하거나 `Payments` 서비스 자체에서 제공하는 방식일 수 있습니다.  
  
**현재 테스트 클라이언트 (`PaymentClient.java`) 고려사항:**  
  
`PaymentClient.java` 역시 현재 인증 로직을 직접적으로 포함하고 있지 않습니다. `Payments` 서비스와의 연동 테스트를 위해서는 `AdminClient`와 유사하게 인증 토큰을 관리하고 요청에 추가할 수 있도록 클라이언트를 확장해야 합니다.  
  
## 3. 테스트 실행 환경 구성  
  
### 3.1. 기본 URL 설정 (`config.properties`)  
  
각 서비스의 기본 URL은 `src/test/resources/config.properties` 파일에 정의되어 있습니다. 테스트 환경에 따라 이 파일의 값을 적절히 수정해야 합니다.  
  
```properties  
# 예시: config.properties  
kiosk.base.url=http://localhost:8080  
admin.base.url=http://localhost:8081  
reservation.base.url=http://localhost:8082  
payment.base.url=http://localhost:8083 # WireMock URL  
```  
  
### 3.2. Docker Compose를 이용한 서비스 실행  
  
프로젝트는 `infra/docker-compose.yml` 및 `infra/docker-compose-infra.yml` 파일을 사용하여 테스트 환경을 구성할 수 있습니다. 이 파일들을 통해 `kiosk`, `admin`, `reservation`, `payment (WireMock)` 서비스를 쉽게 배포하고 실행할 수 있습니다.  
  
**서비스 실행:**  
  
```bash  
# 인프라 서비스(DB, WireMock 등) 실행  
docker-compose -f infra/docker-compose-infra.yml up -d  
# 애플리케이션 서비스(kiosk, admin, reservation) 실행  
docker-compose -f infra/docker-compose.yml up -d```  
  
**서비스 중지:**  
  
```bash  
# 모든 서비스 중지  
docker-compose -f infra/docker-compose.yml downdocker-compose -f infra/docker-compose-infra.yml down```  
  
### 3.3. 테스트를 위한 사전 준비  
  
1.  **Docker 설치:** Docker와 Docker Compose가 시스템에 설치되어 있어야 합니다.  
2.  **JDK 설치:** 테스트 실행을 위해 Java Development Kit (JDK) 17 이상이 설치되어 있어야 합니다.  
3.  **Gradle 설치:** Gradle 빌드 도구가 설치되어 있어야 합니다. ( `./gradlew` 스크립트를 사용한다면 명시적인 Gradle 설치는 불필요할 수 있습니다.)  
4.  **서비스 실행:** 위 `Docker Compose를 이용한 서비스 실행` 섹션의 지침에 따라 모든 필수 서비스가 실행 중인지 확인합니다. 특히 `WireMock`이 `payment.base.url`에 해당하는 포트에서 실행 중이어야 합니다.  
  
## 4. Client 구현 개선 제안  
  
효율적인 인증 토큰 관리를 위해 `com.camping.tests.clients.ApiClient` 또는 각 클라이언트(`AdminClient`, `PaymentClient`)에 다음 기능을 추가하는 것을 제안합니다.  
  
*   **인증 토큰 설정 메서드:** `ApiClient`에 `setAuthToken(String token)`과 같이 `Authorization` 헤더를 `RequestSpecification`에 추가하는 메서드를 구현합니다.  
*   **로그인 헬퍼 메서드:** `AdminClient`에 `login(String username, String password)`와 같은 헬퍼 메서드를 추가하여 로그인 과정을 캡슐화하고 획득한 토큰을 내부적으로 관리하도록 합니다.  
*   **토큰 자동 갱신/만료 처리:** (선택 사항) 토큰 만료를 감지하고 자동으로 갱신하는 로직을 추가하여 테스트의 견고성을 높일 수 있습니다.  
  
이러한 개선을 통해 테스트 시나리오에서 인증 관련 로직의 중복을 줄이고 가독성을 향상시킬 수 있습니다.