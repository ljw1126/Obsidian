- 내부적으로 redis, postgresql을 의존하고 있음
-

📌 **참고**
- https://www.youtube.com/watch?v=85LLdgo5XsY
- [Ramhm/sentry-setup: How to setup Sentry.io (open source) server in Docker Compose](https://github.com/Ramhm/sentry-setup)

### docker-compose 파일 작성
```text
Scripts 
- sentry
  - .env
  - docker-compose.yml
```
- 볼륨 마운트 되면 docker-compose.yml 위치에 `./data` 하위 디렉터리가 생성됨
- 💩 주의. 탐색기로 직접 data 디렉터리 전체 지울 경우 docker daemon 이랑 충돌나서 docker-compose 실행이 안되는 이슈 확인
	- 원인이 daemon 내부에서 볼륨에 대한 정보를 관리하는데, 직접 지워서 동기화 이슈가 발생한 것으로 판다
	- (해결) 컴퓨터 재시작하여 해결 

`.env`
- sentry 컨테이너 내부에서 사용하는 환경 변수 파일 

```text
SENTRY_SECRET_KEY=_9_zr3mh(&9t%wxc((!h6w3izs0(18pk2vg-c5dwp^p&p&!pqc
SENTRY_POSTGRES_HOST=sentry-postgres
SENTRY_POSTGRES_PORT=5432
SENTRY_DB_NAME=sentry
SENTRY_DB_USER=sentry
SENTRY_DB_PASSWORD=89PsZXyRStOT2
SENTRY_REDIS_HOST=sentry-redis
SENTRY_REDIS_PORT=6379
```
- docker-compose.yml 보면 sentry 네트워크를 생성해서 사용한다 
	- 그래서 hostname으로 접근시 실제 포트(ex. 6379) 연결해야 한다.
	- 💩 redis의 포트 포워딩을 6380으로 해서 그렇게 작성했더니 연결 에러 발생 !
		- 그래서 6379 로 다시 변경하니 정상 동작
- `SENTRY_SECRET_KEY`의 경우 가이드에 따라 명령어 실행하면 생성되는 코드를 복붙


`docker-compose.yml`
```yml
version: "3.7"

services:
    sentry-redis:
        image: redis:latest
        container_name: sentry-redis
        hostname: sentry-redis
        ports:
            - "6380:6379"
        restart: always
        networks:
            - sentry
        volumes:
            - "./data/sentry/redis/data:/data"

    sentry-postgres:
        image: postgres:16
        container_name: sentry-postgres
        hostname: sentry-postgres
        restart: always
        environment:
            POSTGRES_USER: sentry
            POSTGRES_PASSWORD: 89PsZXyRStOT2
            POSTGRES_DB: sentry
        networks:
            - sentry
        volumes:
	        # latest 사용시 마운트되는 경로가 달라져서 에러 발생 (아래와 같이 수정)
            - "./data/sentry/postgres:/var/lib/postgresql"

    sentry-base:
        image: sentry:latest
        container_name: sentry-base
        hostname: sentry-base
        restart: always
        ports:
            - "9000:9000"
        env_file:
            - .env
        depends_on:
            - sentry-redis
            - sentry-postgres
        networks:
            - sentry
        volumes:
            - "./data/sentry/sentry:/var/lib/sentry/files"
        environment:
            - SENTRY_EVENT_RETENTION_DAYS=25 #7   # Adjust the retention period as needed

    sentry-cron:
        image: sentry:latest
        container_name: sentry-cron
        hostname: sentry-cron
        restart: always
        env_file:
            - .env
        depends_on:
            - sentry-redis
            - sentry-postgres
        command: "sentry run cron"
        networks:
            - sentry
        volumes:
            - "./data/sentry/sentry:/var/lib/sentry/files"

    sentry-worker:
        image: sentry:latest
        container_name: sentry-worker
        hostname: sentry-worker
        restart: always
        env_file:
            - .env
        depends_on:
            - sentry-redis
            - sentry-postgres
        command: "sentry run worker"
        networks:
            - sentry
        volumes:
            - "./data/sentry/sentry:/var/lib/sentry/files"

networks:
    sentry:
        driver: bridge

```


### 실행 

```shell
docker-compose run --rm sentry-base config generate-secret-key
```
- `--rm sentry-base`
	- 실행이 완료된 후 컨테이너를 즉시 삭제
- `config generate-secret-key`
	- 임시 컨테이너가 시작될 때 실행할 실제 명령어
		- `SENTRY_SECRET_KEY`를 생성하여 표준 출력(콘솔)로 출력한다 
		- 이 문자열을 `.env`에 기재하면 된다✅


```shell
docker-compose run --rm sentry-base upgrade
```
- sentry 어플리케이션의 DB 마이그레이션 및 초기화를 실행하라는 의미
- postgresql에 sentry에서 필요한 스키마가 반영된다.

> 실행 중간에 계정 생성 여부 물어보는데, 개인 이메일/비밀번호 추가하여 생성

> docker-compose down 할시 postgresql 볼륨이 사라져서 sentry 콘솔 접속이 되지 않는 이 확인.. 해당 명령어 재실행

```shell
docker-compose up -d
```

> And Server open `http://localhost:9000` ✅


### 프로젝트 생성 
- 기본적으로 internal이 생성되어 있는데 신규 프로젝트를 생성
- 아래와 같은 형태로 가이드를 제공함 

```text
# Using Package Manager
Install-Package Sentry -Version 1.2.0

# Or using .NET Core CLI
dotnet add package Sentry -v 1.2.0

"http://b58a7f3f8c2a4d288c73874a5694c72a@localhost:9000/2"
```


### 트러블 슈팅

`.env`
```text
SENTRY_SECRET_KEY=_9_zr3mh(&9t%wxc((!h6w3izs0(18pk2vg-c5dwp^p&p&!pqc
SENTRY_POSTGRES_HOST=sentry-postgres
SENTRY_POSTGRES_PORT=5432
SENTRY_DB_NAME=sentry
SENTRY_DB_USER=sentry
SENTRY_DB_PASSWORD=89PsZXyRStOT2
SENTRY_REDIS_HOST=sentry-redis
SENTRY_REDIS_PORT=6379
// 아래 내용 추가
SENTRY_USE_REDIS_CLUSTER=false
SENTRY_CACHE_URL=redis://sentry-redis:6379/0
SENTRY_CELERY_BROKER_URL=redis://sentry-redis:6379/1
SENTRY_RATE_LIMITER_URL=redis://sentry-redis:6379/2
SENTRY_DSN_DEFAULT=redis://sentry-redis:6379/3
```


#### UTC 
- 웹 콘솔 > 사용자 프로필 > Users Settings > Timezone 에서 설정 
	- 시간대별로 나열 되어 있다 
	- `(UTC+0900) Asia/Seoul`
- 이벤트가 저장되는 방식(UTC)를 변경하는게 아니라, 웹 브라우저 콘솔에 표시되는 방식만 변경함 
	- sentry는 데이터 무결성을 위해 서버 내부에서는 여전히 UTC를 사용


#### 재시작 

```shell
docker-compose down 

docker-compose run --rm sentry-base upgrade

docker-compose up -d
```


#### 403 forbidden
timer trigger에서 일부에 예외 발생시켜 sentry 메시지가 가는지 확인하려고 함

[Azure Functions | Sentry for Azure Functions](https://docs.sentry.io/platforms/dotnet/guides/azure-functions-worker/)
[Access to localhost was denied | You don't have authorization to view this page. HTTP ERROR 403 | Sentry](https://sentry.io/answers/access-to-localhost-was-denied-you-don-t-have-authorization-to-view-this-page-http-error-403/)
[getsentry/sentry-javascript: Official Sentry SDKs for JavaScript](https://github.com/getsentry/sentry-javascript)
[🚨 [Sentry] CSRF 인증에 실패했습니다.](https://velog.io/@yakc0202/Sentry-CSRF-%EC%9D%B8%EC%A6%9D%EC%97%90-%EC%8B%A4%ED%8C%A8%ED%96%88%EC%8A%B5%EB%8B%88%EB%8B%A4)

---

