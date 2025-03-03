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