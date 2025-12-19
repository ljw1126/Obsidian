
**Redis 컨테이너 생성**
```shell
$ docker run --name local-redis -p 6379:6379 -d redis
```

**컨테이너 접속** 
```shell
$ docker exec -it local-redis /bin/bash

$ redis-cli
$ monitor
```
- 포트가 다른 경우는 추가 옵션 필요

**local.settings.json 설정**
```text
"Redis:ConnectionString": "localhost:6379,syncTimeout=5000,abortConnect=false,allowAdmin=true"
```
- 테스트시 데이터 초기화를 위해 flush가 필요하여 옵션 추가

