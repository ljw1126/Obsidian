
### 8.1 함수 옮기기 
예시1
```text
trackSummary(..)
- calculateDistance()
- distance(..)
- radians(..)
- calculateTime()

# 리팩터링1.
trackSummary(..)
- calculateDistance()
- distance(..)
- radians(..)
- calculateTime()

top_calculateDistance(..) 최상위로 분리


# 리팩터링2. distacne와 radians 옮기기
trackSummary(..)
- calculateDistance()
	- distance(..)
	- radians(..)
- calculateTime()

top_calculateDistance(..) 최상위로 분리
	- distance(..)
	- radians(..)

# 리팩터링3. 
- calculateDistance()에서 top_calculateDistance() 호출
- 이상 없으면 calculateDistance() 제거
- top_calculateDistance() 직접 호출하도록 변경
- top_calculateDistance() > totalDistance() renaming
- 
```

예시1의 경우 자바스크립트의 함수 중첩 사용하나 자바는 지원하지 않는다.
그래서 **메서드 분리**로 선언하여 구현하였다.

포인트는 함수 중첩을 사용해 함수 옮기기를 하는데 있어 의존하는게 없다면 최상위로 뽑는것으로 보인다

> 중첩 함수를 사용하다보면 숨겨진 데이터끼리 상호 의존하기가아주 쉬우니 중첩함수는 되도록 만들지 말자. p285



```java

public class GpsTrackSummary {  
    private final List<Waypoint> points;  
  
    public GpsTrackSummary(List<Waypoint> points) {  
        this.points = new ArrayList<>(points);  
    }  
  
    public TrackSummary trackSummary() {  
        double totalTime = calculateTime();  
        double pace = totalTime / 60 / totalDistance();  
        return new TrackSummary(totalTime, totalDistance(), pace);  
    }  
  
    private double totalDistance() {  
        double result = 0;  
        for(int i = 1; i < points.size(); i++) {  
            result += points.get(i - 1).distance(points.get(i));  
        }  
        return result;  
    }  
  
    private double calculateTime() {  
        if (points.isEmpty()) {  
            return 0;  
        }  
  
        Waypoint last = points.get(points.size() - 1);  
        Waypoint first = points.get(0);  
        return last.diffTimestamp(first);  
    }  
}


public class Waypoint {  
    private final double latitude;  
    private final double longitude;  
    private final double timestamp;  
  
    public Waypoint(double latitude, double longitude, double timestamp) {  
        this.latitude = latitude;  
        this.longitude = longitude;  
        this.timestamp = timestamp;  
    }  
  
    public double distance(Waypoint other) {  
        int earthRadius = 6371;  
        double dLat = radians(other.latitude - this.latitude);  
        double dLon = radians(other.longitude - this.longitude);  
  
        double a = Math.pow(Math.sin(dLat / 2), 2) + Math.cos(radians(other.latitude))  
                * Math.cos(radians(this.latitude)) * Math.pow(Math.sin(dLon / 2), 2);  
        double c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));  
  
        return earthRadius * c;  
    }  
  
    private double radians(double degrees) {  
        return degrees * Math.PI / 180;  
    }  
  
    public double diffTimestamp(Waypoint other) {  
        return this.timestamp - other.timestamp;  
    }

    // getter 생략
}

```

도움은 되지 않는 참고 주소
https://www.movable-type.co.uk/scripts/latlong.html


예2

```java
public class Account {  
    private final int dayOverdrawn;  
    private final AccountType type;  
  
    public Account(int dayOverdrawn, AccountType type) {  
        this.dayOverdrawn = dayOverdrawn;  
        this.type = type;  
    }  
  
    // 은행 이자 계산  
    public double bankCharge() {  
        double result = 4.5;  
        if(dayOverdrawn > 0) {  
            result += overdraftCharge();  
        }  
        return result;  
    }  
  
    // 초과 인출 이자 계산  
    private double overdraftCharge() {  
        if(type.isPremium()){  
            int baseCharge = 10;  
            if(dayOverdrawn <= 7) {  
                return baseCharge;  
            }  
  
            return baseCharge + (dayOverdrawn - 7) * 0.85;  
        }  
  
        return dayOverdrawn * 1.75;  
    }  
}


public class AccountType {  
    private final boolean premium;  
  
    public AccountType(boolean premium) {  
        this.premium = premium;  
    }  
  
    public boolean isPremium() {  
        return premium;  
    }
}
```
- Account의 overdraftCharge() 함수를 AccountType으로 옮긴다 
	- 이떄 dayOverdrawn은 Account에서 주입하도록한다

