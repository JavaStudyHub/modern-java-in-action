## 10.3 자바로 DSL을 만드는 패턴과 기법

**DSL은 특정 도메인 모델에 적용할 친화적이고 가독성 높은 API를 제공한다.**

우선 몇 가지 도메인 모델을 정의해보며 앞으로 사용할 DSL을 만드는 패턴을 살펴보자.

1. 주어진 시장에 주식 가격을 모델링 하는 순수 자바 빈즈
    
    ```java
    public class Stock {
    		private String symbol;
    		private String market;
    		
    		...
    }
    ```
    

1. 주어진 가격에서 주어진 양의 주식을 사거나 파는 거래
    
    ```java
    public class Trade {
    		public enum Type {BUY, SELL}
    		private Type type;
    		
    		private Stock stock;
    		private int quantity;
    		private double price;
    		
    		...
    }
    ```
    

1. 고객이 요청한 한 개 이상의 거래의 주문
    
    ```java
    public class Order {
    		private String customer;
    		private List<Trade> trades = new ArrayList<>();
    		
    		...
    }
    ```
    

이번에는 고객이 요청한 두 거래를 포함하는 주문을 만들어보자.

```java
Order order = new Order();
order.setCustomer("BigBank");

Trade trade1 = new Trade();
trade1.setType(Trade.Type.BUY);

Stock stock1 = new Stock();
stock1.setSymbol("IBM");
stock1.setMarket("NYSE");

trade1.setStock(stock1);
....
order.addTrade(trade1);

Trade trade2 = new Trade();
trade2.setType(Trade.Type.BUY);

...
order.addTrade(trade2);
```

한 눈에 봐도 알 수 있지만 위의 코드는 상당히 장황하다. 따라서 좀 더 직접적이고, 직관적으로 도메인 모델을 반영할 수 있는 DSL이 필요하다.

<br>

### 10.3.1 메서드 체인

메서드 체인으로 거래 주문을 정의해보자

```java
Order order = forCustomer("BigBank")
				.buy(80)
				.stock("IBM")
				.on("NYSE")
				.at(125.00)
				.sell(50)
				.stock("GOOGLE")
				.on("NASDAQ")
				.at(375.00)
				.end();
```

상당히 코드가 개선된 것을 볼 수 있다. 이 코드는 비 개발자인 도메인 전문가가 보아도 쉽게 이해할 수 있을만한 코드이다.

위처럼 코드를 작성하기 위해서는 플루언트 API로 도메인 객체를 만드는 몇 개의 빌더를 구현해야한다.

```java
public class MethodChainingOrderBuilder {

		public final Order order = new Order();
		
		public static MethodChainingOrderBuilder forCustomer(String customer) {
				order.setCustomer(customer);
		}
		
		public static MethodChainingOrderBuilder forCustomer(String customer) {
				return new MethodChainingOrderBuilder(customer);
		}
		
		public TradeBuilder buy(int quantity) {
				return new TradeBuilder(this, Trade.Type.BUY, quantity);
		}
		
		...
}

public class TradeBuilder {
		private final MethodChainingOrderBuilder builder;
		public final Trade trade = new Trade();
		
		...
		
		public StockBuilder stock(String symbol) {
				return new StockBuilder(builder, trade, symbol);
		}
}

public class StockBuilder {
		private final MethodChainingOrderBuilder builder;
		public final Trade trade;
		private final Stock stock = new Stock();
		
		...
		
		public TradeBuilderWithStock on(String market) {
				...
		}
}

public class TradeBuilderWithStock {
		private final MethodChainingOrderBuilder builder;
		private final Trade trade;
		
		...
}
```

이처럼 자기 자신을 담는 객체를 만들어서 반환하는 식으로 빌더를 구현해야한다.

이렇게 구현을 하게되면 MethodChainingBuilder가 끝날 때까지 다른 거래를 플루언트 방식으로 추가할 수 있다.

또한 각각의 빌드 클래스 간에 순서를 지정해줌으로써 절차에 따라 메서드들을 호출하도록 강제할 수 있다.

이렇게 빌더를 구현함으로써 메서드 이름이 인수의 이름을 대신하도록 만들어 DSL의 가독성을 높일 수 있다.

하지만 빌더를 구현해야한다는 점, 상위 수준의 빌더를 하위 수준의 빌더와 연결할 많은 접착 코드가 필요하다는 것이 메서드 체인의 단점이다.

<br>

### 10.3.2 중첩된 함수 이용

> **다른 함수 안에 함수를 이용해 도메인 모델을 만드는 형태**
> 

```java
Order order = order("BigBank", 
				buy(80,
					stock("IBM", on("NYSE")), at(125.00),
				sell(50,
					stock("GOOGLE", on("NASDAQ")), at(375.00)
				));
```

```java
public class NestedFunctionOrderBuilder {
		
		...
		
		public static Trade buy(int quantity, Stock stock, double price) {
				return buildTrade(....);
		}
		
		public static Trade sell(int quantity, Stock stock, double price) {
				return buildTrade(....);
		}
		
		private static Trade buildTrade(int quantity, Stock stock, double price, Trade.Type buy) {
				....
				return trade;
		}
		
		...
```

