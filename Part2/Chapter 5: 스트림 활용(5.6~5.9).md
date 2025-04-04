# Chapter 5. 스트림 활용 (5.6 ~ 5.9)



## 5.6 실전 연습

1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정렬
2. 거래자가 근무하는 모든 도시를 중복 없이 나열
3. 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬
4. 모든 거래자의 이름을 알파벳순으로 정렬
5. 밀라노에 거래자가 존재하는지 확인
6. 케임브리지에 거주하는 거래자의 모든 트랜잭션 값 출력
7. 전체 트랜잭션 중 최댓값 
8. 전체 트랜잭션 중 최솟값

### 문제 풀이

위의 예시들에서 사용할 데이터

```java
Trader raoul = new Trader("Raoul", "Cambridge");
Trader mario = new Trader("Mario", "Milan");
Trader alan = new Trader("Alan", "Cambridge");
Trader brian = new Trader("Brian", "Cambridge");

List<Transaction> transactions = Arrays.asList(
		new Transaction(brian, 2011, 300),
		new Transaction(raoul, 2012, 1000),
		new Transaction(raoul, 2011, 400),
		new Transaction(mario, 2012, 710),
		new Transaction(mario, 2012, 700),
		new Transaction(alan, 2012, 950)
);
```

```java
public class Trader {
		private final String name; 
		private final String city;
		
		public Trader(String nz String c) {
				this.name = n;
				this.city = c;
		}
		
		public String getName() {
				return this.name;
		}
		
		public String getCity() {
				return this.city;
		}
		
		public String toString() {
				return "Trader:"+this.name + " in " + this.city;
		}
}

public class Transaction {
		private final Trader trader;
		private final int year;
		private final int value;
		
		public Transaction(Trader trader, int year, int value) {
				this.trader = trader;
				this.year = year; this.value = value;
		}
		
		public Trader getTrader(){
				return this.trader;
		}
		
		public int getYear(){
				return this.year;
		}
		
		public int getValue(){
				return this.value;
		}
		
		public String toString() {
				return "{" + this.trader + ", " + 
							 "year: " + this.year + ", " + 
							 "value: " + this.value + "}";
		}
}
```

- 문제 풀이
    
    
    1. 2011년에 일어난 모든 트랜잭션을 찾아 값을 오름차순으로 정렬
        
        ```java
        List<Transaction> result = transactions.stream()
        						.filter(transaction -> transaction.getYear() == 2011)
        						.sorted(comparing(Transaction::getValue))
        						.collect(toList());
        ```
		<br>
        
    2. 거래자가 근무하는 모든 도시를 중복 없이 나열
        
        ```java
        List<String> result = transactions.stream()
        						.map(transaction -> transaction.getTrader().getCity())
        						.distict()
        						.collect(toList());
        ```
        <br>
        → 스트림을 집합으로 변환하는 toSet() 을 사용할 수도 있다.
        
        ```java
        Set<String> result = transactions.stream()
        						.map(transaction -> transaction.getTrader().getCity())
        						.collect(toSet());
        ```
        <br>
    3. 케임브리지에서 근무하는 모든 거래자를 찾아서 이름순으로 정렬
        
        ```java
        List<Trader> result = transactions.stream()
        						.map(Transaction::getTrader)
        						.filter(trader -> trader.getCity().equals("Cambridge"))
        						.distinct()
        						.sorted(comparing(Trader::getName))
        						.collect(toList());
        ```
        <br>
    4. 모든 거래자의 이름을 알파벳순으로 정렬
        
        ```java
        String result = transactions.stream()
        						.map(transaction -> transaction.getTrader().getName())
        						.distinct()
        						.sorted()
        						.reduce("", (n1, n2) -> n1 + n2);
        ```
        
        → 위의 코드의 경우 매번 새로운 문자열을 만들게 되는데, 이 때 joining 을 사용하게 되면 내부적으로 StringBuilder 를 활용하여 값을 만들어준다.
        
        ```java
        String result = transactions.stream()
        						.map(transaction -> transaction.getTrader().getName())
        						.distinct()
        						.sorted()
        						.collect(joining());
        ```
        <br>
    5. 밀라노에 거래자가 존재하는지 확인
        
        ```java
        boolean result = transactions.stream()
        				.anyMath(transaction -> transaction.getTrader().getCity().equals("Milan"));
        ```
        <br>
    6. 케임브리지에 거주하는 거래자의 모든 트랜잭션 값 출력
        
        ```java
        transactions.stream()
        							.filter(transaction -> transaction.geTrader().getCity().equals("Cambridge"))
        							.map(Transaction::getValue)
        							.forEach(Syste.out::println);
        ```
        <br>
    7. 전체 트랜잭션 중 최댓값 
        
        ```java
        Optional<Integer> highestValue = transactions.stream()
        									.map(Transaction::getValue)
        									.reduce(Integer::max);
        ```
		<br>
        
    8. 전체 트랜잭션 중 최솟값
        
        ```java
        Optional<Transaction> smallestTransaction = transactions.stream()
        									.reduce((t1, t2) -> 
        													 t1.getValue() < t2.getValue() ? t1 : t2);
        ```
        
    
    → 스트림은 최댓값이나 최솟값을 계산하는 데 사용할 키를 지정하는 Comparator 를 인수로 받는 min 과 max 메서드를 제공한다.
 
	<br>
    
    ```java
    Optional<Transaction> smallestTransaction = transactions.stream()
    										.min(comparing(Transaction::getValue));
    ```
    