```java

public class AccountType {  
  private final boolean premium;  
  
  public AccountType(boolean premium) {  
    this.premium = premium;  
  }  
  
  private boolean isPremium() {  
    return premium;  
  }  
  
  // 초과 인출 이자 계산  
  public double overdraftCharge(int dayOverdrawn) {  
    if (isPremium()) {  
      int baseCharge = 10;  
      if (dayOverdrawn <= 7) {  
        return baseCharge;  
      }  
  
      return baseCharge + (dayOverdrawn - 7) * 0.85;  
    }  
  
    return dayOverdrawn * 1.75;  
  }  
}

```




### 8.6 문장 슬라이드 하기

서비스 로직에 가깝다고 생각함
```java
public class BillingService {  
  
  private final PricingPlanRepository pricingPlanRepository;  
  private final OrderRepository orderRepository;  
  
  public BillingService(  
      PricingPlanRepository pricingPlanRepository, OrderRepository orderRepository) {  
    this.pricingPlanRepository = pricingPlanRepository;  
    this.orderRepository = orderRepository;  
  }  
  
  public double processOrder(Long orderId) {  
    PricingPlan pricingPlan = pricingPlanRepository.findPricingPlan();  
    Order order = orderRepository.findById(orderId);  
  
    double charge = 0.0;  
    int baseCharge = pricingPlan.getBase();  
    int chargePerUnit = pricingPlan.getUnitCharge();  
    int units = order.getUnits();  
    double discount = 0;  
    charge = baseCharge + units * chargePerUnit;  
    int discountableUnits = Math.max(units - pricingPlan.getDiscountThreshold(), 0);  
    discount = discountableUnits * pricingPlan.getDiscountFactor();  
    if (order.isRepeatOrder()) discount += 20;  
    charge = charge - discount;  
    return charge;  
  }  
}
```


그래서 Repository는 단일 인터페이스만 가지므로 아래와 가팅 람다로 처리하여 단위 테스트 함
```java
class BillingServiceTest {  
  
  @Test  
  void processOrderTest() {  
    PricingPlan pricingPlan = new PricingPlan(100, 10, 5, 0.1);  
    Order order = new Order(10, false);  
    BillingService billingService = new BillingService(() -> pricingPlan, (orderId) -> order);  
  
    double actual = billingService.processOrder(9999L);  
  
    assertThat(actual).isEqualTo(199.5);  
  }  
}
```

> 명령 - 질의 분리 원칙을 지켜가며 코딩 ?


```java

public double processOrder(Long orderId) {  
  // 가격 정책과 주문 정보를 가져온다  
  PricingPlan pricingPlan = pricingPlanRepository.findPricingPlan();  
  Order order = orderRepository.findById(orderId);  
  
  // 기본 청구 금액을 계산한다  
  int baseCharge = pricingPlan.getBase();  
  int chargePerUnit = pricingPlan.getUnitCharge();  
  int units = order.getUnits();  
  double charge = baseCharge + units * chargePerUnit;  
  
  // 할인 금액을 계산한다  
  int discountableUnits = Math.max(units - pricingPlan.getDiscountThreshold(), 0);  
  double discount = discountableUnits * pricingPlan.getDiscountFactor();  
  if (order.isRepeatOrder()) {  
    discount += 20;  
  }  
  
  return charge - discount;  
}

```

문장 교환하기(Swap Statement)라는 이름이 거의 똑같은 리팩터링도 있다.

### 8.7 반복문 쪼개기

반복문 쪼개기의 묘미는 그 자체가 아닌, 다음 단계로 가는 디딤돌 역할에 있다.

```java
public class YoungestTotalSalary {  
  private final List<Person> peoples;  
  
  public YoungestTotalSalary(List<Person> peoples) {  
    this.peoples = new ArrayList<>(peoples);  
  }  
  
  public String getYoungestTotalSalary() {  
    int youngest = peoples.isEmpty() ? Integer.MAX_VALUE : peoples.get(0).getAge();  
    long totalSalary = 0;  
    for (Person person : peoples) {  
      if (person.getAge() < youngest) youngest = person.getAge();  
      totalSalary += person.getSalary();  
    }  
  
    return String.format("최연소: %s, 총 급여: %s", youngest, totalSalary);  
  }  
}
```


