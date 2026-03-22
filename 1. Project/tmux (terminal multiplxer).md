
> https://www.youtube.com/watch?v=vlg8X0N8z08

깃헙 
https://github.com/tmux/tmux

tmux 없이 할때 
- 터미널 에뮬레이터가 닫히면 shell도 닫힌다 

tmux 있을대
- (터미널 에뮬레이터 -> tmux 클라이언트) -> (tmux 서버 -> shell)
	- 터미널 에뮬레이터가 닫혀도 tmux 서버가 살아있다면 접속 가능 
		- `tmux attach`

**터미널 에뮬레이터**  : 화면을 그려주는 앱
- iTerm
- `Ghostty`(✨추천)
- Wezterm
- Kitty
- Warp
- `cmux` (✨추천)

**터미널 멀티플렉서** : 세션을 관리하는 서버
- **tmux**
- screen

> 멀티플렉서는 에뮬레이터에서 다 돌아감 

**tmux 3계층 구조**
1)세션 안에
- 2)윈도우
	- 3)Pane

🔎 tmux cheat sheet
- https://tmuxcheatsheet.com/
- ✅ Prefix (예약어) 
	- `ctrl + b` 를 누른 후 다른 키 입력 => ctrl + space로 설정하기도 한다
- Pane 
	- 수직 분할 : `%`
	- 수평 분할 : `"`
	- 이동 :  `상하좌우 방향키`
- 윈도우 전환 
	- 이전 : `p`
	- 다음 :  `n`
	- 인덱스 : 예약어 누르고 윈도우 번호(인덱스) 누른다
	- detached : `d`
	- `tmux ls`
	- attach : `a` 또는 `attach`
	- session 확인 : `s` // 세션 뭐있는지 방향키로 선택가능하네
	- window 확인 : `w` // window 리스트를 s 보다 선호한다고 한다📌
		- 방향키로 이동 가능
		- 또는 좌측 인덱스 번호로 바로 이동 가능 📌
- session 이름 변경 : `$`
- window 이름 변경 : `"`
- 세션 생성 : 예약어 입력 후 `:new` 
- 윈도우 생성 : 예약어 입력 후 `:c` 
- pane 확대  : `z` 📌 (한번더 누르면 원상 복귀)
- pane -> pane으로 메시지를 보내거나 캡처도 가능한다네
	- `스크립트 자동화`도 가능하다네
	- 백엔드, 프론트, 클러드 윈도우와 패널을 나누넨 
- ex. 클러드한테`0번 pane에 있는 로그를 가져와서 분석해줘`


**tmux 설정** 
- `~/.tmux.conf`
- 참고 
	- https://gist.github.com/devbrother2024/897c72793673a3d4a7019edc1e07e1ea

```text
# 필수 활성화 (tmux 저장 가능한 라인수)
set-option -g history-limit 50000

# 마우스로 pane 이동, 창 크기 조절 가능
set -g mouse on

# Pane 이동이 옵션키 + vi 방향키로 설정 하네
..

# 플러그인 

# 테마

# yazi 파일 탐색기 (예약어 + tab)
```