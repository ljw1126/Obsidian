- mermaid 사용
- 최대 3개 지원


`flowchart` : 작업 절차 설명 
```text
---
config:
  theme: mc
---
flowchart LR
    A[단위 테스트 기반 <br/> 탐지 기능 구현] --> B[객체/레이어 <br/> 아키텍처 설계]
    B --> C[응답 포맷 기능 구현]
    C --> D[DI / 전체 연결 구성]
    D --> E[**result.json** <br/>결과 확인]

```


`sequenceDiagram` : 웹 API 요청 처리 
```text
sequenceDiagram
    autonumber

    actor client
    participant LogController

    box 로그 파일 처리
        participant LogAnalyzer
        participant FileReader
        participant LogAnalysisService
    end 

    client->>LogController: [POST] <br/> /api/v1/log/analysis-report
    LogController->>LogAnalyzer: analyze()
    activate LogAnalyzer
    LogAnalyzer->>FileReader: readAll()
    FileReader-->>LogAnalyzer: json 파일 읽어 전달(LogFile[])
    LogAnalyzer->>LogAnalysisService: analyze(files)
    LogAnalysisService-->>LogAnalyzer: AnalysisReport 생성·전달
    LogAnalyzer-->> LogController:
    deactivate LogAnalyzer
    LogController-->> client: result.json 생성<br/>201 created 


```


`classdiagram` : 도메인 서비스 
```text
---
config:
  theme: neo
---
classDiagram
direction TB
    class MissingSequenceDetector {
	    -
	    +detect(file: LogFile) MissingSequenceAnalysisResult
    }
    class DuplicatedLogDetector {
	    +IDuplicationStrategy~IdenticalLogResult~ identiclaLogStrategy
	    +IDuplicationStrategy~SequenceDuplicateResult~ sequenceDuplicateStrategy
	    +IDuplicationStrategy~EventDuplicateResult~ eventDuplicateStrategy
	    +detect(file: LogFile) DuplicationAnalysisResult
    }
    class WrongLocationDetector {
	    -
	    +detect(file: LogFile) WrongLocationAnalysisResult
    }
    class UpdateNumberDetector {
	    +IUpdatedNumberStrategy~AddZeroResult~ addZeroStrategy
	    +IUpdatedNumberStrategy~UpdateZeroResult~ updateZeroStrategy
	    +IUpdatedNumberStrategy~MissingUpdateResult~ missingUpdateStrategy
	    +detect(file: LogFile) UpdateNumberAnalysisResult
    }
    class LogAnalysisService {
	    +MissingSequenceDetector missingSequenceDetector
	    +DuplicatedLogDetector duplicatedLogDetector
	    +WrongLocationDetector wrongLocationDetector
	    +UpdateNumberDetector updateNumberDetector
	    +analyze(files: LogFile[]) AnaylisisResult
    }
    class IdenticalLogStrategy {
	    -
	    + detect(file: LogFile) IdenticalLogResult[]
    }
    class SequenceDuplicateStrategy {
	    -
	    + detect(file: LogFile) SequenceDuplicateResult[]
    }
    class EventDuplicateStrategy {
	    -
	    + detect(file: LogFile) EventDuplicateResult[]
    }
    class IUpdatedNumberStrategy~T~ {
	    detect(file: LogFile) T[]
    }
    class AddZeroStrategy {
	    -
	    + detect(file: LogFile) AddZeroResult[]
    }
    class UpdateZeroStrategy {
	    -
	    + detect(file: LogFile) UpdateZeroResult[]
    }
    class MissingUpdateStrategy {
	    -
	    + detect(file: LogFile) MissingUpdateResult[]
    }
    class IDuplicationStrategy~T~ {
	    detect(file: LogFile) T[]
    }

	  <<interface>> IUpdatedNumberStrategy
	  <<interface>> IDuplicationStrategy

    LogAnalysisService ..> MissingSequenceDetector
    LogAnalysisService ..> DuplicatedLogDetector
    LogAnalysisService ..> WrongLocationDetector
    LogAnalysisService ..> UpdateNumberDetector
    DuplicatedLogDetector ..> IDuplicationStrategy
    IDuplicationStrategy <|-- IdenticalLogStrategy
    IDuplicationStrategy <|-- SequenceDuplicateStrategy
    IDuplicationStrategy <|-- EventDuplicateStrategy
    UpdateNumberDetector ..> IUpdatedNumberStrategy
    IUpdatedNumberStrategy <|-- AddZeroStrategy
    IUpdatedNumberStrategy <|-- UpdateZeroStrategy
    IUpdatedNumberStrategy <|-- MissingUpdateStrategy

```