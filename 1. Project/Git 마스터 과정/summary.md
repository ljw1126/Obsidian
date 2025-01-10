
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
## 기본 명령어
### workflow
local에서 아래 3가지 단계로 나누어진다
- **working directory**
	- 현재 작업 중인 파일 디렉토리 영역
	- untracked, tracked 파일로 구분
		- untracked : 신규 생성된 파일로 깃 이력에서 추적하지 않는 파일에 표시
		- tracked : 깃 이력에서 추적/관리하고 있는 파일
			- <font color="#ffff00">unmodified</font>와 <font color="#ffff00">modified</font> 상태를 가진다
	- `git add .` 명령어를 통해 staging area로 작업 파일을 올릴 수 있다
- **staging area**
	- 깃 디렉토리에 저장하기 전 단계
	- 의미있는 작업 단위를 나누어 커밋을 작성
	- `git commit -m "메시지"` 명령어를 통해 깃 디렉토리에 이력 저장한다
- **.git directory** 
	- 깃 이력, 스냅샷을 관리하는 저장소


### add 
working directory에서 작업한 파일을 저장하기 위해 staging area로 올린다
```text
- 파일 만들기 
echo hello world! > a.txt
echo hello world! > b.txt
echo hello world! > c.txt

- 상태확인 
git status 

- git에서 tracking 하도록 add 
git a.txt
git *.txt         #존재하는 txt파일 전부  

- staging area 다 삭제하기 ( = untrack상태로)
git rm --cached *
```

>[!note] git add * 와 git add . 명령어의 미묘한 차이가 존재한다


### ignore
깃 이력을 통해 관리 대상에서 제외할 파일 또는 폴더 정보를 나타냄
```text
vim .gitignore  # 루트 디렉토리 생성

ㅁ 특정파일만 안하고 싶은 경우 
log.log 

ㅁ 특정확장자만 안하고 싶은 경우 
*.log

ㅁ 특정폴더 or 폴더내 파일만 안하고 싶은 경우  
> build/     build/*.log
```

>[!info] gitignore.io 사이트를 통해 쉽게 원하는 .gitignore 파일을 생성가능
>https://www.toptal.com/developers/gitignore


### status
현재 깃 프로젝트의 파일 상태를 표시한다
```text
git status -h      # 설명
git status         # 전체 내용 나타냄
git status -s      # 간단하게 표시
```


### diff
깃 프로젝트에서 변경된 파일 내용을 비교하여 보여준다
```text
git diff -h        # 도움말 
git diff           # working dir 에 있는 변경사항만 확인 

/***************************************************************/
diff --git a/c.txt b/c.txt       
index a042389..f5be8ac 100644    # 인덱스인데 넘어가기
--- a/c.txt
+++ b/c.txt
@@ -1 +1,2 @@                # -는 이전 파일 첫줄, +는 이후 파일의 1~2 줄 확인해
hello world!
+add
/***************************************************************/

```

**특정 단계에 있는 파일만 비교할 경우**
```bash
git diff --staged      # staging area에 있는 이력만 비교 
git diff --cached      # staged와 동의어로 사용됨 
```

**editor 연결**
터미널의 기본 에디터말고 vscode와 연동하여 변경 내용을 확인할 수 있다
```bash
# 글로벌 환경 설정 열기
git config --global -e 

# 아래 설정을 추가하고 저장
   [diff]
	  tool=vscode
   [difftool "vscode"]
	  cmd=code --wait --diff $LOCAL $REMOTE
   [merge]
	  tool=vscode
   [mergetool "vscode"]
	  cmd=code --wait $MERGED

# 실행
git difftool            
git difftool --staged   # staging area 파일 대상 비교 

```


### commit\*
작업한 파일들을 커밋이라는 하나의 단위로 분할 하려 깃 이력에 저장한다
```bash
git commit -m "메시지"
git commit -am "메시지" #(add+commit) working, staging 모든 파일을 commit하겠다
```

**Tip.**
1. 작은 단위로 의미있는 작업을 넣어서 commit하는게 좋다 (<u>의미없는 commit 생성은 지양해야 한다</u>)
2. 동사와 명사 형식으로 커밋 메시지를 작성한다
3. history(log)에 표시되는 내용만 작업해야지, <u>log에 기재하지 않은 작업을 해서 포함하게 되면 혼동 발생하여 협업에 방해될 수 있다</u>
4. commit은 너무 커도 문제 있고, 너무 작아도 문제 있다
5. (추가) remote에 반영하기 전 rebase를 통해 commit 이력 합치기/분리하기로 정리하는 것이 좋다


**Tip. commit template 설정**
// 작성 예정


### 파일 변경시 유용한 팁\*
평소 아래와 같이 파일을 삭제 후 status 확인할 경우 staging area에 포함되지 않아 직접 추가해줘야 했다
```bash
rm c.txt
git status 
git add .
```

반면 아래와 같이 할 경우 staging area에 자동으로 포함되게 된다
```bash
git rm d.txt
```

mv 명령어의 경우에도 동일하게 staging area에 자동으로 포함되게 할 수 있다
```bash
git mv cc.txt ccc.txt 
```


### log
터미널을 통해 커밋 이력, 버전 정보를 확인할 수 있다


// terminal plugin

// 로그 포맷을 공식 사이트 지원

// 태그 자리수별 의미 중요



---
## Reference.
- 강의 : https://academy.dream-coding.com/courses/git
- 깃 공식 : https://git-scm.com/
- 필수 리눅스 명령어 (유튜브) : https://www.youtube.com/watch?v=EL6AQl-e3AQ
- iterm2 setup guide : https://gist.github.com/kevin-smets/8568070'
- gitignore : https://www.toptal.com/developers/gitignore


```bash

```