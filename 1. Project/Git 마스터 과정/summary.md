# Git 마스터 과정

>[!info] What is Git
>- 명령어 단위로 이루어진 프로그램
>- VCS ( Version Control System : 버전관리시스템 ) 중 하나 

- 과거에는 폴더에 이름을 붙여 관리하기도 함 (불편)
- **CVC (Centralized Version Control)** 
     - CVS, SubVersion, Perforce  
     - 예로 SVN
     - 단점
	     - 중앙 서버에 문제가 생기면 많은 개발자들이 업무불가하게 된다
		 - 오프라인에 인터넷 없을때 사용 x
  - **DVC (Distributed Version Control:분산버전제어)**
	 - 예로 Git
	 - 각 개발자들이 동일한 history를 가지고 있어서, 서버 문제 생겨도 ok 
     - 인터넷 없어도 ok (history로 서버 복원 가능 , 오프라인 작업가능)
  - 웹 기반 버전 관리 저장소로 대표적으로 github와 bitbucket 있음

>[!note] delta-based version control와 stream of snapshots 방식의 차이
>1) delta-based version control 의 경우 
>    버전 별로 달라진 내용만 가지고 있어, 변경사항을 계산해서 적용하는데 시간이 많이 걸림
>2) stream of snapshots (git 해당)
>    커밋 단위로 프로젝트 스냅샷을 가지고 있다. 변경되지 않은 파일의 경우 링크로 연결되기 때문에 버전들 사이를 자유자재로 오류없이 빠르게 이동 가능하다 (상대적으로 가볍다)


**Git을 사용해야 하는 이유**
- most commonly used 
- free
- open source 
- lightning fast 
- work offline 
- undo mistakes : 실수만회 쉽게 가능
- easy and fast branching/merging : 기능별 branch 만들어서 협업을 효율적으로 가능

---
## Setup
```bash
git config --list

git config --global -e          # 기본 편집기로 .gitconfig 열기
```


**default editor**
```bash
git config --global core.editor "code"      # VScode 열리고 터미널 커맨드 입력가능 

git config --global core.editor "code --wait"  # 편집 끝날때까지 터미널은 대기 상태

git config --global -e      # 확인
```


**사용자 정보 설정**
```bash
git config --global user.name "이름"

git config --global user.email "이메일"  

git config user.name  # 확인
```

**Set Auto CRLF**
```bash
git config --global core.autorlf true  #for window

git config --global core.autorlf input #for mac
```

>[!info] core.autorlf 속성
> 운영체재별로 개행문자가 다른다
> - window 에서는 텍스트 줄바꿈시 text\r\n'  (\r: carriage-return , \n:line feed)
 >- mac에서는 텍스트 줄바꿈시 'text\n'   
 > 
 > 이로인해 서로 다른 운영체재에서 개행문자로 인해 수정하지 않았는데 변경이력 전체가 뜨는 이슈가 발생가능
 > 
 > 그래서 해당 core.autorlf 속성을 설정한다. 설정하게 되면  
 > i) win -> git 넣을때 \r을 삭제해주고, win <- git 받을때 \r 붙여줌
 > ii) mac -> git 넣을때 \r를 삭제해줌 ( mac은 원래 안하는데 이메일 같은 내용 복붙시 붙을 수 있어서 설정함)


>[!info] global config에 아래 설정 추가
>[push] 
>    default = current // 로컬 브랜치 이름이 항상 리모트와 동일하게 설정함
>[pull]
>    rebase = true       // pull 명령어는 merge와 rebase 옵션 중 선택해서 동작 가능


**Git Alias (축약어) 설정 예시**
```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status 
```


**도움말**
```bash
git config -h
```

---



---
## Reference.
- 강의 : https://academy.dream-coding.com/courses/git
- 깃 공식 : https://git-scm.com/
- 필수 리눅스 명령어 (유튜브) : https://www.youtube.com/watch?v=EL6AQl-e3AQ
- iterm2 setup guide : https://gist.github.com/kevin-smets/8568070


```bash

```