
## 소개

### 개요

임의 Wrapper 타입을 생성할 수 있다
```java
@Test  
void createWrapperType() {  
    FixtureMonkey fixtureMonkey = FixtureMonkey.create();  
  
    String randomString = fixtureMonkey.giveMeOne(String.class);  
    Integer randomNumber = fixtureMonkey.giveMeOne(Integer.class);  
  
    assertThat(randomString).isNotNull();  
    assertThat(randomNumber).isNotNull();  
}
```

```text
// 첫번째 실행
죨䉠鐃᷎醺ۖ呁㾀坙눆Ǧ䠷登唹䩎┎괖㹱᪺ᑜ転츈捠⮉ဎ殓
2

// 두번째 실행
楎�훷Ꭹ䩋铂짶嚎귐栅㍒쵧遰见悱킸羨ꮎ̋偤喤缇鸰뗿
410
```


임의 데이터로 복잡한 객체 생성할 수 있다
- 이때 기본 생성자와 setter가 있어야 reflection api로 임의 데이터 주입한다
- setter가 없는 경우 기본 생성자로 초기화한 빈 객체를 반환한다 
```java
public class User {  
    private String name;  
    private int age;  
    private List<Address> addresses;  
  
    private User() {  
    }  
    
    public User(String name, int age, List<Address> addresses) {  
        this.name = name;  
        this.age = age;  
        this.addresses = addresses;  
    }  
    
    public void setName(String name) {  
        this.name = name;  
    }  
    
    public void setAge(int age) {  
        this.age = age;  
    }  

	public void setAddresses(List<Address> addresses) {  
        this.addresses = addresses;  
    }  

}


public class Address {  
    private String city;  
    private String street;  
  
    private Address() {  
    }  
    public Address(String city, String street) {  
        this.city = city;  
        this.street = street;  
    }  
    public void setCity(String city) {  
        this.city = city;  
    }  
    public void setStreet(String street) {  
        this.street = street;  
    }
}
```

```text
User{name='uᲟ糙鼅똄咛钕琜譭ň᎒쯸⡖䜰㫯탿螹꣜寎ຏ₡₤顫쉪ࠕ', age=-1, addresses=[]}
```


객체를 여러 개 생성할 수 있다.
```java
@Test  
void createMultiUser() {  
    List<User> users = fixtureMonkey.giveMe(User.class, 3);  
  
    assertThat(users).hasSize(3);  
}
```

```text
User{name='null', age=-4777, addresses=[fixturemonkey.Address@2783717b]}

User{name='䂊蝘喬惪瑥쵺濝輇Ⳟ뾟缔蕊擯锣췗蜜즥っ즥ﲤ', age=-1, addresses=[fixturemonkey.Address@76f7d241, fixturemonkey.Address@4a335fa8, fixturemonkey.Address@3f363cf5]}

User{name='ὔ墷춞阆톟Ȭ㟦ど귟터ⲱㆡ囋홼쀍玿氠ꧨ᳥茆튄봓', age=-1856768, addresses=[fixturemonkey.Address@3829ac1, fixturemonkey.Address@4baf352a, fixturemonkey.Address@1bb1fde8]}
```


중첩된 객체도 생성해준다 
- `User`객체가 가진 컬렉션의 사이즈를 세부적으로 지정할 수 있어 보인다 
- `sample()` 호출 전까지 생성되지 않는다 (**지연전략**)
```java
@Test  
void createWithAddresses() {  
    ArbitraryBuilder<User> addresses = fixtureMonkey.giveMeBuilder(User.class)  
            .size("addresses", 2);  
  
    User user = addresses.sample();  
  
    assertThat(user).isNotNull();  
    assertThat(user).isNotEqualTo(addresses.sample());  
}
```


## Fixture Monkey를 사용하는 이유 

**✅ 한 줄로 끝나는 테스트 객체 생성**

>[!info] 
>- 테스트 객체 생성을 위한 **보일러 플레이트** 코드는 이제 그만!
>- Fixture Monkey는 한 줄의 코드로 어떤 테스트 객체든 생성함
>- 지루한 테스트 준비 작업을 간결하고 우아한 솔루션으로 바꿔 보기!

