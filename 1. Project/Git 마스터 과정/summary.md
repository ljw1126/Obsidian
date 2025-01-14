- [[#Setup|Setup]]
- [[#기본 명령어|기본 명령어]]
	- [[#기본 명령어#workflow|workflow]]
	- [[#기본 명령어#add|add]]
	- [[#기본 명령어#ignore|ignore]]
	- [[#기본 명령어#status|status]]
	- [[#기본 명령어#diff|diff]]
	- [[#기본 명령어#commit*|commit*]]
	- [[#기본 명령어#파일 변경시 유용한 팁*|파일 변경시 유용한 팁*]]
	- [[#기본 명령어#log|log]]
	- [[#기본 명령어#log 꾸미기*|log 꾸미기*]]
	- [[#기본 명령어#show|show]]
	- [[#기본 명령어#diff|diff]]
	- [[#기본 명령어#Tag*|Tag*]]
- [[#Branch|Branch]]
	- [[#Branch#merge|merge]]
	- [[#Branch#merge --no-ff 옵션|merge --no-ff 옵션]]
	- [[#Branch#Three-way merge|Three-way merge]]
	- [[#Branch#merge conflict*|merge conflict*]]
	- [[#Branch#P4Merge 오픈소스 도구|P4Merge 오픈소스 도구]]
	- [[#Branch#Rebase|Rebase]]
	- [[#Branch#rebase --onto*|rebase --onto*]]
	- [[#Branch#cherry-pick|cherry-pick]]
- [[#Stash|Stash]]
- [[#실수를 만회하는 방법들|실수를 만회하는 방법들]]
	- [[#실수를 만회하는 방법들#커밋 전 변경 내용 취소하기|커밋 전 변경 내용 취소하기]]
	- [[#실수를 만회하는 방법들#커밋 메시지 수정|커밋 메시지 수정]]
	- [[#실수를 만회하는 방법들#reset|reset]]
	- [[#실수를 만회하는 방법들#reflog|reflog]]
	- [[#실수를 만회하는 방법들#revert|revert]]
	- [[#실수를 만회하는 방법들#이전 커밋 수정하기|이전 커밋 수정하기]]
	- [[#실수를 만회하는 방법들#필요없는 커밋 삭제|필요없는 커밋 삭제]]
	- [[#실수를 만회하는 방법들#코끼리 커밋 분할하기|코끼리 커밋 분할하기]]
	- [[#실수를 만회하는 방법들#여러 개 커밋 합치기|여러 개 커밋 합치기]]
- [[#GitHub|GitHub]]
	- [[#GitHub#fetch vs pull  차이|fetch vs pull  차이]]
	- [[#GitHub#fetch 심화|fetch 심화]]
	- [[#GitHub#pull 심화|pull 심화]]
	- [[#GitHub#blame|blame]]
	- [[#GitHub#bisect|bisect]]
- [[#Reference.|Reference.]]

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
>[!tip] 터미널을 사용할 경우 oh my zsh에 git plugins을 설치하는 것을 권장 
>- 다양한 alias와 편의 기능 지원

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


### commit*
작업한 파일들을 커밋이라는 하나의 단위로 분할 하려 깃 이력에 저장한다
```bash
git commit -m "메시지"
git commit -am "메시지" #(add+commit) working, staging 모든 파일을 commit하겠다
```


>[!tip]
>1. 작은 단위로 의미있는 작업을 넣어서 commit하는게 좋다 (<u>의미없는 commit 생성은 지양해야 한다</u>)
>2. 동사와 명사 형식으로 커밋 메시지를 작성한다
>3. history(log)에 표시되는 내용만 작업해야지, <u>log에 기재하지 않은 작업을 해서 포함하게 되면 혼동 발생하여 협업에 방해될 수 있다</u>
>4. commit은 너무 커도 문제 있고, 너무 작아도 문제 있다
>5. (추가) remote에 반영하기 전 rebase를 통해 commit 이력 합치기/분리하기로 정리하는 것이 좋다


>[!tip] git commit message template 설정하여 규격을 맞추어 커밋을 작성하는 것을 권장한다


### 파일 변경시 유용한 팁*
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
```bash
git log --patch 
git log --p
git log --p <commit>
```

**--oneline**
```bash
git log --oneline     # 내림차순, 최근 이력부터 나열
----------------------------------------------------------
d643a6e (HEAD -> master) Update Welcome page
b8e485f Add light theme
bd7bd28 Add About page
328708d Add Welcome page
0ad2dbb Add UserRepository module
9186a41 Add LoginService module
1563681 Initialise project
----------------------------------------------------------


 git log --oneline --reverse     # 오름차순, 첫 이력부터 나열
----------------------------------------------------------
1563681 Initialise project
9186a41 Add LoginService module
0ad2dbb Add UserRepository module
328708d Add Welcome page
bd7bd28 Add About page
b8e485f Add light theme
d643a6e (HEAD -> master) Update Welcome page
----------------------------------------------------------
```


>[!iinfo] HEAD 
>현재 포인터가 가르키고 있는 커밋을 표시
>- head ~ 1 : head에서 이전 커밋
>- head ~ 2 : head에서 두 단계 전 커밋


### log 꾸미기*

**공식 문서** 
https://git-scm.com/docs/pretty-formats

```bash
git log --pretty=oneline

# 포맷을 지정 가능 ( 공식 메뉴얼 사이트 참고하기 )
git log --pretty=format:"%h %an"         # 해쉬코드와 작성자    
git log --pretty=format:"%h %an %ar %s"     # 해쉬코드,작성자, 시간, 타이틀

# fix 브랜치에서 master 브랜치와 함께 로그 보려면  
git log --oneline --graph --all

---------------------------------------------------------- 
* d643a6e (master) Update Welcome page
| * c38c4c4 (HEAD -> fix) Fix light theme
|/
* b8e485f Add light theme
* bd7bd28 Add About page
* 328708d Add Welcome page
* 0ad2dbb Add UserRepository module
* 9186a41 Add LoginService module
* 1563681 Initialise project
----------------------------------------------------------  
```

**global config alias 설정(권장)**
```bash
git config --global alias.hist "log --graph --all --pretty=format:'%C(yellow)[%ad]%C(reset) %C(green)[%h]%C(reset) | %C(white)%s %C(bold red){{%an}}%C(reset) %C(blue)%d%C(reset)' --date=short"

git hist  #지정한 약어로 호출하면 [날짜][해시코드]|메시지{{작성자}} 표출됨 
```

**로그 심화**
- 옵션 통해 필터링 기능을 지원한다
- `-S` 옵션으로 깃 메시지에 문자열 검색이 가능한게 눈에 띈다
```bash
git log -3                     # 세줄만 출력 
git log --oneline -3           # 세줄만 출력 
git log --author="ellie"       # 작성자가 ellie 인거만 검색 
git log --before="2020-09-08"  # 2020-09-08 날짜 이전 꺼만 검색
git log --grep="project"       # 커밋중 project 문자 포함된 것만 검색 

git log -S "문자열"         # (유레카) 변경 소스 코드 내용안에서 문자열 검색하고 싶을때 사용
git log -S "about" -patch(또는 p)     # 자세히 보기 위해 옵션 p 추가

git log 파일이름
> git log about.txt            # about 파일에 대한 커밋을 볼 수 있음 
> git log -p about.txt         # 자세히
> git log -s about.txt         # 간략히 

git log HEAD  (= git log 동일 )
git log HEAD~1                 # HEAD의 이전 부모부터 보고 싶다면
git log HEAD~2 
```


### show
지정한 커밋의 변경 이력을 확인할 수 있다
```bash
git show 해시코드              # 해당하는 커밋의 내용을 정확히 확인가능 

---------------------------------------------------------- 
$ git show 0ad2dbb
commit 0ad2dbb68fd004185753efdc3a43776e4396f58e
Author: Ellie <dream.coder.ellie@gmail.com>
Date:   Wed Oct 28 22:22:11 2020 +0900

	Add UserRepository module

diff --git a/user_repository.txt b/user_repository.txt
new file mode 100644
index 0000000..c362156
--- /dev/null
+++ b/user_repository.txt
@@ -0,0 +1 @@
+user repository
---------------------------------------------------------- 
```

*커밋에서 원하는 파일만 확인하고 싶을 경우*
```bash
git show 0ad2dbb:user_repository.txt
```


### diff 
변경 내용을 확인한다

*두 가지 커밋을 비교할 경우*
```bash
$ git diff b8e485f c38c4c4

diff --git a/light_theme.txt b/light_theme.txt
index 62fd177..89ae3ea 100644
--- a/light_theme.txt
+++ b/light_theme.txt
@@ -1 +1,2 @@
theme
+fix theme
```

두 가지 커밋에서 원하는 파일만 비교하고 싶을 때
```bash
git diff b8e485f c38c4c4 light_theme.txt
```


### Tag*
- 특정 커밋을 북마크 하고 싶을때 사용
- 수많은 history 중에 내가 원하는 commit 으로 빠르게 이동할 수 있음
- 보통은 'release 버전'을 많이 한다 
	- ex) v1.0.0 , v.2.0.0 

>[!info] semantic versioning
>v2.0.0 = v.major.minor.fix 
>- major version : 어떤 특정기능 추가되었을때, 전체적인 업데이트 발생했을때 
>- minor : 그 커다란 기능 중 조금의 기능이 업데이트되거나 개선되었을때
>- fix : 기존에 존재하는 기능에서 오류수정했을때


**태그 생성**
```bash
# 형식 git tag 문자열 해시코드
git tag v1.0.0 0ad2dbb  
```

**태그 버전별 상세 메모 추가**
```bash
# git tag 태그명 해쉬코드 -am "내용"
git tag v1.0.1 328708d -am "Relase note 1.0.0 ... "

git show v1.0.1
----------------------------------------------------------  
tag v.1.0.1
Tagger: leejinwoo <zral1004@gmail.com>
Date:   Sat Dec 26 13:19:34 2020 +0900

Relase note 1.0.0 ...

commit 328708d03533fa78d96d9984e9d97ab76488671c (tag: v.1.0.1)
// .. 
```

**태그 명령어**
```bash
# 목록 확인
git tag

# 태그 목록 중 특정 문자열만 포함된거만 검색 
git tag -l "v.1.0.*"

# 태그 삭제
git tag -d <tag> 

# 태그로 checkout 
git checkout <tag>

# 브랜치 생성하면서 태그 생성 (git checkout -b 브렌치명 태그명)
git checkout -b testing v.2.0.0

# remote 반영 
git push origin <tag>             //단일 태그
git push origin --tags           //모든 태그

# remote 태그 삭제
git push origin --delete <tag>

```

---
## Branch

>[!info] 브랜치를 왜 만들어야 할까?
>- 브랜치를 만들지 않고 작업할 경우 master (main) 브랜치에 작업 이력이 쌓이게 된다
>- branch를 만들어서 일하게 되면 동시 다발적으로 다수의 개발자가 작업을 해 나갈 수 있기 때문에 병렬적으로 업무를 진행할 수 있는 큰 장점이 있다
>	- 보통 A기능 완성 > 검증 완료 > 코드 리뷰 순으로 완료되면 master branch merge 수행한다
>	- A기능 branch를 master에 합치는 경우도 있지만, A기능 branch의 history를 하나로 합쳐서(squash, rebase) master에 merge 하는 것을 더 선호된다 (merge된 branch는 필요없다면 정리한다)  
>- 브랜치로 병렬작업을 하더라도 master 브랜치를 pointer로 가르키고 있고, 스냅샵(branch생성시)의 경우 변경 안 된 파일은 링크만 가지기 때문에 branch간 전환이 빠르게 된다함 (Git의 장점)
>


**브랜치 생성**
```bash 
$ git branch new-branch  //new-branch 생성

----------------------------------------------------------
$ git hist        // new-branch라는 새로운 pointer가 동일한 commit을 가르키고 있다
* [2020-10-28] [3345766] | feature a {{Ellie}}  (HEAD -> master, new-branch, feature-a)
* [2020-10-28] [d643a6e] | Update Welcome page {{Ellie}}
| * [2020-10-28] [c38c4c4] | Fix light theme {{Ellie}}  (fix)
|/
* [2020-10-28] [b8e485f] | Add light theme {{Ellie}}
* [2020-10-28] [bd7bd28] | Add About page {{Ellie}}
* [2020-10-28] [328708d] | Add Welcome page {{Ellie}}
* [2020-10-28] [0ad2dbb] | Add UserRepository module {{Ellie}}
* [2020-10-28] [9186a41] | Add LoginService module {{Ellie}}
* [2020-10-28] [1563681] | Initialise project {{Ellie}}
----------------------------------------------------------  
```

**브랜치 확인**
```bash
$ git branch             //branch 목록 

$ git branch --all       //remote 서버에 있는 branch 까지 목록으로 보여줌  

$ git branch -v                //간단한 commit 까지 

$ git branch --merged          //현재 branch에 merge가 된 branch를 확인가능 

$ git branch --no-merged       //master branch에 merge안된, master에서 파생된 변경사항의 branch나타냄

```

**브랜치 이동**
```bash
$ git switch new-branch        //새로운 branch로 이동 

$ git switch -C new-branch2    //새로운 branch를 만들고 동시에 switch 때림 

$ git chekcout new-branch 

$ git checkout 해시코드        //해시코드가 있는 버전으로 변경가능, branch도 변경가능 
							 //원하는 버전으로 이동뒤 거기에 있는 branch로 checkout이 되네(HEAD가 가르킴) ?! 
							 
$ git checkout -b 브랜치명     // 브랜치 생성과 checkout 동시에 (자주 사용*) 
```


**브랜치 삭제**
```bash
$ git branch -d 브랜치명

# remote 서버에 반영 
$ git push origin --delete 브랜치명
```


**브랜치 이름 변경**
```bash
# branch 이름 변경하기 ,  git branch --move 변경전 변경후명
$ git branch --move fix fix_welcome 
```


**브랜치 remote 서버 반영**
```bash
$ git push --set-upstream origin fix-welcome      //remote 반영하기
```

>[!note] 아래와 같이 두 브랜치를 비교하거나, 추가로 두 브랜치 사이의 특정 파일을 지정하여 확인 가능
> $ git log master..test         // master와 test branch 사이의 로그만 확인
> $ git hist master..test 
> $ git diff master..test


### merge 
>[!info] fast-forward merge
>- git merge 중에 가장 깔끔하고 간단핳ㄴ 방법
>- A branch 생성 후 작업하면서 master에 작업한게 없다면, 단순히 A branch로 master branch의 pointer 로 옮김
>- (단점) history(commit)에 merge 되었다는 이력이 남지 않는다
>	- merge commit을 history로 남기고 싶은 경우 --no-ff 옵션 추가 (=Three-way merges)


**예제. merge**
```bash 
$ git hist  

* [2020-10-28] [7619c90] | h {{Ellie}}  (feature-b)
* [2020-10-28] [769df87] | g {{Ellie}}
| * [2020-10-28] [aaf6522] | f {{Ellie}}  (feature-a)
| * [2020-10-28] [59127a9] | e {{Ellie}}
|/
* [2020-10-28] [2797019] | d {{Ellie}}  (HEAD -> master)
* [2020-10-28] [e8515d8] | c {{Ellie}}
* [2020-10-28] [d0d15b4] | b {{Ellie}}
* [2020-10-28] [2c9e233] | a {{Ellie}} 
```

`master` 브랜치에 `feature-a` 병합
```bash
$ git checkout master
$ ls // 파일 확인 
$ git checkout feature-a
$ ls // 파일 확인(차이 있음)
$ git checkout master 
$ git merge feature-a      //master에 featrue-a 합침  

$ git hist
* [2020-10-28] [7619c90] | h {{Ellie}}  (feature-b)
* [2020-10-28] [769df87] | g {{Ellie}}
| * [2020-10-28] [aaf6522] | f {{Ellie}}  (HEAD -> master, feature-a)
| * [2020-10-28] [59127a9] | e {{Ellie}}
|/
* [2020-10-28] [2797019] | d {{Ellie}}
* [2020-10-28] [e8515d8] | c {{Ellie}}
* [2020-10-28] [d0d15b4] | b {{Ellie}}
* [2020-10-28] [2c9e233] | a {{Ellie}}

$ git branch -d feature-a  // 보통 병합하면 삭제해 줌

```

### merge --no-ff 옵션
- fast-forward merge를 하면 이력이 안 남는데, 이력을 남길 필요가 있다면 `--no-ff` 옵션 사용
	- `ff` : fast forward

**예시**
```bash
$ git merge --no-ff feature-c    //하게 되면 commit 메시지 남기는 화면으로 넘어감
```


### Three-way merge
- fast-forward 는 기본적으로 commit 이력을 남기지 않는다
- fast-forward merge로 하지 않고 이력을 남기고 싶다면  three-way merge 사용
	- master에 새 이력 발생하여 fast-forward merge 불가한 경우 three-way-merge 사용해야 한다

```bash
$ git hist  // master에 새 커밋 이력이 발생했기때문에 ff가 불가능함
*   [2020-12-26] [5118c78] | Merge branch 'feature-c' {{leejinwoo}}  (HEAD -> ma
ster)
|\
| * [2020-12-26] [6b821f9] | test c branch {{leejinwoo}}
|/
* [2020-10-28] [aaf6522] | f {{Ellie}}
* [2020-10-28] [59127a9] | e {{Ellie}}
| * [2020-10-28] [7619c90] | h {{Ellie}}  (feature-b)
| * [2020-10-28] [769df87] | g {{Ellie}}
|/
* [2020-10-28] [2797019] | d {{Ellie}}
* [2020-10-28] [e8515d8] | c {{Ellie}}
* [2020-10-28] [d0d15b4] | b {{Ellie}}
* [2020-10-28] [2c9e233] | a {{Ellie}}

$  git merge feature-b       //참고로 merge 정책을 no-ff로 글로벌 설정도 가능  
$  git hist           
*   [2020-12-26] [42ac23c] | Merge branch 'feature-b' {{leejinwoo}}  (HEAD -> master)
|\
| * [2020-10-28] [7619c90] | h {{Ellie}}  (feature-b)
| * [2020-10-28] [769df87] | g {{Ellie}}
* |   [2020-12-26] [5118c78] | Merge branch 'feature-c' {{leejinwoo}}
|\ \
| * | [2020-12-26] [6b821f9] | test c branch {{leejinwoo}}
|/ /
* | [2020-10-28] [aaf6522] | f {{Ellie}}
* | [2020-10-28] [59127a9] | e {{Ellie}}
|/
* [2020-10-28] [2797019] | d {{Ellie}}
* [2020-10-28] [e8515d8] | c {{Ellie}}
* [2020-10-28] [d0d15b4] | b {{Ellie}}
* [2020-10-28] [2c9e233] | a {{Ellie}}
```


### merge conflict*
- conflict 예제 파일 사용
- 두 가지 branch에서 동일한 파일 수정 후 merge 하게 될 경우 conflict이 발생한다

`main.txt`에 conflict이 발생했을 때
```bash
$ git hist 
* [2020-10-28] [7bccbf9] | third commit on master {{Ellie}}  (HEAD -> master)
| * [2020-10-28] [5e26876] | second commit on feature {{Ellie}}  (feature)
|/
* [2020-10-28] [c133623] | first commit on master {{Ellie}}  
   
$ git checkout 해시코드       // 각각 cat main.txt 확인해보기 , 동일한 파일 수정했다함 
$ git checkout master
$ git merge feature
Auto-merging main.txt
CONFLICT (content): Merge conflict in main.txt
Automatic merge failed; fix conflicts and then commit the result.

$ git status 
$ cat main.txt      //충돌 파일 확인       
$ open main.txt     //수동으로 파일 열어서 수정후 저장(open은 mac 명령어)  

# conflict 내용부분 수정 후  
$ git add .
$ git merge --continue           // ff merge가 아니라 commit 메시지 남기는게 뜸 
$ git hist

*   [2020-12-26] [59c5390] | Merge branch 'feature' {{leejinwoo}}  (HEAD -> master)
|\
| * [2020-10-28] [5e26876] | second commit on feature {{Ellie}}  (feature)
* | [2020-10-28] [7bccbf9] | third commit on master {{Ellie}}
|/
* [2020-10-28] [c133623] | first commit on master {{Ellie}}

$ git show 59c5390    
```


 >[!warning] merge conflict만 해결해야지 , 다른 내용을 추가로 넣는거는 지양해야 한다


```bash
$ git config --global -e  //아래 내용 추가, 저장
[merge]
	tool=vscode
[mergetool "vscode"]
  cmd=code --wait $MERGED

# conflict 발생했을 경우   
$ git mergetool
$ git status               //수정후 보면
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
		modified:   main.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)
		main.txt.orig          // conflict 되어 있는 소스가 있음 

$ git config --global mergetool.keepBackup false  
$ git merge --abort       // merge 취소하기
$ git clean -dn           // 삭제 대상 파일 확인 
$ git clean -fd           // 쓸데없는 파일 삭제 *.orig 
$ git merge --continue    // merge 끝 .. 
```

>[!info] merge시 생성되는 orig 확장자 파일이 생성되지 않도록 비활성화
>$ git config --global mergetool.keepBackup false  


### P4Merge 오픈소스 도구
- conflict 해결을 지원하는 오픈소스 도구
- https://www.perforce.com/downloads/visual-merge-tool 다운로드 후 설치

```text
[merge]
	tool=p4merge
[mergetool "p4merge"]
	path = "C:/ProgramData/Microsoft/Windows/Start Menu/Programs/Perforce/p4merge"
```
참고. 경로는 os마다 틀리니 적절히 수정


### Rebase
- `rebase 예제`를 사용한다
- ff merge는 이력이 깔끔해지지만, 3 way merge는 추가 커밋 이력이 남아 지저분한 감이 있다
	- rebase > master branch 에 최신 history 로 A브랜치 포인터 이동하면 ff merge가 됨 (이력 깔끔)
	- 단, 다른 개발자와 동일한 A branch 사용하고 있다면, 내가 rebase 하는 A branch 의 history는 전혀 다른 commit 되므로, 다른 개발자와 충돌(merge conflict) 발생가능

>[!warning] rebase를 할때 이미 history가 remote 서버에 반영되어 있으면 절대 rebase 하면 안된다
>로컬에 있는 반영 안 된 commit 이력에 한해서 정리 용도로 rebase를 자유롭게 해도 됨 (오히려 권장)

```bash
$ git hist //feature-b 브랜치의 포인터를 최신 master branch 포인터로 이동시킴
* [2020-10-28] [7619c90] | h {{Ellie}}  (feature-b)
* [2020-10-28] [769df87] | g {{Ellie}}
| * [2020-10-28] [aaf6522] | f {{Ellie}}  (HEAD -> master, feature-a)
| * [2020-10-28] [59127a9] | e {{Ellie}}
|/
* [2020-10-28] [2797019] | d {{Ellie}}
* [2020-10-28] [e8515d8] | c {{Ellie}}
* [2020-10-28] [d0d15b4] | b {{Ellie}}
* [2020-10-28] [2c9e233] | a {{Ellie}}

$ git checkout feature-b 
$ git rebase master   //최신 master commit으로 feature-b의 포인터 base를 옮기게 됨 
$ git hist     // 일직선이 되어 있음 
* [2020-10-28] [49fdf8f] | h {{Ellie}}  (HEAD -> feature-b)
* [2020-10-28] [c272a99] | g {{Ellie}}
* [2020-10-28] [aaf6522] | f {{Ellie}}  (master, feature-a)
* [2020-10-28] [59127a9] | e {{Ellie}}
* [2020-10-28] [2797019] | d {{Ellie}}
* [2020-10-28] [e8515d8] | c {{Ellie}}
* [2020-10-28] [d0d15b4] | b {{Ellie}}
* [2020-10-28] [2c9e233] | a {{Ellie}}

$ git checkout master 
$ git merge feature-b 
$ git hist   // feature-b 와 feature-a 지우면 아주 깔끔해짐
* [2020-10-28] [49fdf8f] | h {{Ellie}}  (HEAD -> master, feature-b)
* [2020-10-28] [c272a99] | g {{Ellie}}
* [2020-10-28] [aaf6522] | f {{Ellie}}  (feature-a)
* [2020-10-28] [59127a9] | e {{Ellie}}
* [2020-10-28] [2797019] | d {{Ellie}}
* [2020-10-28] [e8515d8] | c {{Ellie}}
* [2020-10-28] [d0d15b4] | b {{Ellie}}
* [2020-10-28] [2c9e233] | a {{Ellie}}
```


### rebase --onto*

profile , profile-ui branch가 있는데 profile-ui만 master에 merge 하기로 한다
```bash
$ git hist
* [2020-10-28] [f2b9178] | Add ProfileService Interface {{Ellie}}  (profile)
| * [2020-10-28] [5a090bf] | Add profile UI {{Ellie}}  (profile-ui)
|/
* [2020-10-28] [cd9c9e7] | Add tests for ProfileService {{Ellie}}
* [2020-10-28] [dc89240] | Add ProfileService {{Ellie}}
* [2020-10-28] [bbac9d0] | Add LoginService {{Ellie}}  (HEAD -> master)

# master에 업데이트 할거고 profile 에서 파생된 profile-ui를 올려두자
$ git rebase --onto master profile profile-ui        
$ git hist
* [2020-10-28] [12d5e88] | Add profile UI {{Ellie}}  (HEAD -> profile-ui)
| * [2020-10-28] [f2b9178] | Add ProfileService Interface {{Ellie}}  (profile)
| * [2020-10-28] [cd9c9e7] | Add tests for ProfileService {{Ellie}}
| * [2020-10-28] [dc89240] | Add ProfileService {{Ellie}}
|/
* [2020-10-28] [bbac9d0] | Add LoginService {{Ellie}}  (master)

# rebase 하니 HEAD가 profile-ui로 가있음, 이제 ff merge 가능해짐 
$ git checkout master        
$ git merge profile-ui
$ git hist
* [2020-10-28] [12d5e88] | Add profile UI {{Ellie}}  (HEAD -> master, profile-ui
)
| * [2020-10-28] [f2b9178] | Add ProfileService Interface {{Ellie}}  (profile)
| * [2020-10-28] [cd9c9e7] | Add tests for ProfileService {{Ellie}}
| * [2020-10-28] [dc89240] | Add ProfileService {{Ellie}}
|/
* [2020-10-28] [bbac9d0] | Add LoginService {{Ellie}}
```


### cherry-pick
- 특정 commit만 원하는 branch에 가져올 수 있도록 한다

**예시**
```bash
$ git hist
* [2020-10-28] [12d5e88] | Add profile UI {{Ellie}}  (HEAD -> master, profile-ui)
| * [2020-10-28] [f2b9178] | Add ProfileService Interface {{Ellie}}  (profile)    //얘만 master 에 가져오고 싶다
| * [2020-10-28] [cd9c9e7] | Add tests for ProfileService {{Ellie}}
| * [2020-10-28] [dc89240] | Add ProfileService {{Ellie}}
|/
* [2020-10-28] [bbac9d0] | Add LoginService {{Ellie}}

$ git cherry-pick f2b9178
$ git hist      //master 위에 새로운 commit 생김 !
* [2020-10-28] [1baca49] | Add ProfileService Interface {{Ellie}}  (HEAD -> master)
* [2020-10-28] [12d5e88] | Add profile UI {{Ellie}}  (profile-ui)
| * [2020-10-28] [f2b9178] | Add ProfileService Interface {{Ellie}}  (profile)
| * [2020-10-28] [cd9c9e7] | Add tests for ProfileService {{Ellie}}
| * [2020-10-28] [dc89240] | Add ProfileService {{Ellie}}
|/
* [2020-10-28] [bbac9d0] | Add LoginService {{Ellie}}
```

>[!info] 명령어의 동작원리를 이해하고 sourcetree와 같은 오픈소스 도구를 활용하면 도움을 받을 수 있다
>- 명령어로만 이력 정리하기 어려운 경우 오픈 소스 도구를 활용

---
## Stash
- 작업한 내역을 commit 할 단계는 아닌데 branch 변경해야 되는 경우 사용한다
	- commit하기 모호하고, 삭제하기도 모호하니 이때 <u>이력을 임시 저장</u>하기 위해 stash 사용
	- Git에는 Stash Stack 존재한다
	  
git-branch 예제 사용
```bash
$ git hist
* [2020-10-28] [3345766] | feature a {{Ellie}}  (HEAD -> master, feature-a)
* [2020-10-28] [d643a6e] | Update Welcome page {{Ellie}}
| * [2020-10-28] [c38c4c4] | Fix light theme {{Ellie}}  (fix)
|/
* [2020-10-28] [b8e485f] | Add light theme {{Ellie}}
* [2020-10-28] [bd7bd28] | Add About page {{Ellie}}
* [2020-10-28] [328708d] | Add Welcome page {{Ellie}}
* [2020-10-28] [0ad2dbb] | Add UserRepository module {{Ellie}}
* [2020-10-28] [9186a41] | Add LoginService module {{Ellie}}
* [2020-10-28] [1563681] | Initialise project {{Ellie}}
```

track 되고 있는 수정파일을 stash 저장하는 경우 
```bash
$ echo add >> about.txt  //파일수정하여 작업내역 생성 
$ git stash push       //타이틀 없이 저장됨 

# git stash push -m "타이틀명"
$ git stash push -m "first try"  // working dir에 있는 수정 파일(tracked)이 stash 에 저장됨 
```

staging area에 올라가있는 작업내역을 stash 하고 싶은 경우
```bash
$ echo add >> about.txt 
$ git add . 

# staging area에 파일이 올라가 있는거 확인가능
$ git status -s          

# --keep-index 옵션 통해 staging area에 올린 작업내역을 유지하면서 올림
$ git stash push -m "second try" --keep-index      
```

신규 생성된 untrack 파일도 stash 저장하고 싶은 경우  ( -u 옵션 )
```bash
$ echo new > new.txt         //신규 파일 생성
$ git stash                  //그냥 저장
$ git status -s              //확인시 untrack 파일은 stash에 저장되지 않는 것을 확인가능
$ git stash -u               //해당 옵션 통해서 untrack 파일도 stash 저장가능해짐
```

**stash 목록 확인**
```bash
$ git stash list 
stash@{0}: WIP on master: 3345766 feature a  
stash@{1}: On master: second try
stash@{2}: On master: first try

$ git stash show id값           // 수정 내용 간단히 확인 가능
git stash show stash@{3} -p    // 옵션 추가하여 좀더 자세히
```

**stash 반영**
```bash
$ git stash apply              // 아무것도 없으면 stack 맨위에 있는게 자동 적용됨 

$ git stash apply stash@{1}    // stash 목록은 유지하면서 다시 불러오기
```

>[!info] apply 와 pop 
>- 둘 다 stash stack에 저장된 이력을 working directory에 반영함
>	- apply : stash stack 이력을 유지한 상태에서 반영 
>	- pop : stash stack 이력을 제거하고 반영


**stash 삭제**
```bash
$ git stash drop id값          // stash 지정해서 삭제

$ git stash clear             // stash 전체 삭제 
```

**새로운 branch를 만들면서 stash 적용하고 싶을 때**
```bash
$ git stash branch 브랜치이름 
```


---
## 실수를 만회하는 방법들

git undo 예제 파일 사용

```bash 
 $ git hist
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}  (HEAD -> master)
* [2020-11-01] [e94152f] | . {{Ellie}}         # 의미없는 커밋
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}       # WIP(working in progress) : 아직 일이 진행중이다(개발용어), wip 뒤에 내용을 적어주는 게 좋은 커밋
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}
```

### 커밋 전 변경 내용 취소하기

**checkout**
```bash
$ git checkout hash..          # 특정커밋으로 전환
$ git checkout branchName      # 특정브랜치 전환 >> git switch 
$ git checkout tagName         # 태그 지정된 history로 전환 ( 특정commit 으로 전환 )
$ git checkout --file file명    # 파일 초기화 가능 >> git restore
```


**restore**
```bash
$ git restore fileName      # 특정 파일 초기화

$ git restore .             # workgin dir 전체 초기화
```

staging area에 있는 파일을 복구
```bash
# 특정 파일만 working area로 복구
$ git restore --staged fileName         

# staging area에 있는 전체 파일을 working dir로 복구
$ git restore --staged .                 
```


**reset**
```bash
$ git add .                //staging area에 다 올리고
$ git reset HEAD .         //staging area에 있는 애들이 working dir로 복구됨 ( git restore --staged . 이랑 같음 )   
$ git reset HEAD .
  Unstaged changes after reset:
  M       payment-ui.txt

# 깨끗하게 working dir 수정전처럼 클린 시켜버리기 
$ git add .
$ git restore --staged 
$ git restore .              // staging area에 있는거는 영향을 끼치지 않음 , working dir 에 있는 수정 내역 다 초기화 시킴
							 // git reset HEAD . 와 같
```


>[!note] --source 옵션은 개인적으로 이해가 되지 않으니, 필요시 찾아보기
>$ git restore --source= 
>- hash 코드나 head 포인터 사용해도 됨

```bash 
$ git hist 
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}  (HEAD -> master)
* [2020-11-01] [e94152f] | . {{Ellie}}
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

$ git show hash코드           // history 내역확인 
$ git restore --source=HEAD~2 payment-ui.txt      // HEAD에서 2번째 이전으로 해당파일을 restore 하겠다 
git restore --source=fa7bbd6 payment-ui.txt 
$ git hist       //변함없음 
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}  (HEAD -> master)
* [2020-11-01] [e94152f] | . {{Ellie}}
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}  

$ git status
On branch master
Changes not staged for commit:
(use "git add/rm <file>..." to update what will be committed)
(use "git restore <file>..." to discard changes in working directory)
	  deleted:    payment-ui.txt         //변함 있음
```


### 커밋 메시지 수정
```bash
$ git hist
* [2021-01-02] [8326209] | 수정테스트 {{leejinwoo}}  (HEAD -> master)
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}
* [2020-11-01] [e94152f] | . {{Ellie}}
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

$ git commit --amend -m "Add new file"
$ git hist
* [2021-01-02] [5018435] | Add new file {{leejinwoo}}  (HEAD -> master)
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}
* [2020-11-01] [e94152f] | . {{Ellie}}
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

$ git show HEAD       // 커밋 메시지, 추가내용 확인가능 
```


**최신HEAD에 수정된 파일 뒤늦게 반영하고 싶은 경우 (아직 remote 서버에 업로드 하지 않았을 때 유용)**
```bash
$ echo add > add.txt 
$ git add . 
$ git commit --amend             //history(commit내역)은 그대로임 
$ git show HEAD         // add.txt가 추가된것을 확인가능 !! 
commit 086e40455cc6724669552213f6da890071d16a46 (HEAD -> master)
Author: leejinwoo <zral1004@gmail.com>
Date:   Sat Jan 2 00:28:38 2021 +0900

  Add new file
  2

diff --git a/add.txt b/add.txt
new file mode 100644
index 0000000..9dd7446
--- /dev/null
+++ b/add.txt
@@ -0,0 +1,3 @@
+add
+
+I'm ellie
diff --git a/payment-ui.txt b/payment-ui.txt
deleted file mode 100644
index 86fbd66..0000000
--- a/payment-ui.txt
+++ /dev/null
@@ -1 +0,0 @@
-✨ UI Completed ✨
```


### reset
- git_undo 예제 사용

`--mixed` 옵션 경우
```bash
$ git hist
* [2021-01-02] [086e404] | Add new file 2 {{leejinwoo}}  (HEAD -> master)
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}
* [2020-11-01] [e94152f] | . {{Ellie}}       // 요기로 리셋하고 싶다함 
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}


$ git reset HEAD~2 또는 git reset e94152f
$ git hist  # 초기화한 커밋은 hist에서 사라졌지만 작업내역은 working dir에 남아있음 
* [2020-11-01] [e94152f] | . {{Ellie}}  (HEAD -> master)
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

```

>[!info] git reset을 하게 되면 기본 옵션으로 --mixed 준 것과 같다

`--soft` 옵션 경우
```bash
$ git reset --soft HEAD~1 or   git reset --soft fa7bbd6
$ git status
	On branch master
	Changes to be committed:
	  (use "git restore --staged <file>..." to unstage)
			new file:   payment-ui.txt
			
$ git hist
  * [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}  (HEAD -> master)
  * [2020-11-01] [1d11be8] | WIP {{Ellie}}
  * [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

  * [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
  * [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}
```


`--hard` 옵션 경우
```bash
$ git reset --hard HEAD        # 로컬에서 작업하던 내용이 초기화됨
$ git reset --hard 98955fc     # 또는 git reset --hard HEAD~2
$ git status
	On branch master
	nothing to commit, working tree clean

 $ git hist
   * [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}
	(HEAD -> master)
   * [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
   * [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}
```


### reflog 
- reference log
- log를 참조한다 뜻
- 바로 이전 HEAD가 가르키던 내용을 다 기억하고 있어 원하는 시점으로 복구 가능 

git_undo 예제 사용
```bash
$ git reflog
98955fc (HEAD -> master) HEAD@{0}: reset: moving to HEAD~2
fa7bbd6 HEAD@{1}: reset: moving to HEAD~1
e94152f HEAD@{2}: reset: moving to HEAD~2
086e404 HEAD@{3}: commit (amend): Add new file
5018435 HEAD@{4}: commit (amend): Add new file
8326209 HEAD@{5}: commit: 수정테스트
0ddd7ab HEAD@{6}: commit: Add payment UI       //여기로 돌아가고 싶으면 hash code 복사하고
e94152f HEAD@{7}: commit: .
fa7bbd6 HEAD@{8}: commit: Add payment client
1d11be8 HEAD@{9}: commit: WIP
98955fc (HEAD -> master) HEAD@{10}: reset: moving to HEAD^
0440fb9 HEAD@{11}: commit: WIP
98955fc (HEAD -> master) HEAD@{12}: commit: Add payment library and Add payment
service
707de7d HEAD@{13}: commit: Setup Dependencies
20ee16f HEAD@{14}: commit (initial): Initialise Project

$ git reset --hard 0ddd7ab
$ git hist
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}  (HEAD -> master)
* [2020-11-01] [e94152f] | . {{Ellie}}
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

$ git reflog 보면 head 변경된 이력이 남아 있음  # 이하 생략
```


>[!info] IDE 사용할 경우 local history extension 통해 이전 버전으로 언제든지 돌아갈 수 있다


### revert
- git_undo 예제 사용
- master branch에 심각한 문제를 일으키는 commit을 rollback 하고 싶을 때 주로 revert를 한다
	- 이미 remote server의 master branch에 commit된 경우 reset, rebase를 사용하기 보다 revert를 사용하는게 맞다!
- revert는 새로운 commit을 만들어서 이미 추가된 내용을 변경하는 것으로, history를 수정하지 않기 때문에 언제든지 자유롭게 이용할 수 있다
	- commit이 신규 생성되고 revert된 커밋의 내역이 복구된다

`fa7bbd6` 커밋을 취소한다
```bash
$ git hist
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}  (HEAD -> master)
* [2020-11-01] [e94152f] | . {{Ellie}}
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}} #여기를 취소
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}


$ git revert fa7bbd6 또는 git revert HEAD~2
[master 9145be7] Revert "Add payment client"
1 file changed, 1 insertion(+), 1 deletion(-)


$ git hist   //최신 HEAD 보면 revert 처리된걸 확인가능 
* [2021-01-02] [9145be7] | Revert "Add payment client" {{leejinwoo}}  (HEAD -> m
aster)
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}
* [2020-11-01] [e94152f] | . {{Ellie}}
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

$ git show fa7bbd6  // Add payment client

commit fa7bbd67db54237abe5782e9eefa10781d211fbe
Author: Ellie <dream.coder.ellie@gmail.com>
Date:   Sun Nov 1 11:14:38 2020 +0900

  Add payment client

diff --git a/payment-client.txt b/payment-client.txt
index 9c595a6..24e230f 100644
--- a/payment-client.txt
+++ b/payment-client.txt
@@ -1 +1 @@
-temp        // 이전에는 temp가 빼졌음 
+Implement Payment Client
\ No newline at end of file

$ git show HEAD 또는 git show 9145be7        //revert내역을 보면 

commit 9145be7258192c1f0d8095e31928ca911398cb50 (HEAD -> master)
Author: leejinwoo <zral1004@gmail.com>
Date:   Sat Jan 2 00:54:08 2021 +0900

	Revert "Add payment client"

	This reverts commit fa7bbd67db54237abe5782e9eefa10781d211fbe.

diff --git a/payment-client.txt b/payment-client.txt
index 24e230f..9c595a6 100644
--- a/payment-client.txt
+++ b/payment-client.txt
@@ -1 +1 @@
-Implement Payment Client
\ No newline at end of file
+temp      //다시 temp가 추가된 것을 호가인가능 
```


`--no-commit` 옵션
```bash
$ git revert --no-commit 1d11be8   # 자동 commit 메시지 남기지 않고 rollback이력을 staging area에 추가해줌 

$ git status  # staging area에 변경 파일이 있음 , 수동 commit 메시지 기록가능
```
- 보통은 `--no-commit` 옵션은 사용하지 않는다 함
	- 보틍은 자동으로 커밋 남겨지도록 하는게 맞음
	- 그리고 revert commit에서 추가로 내용 더하거나 변경하는 행위는 절대 해서는 안됨
	- 기존의 다른 commit과 충돌이 나는 경우, 내가 메뉴얼 적으로 수정해줘야 하는 경우 revert가 유용


### 이전 커밋 수정하기
- git undo 예제 사용 
- <u>remote 서버에 이력이 업로드 된 경우 사용해서는 x</u>
- 개인 local commit history 변경의 경우 사용 o 

>[!warning] rebase 하는 순간 뒤에 있는 모든 포인터에 대해 history가 업데이트 된다 (ex. commit id 변경)

```bash
$ git hist
* [2020-11-01] [0ddd7ab] | Add payment UI {{Ellie}}  (HEAD -> master)
* [2020-11-01] [e94152f] | . {{Ellie}}
* [2020-11-01] [fa7bbd6] | Add payment client {{Ellie}}
* [2020-11-01] [1d11be8] | WIP {{Ellie}}   // WIP이 가르키는 이전 해쉬코드 부터 시작
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

$ git rebase i 1d11be8     //i:인터렉트 ( 1d11be8 부터 이후 history를 인터렉티브하게 rebase 할거다)

  pick 1d11be8 WIP   >> pick 대신 reword 또는 r 설정 , 저장하면 메시지 수정창뜸 
  pick fa7bbd6 Add payment client
  pick e94152f .
  pick 0ddd7ab Add payment UI

  # Rebase 98955fc..0ddd7ab onto 98955fc (4 commands)
  #
  # Commands:
  # p, pick <commit> = use commit
  # r, reword <commit> = use commit, but edit the commit message //괜찮지만 메시지 변경하겠다 
  # e, edit <commit> = use commit, but stop for amending //커밋을 쓸건데 변경사항을 바꾸겠다
  # s, squash <commit> = use commit, but meld into previous //commit 여러 commit을 하나로 묶어 주는것
  # f, fixup <commit> = like "squash", but discard this commit's log message //squash와 비슷하나 메시지를 남기지 않음
  # x, exec <command> = run command (the rest of the line) using shell // (잘안씀) shell 명령어를 쓰고 싶을때
  # b, break = stop here (continue rebase later with 'git rebase --continue') // 
  # d, drop <commit> = remove commit //해당 commit 제거시 사용 
  # l, label <label> = label current HEAD with a name
  # t, reset <label> = reset HEAD to a label
  # m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
  # .       create a merge commit using the original merge commit's
  # .       message (or the oneline, if no original merge commit was
  # .       specified). Use -c <commit> to reword the commit message.
  #
  # These lines can be re-ordered; they are executed from top to bottom.
  #
  # If you remove a line here THAT COMMIT WILL BE LOST.
  #
  # However, if you remove everything, the rebase will be aborted.
  #

# WIP 커밋 메시지 변경위해 r 옵션으로 변경후 저장 > commit 메시지 변경, 저장 하면 되
[detached HEAD b9ff720] Commit message -edited (WIP)
Author: Ellie <dream.coder.ellie@gmail.com>
Date: Sun Nov 1 11:14:00 2020 +0900
1 file changed, 1 insertion(+)
create mode 100644 payment-client.txt
Successfully rebased and updated refs/heads/master.


# git hist 
* [2020-11-01] [ca528df] | Add payment UI {{Ellie}}  (HEAD -> master)
* [2020-11-01] [2fb9760] | . {{Ellie}}
* [2020-11-01] [6ba70c7] | Add payment client {{Ellie}}
* [2020-11-01] [b9ff720] | Commit message -edited (WIP) {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}
```


### 필요없는 커밋 삭제 
- git undo 예제 사용 

2fb9760 해당하는 commit을 삭제해본다 
```bash
$ git hist 
* [2020-11-01] [ca528df] | Add payment UI {{Ellie}}  (HEAD -> master)
* [2020-11-01] [2fb9760] | . {{Ellie}}      # 삭제 대상 
* [2020-11-01] [6ba70c7] | Add payment client {{Ellie}}
* [2020-11-01] [b9ff720] | Commit message -edited (WIP) {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

$ git rebase -i HEAD~2 
# 편집창 뜨면 해당 commit hash 코드 옵션을 pick 아닌 d ( drop : commit 삭제 ) 변경후 저장 

error: could not apply ca528df... Add payment UI
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort"
.
Could not apply ca528df... Add payment UI

// 여기서 충돌났다 함 
// drop한 commit에서 지워진 파일이 이어지는 다음 commit에서 그 파일이 수정됬다는 의미 
CONFLICT (modify/delete): payment-ui.txt deleted in HEAD and modified in ca528df
(Add payment UI). Version ca528df (Add payment UI) of payment-ui.txt left in tr
ee. 

$ git status
  interactive rebase in progress; onto 6ba70c7
  Last commands done (2 commands done):
	drop 2fb9760 .
	pick ca528df Add payment UI
  No commands remaining.
  You are currently rebasing branch 'master' on '6ba70c7'.
	(fix conflicts and then run "git rebase --continue")
	(use "git rebase --skip" to skip this patch)
	(use "git rebase --abort" to check out the original branch)

  Unmerged paths:
	(use "git restore --staged <file>..." to unstage)
	(use "git add/rm <file>..." as appropriate to mark resolution)
		  deleted by us:   payment-ui.txt

  no changes added to commit (use "git add" and/or "git commit -a")

# 그냥 해당파일 쓰라고 하면 
$ git add .
$ git status 
$ git rebase --continue # 메시지 입력하는 창뜸 , 수정하거나 말거나 한 뒤 저장 

$ git hist      //해당 2fb9760 커밋이 사라진 것을 확인가능 !
* [2020-11-01] [d83fdc9] | Add payment UI(rebase) {{Ellie}}  (HEAD -> master)
* [2020-11-01] [6ba70c7] | Add payment client {{Ellie}}
* [2020-11-01] [b9ff720] | Commit message -edited (WIP) {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

```


### 코끼리 커밋 분할하기 
- git undo 그대로 사용

>[!tip] commit 단위에 대한 유용한 상식
>- 보통 새로추가되는 dependency를 하나의 commit으로 하는것이 바람직하다
>- 나중에 dependency 제거하거나 해당 버전이 안맞으면 그 부분만 revert 할 수 있기 때문이다.
>- 그래서 commit 하나에는 한가지만 하는게 좋다
>	- 한 가지 기능 추가
>	- 한 가지 라이브러리 추가
>	- 한 가지 버그 수정


`98955fc` 커밋을 두가지로 나눠보기 ( 라이브러리 추가, 서비스 추가로 구분)
```bash
$ git hist     
* [2020-11-01] [d83fdc9] | Add payment UI(rebase) {{Ellie}}  (HEAD -> master)
* [2020-11-01] [6ba70c7] | Add payment client {{Ellie}}
* [2020-11-01] [b9ff720] | Commit message -edited (WIP) {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}} # 이 커밋을 두가지로 나눌거임 
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}


$ git rebase -i 707de7d  # 98955fc 이전의 707de7d 선택해줌 
pick 98955fc Add payment library and Add payment service >> pcik을 e로 수정
pick b9ff720 Commit message -edited (WIP)
pick 6ba70c7 Add payment client
pick d83fdc9 Add payment UI(rebase)

# Rebase 98955fc..d83fdc9 onto 98955fc (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#

$ git hist // HEAD가 수정 원하는 commit에 멈춰 있음 
* [2020-11-01] [d83fdc9] | Add payment UI(rebase) {{Ellie}}  (master)
* [2020-11-01] [6ba70c7] | Add payment client {{Ellie}}
* [2020-11-01] [b9ff720] | Commit message -edited (WIP) {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}
  (HEAD) 
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

# 우선 commit을 다시 나의 working dir 로 가져와야함  
# reset의 --mixed ( default ) 사용함 ( point )
$ git reset HEAD~1 //이전 commit으로 돌아가면서 수정파일을 working dir로 가져옴 
$ git hist
* [2020-11-01] [d83fdc9] | Add payment UI(rebase) {{Ellie}}  (master)
* [2020-11-01] [6ba70c7] | Add payment client {{Ellie}}
* [2020-11-01] [b9ff720] | Commit message -edited (WIP) {{Ellie}}
* [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}

* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}  (HEAD)
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

$ git status        // 수정된 파일 package.json, 신규생성된 payment-service.txt 확인됨
$ git add package.json 
$ git status 
$ git commit -m "Add payment lib"
$ git hist
* [2021-01-02] [b688176] | Add payment lib {{leejinwoo}}  (HEAD) // 신규 생성됨 
| * [2020-11-01] [d83fdc9] | Add payment UI(rebase) {{Ellie}}  (master)
| * [2020-11-01] [6ba70c7] | Add payment client {{Ellie}}
| * [2020-11-01] [b9ff720] | Commit message -edited (WIP) {{Ellie}}
| * [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}
|/
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

# 마찬가지 남은 payment-service.txt를 커밋해주면 됨 
$ git add .
$ git commit -m "Add payment-service"
$ git hist
* [2021-01-02] [3a74f69] | Add payment-service {{leejinwoo}}  (HEAD) //신규 생성됨 
* [2021-01-02] [b688176] | Add payment lib {{leejinwoo}}
| * [2020-11-01] [d83fdc9] | Add payment UI(rebase) {{Ellie}}  (master)
| * [2020-11-01] [6ba70c7] | Add payment client {{Ellie}}
| * [2020-11-01] [b9ff720] | Commit message -edited (WIP) {{Ellie}}
| * [2020-11-01] [98955fc] | Add payment library and Add payment service {{Ellie}}
|/
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}

//맘에 든다면 
$ git rebase --continue
$ git hist //결과확인 
* [2020-11-01] [07d7784] | Add payment UI(rebase) {{Ellie}}  (HEAD -> master)
* [2020-11-01] [dd3dfe9] | Add payment client {{Ellie}}
* [2020-11-01] [17fb64e] | Commit message -edited (WIP) {{Ellie}}
* [2021-01-02] [3a74f69] | Add payment-service {{leejinwoo}}
* [2021-01-02] [b688176] | Add payment lib {{leejinwoo}}
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}} 

```


### 여러 개 커밋 합치기 
- git undo 예제 사용

```bash
$ git hist
* [2020-11-01] [07d7784] | Add payment UI(rebase) {{Ellie}}  (HEAD -> master)
* [2020-11-01] [dd3dfe9] | Add payment client {{Ellie}}

#  4개를 하나의 커밋으로 만듦 > Add payment service로 만들어 봄  ( 테스트니 )
* [2020-11-01] [17fb64e] | Commit message -edited (WIP) {{Ellie}}
* [2021-01-02] [3a74f69] | Add payment-service {{leejinwoo}}
* [2021-01-02] [b688176] | Add payment lib {{leejinwoo}}
* [2020-11-01] [707de7d] | Setup Dependencies {{Ellie}}

* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}


$ git rebase -i 20ee16f

pick 707de7d Setup Dependencies   # 대표적인건 pick으로 두고 합칠걸 s(squash)한다
s b688176 Add payment lib
s 3a74f69 Add payment-service
s 17fb64e Commit message -edited (WIP)
pick dd3dfe9 Add payment client
pick 07d7784 Add payment UI(rebase)

# Rebase 20ee16f..07d7784 onto 20ee16f (6 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#

# commit message edit 창 뜸 
# Add payment-service 남겨두고 종료 
$ git hist  # 뒤에 커밋들 해쉬코드가 변경된걸 확인가능 함 !
* [2020-11-01] [ed2c9ac] | Add payment UI(rebase) {{Ellie}}  (HEAD -> master)
* [2020-11-01] [fb4d5f3] | Add payment client {{Ellie}}
* [2020-11-01] [93a121b] | Add payment-service {{Ellie}}
* [2020-11-01] [20ee16f] | Initialise Project {{Ellie}}
```

---
## GitHub
https://github.com/ljw1126

- 로컬에서만 작업시 컴퓨터 문제생기면 최악의 경우 history를 모두 잃게 됨 
- GitHub와 같은 remote 서버에 올려 안전성과 접근성을 높이고, 팀원간의 협업을 이어갈 수 있음
- GitHub를 통해 각 개발자들이 history를 가지고 있음으로서 서로 공유가능하고, 문제 발생시 복원가능 , 오프라인에서 작업가능해 지는 장점이 있다

>[!note]
>- clone : remote > local 로 프로젝트 전체 다운로드
>- push : local > remote 로 history 업로드
>- pull : remote > local 로 history 다운로드


**깃허브 포로젝트를 내 PC에 가져오기**
```bash
# <>Code 탭에서 [Code]버튼 누르면 clone 주소 확인가능 (다운로드도 가능)

# 서버 정보 확인 
$ git remote 
$ git remote -v           //origin이 가르키는 정보 상세 확인가능 

# 새로운 remote 정보 추가 
$ git remote add server 다른링크     //server라는 이름으로 추가함  
$ git remote -v 

# 설정 정보 조회 
$ git remote show       //remote 간략 목록 
$ git remote show origin   //origin 에 대한 상세 정보 출력

```

**로컬 커밋을 서버에  저장하기**
```bash
# 사용자 이름과 이메일을 깃허브에 사용하는 것과 동일해야지 git hub 올라갈때 혼동 안됨 
$ git config --global -e 

$ echo add > add.txt
$ git add .
$ git commit -m "Add new file"
$ git hist         // local이 origin 보다 HEAD가 앞서게 됨 
$ git push         // git hub 계정 정보를 작성입력하면 push 됨 
```

**SSH키 등록해서 간편하게 push 하기**
- 매번 remote 서버에 push 할 때 id, pwd 적기 귀찮음 
- 이때 ssh키를 등록하면 로그인할 필요없이 이력을 서버에 올리기 가능 
- 서버에는 public key 생성하고 , local에는 privite key 생성해서 사용함
```bash
1. git hub 사용자 프로필 누르면 [Settings] 선택 
2. 왼쪽 [SSH and GPG keys] 메뉴 선택 
3. 'generating SSH keys' 가이드 페이지 설명대로 해보기 

$ ssh-keygen -t ed25519 -C "이메일주소"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/c/Users/wkrdm/.ssh/id_ed25519):
Created directory '/c/Users/wkrdm/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/wkrdm/.ssh/id_ed25519
Your public key has been saved in /c/Users/wkrdm/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:sUoU7N6lAADflsVRxSkijGDmN7h+vbLEtymGUdAe2tY zral1004@gmail.com
The key's randomart image is:
+--[ED25519 256]--+
|o=o+ o+o.o..     |
|+oo++o+.. o      |
| o*o*+....       |
| .o*.Eo  o.      |
| .o  ..oSo       |
|... ....o        |
| .o+ o.          |
| .o+. +          |
|  ..++           |
+----[SHA256]-----+

※ .ssh 폴더에 id_ed25519(private key) , id_ed25519.pub( 공개키 , 깃허브 등록용 ) 파일 생성됨 

4. Adding a new SSH key to your GitHub account 가이드 페이지 내용대로 
생성한 public key 복사해서 아까 [SSH and GPG keys]에 'New SSH key' 눌러서 추가하면 됨 
> /c/Users/wkrdm/.ssh/id_ed25519 있는 id_ed25519.pub파일을 editor로 열어서 복붙하면 됨 ( 요 파일에 내용을 git hub 에 복붙 )       

5. 그리고 git push ( 유용함 )

```


>[!warning] ssh로 push/pull하고 싶은 경우 remote origin 주소가 http(s) 형식으로 잡혀 있는 경우 안된다
>- 예) https://github.com/ljw-zral1004/study.git  >> git@github.com:ljw-zral1004/study.git
>- 이때는 뒤의 주소 형식으로 remote 주소 정보를 변경해줘야 한다
>- 참고. https://goddaehee.tistory.com/254


```bash
# .ssh 폴더 확인 
$ ls -al ~/.ssh   //폴더 없는 경우 생성하고 권한을 700 주면 됨 

# ssh-agent 실행여부 확인 
$ eval "$(ssh-agent -s)"
Agent pid 806     // 이런 출력이 뜨면 정상동작 중 

# ssh-agent 에 ssh key 등록하기
$ ssh-add ~/.ssh/id_ed25519  //비밀키 파일을 등록하는 듯 함 

# 생성한 키 확인 방법   // 그냥 에디터로 파일 읽어도 됨 ( id_ed25519.pub )
$ cat(또는 vi) ~/.ssh/id_rsa.pub
$ bcopy < ~/.ssh/id_rsa.pub

# git hub 접속해서 , [프로필 아이콘] 클릭 > Settings 클릭 > SSH and GPG keys 클릭 > [New SSH key] 클릭 > id_ed25519.pub에 있는 공개키 등록 
$ git push/pull 정상동작 확인 
```


>[!warning] git push -f 옵션을 사용하면 remote 서버에 작업한 내용은 지워지고 로컬 history로 대체됨
>- 정말 강제하는 경우가 아니라면 사용해서는 안된다
>- git pull , fetch로 local history를 remote에 맞춘 다음 나의 commit들을 rebase 해서 그다음 push 하는 게 맞음 
>- 간혹 기존의 git history를 rebase를 이용해서 변경하거나 history를 변경했다면 부득이하게 옵션 -f 로 push 해야함


**로컬 프로젝트에 깃허브 프로젝트 연결하기**
```bash
$ git init   // .git 있는 경우 생략 

$ git remote // 목록 없음 

$ git remote add origin 저장소주소 

$ git remote -v // origin 추가된거 확인됨 

$ git push  
```


### fetch vs pull  차이
- fetch의 경우 remote의 최신 이력을 받아오지만 head는 여전히 로컬 히스토리를 가르킨다
- pull의 경우 remote의 최신 이력을 받아 head의 포인터가 최신 이력으로 이동한다
```text
아래의 history 상황에서 
| local |       [remote server]          
 a <- b           a <- b <-c
    
fetch의 경우 remote의 history를 가져오지만 local HEAD는 여전히 b 를 가르키게 됨 
a <- b <- c
		 origin/master 
   master(HAED)      

pull의 경우 remote의 history를 가져오면서 local 내용과 함께 merge를 하게 됨 
a <- b <- c 
	  origin/master 
	  master(HAED)
```


### fetch 심화
```bash
# remote 서버(git hub)에 수동으로 신규 커밋 생성 후 
----------------------------------------------------------
$ git fetch 

origin/main, origin/HEAD는 최신 commit을 가르키고 있지만 
local HEAD는 작업중인 이력에 위치하게 됨 

$ git fetch 서버명 
$ git fetch origin 브랜치명 
----------------------------------------------------------
```


### pull 심화
```bash
# remote 서버에 최신 버전과 local 버전을 똑같이 맞춰줌 ( merge 하면서 최신 commit으로 head 옮겨짐 ) 
----------------------------------------------------------
$ git pull      // 로컬과 서버가 동기화(sync) 잘된 걸 확인가능 
----------------------------------------------------------

# local과 remote 서로 새로운 commit 있는 경우 , 또는 동일한 파일을 수정한 경우 
----------------------------------------------------------
$ git pull // merge conflict 발생함 ( 브랜치 사이나 서버와 로컬사이)
$ git mergetool      // global config 설정한거 
$ git add .
$ git merge --continue       // 새로운 커밋 메시지 만들어짐 ( 메시지 안적어도 됨)
$ git hist                   // merge 이력이 더럽게 생성됨 
$ git reflog                 // 이전 상태로 돌림 
$ git reset --hard 해쉬코드 | git reset --hard HEAD~1    // 로컬 commit 시점으로 돌아감 
//여기서 헷갈릴 수 있는 게 이미 pull 받아서 remote 이력이 있는거고, 로컬 이력이 reset 된거라 history 모양이 다른걸 확인가능             


$ git pull --rebase          // 서버에 있는 commit 가져와서 로컬에 만든 커밋 위에 적용할거임 
$ git mergetool              // conflict 내용수정 후 종료 
$ git rebase --continue      // commit message 창 뜸 ( 그냥 종료 )
$ git hist                   // 깔끔하게 정리됨 , rebase 한거만 새로운 commit 만들어지고 서버에서 받은 커밋까지는 코드 그대로 인걸 확인가능
$ git push                           
---------------------------------------------------------- 
```


### blame
- vscode에 git lens extension으로도 확인 가능 (현업에서도 유용)
- sourcetree 확인시 이력에서 원하는 파일 마우스 오른쪽 클릭 > annotate selected 누르면 확인가능 

```bash
$ git blame 경로/파일명 
```


### bisect
- 디버깅만으로 버그를 찾지 못하여, 커밋 단위로 검사를 해야 할 때 유용
- bisect는 이진 탐색 알고리즘 통해서 commit을 검사함 
- 수많은 commit 중 잘 동작한 commit 포인트와 이상한 commit 포인트만 잘 설정해두면 이진탐색 알고리즘 이용해서 빠르게 나쁜 commit을 잡아내는 명령어 

```bash
----------------------------------------------------------
$ git checkout 원하는커밋이동 
$ git bisect start   //프로그램 실행함  

# <B> 라는 심볼이 생성되면 
$ git bisect good     //마크 생성해둠 
$ git checkout master //최신버전에서 문제가 발생하는지 확인하기 위해 
$ git bisect bad      // 문제 발생시 마크 bad 생성해둠
  그러면 bisecting 2진탐색알고리즘 시작함 

$ git hist    //HEAD가 움직여져 잇음, 해당 HEAD 에서 이슈 발생하는지 확인후 
$ git bisect good  // 마크해주면 다음 commit으로 이동함 
$ git hist         // HEAD가 또 움직여짐 , 프로그램 돌려서 문제 발생하는지 확인 
$ git bisect good  // 괜찮다는 mark 찍음 
$ git hist 
$ git bisect bad   // 문제발생시 bad mark 찍음 

   그러면 good commit과 bad commit 중간 지점에 원인이 있으니 다시 checkout 됨 

$ git hist 
$ git bisect good        //

$ git bisect reset       //원래 브랜치로 돌아간다함 ( = 원래 이력포인트로 이동하는 듯)
----------------------------------------------------------
```


>[!tip] 터미널 UI 인터페이스 툴 tig
>- https://jonas.github.io/tig/INSTALL.html
>- tig status 후 h 누르면 단축키 확인가능 
>- vim 명령어처럼 '/검색어'형식으로 검색 가능


>[!tip] 깃 설정 공유
```bash
# local 내용 초기화 하고 싶은 경우 보통 reset 활용
$ git reset --hard HEAD

$ git config --global -e

[alias]  # git 다음 alias 쓰면 됨  
	 s = status 
	 # up은 마스터를 최신으로 
	 up = !git fetch origin master && git rebase origin/master
	 co = checkout 
	 ca = !git add -A && git commit -m 
	 cad = !git add -A && git commit -m "."
	 c = commit
	 b = branch 
	 slist = stash list 
	 ssave = stash save 
	 spop = stash pop 
	 apply = stash apply 
	 rc = rebase -continue
```


---
## Reference.
- 강의 : https://academy.dream-coding.com/courses/git
- 깃 공식 : https://git-scm.com/
- 필수 리눅스 명령어 (유튜브) : https://www.youtube.com/watch?v=EL6AQl-e3AQ
- iterm2 setup guide : https://gist.github.com/kevin-smets/8568070'
- gitignore : https://www.toptal.com/developers/gitignore