## 5.7 숫자형 스트림

앞에서 reduce 메서드를 통해 스트림 요소의 합을 구하는 예제를 살펴보았다. 

```java
int calories = menu.stream()
									 .map(Dish::getCalories)
									 .reduce(0, Integer::sum);
```

사실 위의 코드는 박싱 비용이 존재한다. 내부적으로 합계를 계산하기 전에 Integer를 기본형으로 언박싱해주어야한다. 그럼 이 때 직접 sum 메서드를 호출할 수 있으면 더 좋지 않을까?

```java
int calories = menu.stream()
									 .map(Dish::getCalories)
									 .sum();
```

하지만 위의 코드처럼 sum 메서드를 바로 사용할 수는 없다. map 메서드가 Sream<T> 를 반환하기 때문이다.

따라서 스트림의 요소 형식은 Integer 이지만, Stream 인터페이스에는 sum 메서드가 존재하지 않는다. Stream 인터페이스에 sum 메서드가 없는 이유는, 이 제네릭에 Integer와 같이 연산이 가능한 타입이 아니라, Dish 와 같이 덧셈 연산이 불가한 객체들이 들어올 수도 있기 때문이다.

이런 문제들을 효율적으로 처리할 수 있도록, 자바 8은 **기본형 특화 스트림**을 제공한다.

### 기본형 특화 스트림

자바 8에서는 세 가지 기본형 특화 스트림을 제공한다.

1. **IntStream : int 요소에 특화**
2. **DoubleStream : double 요소에 특화**
3. **LongStream : long 요소에 특화**

각각의 인터페이스는 숫자 스트림의 합계를 계산하는 sum, 최댓값 요소를 검색하는 max 와 같이 자주 사용하는 숫자 관련 리듀싱 연산 수행 메서드를 제공한다.

<br>

> [!important]
> **특화 스트림은 오직 박싱 과정에서 일어나는 비효율성을 해결하기 위한 목적이지, 스트림에 추가 기능을 제공하지는 않는다.**

<br>

**숫자 스트림으로 매핑**

스트림을 특화 스트림으로 변경

- **mapToInt**
- **mapToDouble**
- **mapToLong**

```java
int calories = menu.stream()
									 .mapToInt(Dish::getCalories) <- IntStream 반환
									 .sum();
```

mapToInt 메서드는 Stream<T> 타입의 스트림을 IntStream 으로 바꾸어 반환해주는 메서드이다. 

**Stream<Integer> 로 반환하는 것이 아니라 IntStream 으로 반환되는 것임에 유의하자!**

<br>

**객체 스트림으로 복원**

숫자 스트림을 만든 후, 원상태인 특화되지 않은 스트림으로 복원하려면 어떻게 해야할까?

이 때는 **boxed 메서드**를 사용하면 된다.

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
**Stream<Integer> stream = intStream.boxed();**
```

boxed 메서드의 경우 

- **IntStream → Stream<Integer>**
- **DoubleStream → Stream<Double>**
- **LongStream → Stream<Long>**

으로 변환한다.

박싱, 언박싱을 기억하면서 메서드 명을 바라보면 좀 더 직관적으로 이해가 될 것이다.

<br>

**기본값 : OptionalInt**

IntStream의 경우, 스트림의 요소가 비어있으면 0이라는 기본값을 반환한다.

근데 이 0이라는 기본값 때문에 문제가 발생할 수 있는 여지가 존재한다.

만약 스트림에 있는 요소들 중 최대값을 구해야하는 상황이라고 생각해보자. 이 때 요소들 중 실제 최대값이 0인 것이다. 그러면 우리는 최대값을 구했을 때 0이라는 값을 얻게 될 것인데, 이 0이 스트림이 비어있어서 반환된 기본값 0인지, 실제 최대값 0인지를 구분할 수가 없어질 것이다.

그럼 이런 상황은 어떻게 구별해야할까?

이 때 앞서 언급한 바 있는 Optional 이 등장한다. 이 Optional 을 Integer, String, 등의 참보 형식으로 파라미터화할 수 있으며, 또한 OptionalInt, OptionalDouble, OptionalLong 등 세가지 기본형 특화 스트림을 제공한다.

<br>

```java
OptionalInt maxCalories = menu.stream()
															.mapToInt(Dish::getCalories)
															.max();
															
