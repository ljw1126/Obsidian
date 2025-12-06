# 📌 오늘 배운 것 (Today I Learned) 
## 날짜 
`2025년 12월 06일`

## 카테고리
`#CSharp`, `#dotnet dev kor`

## 주제: 
### 1. 문제 상황 또는 학습 배경
- 커뮤니티 통해 닷넷 데브 kor 세미나 알게 됨
- 현재 닷넷 8을 사용하고 있는데 10에서는 어떠한 변화가 있는지 관심가지게 되어 참여

### 2. 핵심 내용 / 개념 정리
- `251111` 닷넷 10.0 발표
- C#으로 쉘 스크립트 만들고 실행하기 가능
	- `dotent cli` 활용
	- dotnet sdk 설치 및 파일 실행 권한만 있으면 바로 쉘 스크립트를 만들 수 있게 됨
- `C# dotnetdev-kr-custome` 익스텐션
	- 깃허브에서 공식 레파지토리를 포그해서 포워딩
- `Eclipse Theia` 코드 에디터 소개
	- VS Code 기반으로 만들어짐
	- 가볍고, mac에서도 호환 가능
		- https://theia-ide.org/
		- https://www.youtube.com/watch?v=7vgWZkYtq64
	- 테스트 실행 속도 미확인
- dotnet cli가 설치되어 있으면 curl로 C# 스크립트를 내려받아 파이프라인으로 전달하여 커맨드 라인 통해 바로 실행 가능함을 확인
- 파이썬의 `Fast API`가 있다면, 닷넷 진영에는 `미니멀 API`가 있다
- 닷넷 `aspire 13`, `Podman` 소개
	- aspire는 개발에 특화된 도구이다. (운영 사용 x)
		- 오케스트레이션, 배포, 관측까지의 운영 작업 지원해주는 프레임워크
	- Podman은 도커 데스크탑의 대체제이다 
		- 무료 
		- 레드헷 라이센스 필요 x
		- 단, 윈도우 네이티브 컨테이너는 커버 못함
		- 자체 대시보드, 노드 그래프, 로그 필터링 등 지원 (메트릭 수집, 트레이싱 가능)
- `Garnet`
	- Redis 대체하기 위해 MS에서 만듦 
		- 라이센스가 문제되어서.
	- 닷넷 기반 메모리 캐시 서비스 (오픈 소스)
- 암호 관리 관련
	- 닷넷에 `유저 시크릿`이라는 도구가 있다✅
	- 특정 카테고리의 키와 값을 넣어두면, 프로그램에서 가져와 사용 가능하다
	- https://learn.microsoft.com/ko-kr/aspnet/core/security/app-secrets?view=aspnetcore-10.0&tabs=windows
	- 이를 통해 저장소에 운영환경 변수를 올리는 일을 방지할 수 있다 함
- 그외 `Avalonia`, `AG-UI(Agent User Interface)` 간략 소개


**🔗 링크** 
- [닷넷 데브 코리아](https://forum.dotnetdev.kr/)
- [세미나 영상](https://forum.dotnetdev.kr/t/net-universe-busan-edition-2512/14114)
- 세미나 자료 
	- `깃허브/dotnetdev-kr/dotnet-fba-emplate`

