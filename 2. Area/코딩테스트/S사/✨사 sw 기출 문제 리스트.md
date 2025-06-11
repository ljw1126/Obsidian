- [백준 기출문제 모음](https://www.acmicpc.net/workbook/view/1152)
- 반례 찾기(250610) : `https://testcase.ac/problems/{문제번호}`


👋 : 풀이 경험 없음 
💩 : 직접 풀이 실패
1. [퇴사(실3)](https://www.acmicpc.net/problem/14501)
2. [스타트와 링크(실1)](https://www.acmicpc.net/problem/14889)
3. [시험 감독(브2)](https://www.acmicpc.net/problem/13458)
4. [연산자 끼워넣기(실1)](https://www.acmicpc.net/problem/13458)
5. 👋[로봇 청소기(골5)](https://www.acmicpc.net/problem/14503)💩
	1. 문제 지문 정리가 포인트
	2. BFS로 충분히 풀이 가능한 기본 구현 문제
6. 👋[톱니바퀴(골5)](https://www.acmicpc.net/problem/14891)🙌
	1. 구현/시뮬레이션 문제
	2. 반복문만으로 풀이
7. [치킨배달(골5)](https://www.acmicpc.net/problem/15686)
8. [인구이동(골4)](https://www.acmicpc.net/problem/16234)
9. 👋[컨베이어 벨트 위의 로봇(골5)](https://www.acmicpc.net/problem/20055)
10. 👋[상어 초등학교(골5)](https://www.acmicpc.net/problem/21608) 🙌
	1. 구현 문제 (2시간 소요)
	2. for 반복문 만으로 풀이 가능
	3. 시간 복잡도: 
		1. N이 최대 20일때 400명 학생이 자리 선정하는데 걸리는 시간을 1 ~ 400까지의 합과 같다 
	4. 탐색 조건에서 변수명 실수, 우선순위에 따른 자리 선정 조건문에서 실수
11. 👋[마법사 상어와 비바라기(골5)](https://www.acmicpc.net/problem/21610) 💩
	1. 풀이 실패
	2. 반복문으로 풀이 가능한 구현, 시뮬레이션 문제 (보통 s사는 백트래킹, 구현 위주)
	3. 방문 배열을 매턴 마다 초기화 해주는 거 실수 
		1. 한번 구름이 생긴 위치에 다음 구름이 생길 수 없다는 거지 영영 생기지 않는건 아님
	4. 좌표 변환 실수
		1. (0,0) ~ (n - 1, n - 1) 기준으로 했을 때 모듈러(나머지) 연산으로 양의 좌표 구할 수 있다 ✨
			1. `{java} int dx = (c[0] + dir[d][0] * s + N * s) % N; ` 
			2. `{java}int dy = (c[1] + dir[d][1] * s + N * s) % N;`
		2. **Non-negative Modulus** 또는 **Positive Modulus** 라고 부른다
		3. 현재 (1, 1) 위치에 있는 구름이 왼쪽 방향으로 8칸(`s`) 수평 이동한다면
			1. N = 5이면 s = 8과 곱해서 40이 나오고 앞에 피연사자 더하면 -7이 나온다
			2. `33 % 5 = 3` 으로 (3, 3)으로 이동하는 것을 구할 수 있다
		4. `N * s`를 더하는 게 포인트📌
12.  [뱀(골4)](https://www.acmicpc.net/problem/3190)
13. 👋[주사위 굴리기(골4)](https://www.acmicpc.net/problem/14499)🙌
	1. 직접 풀이 (30분)
	2. 구현, 시뮬레이션 문제 
	3. for 반복문으로 풀이가능 
	4. `Dice(주사위)` 클래스 안에서 방향에 따라 이동한 값을 복사 시켜줌
		1. temp 변수 할당하면 코드량 늘어나기때문에 `int[] result = new int[7]` 생성해서 할당
		2. `동 : 1, 서 : 2, 북 : 3, 남 : 4`
			1. `동: 1 -> 3, 3 -> 6, 6 -> 4, 4 -> 1` (2,5 그대로)
			2. `서: 1 -> 4, 4 -> 6, 6 -> 3, 3 -> 1` (2,5 그대로)
			3. `북: 1 -> 2, 2 -> 6, 6 -> 5, 5 -> 1` (3,4 그대로)
			4. `남: 2 -> 1, 1 -> 5, 5 -> 6, 6 -> 2` (3,4 그대로)
14. 👋[테트로미노(골4)](https://www.acmicpc.net/problem/14500)💩
	1. 답은 구해지나 시간 초과 발생 
	2. 나의 풀이
		1. block 5개를 구한 후 매번 회전, 대칭 호출하도록 처리함 
		2. 이로인해 `시간초과`
	3. 풀이 방법1 (완전 탐색)
		1.  블록 19개에 대한 좌표 정보를 3차원 배열에 저장
		2.  이때 원점 기준 좌표는 제외하고, 나머지 3개 면에 대한 좌표만 기록
		3. 시간복잡도는 `500 * 500 * 19 * 4` 
	4. 풀이 방법2 (dfs, 백트래킹)
		1. 4개의 면을 이어준다는 점에서 재귀 + 백트래킹으로 풀이 가능 
		2. 이때 예외 케이스가 `ㅗ`가 있음 
			1. 깊이 우선 탐색의 경우 `ㅗ`와 같이 이전 위치로 돌아와 탐색이 기본적으로 안됨
			2. 그래서 `depth = 2` 일때 다음 위치를 방문 처리, 합산하고 한번더 같은 위치로 재귀호출하는 방법으로 풀이
	5. 풀이 방법3 (가지치기 추가)
		1. 볼 필요 없는거 early return
		2. `최대값(정답)`을 static 변수로 올린다
		3. 필드에서 최대값, 그리고 현재 depth와 점수 정보를 바탕으로 계산 
			1. `현재 점수 + (4 - depth) * 필드 최대값 <= ans`
			2. 현재 점수에 남은 점수를 더하더라도, 최대값 이하이면 return 
15. [연구소(골4)](https://www.acmicpc.net/problem/14502)
16. [감시(골3)](https://www.acmicpc.net/problem/15683)
17. 👋[드래곤 커브(골3)](https://www.acmicpc.net/problem/15685)
18. 👋[사다리 조작(골3)](https://www.acmicpc.net/problem/15684)
19. 👋[미세먼지 안녕(골4)](https://www.acmicpc.net/problem/17144)
20. 👋[이차원 배열과 연산(골4)](https://www.acmicpc.net/problem/17140)
21. 👋[마법사 상어와 파이어볼(골4)](https://www.acmicpc.net/problem/20056)
22. 👋[나무 재테크(골3)](https://www.acmicpc.net/problem/16235)
23. [아기상어(골3)](https://www.acmicpc.net/problem/16236)
24. 👋[연구소3(골3)](https://www.acmicpc.net/problem/17142)
25. 👋[게리맨더링2(골3)](https://www.acmicpc.net/problem/17779)
26. 👋[마법사 상어와 토네이도(골3)](https://www.acmicpc.net/problem/20057)💩
	1. 달팽이 탬플릿 문제 ➡️ 시작점에서 종료점까지 회전하는 것 부터 실패 ..
	2. 방향을 기준으로 비율도 90도씩 회전하게 된다. 
		1. 매번 계산하기 보다는 방향별로 계산해두는게 편함
		2. 인덱스에 대한 퍼센트를 저장하는 배열도 필요
		3. `a` 좌표에는 나머지를 할당하면 되고
	3. `a` 좌표에 할당하는 모래를 계산할 때 
		1. 실수. 주위에 45%를 날려버리니, 55% 모래가 남을 거라고 착각 
			1. 예로 모래가 5남았을떄 6%는 0 (정수만 취급)이기 때문에 위와같이 계산하면 안됨 
		2. 이동시 날라간 모래의 값을 합산해서 뺴줘야 원하는 결과값이 나온다
27. 👋[마법사 상어와 파이어스톰(골3)](https://www.acmicpc.net/problem/20058)
28. 👋[주사위 굴리기 2(골3)](https://www.acmicpc.net/problem/23288)
29. 👋[경사로(골3)](https://www.acmicpc.net/problem/14890)
30. 👋[2048 Easy (골1)](https://www.acmicpc.net/problem/12100)
31. 👋[새로운 게임2 (골2)](https://www.acmicpc.net/problem/17837)
32. 👋[원판 돌리기(골2)](https://www.acmicpc.net/problem/17822)
33. 👋[주사위 윷놀이(골2)](https://www.acmicpc.net/problem/17825)
34. 👋[모노미노도미노 2(골2)](https://www.acmicpc.net/problem/20061)
35. [청소년 상어(골1)](https://www.acmicpc.net/problem/19236)💩
	1. 백트래킹 
	2. 조건문 처리 주의 
	3. 상태 배열을 깊은 복사해서 해야 함
36. [어른 상어(골1)](https://www.acmicpc.net/problem/19237)
	1. 로직의 실행 순서에 따라 정답 채점 틀릴 수 있다
	2. 상어가 이동 다 끝나고, 1번 상어만 남았는지 검증해야 한다 (**문제 지문 읽어보기**)
37. 👋[스타트 택시(골2)](https://www.acmicpc.net/problem/19238)
38. 👋[상어 중학교(골2)](https://www.acmicpc.net/problem/21609)
39. 👋[구슬 탈출2(골1)](https://www.acmicpc.net/problem/13460)
40. 👋[낚시왕(골1)](https://www.acmicpc.net/problem/17143)
41. 👋[마법사 상어와 블리자드(골1)](https://www.acmicpc.net/problem/21611)
42. 👋[마법사 상어와 복제(골1)](https://www.acmicpc.net/problem/23290)
43. 👋[큐빙(플5)](https://www.acmicpc.net/problem/5373)
44. 👋[온풍기 안녕! (플5)](https://www.acmicpc.net/problem/23289)
45. 👋[어항 정리(플5)](https://www.acmicpc.net/problem/23291)


> 이외에 코드 트리 문제 링크 15개가 주어지는데 회원가입 해도 확인이 되지 않음

**🏹 추가**
1. 👋[피보나치 수의 합 (골1)](https://www.acmicpc.net/problem/2086)
2. 👋[피노나치 수 6 (골2)](https://www.acmicpc.net/problem/11444)