```java
class YoungestTotalSalaryTest {  
  
  @Test  
  void getYoungestTotalSalaryTest() {  
    List<Person> people =  
        List.of(new Person(25, 50000), new Person(30, 60000), new Person(22, 70000));  
    YoungestTotalSalary youngestTotalSalary = new YoungestTotalSalary(people);  
  
    String actual = youngestTotalSalary.getYoungestTotalSalary();  
  
    Assertions.assertThat(actual).isEqualTo("최연소: 22, 총 급여: 180000");  
  }  
}
```

첫번째로 반복문을 복사하고, 두번째로 반복문 복제로 인해 잘못된 결과를 초래할 수 있는 중복을 제거, 문장 슬라이드
```java
public class YoungestTotalSalary {  
  private final List<Person> peoples;  
  
  public YoungestTotalSalary(List<Person> peoples) {  
    this.peoples = new ArrayList<>(peoples);  
  }  
  
  public String getYoungestTotalSalary() {  
    long totalSalary = 0;  
    for (Person person : peoples) {  
      totalSalary += person.getSalary();  
    }  
  
    int youngest = peoples.isEmpty() ? Integer.MAX_VALUE : peoples.get(0).getAge();  
    for (Person person : peoples) {  
      if (person.getAge() < youngest) youngest = person.getAge();  
    }  
  
    return String.format("최연소: %s, 총 급여: %s", youngest, totalSalary);  
  }  
}
```

함수 추출하기와 알고리즘 교체하기 적용
```java
public class YoungestTotalSalary {  
  private final List<Person> peoples;  
  
  public YoungestTotalSalary(List<Person> peoples) {  
    this.peoples = new ArrayList<>(peoples);  
  }  
  
  public String getYoungestTotalSalary() {  
    return String.format("최연소: %s, 총 급여: %s", youngest(), totalSalary());  
  }  
  
  private int youngest() {  
    return peoples.stream()  
        .mapToInt(Person::getAge)  
        .min()  
        .orElseThrow(() -> new IllegalArgumentException("No people available"));  
  }  
  
  private long totalSalary() {  
    return peoples.stream().mapToInt(Person::getSalary).sum();  
  }  
}
```


### 8.8 반복문을 파이프라인으로 바꾸기 

```java
public class IndiaSearch {  
  private IndiaSearch() {}  
  
  public static List<SearchResult> acquireData(String input) {  
    String[] lines = input.split("\n");  
    boolean firstLine = true;  
    List<SearchResult> result = new ArrayList<>();  
    for (String line : lines) {  
      if (firstLine) {  
        firstLine = false;  
        continue;  
      }  
  
      if (line.trim().equals("")) continue;  
  
      String[] record = line.split(",");  
      if (record[1].trim().equals("India")) {  
        result.add(new SearchResult(record[0].trim(), record[2].trim()));  
      }  
    }  
  
    return result;  
  }  
}

```


```java
class IndiaSearchTest {  
  
  @Test  
  void acquireDataTest() {  
    String input =  
        """  
office, country, telephone  
Chicago, USA, +1 312 373 1000  
Beijing, China, +86 4008 900 505  
Bangalore, India, +91 80 4064 9570  
Porto Alegre, Brazil, +55 31 3079 3550  
Chennai, India, +91 44 660 44766  
""";  
  
    List<SearchResult> actual = IndiaSearch.acquireData(input);  
  
    assertThat(actual)  
        .hasSize(2)  
        .containsExactly(  
            new SearchResult("Bangalore", "+91 80 4064 9570"),  
            new SearchResult("Chennai", "+91 44 660 44766"));  
  }  
}
```

흐름을 지켜가면서 파이프라인으로 리팩터링을 하니 재미있음
```java
public class IndiaSearch {  
  private IndiaSearch() {}  
  
  public static List<SearchResult> acquireData(String input) {  
    return Arrays.stream(input.split("\n"))  
        .skip(1)  
        .filter(line -> !line.trim().isEmpty())  
        .map(line -> line.split(","))  
        .filter(record -> record[1].trim().equals("India"))  
        .map(record -> new SearchResult(record[0].trim(), record[2].trim()))  
        .toList();  
  }  
}
```

반복문을 파이프라인으로 대체하는 예를 더 보고 싶은 경우 
- 저자 블로그의 [Refactoring with Loops and Collection Pipelines](https://martinfowler.com/articles/refactoring-pipelines.html) 참고