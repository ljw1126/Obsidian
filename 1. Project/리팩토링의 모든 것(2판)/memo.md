
리팩토링 2판 web edtion 
https://martinfowler.com/articles/access-refactoring-web-edition.html


>[!note] 생각 해보기
>- 책에서 어디가 좋았는가 ? 
>- 의견이 다른 부분이 있는가 ?
>- 질문하고 싶은 부분이 있는가 ?
>- 그 외 코드 퀴즈

### Chapter 1
- 레거시를 리랙터링 하는 과정과 사이클이 개인적으로 좋았다
- "수정 - 컴파일 - 테스트 - 커밋" 사이클이 익숙하지 않았는데, 테스트 준비를 하고 시작하다보니 빠르게 대응할 수 있었던거 같아 좋았다

1. 함수 추출하기
2. 클래스 책임 분리하기
3. 조건부 로직 다형성 적용 하기


스트랭 글러 패턴
https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/modernization-decomposing-monoliths/strangler-fig.html
https://johngrib.github.io/wiki/pattern/strangler/

getter를 사용하는 부분
- 디미터의 법칙(method chain)
- 포워딩 메서드
- .. 객체인지 자료구조인지 (클린코드 p124)
	- 값인 경우 getter 사용, 객체인 경우 getter 지양

일급컬렉션 사용, 컬렉션 조회시 얕은 복사와 깊은 복사
- StatementData 는 데이터만을 가져야 하기 때문에 비즈니스 로직을 포함하는 일급 컬렉션을 가지는건 아닌거 같음.. `{java}List<EnrichPerfomance>`
- toby : dto 에서는 일급 컬렉션을 사용하지 않는다. 비즈니스 계산 로직이 있으니 .. 단순히 데이터 전달하는 용도라서 굳이 한번 더 감쌀 이유는 없는거 같다



