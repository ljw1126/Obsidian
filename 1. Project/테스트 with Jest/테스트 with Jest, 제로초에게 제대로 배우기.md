- `2025-09-08` 구입

## 섹션1. Jest 배워보기

**jest** 
- js + 테스트, 농담이란 뜻
- 페이스북에서 만든 테스트 프레임워크
	- 타입스크립트도 가능
	- Bable / React / Node.js 전부 다 지원
	- Vitest(Vite), Jasmine, Mocha + Sinon + Chai 등의 대체재도 존재

**테스트를 안 하는 이유**
1. 테스트 효과에 대한 불확실성 
2. 귀찮음
3. 스트레스
4. 시간이 오래 걸림

**해야 하는 이유** 
1. 예전에 났던 에러가 또 나는 경우 (`회귀 테스트` 강추✅)
2. 코드가 복잡한데 많이 바꿔야하는 경우
	1. 하나의 코드를 수정했더니 import 한 다른 곳에서 에러가 나는 경우
		1. 이때 변경에 대한 불안감을 해결하는게 `테스트`다
		2. 테스트를 통해 안심하고 수정 가능
		3. <font color="#c00000">테스트가 없으면 런타임에 확인하거나 배포 후 터지고 확인하게 되는데 최악</font>☠️
3. 장기간에 걸쳐 내가 유지보수를 해야 하는 경우 

> 테스트 커버리지보다 쓸모있는 테스트를 작성해야 한다.

**원칙**
- ✅ 테스트는 틈틈이 추가하자 
- ✅ 진짜 도움이 되는 테스트를 작성하자

---

### Jest 설치 및 ESM 세팅

```shell
$ mkdir zerocho-jest

$ npm init -y

$ npm i jest -D
```

window에서 jest 실행시 환경 변수를 추가해서 실행을 해줘야 한다
```shell
$ npm i -D cross-env

$ npx cross-env NODE_OPTIONS="$NODE_OPTIONS --experimental-vm-modules" jest
```
- `package.json`
	- `"type" : "module"` 설정