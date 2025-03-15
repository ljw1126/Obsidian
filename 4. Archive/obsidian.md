
### Reference.
- [지식정리 끝판왕, 옵시디언: 기초 가이드편](https://wikidocs.net/270322)

### Issue

**OS별로 git conflict가 잦은 문제**
- `.obsidian` 디렉터리에서 os 별로 차이가 발생
- 해당 디렉터리를 깃 이력에서 제거

**OS별로 플러그인 공유가 안되는 이슈**
- `.obsidian/community-plugins.json` 파일을 루트 디렉터리로 이동
- **하드 링크**를 걸어서 연결하는 형태로 해결 
	- 소프트 링크(-s옵션) 사용할 경우 옵시디언에서 인식하지 못하고 플러그인 비활성화된다

```shell
ln community-plugins.json .obsidian/community-plugins.json
```

