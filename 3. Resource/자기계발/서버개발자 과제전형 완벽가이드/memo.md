
**시스템 디자인** 
- 규모 추정
- 시스템 성능/비용/확장성 고려 가능

**주요 지표** 
- `MAU`
- `DAU`
- `QPS`: 초당 처리 되는 쿼리 수 (시스템의 처리 능력, 읽기 작업)
- `TPS`: 초당 처리 되는 트랜잭션 수(데이터 베이스의 처리 능력, 쓰기 작업)
- `Peak Users` : 특정 시간대 동시 접속하는 최대 유저 수
- `Data Storage` : 시스템이 처리할 데이터 총량
- `Bandwidth` : 시스템이 초당 사용하는 네트워크 대역폭 
- `Latency` : 지연시간, 요청 처리 평균 시간
- `Throughput` : 단위시간당 처리되는 처리량

**MAU 1천만 가정할 때**
- DAU : 10% 가정 => 100만명의 활성 사용자 예상
- QPS, TPS 
	- QPD = DAU * (세션 당 쿼리수) = 100만 * 세션당 2번 호출 = 200만
	- QPS, TPS가 1:1 비율이라면 100만씩 됨 (실제는 읽기와 쓰기가 8:2 비율이 보통)
	- 하루를 초로 계산하면 86,400(`60*60*24`)초 
	- 초당 QPS, TPS = 100만 / 86,400초 = 11.57로 추정 가능
	- 피크 시간에 얼마나 몰리는지도 예상해야 한다
		- 보통 1.2배, 많으면 2배 이상 예상 가능
- Volume
	- 사용자 검색 요청시 쌓이는 데이터의 크기 예상치
		- ID(8byte), 검색어(50byte), 검색 시간(8byte), meta(64byte) 추정 = 대략 130byte
		- 100만(TPS) * 130byte = 130,000,000 (byte) = 약 130Mb가 하루에 쌓인다


>[!info] 한글, 영문 바이트 수
>- UTF-8 기준 : 영문, 숫자 1byte, 한글 3byte
>- UTF-16 기준 : 영문, 숫자, 한글 전부 2byte
>- EUC-KR 기준 : 영문, 숫자 1byte, 한글 2byte


### 프로젝트 생성
- 프로젝트 명 : libaray-search
- gradle
- Java 17
- Spring web만 우선 의존성 추가


### 멀티 모듈
- [https://docs.gradle.org/current/userguide/java_library_plugin.html](https://docs.gradle.org/current/userguide/java_library_plugin.html)
- [https://docs.gradle.org/current/userguide/intro_multi_project_builds.html](https://docs.gradle.org/current/userguide/intro_multi_project_builds.html)

**Gradle Plugin**
의존성 키워드 관련해서
- `api` : 다른 모듈도 의존성 접근 가능
	- 재사용성 높아짐 
	- 컴파일패스가 커져 빌드 성능 저하 가능
	- 모듈간의 결합도가 높아짐
- `implementation` : 다른 모듈은 의존성에 접근 못함
	- 재사용성은 떨어짐
	- 컴파일 패스가 작아져 빌드 성능 향상
	- 모듈의 캡슐화가 잘됨

**멀티 모듈 구조**
```text
my-multi-module-project/
- build.gradle
- settings.gradle
- module1
	- build.gradle
- module2
	- build.gradle
```


```text
// settings.gradle
rootProject.name = 'my-multi-module-project'
include 'module1', 'module2'
```


```text
// module1/build.gradle
plugins {
	id 'java-library'
}

dependencies {
	// ..
}


// module2/build.gradle
plugins {
	id 'java-library'
}

dependencies {
	implementation project(':module1') // 모듈2는 모듈1을 의존함
	// ..
}

```


**멀티 모듈 생성시 아래와 같이 gradle 구조가 나타남**
- library-search
	- common
	- external
	- search-api 

## 섹션4. 프로젝트 구현

### 외부 API 연동 - OpenAPI & RestClient
- 네이버 개발자 센터 : https://developers.naver.com/main/
- 카카오 개발자 센터 : 