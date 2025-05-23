궁금한 것
- 데이터가 한글이 깨지거나, 음수가 나오는데 조정이 가능한가
	- 양수임을 보장해야 하는데 랜덤값이 음수가 나오니 테스트가 깨진다
- `시작하기 > 사용자 정의 객체 생성하기` 예제에서 서비스 생성하기 귀찮아 인터페이스 사용했는데 이것도 fixture monkey로 대체 가능한가 ?? Mockito 사용하거나 직접 fake, stub 구현이 더 나은거 같기도 하고..
- 인터페이스 구현체를 자체적으로 생성을 못해 `InterfacePlugin.interfaceImplements()` 사용했는데, 메뉴얼에는 마치 구현체 임의로 생성해주는 것처럼 얘기해서 혼란스러움 ([링크](https://naver.github.io/fixture-monkey/v1-1-0-kor/docs/generating-objects/generating-complex-types/#generic-interfaces))


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

>[!info] Fixture Monkey는 
>- 객체 생성을 위한 기본 방법으로 `BeanArbitraryIntrospector` 사용
>- `BeanArbitraryIntrospector` 사용하려면 no-args 기본 생성자 + setter 메서드 필요

### 테스트 객체 생성하기

```java
@Test
void test() {
    // given
    FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
    .objectIntrospector(ConstructorPropertiesArbitraryIntrospector.INSTANCE)
	.build();

    // when
    Product actual = fixtureMonkey.giveMeOne(Product.class);

    // then
    then(actual).isNotNull();
}
```
- `Introspector`를 `ConstructorPropertiesArbitraryIntrospector`로 설정
	- `@ConstructorProperties`이 달린 객체 생성자를 사용하여 fixture 생성하게 됨
- lombok 사용하는 경우 **lombok.config** 파일에 설정 추가 필요⚠️
	- `lombok.anyConstructor.addConstructorProperties=true`


```java
public class Product {  
    private final String name;  
    private final double price;  
  
    @ConstructorProperties({"name", "price"})  
    public Product(String name, double price) {  
        this.name = name;  
        this.price = price;  
    }  
    public String getName() {  
        return name;  
    }  
    public double getPrice() {  
        return price;  
    }
}
```

```text
// 어노테이션이 없는 경우
Product{name='null', reviews=null, category='null', price=0}


// 어노테이션이 추가된 경우 
Product{name='ࣴ໺嚒灣浦紬遙ຣ誹⪍껨⶘组ം뎟㔴潜૨㪯僶慆୼ඍщꛨ嚨͞텳쥛ㅦ᝼駚갍ꪩ♲䃲儌믆颌ﭦ嘊趚𣏕൫康ꫬ硼㵀ࠀ仩鐈䮚뙩撠㈇⵰荒赳쫣줘므횔綒幰懃ꚻꊦ僆繻鷙脦栁倬껟훋먇᲼븢嫕畿뤟ꅲ₸浏됲쫒侀棳뎭곴⦦∦⟝쇝₦똳௓㭭ꥳ礃艹臠㍫䒇뺎ꑳ䵞诜슮㥡הꮌ樎裘蒥豛ᰐᑏ◃됸౅髳㸷徐농智ぴ廐ᄜ篜瑄苴◭㸛甼縌윝⨉ஸ䖺炖뺛޷猣㢇该唲䟂轼숓ڡﯔ挨ᕨ镤璬鐣ﭼ岓ཌྷ냱妠渴켬鄙츚냑樻浿뭀䝞푒兒嬪ﻔ鮘帲ة㯰禮黦ￌ笨䚧먀鶼㝗ጭ熜桸็⣅芒ㄪƙ煯ꥏ纱컭㽞⫓만圢΁롖旗憜䲁⺨嘸ᢛ廍̡䎣鮎縁멻䵘ꄾ㤍ठ僎뛾䋋ꛎ䤶횸纸㷃䦓⫉阦짿໫埽◂鸫찒ຜ⮲嬥䖹', price=96743.45}
```
- **문자열에 null**이 들어갈 수 있고, **price에 음수**가 들어갈 수 있음
	- 랜덤값을 넣어 생성해주기 때문에 예외 케이스 찾기 좋은 듯
	- 🤔 랜덤값이라 테스트가 실패하는 경우 있음


### Bean 유효성 검사 추가하기

fixture monkey `*-starter` 의존성을 추가하면 `*validation`의존성도 포함되어 있음👋

**Gradle**
```text
testImplementation("com.navercorp.fixturemonkey:fixture-monkey-jakarta-validation:1.1.11")
```

**Maven**
```text
<dependency>
  <groupId>com.navercorp.fixturemonkey</groupId>
  <artifactId>fixture-monkey-jakarta-validation</artifactId>
  <version>1.1.11</version>
  <scope>test</scope>
</dependency>
```


```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
    .plugin(new JakartaValidationPlugin()) 
    .build();
```
- 또는 `new JavaxValidationPlugin()`
	- 자바 17 이후로 javax ➡️ jakarta로 패키지명 변경


유효성 제약 검사 조건이 있는 Product 클래스를 생성
```java
@Value
public class Product {
    @Min(1)
    long id;

    @NotBlank
    String productName;

    @Max(100000)
    long price;

    @Size(min = 3)
    List<@NotBlank String> options;

    @Past
    Instant createdAt;
}
```


```text
[Test worker] WARN com.navercorp.fixturemonkey.api.introspector.ConstructorPropertiesArbitraryIntrospector - Given type class fixturemonkey.beanvalidation.Product is failed to generate due to the exception. It may be null.
```
- lombok.config 추가 후 해결됨
	- `lombok.anyConstructor.addConstructorProperties=true`

```java
@Test  
void test() {  
    FixtureMonkey fixtureMonkey = FixtureMonkey.builder()  
            .objectIntrospector(ConstructorPropertiesArbitraryIntrospector.INSTANCE)  
            .plugin(new JakartaValidationPlugin())  
            .build();  
  
    Product actual = fixtureMonkey.giveMeOne(Product.class);  
  
    assertThat(actual).isNotNull();  
    assertThat(actual.getId()).isGreaterThan(0);  
    assertThat(actual.getProductName()).isNotBlank();  
    assertThat(actual.getPrice()).isLessThanOrEqualTo(100000);  
    assertThat(actual.getOptions().size()).isGreaterThanOrEqualTo(3);  
    assertThat(actual.getOptions()).allSatisfy(it -> assertThat(it).isNotEmpty());  
    assertThat(actual.getCreatedAt()).isBefore(Instant.now());  
}
```


### 사용자 정의 객체 생성하기 
- 테스트 객체를 커스터마이즈 가능

상품 가격이 1,000원 이상인 경우에만 10% 할인이 적용하는 서비스를 테스트한다고 가정
```java
@Value  
public class Product {  
    long id;  
    String productName;  
    long price;  
    List<String> options;  
    Instant createdAt;  
}
```

테스트 생성
```java
private FixtureMonkey fixtureMonkey;  
  
@BeforeEach  
void setUp() {  
    fixtureMonkey = FixtureMonkey.builder()  
            .objectIntrospector(ConstructorPropertiesArbitraryIntrospector.INSTANCE)  
            .build();  
}  
  
@Test  
void customizePrice() {  
    Product product = fixtureMonkey.giveMeBuilder(Product.class)  
            .set("price", 2000L)  
            .sample();  
  
    DiscountService discountService = product1 -> 200.0;  
    double discount = discountService.calculateDiscount(product);  
  
    assertThat(discount).isEqualTo(200.0);  
}  
  
private interface DiscountService {  
    double calculateDiscount(Product product);  
}  
  
@Test  
void testProductWithOptions() {  
    Product actual = fixtureMonkey.giveMeBuilder(Product.class)  
            .size("options", 3)  
            .set("options[1]", "red")  
            .sample();  
  
    assertThat(actual.getOptions()).hasSize(3);  
    assertThat(actual.getOptions().get(1)).isEqualTo("red");  
}
```
- 서비스가 없어서 간단히 functionalInterface 선언해 람다식으로 처리

### 주의사항과 팁
1. 필드이름
	1. 클래스에 정의된 필드 이름을 정확히 사용
	2. Tip. IDE의 코드 완성 기능을 사용하여 필드 이름 오타 방지 
	3. Tip. 향상된 코드 완성과 타입 안정성을 위해 [Fixture Monkey Helper](https://plugins.jetbrains.com/plugin/19589-fixture-monkey-helper)를 설치 (Intellij plugin)
2. 컬렉션 인덱싱
	1. 리스트 인덱스는 0부터 시작한다는 것을 기억 
	2. Tip. 특정 인덱스를 설정하기 전에 `size()`를 사용하여 리스트 크기를 먼저 설정하기
3. 타입 안정성
	1. 값의 타입을 올바르게 사용해야 함
	2. Tip. IDE의 타입 힌트를 활용해 올바른 값 타입을 사용하기


### Fixture Monkey 사용 전후 비교 

**before**
```java
// 특정 옵션을 가진 상품 생성
List<String> options = new ArrayList<>();
options.add("option1");
options.add("red");
options.add("option3");
Product product = new Product(1, "테스트 상품", 1000, options, Instant.now());

```

**after**
```java
// 같은 결과를 더 적은 코드로 생성
Product product = fixtureMonkey.giveMeBuilder(Product.class)
    .size("options", 3)
    .set("options[1]", "red")
    .sample();
```


### 초보자를 위한 팁 (읽어보기)

[링크](https://naver.github.io/fixture-monkey/v1-1-0-kor/docs/get-started/tips/)


---
## 객체 생성

### Fixture Monkey

**🧰 FixtureMonkey 인스턴스 생성**
- Java에서는 간단히 `create()` 메서드로 생성할 수 있음
- Kotlin은 전용 플러그인을 추가해야 함
```java
// 기본 설정으로 생성 
FixtureMonkey fixtureMonkey = FixtureMonkey.create();
```

builder를 사용해 테스트 객체 생성 방식을 조정 가능 (예. 인트로스펙터 전략 설정)
```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
    + 옵션들...
    .build();
```

**🧰 객체 생성하기**
`FixtureMonkey` 클래스는 다양한 방식으로 테스트에 필요한 객체를 생성할 수 있는 메서드를 제공
- `giveMeOne()` : 기본 랜덤 값을 가진 단일 객체가 필요할 때
- `giveMe()` : 기본 랜덤 값을 가진 여러 객체가 필요할 때
- `giveMeBuilder()` : 속성을 세부 조정한 객체가 필요할 때
- `giveMeArbitrary()` : jqwik의 Arbitrary API를 활용한 고급 사용 사례

**✅ giveMeOne**
- 특정 타입의 객체 하나가 필요할 때 giveMeOne을 사용
```java
private FixtureMonkey fixtureMonkey = FixtureMonkey.builder()  
        .objectIntrospector(new FailoverIntrospector(  
        Arrays.asList(ConstructorPropertiesArbitraryIntrospector.INSTANCE,  
                            BuilderArbitraryIntrospector.INSTANCE,  
                            FieldReflectionArbitraryIntrospector.INSTANCE,  
                            BeanArbitraryIntrospector.INSTANCE  
                            ), false))  
        .build();  
  
@DisplayName("특정 타입의 객체 하나가 필요할 때 giveMeOne 사용")  
@Test  
void giveMeOne() {  
    Product product = fixtureMonkey.giveMeOne(Product.class);  
  
    assertThat(product).isNotNull();  
}  
  
@DisplayName("빈 리스트나 임의 길이를 가진 문자열 리스트를 반환")  
@Test  
void getMeOne2() {  
    List<String> strings = fixtureMonkey.giveMeOne(new TypeReference<List<String>>() {});  
  
    assertThat(strings).isNotNull();  
}
```

**✅ giveMe**
- 동일한 타입의 객체 여러 개가 필요한 경우 giveMe 사용
- 원하는 개수를 지정해 스트림이나 리스트 형태로 객체를 생성
```java
@Test  
void giveMe() {  
    Stream<Product> productStream = fixtureMonkey.giveMe(Product.class);  
  
    assertThat(productStream).isNotNull();  
}  
  
@DisplayName("인자가 하나면 Stream, 두 개면 List<?> 타입으로 반환한다")  
@Test  
void giveMe2() {  
    Stream<List<String>> stringStream = fixtureMonkey.giveMe(new TypeReference<List<String>>() {});  
    List<List<String>> lists = fixtureMonkey.giveMe(new TypeReference<List<String>>() {  
    }, 3);  
  
    assertThat(stringStream).isNotNull();  
    assertThat(lists).isNotNull()  
            .hasSize(3);  
  
    lists.forEach(System.out::println);  
}
```


**✅ giveMeBuilder**
- 객체의 속성을 세부적으로 조정해야 할 때는 giveMeBuilder를 사용
- 속성 조정 방법에 대한 자세한 내용은 [객체 커스터마이징 문서](https://naver.github.io/fixture-monkey/v1-1-0-kor/docs/customizing-objects/apis/) 참고 
- ArbitraryBuilder에서 최종 객체를 얻으려면 `sample()`, `sampleList()`, `sampleStream()`를 사용합니다
```java
@Test  
void giveMeBuilder() {  
    ArbitraryBuilder<Product> productArbitraryBuilder = fixtureMonkey.giveMeBuilder(Product.class)  
            .set("productName", "테스트 상품");  
  
    Product product = productArbitraryBuilder.sample();  
  
    assertThat(product).isNotNull()  
            .extracting("productName")  
            .matches(s -> s.equals("테스트 상품"));  
}  
  
@Test  
void giveMeBuilder2() {  
    ArbitraryBuilder<Product> productArbitraryBuilder = fixtureMonkey.giveMeBuilder(Product.class)  
            .set("productName", "깨끗한 세상");  
  
    Product product = productArbitraryBuilder.sample();  
    List<Product> products = productArbitraryBuilder.sampleList(3);  
  
    assertThat(product).isNotNull()  
                    .extracting("productName")  
                            .matches(s -> s.equals("깨끗한 세상"));  
    assertThat(products).hasSize(3)  
            .extracting(Product::getProductName)  
            .containsOnly("깨끗한 세상");  
}
```


**✅ giveMeArbitrary**
- Arbitrary 객체가 직접 필요한 경우 getMeArbitrary() 메서드를 사용
- 용도를 잘 모르겠다🤔
	- sample(), sampleStream() 이 있기는 한데 ..

```java
@Test  
void giveMeArbitrary() {  
    Arbitrary<Product> productArbitrary = fixtureMonkey.giveMeArbitrary(Product.class);  
  
    Arbitrary<List<String>> strListArbitrary = fixtureMonkey.giveMeArbitrary(new TypeReference<List<String>>() {});  
  
    assertThat(productArbitrary).isNotNull();  
    assertThat(strListArbitrary).isNotNull();  
}

```

### 복잡한 객체 생성하기
- [공식 문서](https://naver.github.io/fixture-monkey/v1-1-0-kor/docs/generating-objects/generating-complex-types/)

**✅ 테스트에서 복잡한 타입이 중요한 이유**
실제 테스트는 다음과 같은 복잡한 객체로 작업하는 경우가 많음
- 여러 타입 파라미터를 가진 제네릭 타입
- 트리나 그래프 같은 자기 참조 구조
- 복잡한 인터페이스 계층 구조
- sealed 클래스나 추상 클래스

테스트를 위해 이런 타입의 인스턴스를 수동 생성하는 것은 매우 지루하고 오류 발생하기 쉽다💩
➡️이런 경우 Fixture Monkey가 빛을 발합니다. (**최소한의 코드로 가장 복잡한 타입의 유효한 인스턴스까지 자동 생성 가능**)

**✅ Fixture Monkey가 복잡한 타입을 처리하는 방법**
- 런타임에 클래스와 인터페이스의 구조를 분석하여 관계와 제약 조건을 이해함
	- 그런 다음 중첩되고 재귀적인 구조에서도 필요한 모든 필드가 채워진 유효한 인스턴스를 생성
- 인터페이스의 경우 
	- 하나의 인터페이스에 여러 구현체가 있으면 사용 가능한 구현체 중 하나를 무작위로 선택하여 생성함
	- `InterfacePlugin` 설정을 통해 특정 구현체를 명시적으로 지정할 수도 있음 

```java
// 👋 UserService 인터페이스에 대해 여러 구현체 지정 예시
FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
    .plugin(
        new InterfacePlugin()
            .interfaceImplements(UserService.class, 
                List.of(BasicUserService.class, PremiumUserService.class))
    )
    .build();

// 지정된 구현체 중 하나가 무작위로 선택됨
UserService userService = fixtureMonkey.giveMeOne(UserService.class);
```


**✅ Generic Objects ** 
```java
private FixtureMonkey fixtureMonkey = FixtureMonkey.create();  
  
@Test  
void genericObject() {  
    GenericeObject<String> stringGenericeObject = fixtureMonkey.giveMeOne(new TypeReference<GenericeObject<String>>() {});  
  
    assertThat(stringGenericeObject).isNotNull();  
}  
  
@Test  
void genericArrayObject() {  
    GenericArrayObject<Integer> genericArrayObject = fixtureMonkey.giveMeOne(new TypeReference<GenericArrayObject<Integer>>() {});  
  
    assertThat(genericArrayObject).isNotNull();  
}  
  
@Test  
void twoGenericObject() {  
    TwoGenericObject<String, Integer> stringIntegerTwoGenericObject = fixtureMonkey.giveMeOne(new TypeReference<TwoGenericObject<String, Integer>>() {  
    });  
    assertThat(stringIntegerTwoGenericObject).isNotNull();  
}  
  
private static class GenericeObject<T> {  
    T foo;  
  
    public GenericeObject() {}  
  
    public void setFoo(T foo) {  
        this.foo = foo;  
    }}  
  
private static class GenericArrayObject<T> {  
    GenericArrayObject<T> foo;  
  
    public GenericArrayObject() {}  
  
    public void setFoo(GenericArrayObject<T> foo) {  
        this.foo = foo;  
    }}  
  
private static class TwoGenericObject<T, U> {  
    T foo;  
    U bar;  
  
    public TwoGenericObject() {}  
  
    public void setFoo(T foo) {  
        this.foo = foo;  
    }  
    public void setBar(U bar) {  
        this.bar = bar;  
    }}

```


**✅ Generic Interfaces** 
- 예제를 그대로 따라해서 기본 create()로 생성된 Fixture Monkey 사용하면 null이 반환된다.
- 인터페이스와 구현체를 아래와 같이 지정해야 하는 것으로 생각된다
```java
@Test  
void genericInterface() {  
    FixtureMonkey customFixtureMonkey = FixtureMonkey.builder()  
            .plugin(new InterfacePlugin().interfaceImplements(GenericInterface.class, List.of(GenericInterfaceImpl.class)))  
            .build();  
  
    GenericInterface<String> stringGenericInterface = customFixtureMonkey.giveMeOne(new TypeReference<>() {});  
  
    assertThat(stringGenericInterface).isNotNull();  
}
```



**✅ SelfReference** 
자기 참조 타입은 수동으로 생성하기 특히 어렵지만 Fixture Monkey를 사용하면 쉬움
```java
@Test  
void selfReference() {  
    SelfReference selfReference = fixtureMonkey.giveMeOne(SelfReference.class);  
  
    assertThat(selfReference).isNotNull();  
}  
  
public static class SelfReference {  
    String foo;  
    SelfReference bar;  
  
    public SelfReference() {}  
  
    public void setFoo(String foo) {  
        this.foo = foo;  
    }  
    public void setBar(SelfReference bar) {  
        this.bar = bar;  
    }}  
  
private static class SelfReferenceList {  
    String foo;  
    List<SelfReferenceList> bar;  
  
    public SelfReferenceList() {}  
  
    public void setFoo(String foo) {  
        this.foo = foo;  
    }  
    public void setBar(List<SelfReferenceList> bar) {  
        this.bar = bar;  
    }}
```

컬렉션 예시에서 `defaultArbitraryContainerInfo`가 없어서 테스트 못함 (5/23)
```java
// 재귀 깊이를 제어하기 위한 사용자 정의 구성
FixtureMonkey customFixture = FixtureMonkey.builder()
    .defaultArbitraryContainerInfo(new ContainerInfo(2, 2)) // 리스트 크기 제어
    .build();
    
SelfReferenceList refList = customFixture.giveMeOne(SelfReferenceList.class);
```

**✅ Interface** 

```java
@Test  
void paymentProcessor() {  
    FixtureMonkey customFixtureMonkey = FixtureMonkey.builder()  
            .plugin(new InterfacePlugin().interfaceImplements(PaymentProcessor.class, List.of(CreditCardProcessor.class, BankTransferProcessor.class)))  
            .build();  
  
    PaymentProcessor paymentProcessor = customFixtureMonkey.giveMeOne(PaymentProcessor.class);  
  
    assertThat(paymentProcessor).isNotNull();  
}  
  
private interface PaymentProcessor {  
    void processPayment(double amount);  
}  
  
private static class CreditCardProcessor implements PaymentProcessor {  
    @Override  
    public void processPayment(double amount) {  
        // 신용 카드 결제 처리 로직  
    }  
}  
  
private static class BankTransferProcessor implements PaymentProcessor {  
    @Override  
    public void processPayment(double amount) {  
        // 계좌 이체 결제 처리 로직  
    }  
}
```


아래 예제는 테스트 안했으나 위와 똑같아 보임, Interface 타입의 구현체를 지정해서 임의로 만들어지는 걸로 생각됨
```java
public interface Interface {
   String foo();

   Integer bar();
}

public interface InheritedInterface extends Interface {
   String foo();
}

public interface InheritedInterfaceWithSameNameMethod extends Interface {
   String foo();
}

public interface ContainerInterface {
   List<String> baz();

   Map<String, Integer> qux();
}

public interface InheritedTwoInterface extends Interface, ContainerInterface {
}
```


>[!info] Kotlin은 생략 👍

### 객체 생성 방법 지정하기

Fixture Monkey로 객체를 생성하는 기본 방법 (대부분의 경우 충분)
```java
// 기본 방식 - 인트로스펙터가 자동으로 객체 생성 방법 결정
Product product = fixtureMonkey.giveMeOne(Product.class);
```

하지만 특정 생성자나 팩토리 메서드를 사용하고 싶다면` instaniate() `메서드를 사용 ✨
```java
// 특정 생성자 지정
Product product = fixtureMonkey.giveMeBuilder(Product.class)
    .instantiate(constructor())
    .sample();

// 특정 팩토리 메서드 지정
Product product = fixtureMonkey.giveMeBuilder(Product.class)
    .instantiate(factoryMethod("create"))
    .sample();
```


**📋 기본 개념**

**✅ ArbitraryBuilder란?**
- `ArbitraryBuilder`는 객체 생성 설정을 구성하기 위한 빌더 클래스이다
```java
// ArbitraryBuilder 얻기
ArbitraryBuilder<Product> builder = fixtureMonkey.giveMeBuilder(Product.class);
```

**✅ instantiate() 메서드란?**
- `ArbitraryBuilder`에서 객체를 어떻게 생성할지 지정하는 메서드
- **생성자**나 **팩터리 메서드** 중 하나를 선택 가능 
	- 세부적으로 파라미터 타입, 파라미터 이름 설정하여 지정 가능
```java
// Java에서 생성자 지정
.instantiate(constructor())

// Java에서 팩토리 메서드 지정
.instantiate(factoryMethod("methodName"))
```

>[!info] 기본은 no-args 생성자 + setter 조합을 사용, 특별하게 팩토리 메서드 사용하는 경우 `instantiate()` 메서드로 지정


**📋 4. 고급 기능과 주의사항**

**✅ 4.1 필드 vs JavaBeansProperty 선택하기**
1. `field()` 
	1. 클래스의 필드 기반으로 속성 생성 
		1. 장점: 직접 필드에 접근하므로 setter 없어도 됨
		2. 단점: 캡슐화 우회, 유효성 검사 로직 무시
2. javaBeansProperty()
	1. getter/setter 메서드를 기반으로 속성 생성
		1. 장점: 캡슐화 유지, setter 유효성 검사 로직 활용 
		2. 단점: setter가 없으면 속성 설정 불가

> `instantiate()`에서 설정하는거구나

**💁‍♂️Tip**
- setter 메서드에 유효성 검사 로직이 있고 이를 테스트 하고 싶다면 
	- javaBeansProperty()
- setter 메서드가 없거나 유효성 검사를 우회하고 싶다면
	- `field()`

**field 기반 속성 생성** 
```java
@Test
void 필드_기반_속성_생성() {
    Product product = fixtureMonkey.giveMeBuilder(Product.class)
        .instantiate(
            constructor().field()  // 필드 기반 속성 생성
        )
        .sample();
    
    assertThat(product).isNotNull();
}
```

**javaBeansProperty 기반 속성 생성**
```java
@Test
void JavaBeansProperty_기반_속성_생성() {
    Product product = fixtureMonkey.giveMeBuilder(Product.class)
        .instantiate(
            constructor().javaBeansProperty()  // JavaBeansProperty 기반 속성 생성
        )
        .sample();
    
    assertThat(product).isNotNull();
}
```


**✅ 4.2 생성자 이후 속성 설정 주의사항**
- [공식문서](https://naver.github.io/fixture-monkey/v1-1-0-kor/docs/generating-objects/instantiate-methods/#42-%ec%83%9d%ec%84%b1%ec%9e%90-%ec%9d%b4%ed%9b%84-%ec%86%8d%ec%84%b1-%ec%84%a4%ec%a0%95-%ec%a3%bc%ec%9d%98%ec%82%ac%ed%95%ad)
- 생성자를 지정한 이후에 속성 값을 세팅할 수 있다는 내용


**📚 자주 묻는 질문(FAQ)**
- [공식문서](https://naver.github.io/fixture-monkey/v1-1-0-kor/docs/generating-objects/instantiate-methods/#%EC%9E%90%EC%A3%BC-%EB%AC%BB%EB%8A%94-%EC%A7%88%EB%AC%B8-faq)

> [!info] 대부분의 경우 인트로스펙터만으로 충분하지만, 특별한 생성 로직이 필요할때 instantiate를 사용


### 인터페이스 타입 생성하기
- 기본 예제인데 npe 예외 발생함
	- `Cannot invoke "fixturemonky.InterfaceTypeObjectTest$StringSupplier.getValue()" because "stringSupplier" is null`
```java
@Test  
void test() {  
    FixtureMonkey fixtureMonkey = FixtureMonkey.create();  
  
    StringSupplier stringSupplier = fixtureMonkey.giveMeOne(StringSupplier.class);  
  
    assertThat(stringSupplier.getValue()).isNotNull();  
}  
  
private interface StringSupplier {  
    String getValue();  
}
```



### 인트로스펙터
- `인트로스펙터(Introspector)`는 테스트 객체를 생성하는 방법을 결정하는 도구
- 다음과 같은 역할을 수행합니다
	- 생성자나 빌더 중 어떤 방식으로 객체를 생성할지 결정
	- 필드 값을 어떻게 설정할지 결정
	- 코드 베이스의 다양한 클래스 타입을 어떻게 처리할지 결정

**✨인트로스펙터가 중요한 이유**
다양한 프로젝트는 객체 생성을 위한 다양한 패턴을 사용합니다
- 일부는 getter/setter가 있는 간단한 클래스를 사용
- 일부는 생성자를 사용하는 불변 객체를 사용
- 일부는 빌더 패턴을 따름
- Lombok과 같은 프레임워크는 특정 방식으로 코드를 생성

> 적절한 인트로스펙터를 선택하면 **기존 코드를 수정하지 않고**도 Fixture Monkey를 활용할 수 있어 시간과 노력을 절약할 수 있음 👋


👍**권장되는 설정**
- 대부분의 프로젝트에서 잘 작동함
- 아래 서정은 여러 전략을 조합하여 다양한 클래스 타입을 처리하므로, 대부분의 실제 프로젝트에서 추가 설정 없이 잘 작동함
```java
// 대부분의 클래스 타입을 처리할 수 있는 권장 설정
FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
    .objectIntrospector(new FailoverIntrospector(
        Arrays.asList(
            ConstructorPropertiesArbitraryIntrospector.INSTANCE,
            BuilderArbitraryIntrospector.INSTANCE,
            FieldReflectionArbitraryIntrospector.INSTANCE,
            BeanArbitraryIntrospector.INSTANCE
        ),
        false // 더 깔끔한 테스트 출력을 위해 로깅 비활성화
    ))
    .build();
```


🎯**가장 간단한 접근법**
```java
// 기본 설정으로 가장 간단한 접근법 
FixtureMonkey fixtureMonkey = FixtureMonkey.builder().build();
```

> [!info] 기본 접근법은 기본 생성자 + setter 메서드가 있는 간단한 JavaBean 클래스에서만 잘 작동함


**✅ 클래스에 맞는 인트로스펙터 선택하기**
- 상황에 맞게 Introspector를 설정해서 사용할 수 있음 
- 👍권장되는 설정으로 빠르게 시작하고, 커스텀하는 것도 방법일 듯

| 클래스 타입                         | 권장 인트로스펙터                                    | 예시                        |
| ------------------------------ | -------------------------------------------- | ------------------------- |
| 기본 생성자 + Setter<br>(JavaBeans) | `BeanArbitraryIntrospector`                  | getter/setter가 있는 클래스     |
| 생성자가 있는 불변 클래스                 | `ConstructorPropertiesArbitraryIntrospector` | 레코드, 어노테이션된 생성자가 있는 클래스   |
| 흔한 필드 접근 방식의 클래스               | `FieldReflectionArbitraryIntrospector`       | public 필드, 기본 생성자가 있는 클래스 |
| 빌더 패턴을 사용하는 클래스                | `BuilderArbitraryIntrospector`               | `.builder()` 메서드가 있는 클래스  |
| 다양한 패턴이 섞인 코드 베이스              | `FailoverArbitraryIntrospector`<br>          | 다양한 클래스 타입이 있는 프로젝트       |

롬북 사용하지 않고 빌더 패턴을 직접 구현하는 경우 Builder 받는 생성자와 builder() 메소드가 필요
```java
public class User {  
    private final String username;  
    private final String email;  

	// 둘다 필요
    private User(Builder builder) {  
        this(builder.username, builder.email);  
    }  

	public User(String username, String email) {  
	    this.username = username;  
	    this.email = email;  
	}

    public static Builder builder() {  
        return new Builder();  
    }  
    
    public static class Builder {  
        private String username;  
        private String email;  
  
        public Builder username(String username) {  
            this.username = username;  
            return this;  
        }  
        public Builder email(String email) {  
            this.email = email;  
            return this;  
        }  
        public User build() {  
            return new User(username, email);  
        }    }  
    public String getUsername() {  
        return username;  
    }  
    public String getEmail() {  
        return email;  
    }}
```


🎯 **Q&A**

**Q**: 객체가 생성되지 않는 경우
**A**: 클래스가 다음 중 하나를 가지고 있는지 확인
- 기본 생성자와 setter 가진 객체 ➡️ `BeanArbitraryIntrospector` 
- **@ConstrucotrProperties**가 붙은 생성자 가진 객체 ➡️ `ConstructorPropertiesArbitraryIntrospector`
- 빌더 메서드 가진 객체 ➡️ `BuilderArbitraryIntrospector`

**Q**: Lombok을 사용 중인데 객체가 제대로 생성되지 않는 경우
**A**: **lombok.config** 파일에 `lombok.anyConstructor.addConstructorProperties=true`를 추가하고 `ConstructorPropertiesArbitraryIntrospector`를 사용👍

**Q**: 특정 클래스에 대해 사용자 정의 생성 로직이 필요하면 어떻게 하나요?
**A**: 특정 케이스의 경우 `instantiate` 메서드를 사용하여 인스턴스 생성 방법을 지정할 수 있습니다

```java
MySpecialClass object = fixtureMonkey.giveMeBuilder(MySpecialClass.class) .instantiate(() -> new MySpecialClass(specialParam1, specialParam2)) .sample();
```
- 고급 사용자 정의 로직은 `사용자 정의 인트로스펙터` 가이드 참조
	- 대부분의 사용자는 이 기능이 필요하지 않을 것입니다


**⚙️ 사용 가능한 인트로스펙터 (상세 정보)**

**BeanArbitraryIntrospector (기본값)**
- 기본 생성자 + setter 메서드 필요
- Introspector 기본 값이라 생략 가능 (?)
```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder() .objectIntrospector(BeanArbitraryIntrospector.INSTANCE) // 이것이 기본값입니다 .build();
```


**ConstructorPropertiesArbitraryIntrospector**
- 대상: 생성자가 있는 불변 객체
- `@ConstructorProperties`가 있는 생성자가 있거나, 레코드 타입이어야 함
- Lombok의 경우 lombok.config에 설정 추가 필요
	- `lombok.anyConstructor.addConstructorProperties=true`
```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder() .objectIntrospector(ConstructorPropertiesArbitraryIntrospector.INSTANCE) .build();
```


**FieldReflectionArbitraryIntrospector**
- 대상: 필드 접근 방식의 클래스
	- 기본 생성자 필요
	- 필드는 리플렉션을 통해 접근 가능 
```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder() .objectIntrospector(FieldReflectionArbitraryIntrospector.INSTANCE) .build();
```


**BuilderArbitraryIntrospector**
- 대상: 빌더 패턴을 사용하는 클래스 
	- 클래스에 builder() 메소드와 Builder를 주입받는 생성자가 필요
```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder() .objectIntrospector(BuilderArbitraryIntrospector.INSTANCE) .build();
```


**FailoverArbitraryIntrospector (혼합 코드 베이스에 권장)**
- 대상: 다양한 클래스 타입이 섞인 프로젝트
- 장점
	- 여러 인트로스펙터를 순차적으로 시도 
	- 다양한 클래스 패턴에 작동
	- 가장 다용로도 사용 가능한 옵션

```java
FixtureMonkey fixtureMonkey = FixtureMonkey.builder() .objectIntrospector(new FailoverIntrospector( 
	Arrays.asList( ConstructorPropertiesArbitraryIntrospector.INSTANCE, BuilderArbitraryIntrospector.INSTANCE, FieldReflectionArbitraryIntrospector.INSTANCE, BeanArbitraryIntrospector.INSTANCE 
), false )) .build(); // false: 로깅 비활성화
```

>[!warning] 순차적으로 인트로스펙터를 시도하기 때문에 객체 생성 비용이 증가할 수 있습니다. 성능이 중요한 경우 클래스 패턴을 알고 있다면 특정 인트로스펙터 사용을 권장


**PriorityConstructorArbitraryIntrospector**
- 대상: 다른 인트로스펙터가 작동하지 않는 특별한 경우
- 장점
	- `@ConstructorProperties` 없이도 사용 가능한 생성자 사용
	- 수정할 수 없는 라이브러리 클래스에 유용

>[!info] 플로그인 추가로 인트로스펙터를 추가 제공
>- [`JacksonObjectArbitraryIntrospector`](https://naver.github.io/fixture-monkey/v1-1-0-kor/docs/plugins/jackson-plugin/jackson-object-arbitrary-introspector): Jackson JSON 객체용
>- [`PrimaryConstructorArbitraryIntrospector`](https://naver.github.io/fixture-monkey/v1-1-0-kor/docs/plugins/kotlin-plugin/introspectors-for-kotlin): Kotlin 클래스용