중첩 함수 방식은 메서드 체인에 비해 함수의 중첩 방식이 도메인 **객체 계층 구조**에 그대로 반영된다는 것이 장점

- **문제점**
    - 더 많은 괄호가 필요하다.
    - 인수 목록을 정적 메서드에 넘겨주어야한다.
    - 인수를 생략하는 경우가 존재할 수 있으므로, 여러 메서드 오버라이드를 구현해야한다.
    - **인수의 의미가 이름이 아니라 위치에 의해 정의**된다.

### 10.3.3 람다 표현식을 이용한 함수 시퀀싱

```java
Order order = order( o -> {
		o.forCustomer( "BigBank" );
		o.buy( t -> {
			t.quantity( 80 );
			t.price( 125.00 );
			t.stock( s -> {
				s.symbol( "IBM" );
				s.market( "NYSE" );
			})；
		})；
		o.sell( t -> {
			t.quantity( 50 );
			t.price( 375.00 );
			t.stock( s -> {
				s.symbol( "GOOGLE" );
				s.market( "NASDAQ" );
			})；
		})；
})；
```

위 코드를 구현을 하기 위해서는 **람다 표현식**을 받아 실햄함으로써 도메인을 만들어 내는 여러 빌더를 구현해야 한다.

```java
public class LambdaOrderBuilder {
		private Order order = new Order();
		
		public static Order order(Consumer<LambdaOrderBuilder> consumer) {
				LambdaOrderBuilder builder = new LambdaOrderBuilder();
				consumer.accept(builder);
				return builder.order;
		}
		
		...
		
		public void buy(Consumer<TradeBuilder> consumer) {
				trade(consumer, Trade.Type.BUY);
		}
		
		public void sell(Consumer<TradeBuilder> consumer) {
				trade(consumer, Trade.Type.SELL);
		}
		
		...
		
		private void trade(Consumer<TradeBuilder> consumer, Trade.Type type) {
				...
		}
}
```

이런 식으로 구현할 시 이전 두 가지 DSL 형식의 장점을 더할 수 있다.

1. **메서드 체인 패턴**
    
    > **플루언트 방식으로 거래 주문 정의 가능**
    > 
2. **중첩 함수 형식**
    
    > **도메인 객체의 계층 구조 유지 가능**
    > 

<br>

이 두 가지를 더해서 플루언트 방식으로 주문을 정의함과 동시에 다양한 람다 표현식의 중첩 수준과 비슷하게 도메인 객체의 계층 구조를 유지할 수 있게 된다.

다만 많은 설정 코드가 필요하며, DSL 자체가 자바 8 람다 표현식 문법에 의한 잡읍의 영향을 받는다는 것이 단점이다.

<br>

### 10.3.4 조합하기

앞서 살펴본 세 가지 DSL 패턴은 각자가 장단점을 갖는다.

이 때 한 DSL에 한 개의 패턴만 사용하라는 법은 없다. 따라서 새로운 DSL을 개발해 주식 거래 주문을 정의할 수 있다.

```java
Order order = forCustomer("BigBank",
					buy(t -> t.quantity(80)
								.stock("IBM")
								.on("NYSE")
								.at(125.00)
					sell(t -> t.quantity(50)
								.stock("GOOGLE")
								.on("NASDAQ")
								.at(125.00)));
```

이처럼 중첩된 함수 패턴과 람다 기법을 혼용할 수 있다.

이렇듯 여러 장점을 모아서 이용할 수 있지만, 러닝 커브가 가파르다는 결점이 존재한다.

<br>

### 10.3.5 DSL에 메서드 참조 사용하기

**예시 - 주문의 총합에 0개 이상의 세금을 추가해 최종값을 계산하는 기능**

```java
public class Tax {
		public static double regional(double value) {
				return value + 1.1;
		}
		
		public static double general(double value) {
				return value * 1.3;
		}
		
		public static double surcharge(double value) {
				return value * 1.05;
		}
}
```

**불리언 플래그를 이용해 주문에 세금 적용**

```java
public static double calculate(Order order, boolean useRegional, boolean useGeneral, boolean useSurcharge) {
		double value = order.getValue();
		if(userRegional) value = Tax.regional(value);
		if(useGeneral) value = Tax.general(value);
		if(useSurcharge) value = Tax.surcharge(value);
		return value;
}
```

지역 세금과 추가 요금을 적용하고, 일반 세금은 뺀 주문의 최종값 계산

```java
double value = calculate(order, true, false, true);
```

이 코드는 딱 봐도 가독성에 문제가 존재한다. 불리언 변수의 올바른 순서를 기억하기 어렵고, 어떤 세금이 적용되었는지도 파악하기 힘들다.

따라서 최소 DSL을 제공하는 TaaxCalculator 클래스를 정의하여 가독성을 높이는 방법을 알아보자.

<br>

**적용할 세금을 정의하는 세금 계산기**

