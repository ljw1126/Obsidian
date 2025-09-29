

**SHIP PARTICULAR**
- **Vessel Particulars** 라고도 함
- **선박 제원 정보**를 의미함 🚢
- 이 정보는 선박의 물리적, 설계, 기능적 특성을 정의하는 모든 정략적이고 확정적인 사항들을 포함합니다. 즉, 선박의 명세서 라고 할 수 있습니다.
- SHIP PARTICULAR에는 일반적으로 다음과 같은 중요한 정보들이 포함됩니다 
	- 1. 선박의 크기 및 용적 관련 정보
	- 2. 톤수 
	- 3. 엔진 및 속도
	- 4. 선박 종류 및 용도
	- 5. 선적 능력(Capacity)
	- 6. 등록 및 식별 정보
- 📌사용 목적 
	- SHIP PARTICULAR는 해운, 물류, 보험, 조선 등 다양한 해사 산업 분야에서 **선박의 성능, 능력, 특성을 파악**하고 **각종 계약, 운항 계획, 세금 및 수수료 산정** 등의 목적으로 매우 중요하게 사용됩니다.

---

> SHIP PARTICULAR 모듈에 포함되는 일부 테이블에 대한 CRUD 구현

**테이블 목록**
- 1. SHIP_INFO
- 2. SHIP_SERVICE
- 3. REPLACE_SHIP_NAME
- 4. SHIP_SATELLITE
- 5. SHIP_MODEL_TEST
- 6. SK_TELINK_COMPANY_SHIP

> 🧐 플로우 차트 기준으로는 한꺼번에 처리되는 느낌인데 .. 

---

**프로젝트 생성** 
- 프로젝트명 : `ShipParticularsApi`
- 닷넷 `v8.0` 

**패키지 설치**
- `[도구 > NuGet 패키지 관리자 > 솔루션용 NuGet 패키지 관리...]` 실행
- 닷넷 코어 버전이 v8.x 라서, 동일한 버전대로 설치.

```text
Microsoft.EntityFrameworkCore.SqlServer  // 8.0.20
Microsoft.EntityFrameworkCore.Tools
Microsoft.EntityFrameworkCore.Design
Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore  //추가 설치