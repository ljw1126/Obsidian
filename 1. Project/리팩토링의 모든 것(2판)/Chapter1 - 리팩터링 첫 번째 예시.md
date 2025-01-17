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

#### 3-1. statement() 함수 쪼개기
`statement(..)`에 있던 <u>swtich문</u>과 <u>누적 계산 변수</u>를 함수 추출하여 리팩토링하는 과정을 실습했다 

**첫번째**로 swtich문을 함수 추출하여 분리하는 과정에서는 아래 기법이 적용되었다
- 함수 추출하기 : `{java}amountFor(..)`
- 임시 변수를 질의 합수로 바꾸기 : `{java}playFor(Performance)`
- 변수 인라인하기 

매개변수 Plays 제거하기 위해 "임시 변수를 질의 함수 추출하기"와 "인라인"기법이 적용되었고, 그 결과 코드 가독성이 향상되어 보였다

> (p35)"나는 김 함수를 잘게 쪼갤 때마다 play 같은 변수를 최대한 제거한다. 이런 임시 변수들 때문에 로컬 범위에 존재하는 이름이 늘어나서 추줄 작업이 복잡해지기 때문이다"
 
```java
// before
private int amountFor(Performance performance, Plays plays) {..}

// after
private int amountFor(Performance performance) {..}
```


 하지만 `{java}plays` 조회하는게 `{java}amountFor(..)`에서 들어나지 않기 때문에 초기화시 `{java}plays` 주입하지 않아 오류 발생하면 코드를 찾아 봐야하므로 명시적인 게 경우에 따라 나을 수도 있을거라는 생각도 들었다 
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


**두번째**로 totalAmount와 volumeCredits 계산 함수 추출하기를 하였다
- **반복문 쪼개기** : 변수 값을 누적시키는 부분 분리
- **문장 슬라이드 하기** : 변수 초기화 문장을 변수 값 누적 코드 바로 앞으로 옮김
- **함수 추출하기** : 적립 포인트 계산 부분을 별도 함수로 추출
- **변수 인라인하기** : totalAmount, volumeCredits 변수를 statement(..)에서 제거

**리팩토링 전**
```java
public String statement(Invoie invoice, Plays plays) {   
    int totalAmount = 0;  
	int volumeCredits = 0;
	// .. 
	
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
	
    // .. 
    return result.toString();
}
```


**리팩토링 후**
```java hl:9-10
public String statement() throws Exception {  
    StringBuilder result = new StringBuilder();  
    result.append(String.format("청구 내역 (고객명: %s)", invoice.getCustomer())).append("\n");  
  
    for(Performance performances : invoice.getPerformances()) {  
        result.append(String.format("%s: $%d %d석\n", playFor(performances).getName(), amountFor(performances) / 100, performances.getAudience()));  
    }  
  
    result.append(String.format("총액: $%d\n", totalAmount() / 100));  
    result.append(String.format("적립 포인트: %d점\n", totalVolumeCredits()));  
    return result.toString();  
}

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

*"굳이? 한번 조회하면 될 걸"*
 이전 코드는 루프를 한 번 돌면서 결과값을 누적하는 반면, 리팩터링한 코드에서는 세 번이나 조회한다. 데이터가 얼마 없기 때문에 for문을 몇번 실행하더라도 크게 성능상 크게 영향은 없을 것으로 생각되었지만, 어떤 영향이 있는지는 떠오르지 않았다. 이에 저자는 아래와 같이 말한다

> (p39) "지역 변수를 제거해서 얻는 가장 큰 장점은 추출 작업이 훨씬 쉬워진다는 것이다. 유효 범위를 신경써야 할 대상이 줄어들기 때문이다. 실제로 나는 추출 작업전에는 거의 항상 지역 변수부터 제거한다"

이를 고려해본다면 함수 분리하면서 totalAmount와 totalVolumeCredits를 관리하는 포인트가 메소드로 명확해진게 두드러지게 느껴진다. 또한 `statement(..)`에 임시 변수가 없어지고 인라인으로 함수 호출하는 형태로 리팩터링이 되면서 코드의 가독성도 높아진 효과가 이번에 리팩터링을 느낄 수 있었다

>[!tip] p47
>"따라서 리팩터링으로 인한 성능 문제에 대한 내 조언은 '특별한 경우가 아니라면 일단 무시하라'는 것이다. 
>리팩터링 때문에 성능이 떨어진다면, 하던 리팩터링을 마무리하고 나서 성능을 개선하자"


#### 3-2.  클래스 분리
**단계 쪼개기**
- 필요한 데이터를 먼저 처리 후 `statement(..)` 파라미터로 전달한다
- statement(..)에서는 전달받은 결과 데이터 기반으로 HTML 렌더링을 생성한다

```java
private final Invoice invoice;  
private final Plays plays;  
  
