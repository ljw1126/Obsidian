
**로깅 프레임워크**
- `Slf4j`는 인터페이스
	- **구현체** : `Logback`, `Log4j`, `Log4j2`, `java.util.logging` 등

>[!note] Portable Service Abstraction (PSA)
>- @Slf4j 와 같이 @Transactional도 대표적인 사례


📁`resources/logback.xml`
```xml
<configuration>  
    <property name="LOG_FILE" value="application-dev.log"/>  
  
    <!-- 콘솔 출력 -->  
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">  
        <encoder>  
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{36} - %msg%n</pattern>  
        </encoder>  
    </appender>  
  
    <!-- 파일 출력 -->  
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">  
        <file>${LOG_FILE}</file>  
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">  
            <fileNamePattern>application.%d{yyyy-MM-dd_HH-mm}.log.gz</fileNamePattern>  
            <maxHistory>5</maxHistory>  
        </rollingPolicy>  
        <encoder>  
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{36} - %msg%n</pattern>  
        </encoder>  
    </appender>  
  
    <!-- Logger 설정 -->  
    <root level="info">  
        <appender-ref ref="CONSOLE" />  
        <appender-ref ref="FILE" />  
    </root>  
</configuration>
```
- **property**
	- 변수처럼 활용될 수 있는 것을 선언
- **appender** (핵심*)
	- CONSOLE, FILE 등으로 로그 나갈지 제어 설정
		- **rollingPolicy** : 시간 단위 정책
		- **maxHistory**: 
			- 분 단위로 롤링 (fileNamePattern 표현식에 의존)
			- 15~20분 로그가 쌓이면 21분 되면 15분 삭제되서 **5**개 유지됨
		- `application.%d{yyyy-MM-dd_HH-mm}.log` 파일이 개별 생성되는데 `application.%d{yyyy-MM-dd_HH-mm}.log.gz` 와 같이 끝에 gz 붙이면 압축되면서 롤링이 된다 
- **root**
	- `level = "info"` 
		- info 레벨 이상의 로그는 아래의 appender 통해 로그 기록 된다

어플리케이션 실행시 xml 파일을 찾았다고 콘솔에 출력이 됨
```text
15:25:38,659 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [logback.xml] at [file:/home/gitRepository/shorten-url-java/build/resources/main/logback.xml]
```


**참고**
- [logback documentation](https://logback.qos.ch/documentation.html)
- [A Guide To Logback - Baeldung](https://www.baeldung.com/logback)

---

**Appender 종류**
- `CONSOLE` ✨
- `FILE` 
	- 하나의 파일에 기록
- `ROLLING_FILE` ✨
	- 로그 파일을 자동으로 롤링(분할)하여 관리한다
- `ASYNC`
	- 비동기적으로 로그를 기록하여 성능을 개선
- `SYSLOG`
	- 시스템 로그
- `SOCKET`
	- 소켓을 통해 로그를 원격 서버로 전송
- EMAIL
	- SMTP, 메일 전송 
- `DB`
	- 데이터베이스에 로그를 기록
- `CUSTOM`
	- 사용자 정의 Appender를 만들어 특정 요구사항에 맞게 로그를 처리한다
---

> Spring profile 별로 logback 설정을 나눌 수 있다 (단, 네이밍으로는 안되고 properties 설정에서 logback 파일 지정을 해줘야한다)
> - application-dev.properties  -> logback-dev.xml 을 따로 지정해줌

```text
// application.properties, 기본 profile 설정
spring.profiles.active=dev

// application-dev.properties
logging.config=classpath:logback-dev.xml

// application-prod.properties
logging.config=classpath:logback-prod.xml
```


`logback.xml`에 설정으로 Profile에 따라 다른 logback 설정 사용 가능하다
- properties에서 할지, logback.xml에 할지 정책을 정하면 될 듯

```xml
<configuration>
	<springProfile name="dev">
		<include resource="logback-dev.xml"/>
	</springProfile>
	
	<springProfile name="prod">
		<include resource="logback-prod.xml"/>
	</springProfile>

	<springProfile name="default">
		<include resource="logback-dev.xml"/>
	</springProfile>
</configuration>
```

---
### 섹션4. 로그 수집

> [!info]
> Logback(로그기록) -> Logstash (수집) -> ElasticSearch (데이터베이스, 저장) <- Kibana(시각화)
- Logback appender 중에서 ElasticSearch로 바로 보내는것도 있다

**로그 수집이 필요한 이유**
- ssh로 직접 서버에 접속해서 로그 확인하는 것은 비효율적인 상황이 발생한다
	- 서비스가 보통 N개를 사용하기 때문에
- logstash 인코더를 사용하면 json으로 포맷 변환해준다 
	- 그리고 elasticsearch 경로 설정해주면 저장을 해준다

 **1. elasticsearch 실행 도커 커맨드**
`docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "xpack.security.http.ssl.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:8.10.0`

**참고.** 맥북 애플 실리콘 (M1, M2 등) 실행시 `--platform linux/amd64` 파라미터도 주셔야합니다. ([링크](https://www.inflearn.com/community/questions/1509617/elasticsearch-logstash-%EC%84%B8%ED%8C%85-%EC%8B%9C-%EC%98%A4%EB%A5%98-%EC%82%AC%ED%95%AD-%EA%B3%B5%EC%9C%A0))

 

**2. logstash 의존성 추가**
`pom.xml`
```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

`build.gradle`
```text
implementation 'net.logstash.logback:logstash-logback-encoder'
```

 **3. logstash Appender 추가 (logback.xml)**
```xml
<configuration>
    <property name="LOG_FILE" value="application.log"/>

    <!-- Logstash로 전송할 Appender -->
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>localhost:5044</destination>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
    </appender>

    <!-- 콘솔 출력 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 파일 출력 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>application.%d{yyyy-MM-dd_HH-mm}.log.gz</fileNamePattern>
            <maxHistory>5</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Logger 설정 -->
    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="FILE" />
        <appender-ref ref="LOGSTASH" />
    </root>
</configuration>
```

 **4. logstash 실행 설정 파일 (logstash.conf)**
```
input {
    tcp {
        port => 5044
        codec => json
    }
}

output {
    elasticsearch {
        hosts => ["http://elasticsearch:9200"]
        index => "application-logs-%{+YYYY.MM.dd}"
    }
}
```


**5. logstash 실행 도커 커맨드 (중간에 .\logstash.conf 에서 백슬래시(\)는 윈도우즈 기준이고 기타 운영체제에선 슬래시로 작성)**
`docker run -d --name logstash -p 5044:5044 -p 9600:9600 -v .\logstash.conf:/usr/share/logstash/pipeline/logstash.conf docker.elastic.co/logstash/logstash:8.10.0`

**6. 도커 컨테이너간 통신을 위한 네트워크 생성**
```
docker network create elastic-network
docker network connect elastic-network elasticsearch
docker network connect elastic-network logstash
```
- 같은 네트워크에 묶여야 통신이 가능

```shell
$ docker network ls # 네트워크 확인
$ docker network inspect elastic-network # 네트워크 내부 컨테이너 확인
```


>[!info]
>- 엘라스틱서치 : http://localhost:9200/_cat/indices



