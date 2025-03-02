
캡슐화란 
- `데이터와 행위를 묶는 것은 캡슐화가 아니다 by 토비님`
- 역할과 책임, 응집도와 결합도 키워드가 붙어 다는듯 하다

### 7.1
- 자바의 깊은 복사는 어떻게?
	- `rawData()` getter 호출시 javascript는 loadash를 사용하여 깊은 복사 수행 
		- **모든 중첩 구조 포함됨**
		- 자바에서 사용 가능한 deep copy 종류 : https://myborn.tistory.com/23
- 그리고 setUsage(..) 호출시 3 depth는 들어가는데 .. 위임하는게 맞나 싶다

처음부터 혼란 스럽다.. depth가 깊어지다보니
메서드 체인 방식으로 자바스크립트에서 호출했는데
실무에서도 depth가 1이상 넘어본적 없는데, 
내부적으로 setter를 호출하여 파라미터를 넘기는게 맞는건가??


### 7.2 컬렉션 캡슐화하기
- [컬렉션 파이프라인 패턴](https://martinfowler.com/articles/collection-pipeline/)
- (일급 컬렉션은 아니고) 컬렉션을 가진 객체에서 추가/삭제 제어권을 가지도록 한다
- 테스트 연습하기 괜찮음

```java
@DisplayName("주입한 컬렉션의 참조를 끊지 않는 경우, 외부에서 코스 추가시 원본도 영향을 받는다")  
@Test  
void addCourse1() {  
    List<Course> courses = new ArrayList<>();  
    courses.add(new Course("코스1"));  
    courses.add(new Course("코스2"));  
    Person person = new Person("name", courses);  
  
    courses.add(new Course("추가 코드", false)); // 💩
  
    assertThat(person.getCourses()).hasSize(3);  
}  
  
@DisplayName("게터로 코스 컬렉션을 가져와 수정할 경우, 원본 컬렉션도 수정된다")  
@Test  
void addCourse2() {  
    List<Course> courses = new ArrayList<>();  
    courses.add(new Course("코스1"));  
    courses.add(new Course("코스2"));  
    Person person = new Person("name", courses);  
  
    person.getCourses().add(new Course("추가 코드", false));  // 💩
  
    assertThat(person.getCourses()).hasSize(3);  
}
```



courses 컬렉션에 대한 게터를 제거 하고 메서드 지원함으로써 안정적으로 컬렉션 제어 가능해짐
```java
public class Person {  
    private final String name;  
    private final List<Course> courses;  
  
    public Person(String name, List<Course> courses) {  
        this.name = name;  
        this.courses = new ArrayList<>(courses);
    }  

    public void addCourse(Course aCourse) {  
        this.courses.add(aCourse);  
    }
    
	public boolean removeCourse(Course aCourse) {  
	    return this.courses.remove(aCourse);  
	}

	public int numAdvancedCourses() {  
	    return (int) this.courses.stream()  
	            .filter(Course::isAdvanced)  
	            .count();  
	}

    public String getName() {
        return this.name;
    }
}
```

개별 원소를 추가하고 제거하는 메서드를 제공했기 때문에 컬렉션에 대한 getter, setter를 제거한다
만일 세터나 게터를 제공해야 할 특정 이유가 있다면 컬렉션의 복제본을 저장하거나 제공하도록 한다

p250 내 경험에 따르면 컬렉션에 대해서는 어느 정도 강박증을 갖고 불필요한 복제복을 만드는 편이 예상치 못한 수정이 촉발한 오류를 디버깅하는 것보다 낫다.


### 7.3 기본 값을 객체로 만들기
- 테스트 연습하기 괜찮음

문자열을 Priority 라는 값 객체로 감싸서 관리하네 
- 앞에서 `문자열 값` 이라는 표현을 사용했었는데 좋지 않아 보인다
- **추가로 Priority를 enum으로 변경하는ㄴ게  타입 안정성, 재활용성을 보장할 수 있어 보인다**

```java
public class Order {  
    private Priority priority;  
  
    public Order(Priority priority) {  
        this.priority = priority;  
    }  
  
    public String getPriorityString() {  
        return priority.getValue();  
    }  
  
    public void setPriority(String aString) {  
        this.priority = new Priority(aString);  
    }  
}


public class Priority {  
    private final String value;  
  
    public Priority(String value) {  
        this.value = value;  
    }  
  
    public String getValue() {  
        return value;  
    }  
}
```



```java
class OrderTest {  
  
    @Test  
    void test() {  
        List<Order> orders = new ArrayList<>();  
        orders.add(new Order(new Priority("high")));  
        orders.add(new Order(new Priority("rush")));  
        orders.add(new Order(new Priority("normal")));  
  
        int highPriorityCount = (int) orders.stream()  
                .map(Order::getPriorityString)  
                .filter(p -> p.equals("high") || p.equals("rush"))  
                .count();  
  
        assertThat(highPriorityCount).isEqualTo(2);  
    }  
}
```


기존 코드의 문제점 
- `Priority` 값이 문자열(`String`)로 관리되기 때문에 **잘못된 값이 들어와도 런타임에서야 오류를 발견할 수 있음**  
    → 예: `new Priority("hogh")` (오타)
- 테스트 시 `"high"` 또는 `"rush"` 같은 **매직 스트링**을 직접 비교해야 함
- 새로운 우선순위 값이 추가될 때 `"high"`, `"rush"` 등의 문자열을 **여러 곳에서 일일이 추가해야 하는 문제** 발생


```java
public enum Priority2 {  
    LOW("low"),  
    NORMAL("normal"),  
    HIGH("high"),  
    RUSH("rush");  
  
    private final String value;  
  
    Priority2(String value) {  
        this.value = value;  
    }  
  
    public String getValue() {  
        return value;  
    }  
  
    public static Priority2 from(String value) {  
        return Arrays.stream(values())  
                .filter(p -> p.value.equalsIgnoreCase(value))  
                .findFirst()  
                .orElseThrow(() -> new IllegalArgumentException(value + "는 유효하지 않은 우선순위입니다"));  
    }  
  
    public boolean isHigher() {  
        return this == HIGH || this == RUSH;  
    }  
}
```


```java
public class Order {  
    private Priority priority;  
    private Priority2 priority2;  
  
    public Order(Priority priority) {  
        this.priority = priority;  
    }  
  
    public Order(Priority2 priority2) {  
        this.priority2 = priority2;  
    }  
  
    public boolean highMoreThan() {  
        return priority2.isHigher();  
    }  
  
    public String getPriorityString() {  
        return priority.getValue();  
    }  
  
    public void setPriority(String aString) {  
        this.priority = new Priority(aString);  
    }  
}
```


```java
class OrderTest {  
  
    @Test  
    void test() {  
        List<Order> orders = new ArrayList<>();  
        orders.add(new Order(Priority2.HIGH));  
        orders.add(new Order(Priority2.RUSH));  
        orders.add(new Order(Priority2.NORMAL));  
  
        int highPriorityCount = (int) orders.stream()  
                .filter(Order::highMoreThan)  
                .count();  
  
        assertThat(highPriorityCount).isEqualTo(2);  
    }  
}
```

불필요해진 필드와 메서드를 제거하고 Priority2를 Priority로 renaming
```java
public class Order {  
    private Priority priority;  

    public Order(String aString) {
       this(Priority.from(aString));
    }
  
    public Order(Priority priority) {  
        this.priority = priority;  
    }  
  
    public boolean isHigher() {  
        return priority.isHigher();  
    }  
  
    public String getPriorityString() {  
        return priority.getValue();  
    }  
  
    public void setPriority(String aString) {  
        this.priority = Priority.from(aString);  
    }  
}


class OrderTest {  
  
    @Test  
    void test() {  
        List<Order> orders = new ArrayList<>();  
        orders.add(new Order(Priority.HIGH));  
        orders.add(new Order(Priority.RUSH));  
        orders.add(new Order(Priority.NORMAL));  
  
        int highPriorityCount = (int) orders.stream()  
                .filter(Order::isHigher)  
                .count();  
  
        assertThat(highPriorityCount).isEqualTo(2);  
    }  
}
```


### 7.4 임시 변수를 질의 함수로 바꾸기
> Replace Temp with Query

- 테스트 연습하기 괜찮음

변수 대신 함수로 만들어두면 비슷한 계산을 수행하는 다른 함수에서도 사용할 수 있어 **코드 중복**이 줄어든다



✨리팩터링 후
```java
public class Order {  
    private final int quantity;  
    private final Item item;  
  
    public Order(int quantity, Item item) {  
        this.quantity = quantity;  
        this.item = item;  
    }  
  
    public double price() {  
        return basePrice() * discountFactor();  
    }  
  
    private int basePrice() {  
        return this.quantity * this.item.getPrice();  
    }  
  
    private double discountFactor() {  
        double discountFactor = 0.98;  
        if(basePrice() > 1000) discountFactor -= 0.03;  
        return discountFactor;  
    }  
}
```


### 7.5 클래스 추출하기
- 테스트와 함께 점진적으로 리팩터링
	- 초기화 테스트 생성해서 VO생성, equals, hashCode 재생성 및 리네이밍을 반복
- VO 만드는 걸 뜻하는 듯함

```java
public class Person {  
    private String name;  
    private String officeAreaCode;  
    private String officeNumber;  
  
    public Person(String name, String officeAreaCode, String officeNumber) {  
        this.name = name;  
        this.officeAreaCode = officeAreaCode;  
        this.officeNumber = officeNumber;  
    }  
  
    // getter, setter
}
```


### 7.6 클래스 인라인하기


```java
public class TrackingInformation {  
    private String shippingCompany; // 배송 회사  
    private String trackingNumber; // 추적 번호  
  
    public TrackingInformation(String shippingCompany, String trackingNumber) {  
        this.shippingCompany = shippingCompany;  
        this.trackingNumber = trackingNumber;  
    }  
  
    public String display() {  
        return String.format("%s: %s", this.shippingCompany, this.trackingNumber);  
    }  
}


public class Shipment {  
    private TrackingInformation trackingInformation;  
  
    public Shipment(TrackingInformation trackingInformation) {  
        this.trackingInformation = trackingInformation;  
    }  
  
    public String display() {  
        return trackingInformation.display();  
    }  
  
    public TrackingInformation getTrackingInformation() {  
        return trackingInformation;  
    }  
  
    public void setTrackingInformation(TrackingInformation trackingInformation) {  
        this.trackingInformation = trackingInformation;  
    }  
}
```

리팩터링 후 - 점진적으로 개선한다
```java
public class Shipment {  
    private String shippingCompany; // 배송 회사  
    private String trackingNumber; // 추적  
  
    public Shipment(String shippingCompany, String trackingNumber) {  
        this.shippingCompany = shippingCompany;  
        this.trackingNumber = trackingNumber;  
    }  
  
    public String display() {  
        return String.format("%s: %s", this.shippingCompany, this.trackingNumber);  
    }  
  
    public void setShippingCompany(String arg) {  
        this.shippingCompany = arg;  
    }  
  
    public void setTrackingNumber(String arg) {  
        this.trackingNumber = arg;  
    }  
}
```


### 7.7 위임 숨기기 
> Hide Delegate

```java
public class Person {  
    private final String name;  
    private Department department;  
  
    public Person(String name, Department department) {  
        this.name = name;  
        this.department = department;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public Department getDepartment() {  
        return department;  
    }  
  
    public void setDepartment(Department department) {  
        this.department = department;  
    }  
  
    public static class Department {  
        private String chargeCode;  
        private String manager;  
  
        public Department(String chargeCode, String manager) {  
            this.chargeCode = chargeCode;  
            this.manager = manager;  
        }  
  
        public String getChargeCode() {  
            return chargeCode;  
        }  
  
        public void setChargeCode(String chargeCode) {  
            this.chargeCode = chargeCode;  
        }  
  
        public String getManager() {  
            return manager;  
        }  
  
        public void setManager(String manager) {  
            this.manager = manager;  
        }  
    }  
}

```


```java
import static org.assertj.core.api.Assertions.assertThat;  
  
class PersonTest {  
  
    @Test  
    void manager() {  
        Person person = new Person("tester", new Person.Department("001", "toby"));  
    
        String manager = person.getDepartment().getManager(); // 💩 
  
        assertThat(manager).isEqualTo("toby");  
    }  
}
```

- 클라이언트는 부서 클래스의 작동 방식, 다시 말해 부서 클래스가 관리자 정보를 제공한다는 사실을 알아야 한다
- 이러한 의존성을 줄이려면 클라이언트가 부서 클래스를 알수 없게 숨기고, 대신 사람 클래스에 간단한 위임 메서드를 만들면 된다


리팩터링 후
```java

public class Person {
	//..
	
	public String manager() {  // 위임 메서드 (= 포워딩 메서드?)
	    return this.department.getManager();  
	}
}

```

```java
class PersonTest {  
  
    @Test  
    void manager() {  
        Person person = new Person("tester", new Person.Department("001", "toby"));  
  
        String manager = person.manager();  //✨
  
        assertThat(manager).isEqualTo("toby");  
    }  
}
```


### 7.8 중재자 제거하기
> Remove Middle Man



> [!note] 이 냄새는 데메테르 법칙을 너무 신봉할 때 자주 나타난다. 나는 이 법칙을 '이 따금 유용한 데메테르의 제안' 정도로 부르는게 훨씬 낫다고 생각한다 


예제는 복사 붙여 넣기 하고 끝이라서 의미 없을듯 .. 설명이랑 링크 주소나 읽어보자


### 7.9 알고리즘 교체하기 
> Substitute Algorithm

```java
public class Person {  
  
    public String foundPerson(String[] peoples) {  
        for(String people : peoples) {  
            if(people.equals("Don")) return "Don";  
  
            if(people.equals("John")) return "John";  
  
            if(people.equals("Jane")) return "Jane";  
        }  
  
        return "";  
    }  
}
```


리팩터링 후 - `foundPerson2(..)` 생성해서 구현 > 테스트 > 기존 메서드 삭제 > remaning
```java
public class Person {  
  
    public String foundPerson(String[] peoples) {  
        final List<String> candidate = List.of("Don", "John", "Jane");  
        return Arrays.stream(peoples)  
                .filter(candidate::contains)  
                .findFirst()  
                .orElse("");  
    }  
      
}
```