public Statement(Invoice invoice, Plays plays) {  
    this.invoice = invoice;  
    this.plays = plays;  
}  
  
public String statement() throws Exception {  
    StatementData data = new StatementData(invoice);  
    return renderPlainText(data);  
}

private String renderPlainText(StatementData data) {
  // 청구서 생성
}

```


**참고. 클래스 다이어그램**
<img src="https://github.com/ljw1126/user-content/blob/master/refactoring2/%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81%201%EC%9E%A5_%ED%81%B4%EB%9E%98%EC%8A%A4%EB%B6%84%EB%A6%AC.png?raw=true">

Statement에는 텍스트와 html을 렌더링 하는 책임만을 가지게 되도


**리팩토링 결과**
```java
public class Statement {  
  
    public String statement(Invoice invoice, Plays plays) throws Exception {  
        return renderPlainText(StatementData.create(invoice, plays));  
    }  
  
    private String renderPlainText(StatementData data) {  
        // 랜더링..
        return result.toString();
    }  
  
    public String htmlStatement(Invoice invoice, Plays plays) throws Exception {  
        return renderHtml(StatementData.create(invoice, plays));  
    }  
  
    private String renderHtml(StatementData data) {  
        // ..
        return result.toString();
    }  
}
```

#### 3-3. 다형성

각 Performance별 amount와 volumeCredits 계산하는 책임을 PerformanceCalculator에 위임하였다.
```java
public class PerformanceCalculator {  
    private final Performance performance;  
    private final Play play;  
  
    public PerformanceCalculator(Performance performance, Play play) {  
        this.performance = performance;  
        this.play = play;  
    }  
  
    public int amount() throws Exception {  
        int result = 0;  
        switch (play.getType()) {  
            case TRAGEDY :  
                result = 40_000;  
                if(performance.getAudience() > 30) {  
                    result += 1_000 * (performance.getAudience() - 30);  
                }  
                break;  
            case COMEDY :  
                result = 30_000;  
                if(performance.getAudience() > 30) {  
                    result += 10_000 + 500 * (performance.getAudience() - 20);  
                }  
                result += 300 * performance.getAudience();  
                break;  
            default :  
                throw new Exception(String.format("알 수 없는 장르: %s", play.getType()));  
        }  
  
        return result;  
    }  
  
    public int volumeCredits() {  
        int result = 0;  
        result += Math.max(performance.getAudience() - 30, 0);  
  
        if(play.getType().equals(PlayType.COMEDY)) {  
            result += (performance.getAudience() / 5);  
        }  
  
        return result;  
    }  
  
    public Play getPlay() {  
        return play;  
    }  
}
```


참고. 클래스 다이어그램
<img src="https://github.com/ljw1126/user-content/blob/master/refactoring2/%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%811%EC%9E%A5_%EB%8B%A4%ED%98%95%EC%84%B1.png?raw=true">



---
### 4. 인용구

>[!note]  p28
>"리팩터링하기 전에 제대로 된 테스트부터 마련한다. 테스트는 반드시 자가진단하도록 만든다"


>[!tip] p34
>자바스크립트와 같은 동적 타입 언어를 사용할 때는 타입이 드러나게 작성하면 도움된다. 그래서 나는 매개변수 이름에 접두어로 타입 이름을 적는데, 지금처럼 매개변수의 역할이 뚜렷하지 않을 때는 부정 관사(a/an)을 붙인다. 이 방식은 켄트 백에게 배웠는데 ..


>[!note] p35
>"컴퓨터가 이해하는 코드는 바보도 작성할 수 있다. 사람이 이해하도록 작성하는 프로그래머가 진정한 실력자다"


>[!note] p 47 statement(..) 함수 쪼개기에서
>솔직히 나라고 해서 항상 단계를 이 처럼 잘게 나누는 것은 아니지만, 그래도 상황이 복잡해지면 단계를 더 작게 나누는 일을 가장 먼저 한다. 특히 리팩터링 중간에 테스트가 실패하고 원인을 바로 찾지 못하면 가장 최근 커밋으로 돌아가서 테스트에 실패한 리팩터링의 단계를 더 작게 나눠 다시 시도한다. 
>
>이렇게 하면 문제를 해결할 수 있다. 
>커밋을 자주 했기 때문이기도 하고, 코드가 복잡할수록 단계를 작게 나누면 작업 속도가 빨라지기 때문이다

---
### 
함수 추출하기, 임시 변수를 질의 함수로 바꾸기, 변수를 인라인하기
