
### Sorted Sets
- ordered collection(unique strings)
	- ex. leader board, rate limit 기능 구현시 적합
- key score member \[score member...\]
	- key 그룹에 member(element)가 score 기준으로 정렬된다
	- 기본적으로 **score 기준 오름차순 정렬**
	- REV 옵션 통해 내림차순 정렬 조회 가능
- command
	- `ZADD`
	- `ZREM`
	- `ZRANGE`
	- `ZCARD`
	- `ZRANK` / `ZREVRANK`
	- `ZINCRBY`: 스코어를 변경시키면 순위가 바뀔 수 있다

**Redis 컨테이너 접속**
```shell
$ docker exec -uroot -it {레디스 컨테이너명} redis-cli

$ docker exec -uroot -it {레디스 컨테이너명} /bin/sh
$ redis-cli -p 6379 -hlocalhost
```


[ZADD](https://redis.io/docs/latest/commands/zadd/)
```shell
127.0.0.1:6379> ZADD "game1:scores" 100 user1 200 user2 300 user3
3
127.0.0.1:6379> ZADD "game1:scores" 50 user4 150 user5 350 user6
3
```

[ZRANGE](https://redis.io/docs/latest/commands/zrange/)
- [ZRANGEBYSCORE](https://redis.io/docs/latest/commands/zrangebyscore/)는 Redis v6.2.0 부터 depreacted 됨
- 점수 기준으로 오름차순 정렬이 기본 
```shell
// score 기준 오름차순
127.0.0.1:6379> ZRANGE "game1:scores" 0 +inf BYSCORE LIMIT 0 10 WITHSCORES
 1) "user4"
 2) "50"
 3) "user1"
 4) "100"
 5) "user5"
 6) "150"
 7) "user2"
 8) "200"
 9) "user3"
10) "300"
11) "user6"
12) "350"

// score 기준 내림차순
127.0.0.1:6379> ZRANGE "game1:scores" +inf 0 BYSCORE REV LIMIT 0 10 WITHSCORES
 1) "user6"
 2) "350"
 3) "user3"
 4) "300"
 5) "user2"
 6) "200"
 7) "user5"
 8) "150"
 9) "user1"
10) "100"
11) "user4"
12) "50"

```


[ZREM](https://redis.io/docs/latest/commands/zrem/)
- `Removes the specified members from the sorted set stored at `key`. Non existing members are ignored.`
```shell
127.0.0.1:6379> ZREM "game1:scores" user3
1
127.0.0.1:6379> ZREM "game1:scores" user99999
0
127.0.0.1:6379> ZRANGE "game1:scores" +inf 0 BYSCORE REV LIMIT 0 3 WITHSCORES
1) "user6"
2) "350"
3) "user2"
4) "200"
5) "user5"
6) "150"
```

[ZCARD](https://redis.io/docs/latest/commands/zcard/)
- `ZCARD`는 key에 저장된 요소의 전체 개수를 반환
- `ZCOUNT`는 min, max 범위 내에 해당하는 score 요소 개수를 반환함
	- `ZCOUNT key min max`
```shell
127.0.0.1:6379> ZCARD "game1:scores"
5
```


[ZINCRBY](https://redis.io/docs/latest/commands/zincrby/)
- `ZINCRBY key increment member`
- 키 그룹에 소속된 특정 요소의 score를 갱신
```shell
127.0.0.1:6379> ZRANGE "game1:scores" +inf 0 BYSCORE REV LIMIT 0 10 WITHSCORES
 1) "user6"
 2) "350"
 3) "user2"
 4) "200"
 5) "user5"
 6) "150"
 7) "user1"
 8) "100"
 9) "user4"
10) "50"

127.0.0.1:6379> ZINCRBY "game1:scores" 500 "user4"
"550"

127.0.0.1:6379> ZRANGE "game1:scores" +inf 0 BYSCORE REV LIMIT 0 10 WITHSCORES
 1) "user4"
 2) "550"
 3) "user6"
 4) "350"
 5) "user2"
 6) "200"
 7) "user5"
 8) "150"
 9) "user1"
10) "100"
```