int max = maxCalories.orElse(1);
```

이렇게 사용하면 스트림이 비어있는 상황과 실제 최대값이 0인 상황등을 구분할 수 있게된다.

<br>

### 숫자 범위

프로그램에서는 특정 범위의 숫자를 이용해야하는 상황이 자주 발생한다.

자바 8의 IntStream, LongStream 에서는 **range** 와 **rangeClosed** 라는 두 가지 정적 메서드를 제공한다.

**이 두 메서드 모두 첫번째 인수로 시작값, 두번째 인수로 종료값을 가지며, range의 경우 시작값과 종료값이 결과에 포함되지 않고, rangeCloseD의 경우 시작값과 종료값이 결과에 포함된다.**

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100)
																 .filter(n -> n % 2 == 0);
																 
System.out.println(evenNumbers.count());
												
```

→ 1 ~ 100 의 숫자중 짝수의 개수를 반환

> [!important]
> **flatMap 메서드 : flatMap 메서드는 생성된 각각의 스트림을 하나의 평준화된 스트림으로 만들어주는 동작을 한다.**


<br>


## 5.8 스트림 만들기

stream 메서드를 통해서 컬렉션으로부터 스트림을 얻을 수 있었다.

하지만 그 뿐 아니라 일련의 값, 배열, 파일, 심지어 함수를 이용하는 등의 다양한 방식으로도 스트림을 만들 수 있다.



### 값으로 스트림 만들기

임의의 수를 인수로 받는 정적 메서드 Stream.of 를 사용해서 스트림을 만들 수 있다.

```java
Stream<String> stream = Stream.of("Modern", "Java", "In", "Action");
```

<br>

### null이 될 수 있는 객체로 스트림 만들기

자바 9에서는 **null 이 될 수 있는 객체**를 스트림으로 만들 수 있는 ofNullable 이라는 새로운 메서드가 추가되었다.

예를 들어 System.getProperty 는 제공된 키에 대응되는 속성이 없으면 null 을 반환한다. 이 메서드를 적용하여 스트림을 만드는 예시를 확인해보자.

<br>

**기존 형태**

```java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = (homeValue == null) ? Stream.empty() : Stream.of(value)); 
```

<br>

**ofNullable 적용 형태**

```java
String homeValue = System.getProperty("home");
Stream<String> homeValueStream = Stream.ofNullable(System.getProperty("home"));
```

→ ofNullable 을 활용하니, null 이 가능해서 NullPointerException 을 피하기 위해서는 homeValue == null 과 같은 null 체크 로직이 필요했던 기존 형태에서, 

null 일 경우 알아서 빈 스트림을 반환해주도록 하나의 메서드로 처리가 가능하게 간략화되었다.

즉 null 체크를 위한 부가적인 과정이 사라졌다고 생각하면 된다.

<br>

### 배열로 스트림 만들기

배열을 인수로 받는 정적 메서드 **Arrays.stream** 을 이용해서 스트림을 만들 수 있다.

```java
int[] numbers = {1, 2, 3, 4, 5, 6}

IntStream stream = Arrays.stream(numbers);
```

**이 때 기본형 배열을 넘길 경우 IntStream, DoubleStream 과 같은 특화 스트림을 반환하고, 참조형 배열을 넘길 경우 Stream<T> 형식의 일반적인 스트림을 반환한다.**

<br>

### 파일로 스트림 만들기

파일을 처리하는 등의 I/O 연산에 사용하는 자바의 NIO API(비블록 I/O) 도 스트림 API를 활용할 수 있도록 업데이트 되었다.

java.nio.file.Files 의 많은 정적 메서드가 스트림을 반환하는데, 예를 들어 Files.lines 는 주어진 파일의 행 스트림을 문자열로 반환한다.

```java
Stream<String> lines = Files.lines(Paths.get("data.txt");
```

<br>

### 함수로 무한 스트림 만들기

스트림 API는 함수에서 스트림을 만들 수 있는 두 정적 메서드 

1. **Stream.iterate**
2. **Stream.generate** 

를 제공한다.

