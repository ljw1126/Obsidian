
[microsoft/mssql-server - Docker Image | Docker Hub](https://hub.docker.com/r/microsoft/mssql-server/)

```shell
docker run -p 1433:1433 --name sqlserver -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=qwer789&*(" -e "MSSQL_PID=Developer" mcr.microsoft.com/mssql/server:2022-latest
```


`컨테이너 및 sqlserver 접속`
```shell
$ docker exec -it sqlserver /bin/bash

mssql@d95d75f18f8f:/$ /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'qwer123@' -C
1> SELECT DB_NAME();
2> GO

--------------------------------------------------------------------------------------------------------------------------------
master
```


`Db/계정 생성, 권한 할당`
```shell
CREATE DATABASE TestManagerDb;
GO

-- 1. 로그인 (SQL Server 인증 사용자) 생성
CREATE LOGIN Tester WITH PASSWORD = 'qwer789&*('
GO

-- 2. 데이터베이스 'TestManagerDb'에 사용자 매핑
USE TestManagerDb;
CREATE USER Tester FOR LOGIN Tester;

-- 3. 사용자에게 'db_datareader' 및 'db_datawriter' 권한 부여 (CRUD 권한)
EXEC sp_addrolemember 'db_datareader', 'Tester'; 
EXEC sp_addrolemember 'db_datawriter', 'Tester'; 
EXEC sp_addrolemember 'db_ddladmin', 'Tester';  // 테이블 생성 위한 권한
GO

-- 4. sqlcmd 종료 
EXIT
```


`데이터베이스` 확인
```shell
1> SELECT name FROM sys.databases;
2> GO
name
--------------------------------------------------------------------------------------------------------------------------------
master
tempdb
model
msdb
TestManagerDb
```


`테이블` 확인
```shell
USE [DB_이름]; -- 사용할 DB 지정 (예: USE TestManagerDb;)
SELECT name FROM sys.tables;
GO
```

> **요약하자면,** MSSQL에서는 `SHOW` 명령 대신 **`SELECT` 문과 시스템 뷰**를 사용하여 메타데이터를 조회합니다.


