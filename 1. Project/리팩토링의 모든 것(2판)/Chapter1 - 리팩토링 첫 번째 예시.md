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


### 1. 예시 코드
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
### 2. 예시 프로그램을 처음 본 나의 소감 

처음 예시를 봤을 때 목적에 맞게 충실하게 기능 구현이 되어 있는 것으로 보였다 
더군다나 코드 양이 많지 않기 때문에 순차적으로 코드를 읽으면 나름 가독성 있어 보였다

<img src = "https://raw.githubusercontent.com/ljw1126/user-content/refs/heads/master/meme/%EA%B7%B8%EB%9F%B4%EC%8B%B8%ED%95%9C%EB%8D%B0.webp"/>


하지만 장르가 추가되거나 계산식이 변경된다면 어떻게 될까? 
내가 생각하기에 `statement(..)`에는 두 가지의 문제가 보였다

**1️⃣ SRP(단일 책임 원칙) 위반**
- `statement(..)`에는 **공연료 계산**과 **공연료 청구서**라는 두 가지 책임을 가지고 있다
- 이는 곧 공연료 청구서의 요구사항 변경이 계산 로직에도 영향을 줄 수 있다는 것으로 여러 책임을 가지고 있는 것만으로 유지보수 비용이 늘어날 것으로 보였다 (`복잡성↑`)
- <u>따라서 두 책임을 분리하여 각각의 클래스로 만들고 협력하는 것이 필요해 보였다</u>

**2️⃣ OCP(개방 폐쇄 원칙) 위반**
- `statement(..)`에서는 두 가지 장르`(비극 : tragedy, 희극 : comedy)`만을 지원하고 있다
- 만약 장르가 추가된다면  `switch문` 수정은 불가피할 것이고, 그 수가 늘어날수록 가독성 저하, 복잡성 증가로 이어져 유지보수 비용이 늘어날 것으로 보였다
- <u>따러서  기존 코드의 변경없이 유연하게 확장하고 관리하기 위해서 인터페이스 정의하고 구현하는 방식의 설계가 필요해 보였다 (다형성)</u>
	- `장르별 계산 인터페이스`에 의존하게 되면 구현체의 변경이 발생하더라도 `statement(..)`는 영향을 받지 않고 인터페이스에 정의된 계산 메서드만 호출하여 사용하면 된다 


>[!note] p27
> "프로그램이 새로운 기능을 추가하기에 편한 구조가 아니라면, 먼저 기능을 추가하기 쉬운 형태로 리팩터링하고 나서 원하는 기능을 추가한다" 

---
### 3.  리팩토링

#### 첫번째. 함수 추출하기
`statement(..)`에 있던 <u>swtich문</u>과 <u>누적 계산 변수</u>를 함수 추출하여 리팩토링하는 과정을 실습했다 

swtich문을 함수 추출하여 분리하는 과정에서는 아래 기법이 적용되었다
- 함수 추출하기 : `{java}amountFor(..)`
- 임시 변수를 질의 합수로 바꾸기 : `{java}playFor(Performance)`
- 변수 인라인하기 

함수 추출에서 더 나아가 매개변수 Plays 제거하는 과정을 저자는 설명하고 있었다
> (p35)"나는 김 함수를 잘게 쪼갤 때마다 play 같은 변수를 최대한 제거한다. 이런 임시 변수들 때문에 로컬 범위에 존재하는 이름이 늘어나서 추줄 작업이 복잡해지기 때문이다"
 
```java
// before
private int amountFor(Performance performance, Plays plays) {..}

// after
private int amountFor(Performance performance) {..}
```


 "임시 변수를 질의 함수 추출하기"와 "인라인"기법을 적용함으로써 코드 가독성이 향상되어 보였다. 하지만 `{java}plays` 조회하는게 `{java}amountFor(..)`에서 들어나지 않기 때문에 초기화시 `{java}plays` 주입하지 않아 오류 발생하면 코드를 찾아 봐야하므로 명시적인 게 경우에 따라 나을 수도 있을거라는 생각도 들었다 
```java hl:3,11
private int amountFor(Performance performance) {  
    int result;  
    switch (playFor(performance).getType()) {
        // 생략..
    }

    return result;
}

private Play playFor(Performance performances) {  
    return plays.get(performances.getPlayId());  
}
```




totalAmount와 volumeCredits 계산 함수 추출

```java
for(Performance performances : invoice.getPerformances()) {  
	// 포인트를 적립한다  
	volumeCredits += Math.max(performances.getAudience() - 30, 0);  

	// 희극 관객 5명마다 추가 포인트를 제공한다  
	if(play.getType().equals(PlayType.COMEDY)) {  
		volumeCredits += (performances.getAudience() / 5);  
	}  

	result.append(String.format("%s: $%d %d석\n", play.getName(), thisAmount / 100, performances.getAudience()));  
	totalAmount += thisAmount;  
}  
```



```java
private int totalAmount() throws Exception {  
    int result = 0;  
    for(Performance performances : invoice.getPerformances()) {  
        result += amountFor(performances);  
    }  
    return result;  
}  

private int amountFor(Performance perfromance) { 
  // .. 
}

private Play playFor(Perfromance performance) {
  // ..
}
  
private int totalVolumeCredits() {  
    int result = 0;  
    for(Performance performances : invoice.getPerformances()) {  
        result += volumeCreditsFor(performances);  
    }  
    return result;  
}

private int volumeCreditsFor(Performance performances) {  
    int result = 0;  
    result += Math.max(performances.getAudience() - 30, 0);  
  
    if(playFor(performances).getType().equals(PlayType.COMEDY)) {  
        result += (performances.getAudience() / 5);  
    }  
  
    return result;  
}
```







---
### 4. 인용구

>[!note]  p28
>"리팩터링하기 전에 제대로 된 테스트부터 마련한다. 테스트는 반드시 자가진단하도록 만든다"


>[!tip] p34
>자바스크립트와 같은 동적 타입 언어를 사용할 때는 타입이 드러나게 작성하면 도움된다. 그래서 나는 매개변수 이름에 접두어로 타입 이름을 적는데, 지금처럼 매개변수의 역할이 뚜렷하지 않을 때는 부정 관사(a/an)을 붙인다. 이 방식은 켄트 백에게 배웠는데 ..


>[!note] p35
>"컴퓨터가 이해하는 코드는 바보도 작성할 수 있다. 사람이 이해하도록 작성하는 프로그래머가 진정한 실력자다"


---
### 
함수 추출하기, 임시 변수를 질의 함수로 바꾸기, 변수를 인라인하기