이 두 연산을 이용해서 **무한 스트림**, 즉 고정된 컬렉션에서 고정된 크기로 스트림을 만들었던 것과 달리, **크기가 고정되지 않은 스트림**을 만들 수 있다.

> [!tip]
> **무한 스트림 : 크기가 고정되지 않은 스트림**

iterate와 generate 에서 만든 스트림은, 요청할 때마다 주어진 함수를 이용해서 값을 만든다. 따라서 무제한으로 값을 계산할 수 있다.

하지만 보통 무한한 값을 출력하지 않도록 하기 위해 limit(n) 함수를 연결해서 사용한다.

<br>

**iterate 메서드**

```java
Stream.iterate(0, n -> n + 2)
			.limit(10)
			.forEach(System.out::println);
```

iterate 메서드는 초깃값 과 람다 를 인수로 받아서 새로운 값을 끊임없이 생산할 수 있다.

예제에서는 람다 n → n + 2, 즉 이전 결과에 2를 더한 값을 계속해서 반환한다.

결과적으로 예제의 iterate 메서드는 짝수 스트림을 생성하게된다.

기본적으로 iterate는 기존 결과에 의존해서 순차적으로 연산을 수행한다.

이렇듯 **iterate는 요청할 때마다 값을 계속해서 생산할 수 있으며, 끝이 없기 때문에 무한 스트림**을 만든다. 

이러한 스트림을 **언바운드 스트림** 이라고 표현한다.

일반적으로 연속된 일련의 값을 만들 때에는 iterate를 사용한다.

<br>

- **퀴즈 5-4**
    
    
    **피보나치 수열 만들기**
    
    ```java
    Stream.iterate(new int[]{0, 1},
    							t -> new int[]{t[1], t[0] + t[1]})
    			.limit(20)
    			.map(t -> t[0])
    			.forEach(System.out::println);
    ```
    
    위 코드를 실행시키면, 최초의 요소는 (0, 1) 그 다음에는 (0, 1) 을 활용해서 새롭게 (1, 1) 이라는 요소를 만들고, 그 다음에는 (1, 2) → (2, 3) → (3, 5) → (5, 8) → (8, 13)
    
    이런 식으로 이전의 요소를 가지고 계속해서 연산을 수행하며 결과를 만들어낼 것이다.
    
<br>

iterate 메서드는 추가적으로 프레디케이트를 지원한다. 예를 들어 0에서 시작해서 100 보다 크면 숫자 생성을 중단하는 코드를 다음처럼 구현할 수 있다.

```java
IntStream.iterate(0, n -> n < 100, n -> n + 1)
				 .forEach(System.out::println);
```

이렇듯 **iterate 메서드는 두번째 인수로 프레디케이트를 받아 언제까지 작업을 수행할 것인지의 기준으로 사용**한다.

그럼 이 때 filter 메서드를 사용하면 되는 것 아닌가? 하는 생각이 들 수 있다.

```java
IntStream.iterate(0, n -> n + 1)
				 .filter(n -> n < 100)
				 .forEach(System.out::println);
```

하지만 이 방식은 계속해서 생성되는 값들중 100을 넘어가는 값들은 스트림의 요소에 포함하지 않도록 구현은 가능하지만, 값 생성의 종료 조건이 될 순 없다. 그러다보니 위의 코드는 사실 종료되지 않는 코드가 된다. filter 메서드는 이 작업을 언제 중단해야하는지 알 수 없기 때문이다.

따라서 동일한 동작을 다르게 구현하기 위해서는 스트림 쇼트서킷을 지원하는 takeWhile 을 이용해야한다.

```java
IntStream.iterate(0, n -> n + 4)
				 .takeWhile(n -> n < 100)
				 .forEach(System.out::println);
```

<br>

**generate 메서드**

iterate 와 비슷하게 generate 도 값을 계산하는 무한 스트림을 만들 수 있다. 하지만 iterate와 달리, **generate 는 생상된 각 값을 연속적으로 계산하지 않는다. generate는 Supplier<T>를 인수로 받아서 새로운 값을 생산한다.** 

```java
Stream.generate(Math::random)
			.limit(5)
			.forEach(System.out::println);
```

이 코드는 0에서 1 사이에서 임의의 double 숫자 5개를 만든다.

그럼 도대체 generate를 어떤 상황에서 활용할 수 있는 걸까??

방금 사용한 Supplier<T> 는 상태가 없다. 즉 **나중에 계산에 사용할 어떤 값도 저장해두지를 않는다.**

하지만 이 Supplier 에 꼭 상태가 없어야하는 것은 아니다. Supplier가 상태를 저장한 다음에 스트림의 다음 값을 만들 때 상태를 고칠 수도 있다. 이 때 피보나치 수열을 구현해보면서 iterate와 generate 를 비교해보자.

