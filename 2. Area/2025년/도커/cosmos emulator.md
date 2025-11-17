### Cosmos Db
[Get started with Azure Cosmos DB for NoSQL using .NET | Azure Docs](https://docs.azure.cn/en-us/cosmos-db/nosql/how-to-dotnet-get-started)
[Use the emulator for development and CI - Azure Cosmos DB | Microsoft Learn](https://learn.microsoft.com/en-us/azure/cosmos-db/how-to-develop-emulator?tabs=docker-linux%2Ccsharp&pivots=api-nosql)
- DB 안에 Container가 있다. 
- 접속시 
	- endpoint, tokenCredential이 필요 (options는 optional)

```cs
// New instance of CosmosClient class using a connection string
using CosmosClient client = new(
    accountEndpoint: Environment.GetEnvironmentVariable("COSMOS_ENDPOINT")!,
    tokenCredential: credential
);
```

> [!note] 공식 문서의 경우 코드 레벨에서 db와 container 생성하는 것을 설명한다


#### 에뮬레이터 설치 
[Use the emulator for development and CI - Azure Cosmos DB | Microsoft Learn](https://learn.microsoft.com/en-us/azure/cosmos-db/how-to-develop-emulator?tabs=docker-linux%2Ccsharp&pivots=api-nosql)

```shell
$ docker pull mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:latest

$ docker images

$ docker run \
    --publish 8081:8081 \
    --publish 10250-10255:10250-10255 \
    --name cosmos-emulator \
    --detach \
    mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:latest
    
$ docker logs cosmos-emulator

// 인증서 가져오기
$ curl --insecure https://localhost:8081/_explorer/emulator.pem > ~/emulatorcert.crt

// 기존 인증서 삭제 (컨테이너 재설치시 필요)
sudo rm /usr/local/share/ca-certificates/emulatorcert.crt

// 인증서 파일을 시스템의 신뢰 저장소 디렉터리로 복사
// 루트 비번은 계정이랑 동일
$ sudo cp ~/emulatorcert.crt /usr/local/share/ca-certificates/

// 시스템 전체의 신뢰할 수 있는 인증서 목록을 갱신
$ sudo update-ca-certificates
```


로컬 접속 
> https://localhost:8081/_explorer/index.html

db와 container 3개 초기화 (db명/container명/partition key)
- ShipDb|LoggerData|`/sk`
- ShipDb|IotHub|`/DeviceID`
- ManagerDb|VessellinkData|`/_partitionKey`

참고. docker-compose 예시
```text
version: '3.8'

services:
  # 1. Cosmos DB 에뮬레이터 서비스
  cosmosdb:
    image: mcr.microsoft.com/cosmosdb/linux/azure-cosmosdb-emulator:latest
    container_name: cosmosdb-emulator
    # 포트 노출 (8081: 메인 엔드포인트, 10250-10255: 데이터 엔드포인트)
    ports:
      - "8081:8081"
      - "10250-10255:10250-10255"
    # 데이터 영속성을 위한 볼륨
    volumes:
      - cosmos-data:/data
  
  # 2. DB 및 컨테이너 초기화 서비스 (별도의 .NET 프로젝트)
  db-initializer:
    # Dockerfile이 있는 초기화 프로젝트의 경로 지정
    build: 
      context: ./cosmos-initializer 
    container_name: cosmosdb-initializer
    
    # Cosmos DB 에뮬레이터가 완전히 뜰 때까지 기다립니다.
    depends_on:
      - cosmosdb
      
    # 환경 변수를 통해 인증 정보를 전달합니다.
    environment:
      # 💡 도커 네트워크에서는 'localhost' 대신 서비스 이름 'cosmosdb'를 사용해야 합니다.
      - CosmosDbEndpoint=https://cosmosdb:8081/
      - CosmosDbKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDUgOfT6pBX42nm/EZWmi2BPSt6FNmF2ZBAyW/XSUNR6nBfWJg==
      
      # 💡 초기화 서비스 코드에서 읽을 사용자 정의 환경 변수
      # 이 변수를 초기화 코드가 파싱하여 여러 DB/컨테이너를 생성합니다.
      # 예: MigrationDB|ShipContainer|/shipKey;LogDB|AuditContainer|/logId
      - InitStructure="MigrationDB|ContainerA|/AKey;LogDB|ContainerB|/BKey" 
      
    # DB 생성 작업 완료 후 컨테이너 자동 종료
    restart: "no"

volumes:
  cosmos-data:
```

> 우선은 수작업으로 웹 콘솔로 접속해서 db와 container를 생성함

참고. [Azure Cosmos DB Gallery](https://azurecosmosdb.github.io/gallery/?tags=example)