```java
public class TaxCalculator {
		private boolean useRegional;
		private boolean useGeneral;
		private boolean useSurcharge;
		
		public TaxCalculator withTaxRegional() {
				useRegional = true;
				return this;
		}
		
		public TaxCalculator withTaxGeneral() {
				useGeneral = true;
				return this;
		}
		
		public TaxCalculator withTaxSurcharge() {
				useSurcharge = true;
				return this;
		}
		
		public double calculate(Order order) {
				return calculate(order, useRegional, useGeneral, useSurcharge);
		}
}
```

**TaxCalculator 를 통해 지역 세금과 추가 요금 부과**

```java
double value = new TaxCalculator().withTaxRegional()
																	.withTaxSurcharge()
																	.calculate(order);
```

이 방식은 가독성이 좋아지고 로직의 의도를 명확히 이해할 수 있다는 장점이 있지만, 코드가 장황하다는 단점이 존재한다. 

<br>

이번엔 자바의 함수형 기능을 통해 리팩토링을 해보자.

```java
public class TaxCalculator {
		public DoubleUnaryOperator taxFunction = d -> d;
		
		public TaxCalculator with(DoubleUnaryOperator f) {
				taxFunction = taxFunction.andThen(f);
				return this;
		}
		
		public double calculate(Order order) {
				return taxFunction.applyAsDouble(order.getValue());
		}
}
```

- with() 메서드를 통해 새로운 세금이 추가되면, 현재 세금 계산 함수에 이 세금이 조합되는 방식. 즉 한 함수에 모든 추가된 세금이 적용된다.

여기에 메서드 참조를 적용하면 다음과 같이 코드를 작성할 수 있다.

```java
double value = new TaxCalculator().with(Tax::regional)
																	.with(Tax::surcharge)
																	.calculate(order);
```

이처럼 메서드 참조를 활용하면 읽기 쉽고 간결하게 코드를 작성할 수 있다. 심지어 새로운 세금 함수를 Tax 클래스에 추가해도, TaxCalculator 를 바꾸지 않고 바로 사용할 수 있다.

OCP 원칙을 잘 준수하며 유연성도 제공하는 것이다.

<br>

## 10.4 실생활의 자바 8 DSL

### DSL 패턴의 장점과 단점

<img width="1035" alt="스크린샷 2025-05-18 오후 11 55 15" src="https://github.com/user-attachments/assets/4939b9f0-11a0-41e1-8065-c98519f0362e" />


<br>

### 10.4.1 jOOQ

> **SQL을 구현하는 내부적 DSL로, 자바에 직접 내장된 형식 안전 언어.**
> 

- SQL 질의
    
    ```sql
    SELECT * FROM BOOK
    WHERE BOOK PUBLISHED_IN = 2016
    ORDER BY BOOK.TITLE
    ```
    
- JOOQ DSL을 이용한 코드
    
    ```java
    create.selectFrom(BOOK)
    			.where(BOOK.PUBLISHED_IN.eq(2016))
    			.orderBy(BOOK.TITLE)
    ```
    

→ 스트림 API와 조합해 사용할 수 있다는 것 또한 장점이다.

실제 활용 예시

```java
Class.forName("org.h2.Driver");

try(Connection c = getConnection("jdbc:h2:~/sql-goodies-with-mapping", "sa", "")) {
		DSL.using(c)
			 .select(BOOK.AUTHOR, BOOK.TITLE)
			 .where(BOOK.PUBLISHED_IN.eq(2016))
			 .orderBy(BOOK.TITLE)
		.fetch()
		.stream()
		.collect(groupingBy(
				r -> r.getValue(BOOK.AUTHOR),
				LinkedHashMap::new,
				mapping(r -> r.getValue(BOOK.TITLE), toList())))
				.forEach((author, titles) -> 
		System.out.println(author + " is author of " + titles));
}
```

<br>

### 10.4.2 큐컴버

**동작 주도 개발(BDD)**

> **테스트 주도 개발의 확장으로, 다양한 비즈니스 시나리오를 구조적으로 서술하는 간단한 도메인 전용 스크립팅 언어를 사용**
> 

큐컴버는 다른 BDD 프레임워크와 마찬가지로, 명령문을 실행할 수 있는 테스트 케이스로 변환한다.

좀 더 자세히 말하면, 개발자가 비즈니스 시나리오를 평문 영어로 구현할 수 있도록 도와주는 도구라고 할 수 있다.

```
Feature: Buy stock
	Scenario: Buy 10 IBM stocks
		Given the price of a "IBM" stock is 125$
		When i buy 10 "IBM"
		Then the order value should be 1250$
```

큐컴버는 세 가지로 구분되는 개념을 사용한다.

1. Given - 전제 조건 정의
2. When - 시험하려는 도메인 객체의 실질 호출 
3. Then - 테스트 케이스의 결과를 확인하는 assertion

큐컴버의 DSL은 간단하지만, 외부적 DSL과 내부적 DSL이 어떻게 효과적으로 합쳐질 수 있으며 람다와 함께 가독성 있는 함축된 코드를 구현할 수 있는지 보여준다.