여기서 중요한 점은, 병렬 코드에서는 Supplier 에 상태가 있으면 안전하지 않다는 것이다. 따라서 상태를 갖는 Supplier는 실제로는 피해야한다.

우선 피보나치수열을 스트림으로 구현하는 코드를 좀 더 효율적으로 작성하기 위해 일반적인 스트림을 특화 스트림으로 바꾸고, 병렬 환경에서의 상태 유지의 문제를 알아보기 위해 익명 클래스를 사용해보자. 이 때 IntStream 도 마찬가지로 generate 메서드를 지원하는데, 이 IntStream의 generate 메서드는 Supplier<T> 대신에 IntSupplier 를 인수로 받는다.

```java
IntStream twos = IntStream.generate(new IntSupplier() {
		public int getAsInt() {
				return 2;
		}
}
```

→ 람다를 사용하지 않고 익명 클래스를 사용한 상황

여기서 중요하게 알고가야할 점은 **익명 클래스에 상태 필드가 존재할 수 있다**는 점이다. 람다에서는 상태 필드가 존재할 수 없다. 즉 **람다는 상태를 바꾸지 않기 때문에 부작용이 없다.** 하지만 익명 클래스는 상태를 바꿀 수 있기 때문에 병렬 환경에서 부작용이 존재한다.

그럼 본격적으로 피보나치 수열 예시를 알아보자. IntStream, generate 를 활용할 것이므로 IntSupplier 를 구현해주어야한다.

```java
IntStream fib = new IntSupplier() {
		private int previous = 0;
		private int current = 1;
		public int getAsInt() {
				int oldPrevious = this.previous;
				int nextValue = this.previous + this.current;
				this.previous = this.current;
				this.current = nextValue;
				return oldPrevious;
		}
}；

IntStream.generate(fib).limit(10).forEach(System.out::println);
```

위 코드에서는 IntSupplier 인스턴스를 만든다. 만들어진 객체는 기존 피보나치 요소와 두 인스턴수 변수에 어떤 피보나치 요소가 들어가있는지를 추적한다. 따라서 **가변 상태 객체이다.**

getAsInt 를 호출하면, 객체 상태가 바뀌면서 새로운 값을 생산하게 된다. Iterate를 사용했을 때에는 각 과정에서 새로운 값을 생성하면서도, 기존 상태를 바꾸지 않는 **순수한 불변 상태**를 유지했다.

스트림을 병렬로 처리하면서 올바른 결과를 얻기 위해서는 불변 상태 기법을 고수해야한다. 하지만 generate를 통해 상태를 유지하는 과정에서는 불변 상태를 유지할 수 없다.

<br>

## 5.9 요약

- 스트림 API를 이용하면 복잡한 데이터 처리 질의를 표현할 수 있다.
- filter, distinct, takeWhile(자바 9), dropWhile(자바 9), skip, limit 메서드로 스트림을 필터링하거나 자를 수 있다.
- 소스가 정렬되어 있다는 사실을 알고 있을 때 takeWhile과 dropWhile 메서드를 효과적으로 사용할 수 있다.
- map, flatMap 메서드로 스트림의 요소를 추출하거나 변환할 수 있다.
- findFirst, findAny 메서드로 스트림의 요소를 검색할 수 있다. allMatch, noneMatch, anyMatch 메서드를 이용해서 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색할 수 있다.
    - 이들 메서드는 쇼트서킷, 즉 결과를 찾는 즉시 반환하며, 전체 스트림을 처리하지 않는다.
- reduce 메서드로 스트림의 모든 요소를 반복 조합하며 값을 도출할 수 있다. 예를 들어 reduce로 스트림의 최댓값이나 모든 요소의 합계를 계산할 수 있다.
- filter, map 등은 상태를 저장하지 않는 상태 없는 연산이다. reduce 같은 연산은 값을 계산하는 데 필요한 상태를 저장한다. sorted, distinct 등의 메서드는 새로운 스트림을 반환하기에 앞 서 스트림의 모든 요소를 버퍼에 저장해야한다. 이런 메서드를 상태 있는 연산이라고 부른다.
- IntStream, DoubleStream, LongStream 은 기본형 특화 스트림이다. 이들 연산은 각각의 기본형에 맞게 특화되어있다.
- 컬렉션 뿐 아니라 값, 배열, 파일, iterate와 generate 같은 메서드로도 스트림을 만들 수 있다.
- 무한한 개수의 요소를 가진 스트림을 무한 스트림이라 한다.
