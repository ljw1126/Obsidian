다양한 연극을 외주로 받아서 공연하는 극단이 있다고 생각해보자. 공연 요청이 들어오면 연극의 장르과 관객 규모를 기초로 비용을 책정한다. 
- 현재 이 극단은 두 가지 장르`(비극 : tragedy, 희극 : comedy)` 만 공연한다
- 그리고 공연료와 별개로 포인트`(volume credit)`를 지급해서 다음 번 의뢰시 공연로를 할인받을 수 도 있다

**공연할 연극 정보 데이터 예시 (plays.json)** 
```json
{
  "hamlet" : {"name" : "Hamlet", "type" : "tragedy"},
  "as-like" : {"name" : "As You Like It", "type" : "tragedy"},
  "othello" : {"name" : "Othello", "type" : "tragedy"}
}
```

**공연료 청구서에 들어갈 데이터 예시 (invoices.json)**
```json
[
  {
     "customer" : "BigCo",
     "performances" : [
        {
           "playID" : "hamlet",
           "audience" : 55
        },
        {
           "playID" : "as-like",
           "audience" : 35
        },
        {
           "playID" : "othello",
           "audience" : 40
        }
     ]
   }
]
```


### 1.1 예시 코드 비교
공연료 청구서를 출력하는 함수는 아래와 같다

**레거시 코드**
```java
public class Statement {  
  
    public String statement(Invoice invoice, Plays plays) throws Exception {  
        int totalAmount = 0;  
        int volumeCredits = 0;  
        StringBuilder result = new StringBuilder();  
        result.append(String.format("청구 내역 (고객명: %s)", invoice.getCustomer())).append("\n");  
  
        for(Performance performances : invoice.getPerformances()) {  
            Play play = plays.get(performances.getPlayId());  
            int thisAmount = 0;  
  
            switch (play.getType()) {  
                case TRAGEDY :  
                    thisAmount = 40_000;  
                    if(performances.getAudience() > 30) {  
                        thisAmount += 1_000 * (performances.getAudience() - 30);  
                    }  
                    break;  
                case COMEDY :  
                    thisAmount = 30_000;  
                    if(performances.getAudience() > 30) {  
                        thisAmount += 10_000 + 500 * (performances.getAudience() - 20);  
                    }  
                    thisAmount += 300 * performances.getAudience();  
                    break;  
                default :  
                    throw new Exception(String.format("알 수 없는 장르: %s", play.getType()));  
            }  
  
            // 포인트를 적립한다  
            volumeCredits += Math.max(performances.getAudience() - 30, 0);  
  
            // 희극 관객 5명마다 추가 포인트를 제공한다  
            if(play.getType().equals(PlayType.COMEDY)) {  
                volumeCredits += (performances.getAudience() / 5);  
            }  
  
            result.append(String.format("%s: $%d %d석\n", play.getName(), thisAmount / 100, performances.getAudience()));  
            totalAmount += thisAmount;  
        }  
  
        result.append(String.format("총액: $%d\n", totalAmount / 100));  
        result.append(String.format("적립 포인트: %d점\n", volumeCredits));  
        return result.toString();  
    }  
}
```


**리팩토링 결과**
```java
public class Statement {  
  
    public String statement(Invoice invoice, Plays plays) {  
        return renderPlainText(StatementData.create(invoice, plays));  
    }  
  
    private String renderPlainText(StatementData data) {  
        StringBuilder result = new StringBuilder();  
        result.append(String.format("청구 내역 (고객명: %s)", data.getCustomer())).append("\n");  
  
        for(EnrichPerformance performances : data.getEnrichPerformances()) {  
            result.append(String.format("%s: $%d %d석\n", performances.getPlayName(), performances.getAmount() / 100, performances.getAudience()));  
        }  
  
        result.append(String.format("총액: $%d\n", data.getTotalAmount() / 100));  
        result.append(String.format("적립 포인트: %d점\n", data.getTotalVolumeCredits()));  
        return result.toString();  
    }  
  
    public String htmlStatement(Invoice invoice, Plays plays) {  
        return renderHtml(StatementData.create(invoice, plays));  
    }  
  
    private String renderHtml(StatementData data) {  
        StringBuilder result = new StringBuilder();  
        result.append(String.format("<h1>청구 내역 (고객명: %s)</h1>\n", data.getCustomer()));  
  
        result.append("<table>\n");  
        result.append("<tr><th>연극</th><th>좌석 수</th><th>금액</th></tr>\n");  
        for(EnrichPerformance performance : data.getEnrichPerformances()) {  
            result.append(String.format("<tr><td>%s</td><td>%d석</td>", performance.getPlayName(), performance.getAudience()));  
            result.append(String.format("<td>$%d</td></tr>\n", performance.getAmount() / 100));  
        }  
        result.append("</table>\n");  
  
        result.append(String.format("<p>총액: <em>$%d</em></p>\n", data.getTotalAmount() / 100));  
        result.append(String.format("<p>적립 포인트: <em>%d점</em></p>\n", data.getTotalVolumeCredits()));  
        return result.toString();  
    }  
}
```

---
### 1.2 예시 프로그램을 처음 본 나의 소감 

(1년 전의 나)
목적에 맞게 충실하게 기능 구현이 되어 있는 것으로 보였다. 더군다나 코드 양이 많지 않기 때문에 순차적으로 코드를 읽으면 나름 가독성 있게 잘 작성되어 있는 코드로 보이기도 한다 


(1년 후의 나)
하지만 장르가 추가되거나 계산 식이 변경된다면 어떻게 될까? 
내가 생각하기에 처음 `statement(..)`에는 두 가지의 문제가 보였다

SRP 원칙 위반 : `statement(..)`에는 공연료 계산과 출력이라는 두 가지 책임을 가지고 있다. 변경이 되야 하는 이유도 두 가지가 된다. 복잡성이 늘어난다. 클래스는 하나의 책임만을 가져야하고 변경되는 이유도 하나가 되어야 한다. 공연료 계산 로직이 변경되는 경우 출력하는 로직도 변경에 영향을 받을 수 있다. 따라서 이 두 책임을 분리하여 각각의 클래스로 만들어 협력하는 것이 바람직해보인다

OCP 원칙 위반 : swtich문에서 장르가 추가될 경우 기존 코드의 변경이 발생하게 된다. 장르의 수가 늘어갈 수 록 기존 코드의 영향을 주고 가독성도 떨어지고, 복잡성이 늘어나 유지보수하기 힘들어 질 것으로 보인다. 장르별 계산 인터페이스를 정의 하고 다형성을 통해 구현한다면 기존 코드에 변경을 주지 않고 기능 확장을 할 수 있을 것으로 생각된다

이를 해결하기 위해서는
- "공연료 계산"과 "공연료 청구서 출력" 책임을 두 클래스로 분리하고 객체가 협력하는 형태로 변경
- 장르별 인터페이스를 정의하고 다형성을 통해 계산 방법을 
	- 이 경우 장르의 게산이 변경이 발생하더라도 추상화된 인터페이스에 의존하고 있기 때문에 영향이 전파되지 않고, 한 곳만 수정
	- 장르가 추가되더라도 인터페이스 구현체만 추가하면 된다
	- 런타임에 유연하게 장르 조건에 맞는 구현체를 사용함으로써 유연하게 대응가능



