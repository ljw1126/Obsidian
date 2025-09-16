> [!note] 깃을 사용하면서 겪은 상황에 대한 해결책을 기록


### branch

**상황**(`250302`)
- 브랜치를 분리하지 않고 `master`에 커밋 이력이 쌓임
- `ch07` 브랜치에 커밋 이력을 옮기고 `master` 이력을 제거하고 싶음

**방법**
```shell
# 시작 커밋을 기점으로 브랜치를 생성 
git checkout -b ch07 6695a65

# ch07 브랜치로 체크아웃하고 master merge
git merge master

# 로컬 master 이력을 리모트 이력과 동일하기 만들기
git reset --hard origin/master
```

이외 cherry pick 사용하여 from ~ to까지 이력을 새 브랜치로 옮길 수 있다
```shell
# A부터 B까지의 모든 커밋을 가져오기 
git cherry-pick A^..B
```


### merge
>[!info] merge시 생성되는 orig 확장자 파일이 생성되지 않도록 비활성화

```shell
$ git config --global mergetool.keepBackup false
```


### rebase

**상황**(`250509`)
- 접속 대기열 시스템의 커밋 이력 정리 중에 **순서가 바뀐 커밋** 발견
	- `00208e6`가 `9c3e567`보다 먼저 나오길 원함

```text
* [2025-05-09] [00208e6] | feat: AllowedResponse 레코드 추가 {{leejinwoo}}  (master)
* [2025-05-09] [9c3e567] | docs: JMeter 파일 및 가이드 문서 추가 {{leejinwoo}} 
* [2025-05-09] [e7f89c2] | feat: 메인 페이지, 대기열 페이지 수정 {{leejinwoo}} 
* // 이하 생략
```


`reset --soft` 사용할 수도 있지만, rebase를 통해 하는 방법을 알게 됨

```shell
$ git rebase -i HEAD~2
```

편집창에서 순서를 바꿔주면 된다✨
```text
// before
pick 00208e6 커밋 A
pick 9c3e567 커밋 B

// after
pick 9c3e567 커밋 B
pick 00208e6 커밋 A
```


### commit
- [커밋 날짜 및 시간 변경](https://blacklobster.tistory.com/17)

```text
// 하루 전 날짜로 커밋
git commit --date "1 day ago" -m "커밋 메시지"

// 특정 날짜, 시간 정해서 커밋 
git commit --date "Thu 30 Mar 2023 10:00:00 KST" -m "커밋 메시지"

// 바로 최근 커밋에 날짜 변경하기 (이때 뒤에 -m 옵션 줘서 커밋 메시지 적으면 메시지도 변경)
git commit --amend --no-edit --date "1 day ago"

or

git commit --amend --no-edit --date "Thu 30 Mar 2023 10:10:00 KST"

// 특정 커밋의 날짜 변경 (생략)

```