```java
// 이전: 수동으로 객체 생성 
Product product = new Product(); 
product.setId(1L); 
product.setName("Test Product"); 
product.setPrice(1000); 
product.setCreatedAt(LocalDateTime.now()); 

// 이후: Fixture Monkey 사용 
Product product = fixtureMonkey.giveMeOne(Product.class);
```


**⚙️ 직관적인 경로 기반 설정** 
- `[*]` 와일드 카드 연산자로 컬렉션 전체를 손쉽게 조작 가능
- 보일러 플레이트 코드를 크게 줄이고 테스트의 유지보수성을 높임 

```java
class Order {
    List<OrderItem> items;
    Customer customer;
    Address shippingAddress;
}

class OrderItem {
    Product product;
    int quantity;
}

class Product {
    String name;
    List<Review> reviews;
}

// 모든 상품 이름을 "Special Product"로 설정
ArbitraryBuilder<Order> orderBuilder = fixtureMonkey.giveMeBuilder(Order.class)
    .set("items[*].product.name", "Special Product");

// 모든 리뷰 평점을 5점으로 설정
ArbitraryBuilder<Order> orderWithGoodReviews = fixtureMonkey.giveMeBuilder(Order.class)
    .set("items[*].product.reviews[*].rating", 5);

// 모든 수량을 2로 설정
ArbitraryBuilder<Order> orderWithFixedQuantity = fixtureMonkey.giveMeBuilder(Order.class)
    .set("items[*].quantity", 2);
```


**♻️ 재사용 가능한 테스트 스펙**
- `ArbitraryBuilder`의 **지연평가(Lazy evaluation)** 는 객체가 필요할 때만 생성되도록 보장하여 테스트 성능을 최적화합니다

```java
// 재사용 가능한 빌더 정의
ArbitraryBuilder<Product> productBuilder = fixtureMonkey.giveMeBuilder(Product.class)
    .set("category", "Book")
    .set("price", 1000);

// 다양한 테스트에서 재사용
@Test
void testProductCreation() {
    Product product = productBuilder.sample();
    assertThat(product.getCategory()).isEqualTo("Book");
    assertThat(product.getPrice()).isEqualTo(1000);
}

@Test
void testProductWithReviews() {
    Product product = productBuilder
        .size("reviews", 3)
        .sample();
    assertThat(product.getReviews()).hasSize(3);
}

@Test
void testProductWithSpecificReview() {
    Product product = productBuilder
        .set("reviews[0].rating", 5)
        .set("reviews[0].comment", "Excellent!")
        .sample();
    assertThat(product.getReviews().get(0).getRating()).isEqualTo(5);
    assertThat(product.getReviews().get(0).getComment()).isEqualTo("Excellent!");
}
```


**✅ 모든 객체 생성 가능**
- 단순한 POJO 부터 복잡한 객체 그래프까지 처리 가능
- 리스트, 중첩된 컬렉션, 열거형, 제니릭 타입은 물론 상속 관계나 순환 참조를 가진 객체도 생성 가능
- Fixture Monkey에게 너무 복잡한 객체 구조란 없습니다
```java
// 상속
class Foo {
  String foo;
}

class Bar extends Foo {
    String bar;
}

Foo foo = FixtureMonkey.create().giveMeOne(Foo.class);
Bar bar = FixtureMonkey.create().giveMeOne(Bar.class);

// 순환 참조
class Foo {
    String value;
    Foo foo;
}

Foo foo = FixtureMonkey.create().giveMeOne(Foo.class);

// 익명 객체
interface Foo {
    Bar getBar();
}

class Bar {
    String value;
}

Foo foo = FixtureMonkey.create().giveMeOne(Foo.class);
```


**⌛ 실제 테스트 예제**

```java
@Test
void testOrderProcessing() {
    // Given
    Order order = fixtureMonkey.giveMeBuilder(Order.class)
        .set("items[*].quantity", 2)
        .set("items[*].product.price", 1000)
        .sample();
    
    OrderProcessor processor = new OrderProcessor();
    
    // When
    OrderResult result = processor.process(order);
    
    // Then
    assertThat(result.getTotalAmount()).isEqualTo(4000); // 2 items * 2 quantity * 1000 price
    assertThat(result.getStatus()).isEqualTo(OrderStatus.COMPLETED);
}
```

---

## 시작하기 
