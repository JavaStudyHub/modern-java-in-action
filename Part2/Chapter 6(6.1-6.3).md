# Chapter 6. 스트림으로 데이터 수집


앞선 4장과 5장에서는 스트림에서 최종 연산 collect 를 사용하는 방법을 확인했다. 하지만 기존의 예제들은 toList 를 통해 스트림 요소를 항상 리스트로만 변환했었다.

이번에는 collect 를 통해 다양한 요소 누적 방식을 인수로 받아, 스트림을 최종 결과로 도출하는 리듀싱 연산을 수행하는 방법을 알아볼 것이다.

이 때 다양한 요소 누적 방식은 Collector 인터페이스에 정의된다. 

컬렉션(Collection), 컬렉터(Collector), collect 를 혼동하지 않도록 주의하자.

6장에 들어가기에 앞서 collect를 사용했을 시 어떤 식으로 코드를 최적화할 수 있는지 살펴보자

어떤 트랜잭션 리스트에서, 통화별로 트랜잭션을 그룹화하고자할 때, 이 요구사항을 만족시키기 위해 collect 를 사용하는 경우와 사용하지 않는 경우 각각에 대해 어떤 식으로 코드를 작성해야할까?

1. **collect 를 사용하지 않는 경우**
    
    ```java
    Map<Currency, List<Transaction>> transactionByCurrencies = new HashMap<>();
    
    for(Transaction transaction : transactions) {
    	Currency currency = transaction.getCurrency();
    	List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
    		
    	if(transactionsForCurrency == null) {
    			transactionsForCurrency = new ArrayList<>();
				transactionsByCurrencies.put(currency, transactionsForCurrency);
    	}
    	transactionsForCurrency.add(transaction);
    }
    ```
    
2. **collect 를 사용하는 경우**
    
    ```java
    Map<Currency, List<Transaction>> transactionsByCurrencies = transactions.stream().collect(groupingBy(Transaction::getCurrency));
    ```
    

코드가 놀랍도록 간단해진다!! 그럼 이제 본격적으로 이 컬렉터, collect 에 대해 알아보자

<br>

## 6.1 컬렉터란 무엇인가?

이전 예제의 코드는 collect 메서드로 Collector 인터페이스의 구현을 전달한 형태이다.

**즉 Collector 인터페이스의 구현은, 스트림의 요소를 어떤 식으로 도출할 지 지정한다**

<br>

### 6.1.1 고급 리듀싱 기능을 수행하는 컬렉터

함수형 API들의 장점 중 하나로 높은 수준의 조합성과 재사용성을 꼽을 수 있다. 즉, collect 를 통해서 결과를 수집하는 과정을, 간단하면서도 유연한 방식으로 정의할 수 있다는 것이 컬렉터의 최대 강점이다.

스트림에 대해 collect 를 호출하면 스트림의 요소에 대해 리듀싱 연산이 수행된다. 

<img width="480" alt="스크린샷 2025-05-04 오전 2 27 49" src="https://github.com/user-attachments/assets/f6d23347-1293-4a77-8db9-9da245f2003d" />

이 그림은 앞서 본 트랜잭션 예제에서 내부적으로 리듀싱 연산이 일어나는 모습을 보여준다. 즉 명령형 프로그래밍에서는 직접 구현해야했던 작업들이 자동으로 수행되고있는 것이다.

> [!important]
> **Collector 인터페이스의 메서드를 어떻게 구현하느냐에 따라 스트림에 어떤 리듀싱 연산을 수행할 지 결정된다.**

<br>

**Collectors 유틸리티 클래스**는 자주 사용하는 컬렉터 인스턴스를 손쉽게 생성할 수 있는 **정적 팩토리 메서드**를 제공하는데, 그 중 하나가 우리가 자주 사용하는 toList 를 꼽을 수 있다.

```java
List<Transaction> transactions = transacionStream.collect(Collectors.toList());
```

<br>

### 6.1.2 미리 정의된 컬렉터

Collectors 유틸리티 클래스에서는 toList, groupingBy 와 같은 정적 팩토리 메서드 기능들을 제공하며, 이 기능들은 크게 3가지로 구분할 수 있다.

1. **스트림 요소를 하나의 값으로 리듀스하고 요약**
2. **요소 그룹화**
3. **요소 분할**

지금부터 하나씩 알아보자.

<br>

## 6.2 리듀싱과 요약

앞서 언급했듯이 컬렉터(Stream.collect 메서드의 인수)을 통해 스트림의 항목을 컬렉션으로 재구성할 수 있다.

좀 더 일반적으로 말하면, 컬렉터로 스트림의 모든 항목을 하나의 결과로 합칠 수 있으며, 트리를 구성하는 다수준 맵이 될 수도 있고, 메뉴의 칼로리 합계를 가리키는 단순한 정수가 될 수도 있다.

간단한 예시로 **`counting()`** 이라는 팩토리 메서드가 반환하는 컬렉터를 통해 메뉴에서 요리 수를 계산해보자

```java
long howManyDishes = menu.stream().collect(Collectors.counting());

// 아래 같은 코드도 가능

long howManyDishes = menu.stream().count();
```

이처럼 컬렉터에서 지원하는 팩토리 메서드를 통해 요소의 개수를 세어 반환하는 동작을 손쉽게 구현할 수 있다.

<br>

### 6.2.1 스트림값에서 최댓값과 최솟값 검색

미리 정의된 컬렉터를 통해 스트림의 최댓값과 최솟값을 찾을 수 있다.

1. **`Collectors.minBy`** : **스트림에서의 최솟값 계산** 
2. **`Collectors.maxBy`** : **스트림에서의 최댓값 계산**

이 두 컬렉터는 **스트림의 요소를 비교하는 데 사용할 Comparator 를 인수로 받는다.** 

칼로리로 요리를 비교하는 Comparator를 구현한 다음 Collectors.maxBy 로 전달하는 코드를 살펴보자

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);

Optional<Dish> mostCaloriesDish = menu.stream().collect(maxBy(dishCaloriesComparator));
```

<br>

### 6.2.2 요약 연산

**스트림에 있는 객체의 숫자 필드의 합계나 평균 등을 반환하는 연산에도 리듀싱 기능이 자주 사용된다.**

이러한 연산을 **요약 연산** 이라고 부른다.

1. **`Collectors.summingInt (summingLong, summingDouble)`**
    - summingInt 는 객체를 int로 매핑하는 함수를 인수로 받는다. 이 때 summingInt의 인수로 전달된 함수는 객체를 int로 매핑한 컬렉터를 반환한다.
    
    ```java
    int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));
    ```
    
    <img width="508" alt="스크린샷 2025-05-04 오전 2 28 14" src="https://github.com/user-attachments/assets/48387ec8-21ce-4170-affb-d650903d746c" />

    데이터 수집 과정은 위와 같다. 칼로리로 매핑된 각 요리의 값을 탐색하면서 초깃값으로 설정되어있는 누적자에 칼로리를 더한다.
    
2. **`Collectors.averagingInt (averagingLong, averagingDouble)`**
    - averagingInt 또한 객체를 int 로 매핑하는 함수를 인수로 받는다. 이 때 averagingInt의 인수로 전달된 함수는 객체를 int로 매핑한 컬렉터를 반환한다.
    
    ```java
    double avgCalories = menu.stream().collect(averagingInt(Dish::getCalories));
    ```
    

이렇게 컬렉터를 활용하면 스트림의 요소 수를 계산한다던지, 최댓값 혹은 최솟값을 찾거나 합계와 평균을 계산할 수도 있다.

이 때 종종 두 개 이상의 연산을 한 번에 수행해야할 때도 있다. 즉 여러 요소들에 대한 합계도 알아야하고, 평균도 알아야하고, 최대 최소도 모두 알아야하는 상황인 것이다.

이런 경우 스트림이 일회용이기 때문에 특정 컬렉션에 대해 스트림을 여러번 열어주어야해서 다소 귀찮아질 수 있다.

이런 상황에서는 팩토리 메서드 summarizingInt 가 반환하는 컬렉터를 사용하면 좋다.

```java
IntSummaryStatistic menuStatistics = menu.stream().collect(summarizingInt(Dish::getCalories));

IntSumaryStatistics{count=9, sum=4300, min=120, average=477.777, max=800}
```

summarizingInt 를 사용하면 IntSummaryStatistics 클래스로 모든 정보가 수집되고, 이처럼 요소 수, 합계, 최대, 최소, 평균값 등의 모든 정보가 수집된다.

<br>

### 6.2.3 문자열 연결

컬렉터에 **`joining 팩토리 메서드`** 를 이용하면, 스트림의 각 객체에 **toString** 메서드를 호출해서 추출한 모든 문자열을 하나의 문자열로 연결해서 반환한다.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining());

-> 메뉴의 모든 요리명을 연결하여 하나의 문자열로 반환해주는 코드
```

여기서 중요한 점은, joining 메서드가 내부적으로 StringBuilder 를 활용한다는 점이다. 따라서 매번 문자열들을 연결할 때마다 힙 영역에 새로운 문자열 객체가 생기지 않고, 하나의 StringBuilder 객체를 활용함으로써 메모리를 효율적으로 사용하게된다.

연결된 두 요소 사이에 구분 문자열을 넣을 수 있도록 지원하는 joining 팩토리 메서드도 존재한다.

```java
String shortMenu = menu.stream().map(Dish::getName).collect(joining(", "));
```

이렇게 스트림을 하나의 값으로 리듀스해주는 다양한 컬렉터를 살펴보았다. 이번에는 **`Collectors.reducing 팩토리 메서드`** 가 제공하는 범용 리듀싱 컬렉터에 대해서 알아보자.

<br>

### 6.2.4 범용 리듀싱 요약 연산

지금까지 살펴본 모든 컬렉터는 reducing 팩토리 메서드로도 정의할 수 있다.

예를 들어 메뉴의 모든 칼로리 합계를 summingInt 가 아닌 reducing 을 통해 구현해보자.

```java
int totalCalories = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i+j));
```

합계를 구하는 reducing의 경우 인수를 3개 받는다.

- **첫번째 인수** : 리듀싱 연산의 시작값이거나 스트림에 인수가 없을 때는 반환값이다.
- **두번째 인수** : 변환 함수
- **세번째 인수** : 같은 종류의 두 항목을 하나의 값으로 더하는 BinaryOperator

만약 최대값을 찾고싶은 경우에는 아래와 같이 하나의 인수를 가진 reducing 을 이용하면 된다.

```java
Optional<Dish> mostCalorieDish = menu.stream().collect(reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2));
```

하나의 인수를 받는 reducing 의 경우, 세 개의 인수를 갖는 reducing 메서드에서, **첫번째 인수로 스트림의 첫번째 요소**를, **두번째 인수로 자신을 그대로 반환하는 항등 함수**를 받는 상황이라고 생각하면 된다.

즉 빈 스트림이 넘겨졌을 때는 시작값이 없기 때문에 reducing 컬렉터에 대해서도 시작값이 설정되지 않는 상황이 벌어진다. 그러다보니 한 개의 인수를 갖는 reducing은 Optional<Dish> 객체를 반환한다.

> [!NOTE]
> **collect와 reduce의 차이점**
>
> `collect`와 `reduce`는 Java Stream에서 모두 결과를 누적하거나 집계하는 데 사용되지만, 근본적인 목적과 사용 방식에 중요한 차이가 있다.
>
> - **collect** 메서드는 결과를 누적하는 *컨테이너*를 변경하도록 설계된 메서드다. 예를 들어, 스트림의 요소들을 리스트, 집합, 맵 등 다양한 컬렉션이나 원하는 형태로 모을 때 사용된다. 내부적으로 가변 컨테이너(예: ArrayList)에 요소를 추가하는 식으로 동작하며, Collector 인터페이스를 통해 다양한 누적 방식을 지원한다
>
> - **reduce** 메서드는 두 값을 하나로 도출하는 *불변형 연산*이다. 즉, 스트림의 각 요소를 순차적으로 결합해 하나의 결과(예: 합계, 곱, 최대값 등)로 축소한다. 이때 연산 대상과 반환값의 타입이 동일해야 하며, 가변 객체의 상태를 변경하는 방식은 의미론적으로 맞지 않는다
>
> 예를 들어, 아래 코드는 `collect` 대신 `reduce`를 사용해 리스트를 만드는 잘못된 예시다:
>
> ```java
> Stream stream = Arrays.asList(1, 2, 3, 4, 5, 6).stream();
> List numbers = stream.reduce(
>     new ArrayList(),
>     (List l, Integer e) -> {
>         l.add(e);
>         return l;
>     },
>     (List l1, List l2) -> {
>         l1.addAll(l2);
>         return l1;
>     });
> ```
>
> 이 코드는 두 가지 문제가 있다.
>
> 1. **의미론적 문제**: `reduce`는 불변 객체를 대상으로 하는 연산이어야 하지만, 위 예제는 리스트(가변 객체)의 상태를 직접 변경하고 있다. 이로 인해 원래 의도와 다르게 동작할 수 있다
> 2. **실용성 문제**: 병렬 스트림에서 여러 스레드가 동시에 같은 리스트를 수정하면 리스트가 망가질 수 있다. 안전하게 하려면 매번 새로운 리스트를 만들어야 하고, 이는 성능 저하로 이어진다
>
> 따라서, 누적 컨테이너를 사용하거나 병렬 처리가 필요한 경우에는 `collect`를, 단순히 불변 객체의 집계(예: 합계, 곱셈 등)가 필요한 경우에는 `reduce`를 사용하는 것이 바람직하다


<br>

### 컬렉션 프레임워크의 유연성 : 같은 연산도 다양한 방식으로 수행 가능하다.

앞서 reducing 컬렉터를 통해 모든 칼로리 합계를 구했던 코드를, 람다식 대신 Integer 클래스의 sum 메서드를 참조하도록 하면 코드를 좀 더 단순화할 수 있다.

```java
int totalCalories = menu.stream().collect(reducing(0, <- 초깃값
																					Dish :: getCalories,  <- 변환 함수
																					Integer::sum));       <- 합계 함수
```

<img width="532" alt="스크린샷 2025-05-04 오전 2 28 47" src="https://github.com/user-attachments/assets/fac3ff90-5dc1-4fcb-ac70-a133fa1c3a65" />

그 외에도 앞서 보았던 counting 컬렉터를 활용한 요소수 계산도 reducing 팩토리 메서드를 이용해 구현할 수 있다.

다음과 같이 스트림의 Long 객체 형식의 요소를 1로 변환한 다음 모두 더하면 된다.

```java
public static <T> Collector<T, ?, Long> counting() {
		return reducing(0L, e -> 1L, Long::sum);
}
```

이렇듯 스트림을 사용하다보면 하나의 연산을 다양한 방법으로 해결할 수 있다.

따라서 우리는 문제를 해결할 수 있는 다양한 해결 방법을 확인한 다음, 가장 일반적으로 문제에 특화된 해결책을 고르는 것이 바람직하다.

<br>

## 6.3 그룹화

데이터를 특성에 따라 그룹화하는 연산 역시 빈번하게 수행되는 작업 중 하나이다. 이 때 명령형으로 그룹화를 구현하려면 까다롭고 할 일이 많은데, 이 때 자바 8의 함수형을 이용하면 가동성 있는 한 줄의 코드로 그룹화를 구현할 수 있다.

이 때는 팩토리 메서드 **`Collectors.groupingBy`** 를 이용하면 된다.

```java
Map<Dish.Type, List<Dish>> dishesByType = menu.stream().collect(groupingBy(Dish::getType));

결과

{FISH=[prawns, salmon], OTHER=[french fries, rice, pizza], MEAT...}
```

이처럼 각 요리에서 Dish.Type 과 일치하는 모든 요리를 추출하는 함수를 groupingBy 메서드로 전달한 모습을 볼 수 있는데, 이 때 이 함수를 기준으로 스트림이 그룹화 된다.

따라서 이 함수를 **분류 함수** 라고 한다.

<img width="533" alt="스크린샷 2025-05-04 오전 2 29 02" src="https://github.com/user-attachments/assets/59c47b7d-f077-4703-9cae-5d27946717d5" />

→ 그룹화 연산의 결과로 그룹화 함수가 반환하는 키, 그리고 각 키에 대응하는 스트림의 모든 항목 리스트를 값으로 갖는 맵이 반환된다.

만약 단순히 속성을 기반으로 분류를 하는 것이 아니라, 더 복잡한 분류 기준이 필요한 상황이라면 메서드 참조를 분류 함수로 사용할 수 없을 수 있다. 따라서 이런 때에는 분류 함수에 메서드 참조 대신 람다 표현식을 활용해 사용해주면 된다.

```java
public enum CaloricLevel {DIET, NORMAL, FAT}

Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
			groupingBy(dish -> {
					if(dish.getCalories() <= 400) return CaloricLevel.DIET;
					else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
					else return CaloricLevel.FAT;
			}));
			
```

<br>

### 6.3.1 그룹화된 요소 조작

요소를 그룹화 한 다음에는 각 결과 그룹의 요소를 조작하는 연산이 필요하다. 

예를 들면 500 칼로리가 넘는 요리만 필터링 한다고 가정해보자. 우리가 기존에 학습했던 대로라면 filter 를 적용해줌으로써 500 칼로리가 넘는 요소들만 뽑아낸 후 그룹화를 하면 되는 것 아닌가? 라는 생각을 할 수 있다. 

이 경우 아래와 같은 형태로 코드를 작성하게 될 것이다.

```java
public enum Type {FISH, MEAT, OTHER}

Map<Dish.Type, List<Dish>> caloricDishesByType =
				menu.stream().filter(dish -> dish.getCalories() > 500)
										 .collect(groupingBy(Dish::getType));
```

하지만 이 방식으로는 원하는 요구사항을 만족하지 못하는 경우가 생길 수 있다.

만약 스트림 연산의 결과가 맵 형태로 반환되어야하다고 생각해보자. 즉 요리의 타입과 요리가 대응되는 형태인 것이다. 

이 때 우리는 각 타입별로 분류를 하되, 각 타입 내에 칼로리가 500을 넘는 요리가 무엇이 있는지를 보고싶은 상황이다. 또한 해당 타입을 갖는 요리 중 500 칼로리를 넘는 요리가 없다고 하더라도, 키 : [] 와 같이 빈 리스트로 출력되길 원하는 상황이라고 할 때, 위와 같은 코드는 해당 요구사항을 만족하지 못하고 아래와 같은 결과가 도출되는 경우가 생긴다.

```java
{OTHER=[french fries, pizza], MEAR=[port, beef]}
```

결과에서 특정 키 자체가 사라지는 모습을 볼 수 있다.

만약 필터 프레디케이트를 만족하는 FISH 종류 요리가 없다면, 결과 맵에서 FISH 라는 키 자체가 사라져버릴 수 있는 것이다.

따라서 이런 상황을 해결하고싶다면, **Collectors 클래스는 일반적인 분류 함수에 Collector 형식의 두 번째 인수를 갖도록 groupingBy 팩토리 메서드를 오버로드** 함으로써 이 문제를 해결한다.

```java
Map<Dish.Type, List<Dish» caloricDishesByType =
		menu.stream()
				.collect(groupingBy(Dish::getType,
								 filtering(dish -> dish.getCalories() > 500, toList())));
```

이 filtering 메서드는 Collector 클래스의 또 다른 정적 팩토리 메서드로써, Predicate를 인수로 받는다.

이 프레디케이트로 각 그룹의 요소와 필터링 된 요소를 재그룹화 한다. 이렇게 해서 아래 결과 맵에서 볼 수 있는 것처럼 목록이 비어있는 FISH도 항목으로 추가된다.

```java
{OTHER=[french fries, pizza], MEAT=[pork, beef], FISH=[]}
```

그룹화된 항목을 조작하는 다른 기능으로, 매핑 함수를 이용해 요소를 변환하는 작업이 있다.

이에 대해 Collectors 클래스는 또 다른 컬렉터를 인수로 받는 mapping 메서드를 제공한다. 예를 들어 이 함수를 이용해 그룹의 각 요리를 관련 이름 목록으로 변환할 수 있다.

```java
Map<Dish.Type, List<String>> dishNameByType = menu.stream()
																									.collect(groupingBy(Dish::getType, mapping(Dish::getName, toList())));
```

또한 groupingBy 와 연계하여, 세 번째 컬렉터를 사용해 일반 맵이 아닌 flatMap 변환을 수행할 수 있다. 

다음처럼 태그 목록을 가진 각 요리로 구성된 맵이 있다고 가정해보자.

```java
Map<String, List<String>> dishTags = new HashMap<>(); 

dishTags.put("pork",asList("greasy", "salty")); 
dishTags.put("beef", asList("salty", "roasted"));
dishTags.put("chicken", asList("fried", "crisp")); 
dishTags.put("french fries", asList("greasy", "fried")); 
dishTags.put("rice", asList("light", "natural"));
dishTags.put("season fruit", asList("fresh", "natural")); 
dishTags.put("pizza", asList("tasty", "salty")); 
dishTags.put("prawns"z asList("tasty", "roasted"));
dishTags.put("salmon"z asList("delicious", "fresh"));
```

이 때 **`flatMapping`** 컬렉터를 이용하면 각 형식의 요리의 태그를 간편하게 추출할 수 있다.

```java
Map<Dish.Type, Set<String>> dishNamesByType = menu.stream()
																									.collect(groupingBy(Dish::getType, flatMapping(dish -> dishTags.get(dish.getName()).stream(), toSet())));		
```

앞서 학습했던 것처럼 두 수준의 리스트를 한 수준으로 평면화하려면 flatMap 을 수행해야한다. 이전처럼 각 그룹에 대해 수행한 flatMapping 연산 결과를 수집해서 리스트가 아닌 집합으로 그룹화하여 중복 태그를 제거하고있음에 주목하자.

<br>

### 6.3.2 다수준 그룹화

Collectors.groupingBy 를 이용하면 항목을 다수준으로 그룹화할 수 있다. 이 Collectors.groupingBy는 일반적인 분류 함수와 컬렉터를 인수로 받으며, **바깥쪽 groupingBy 메서드에 스트림의 항목을 분류할 두 번째 기준을 정의하는 내부 groupingBy 를 전달**해서 두 수준으로 스트림의 항목을 그룹화할 수 있다.

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream().collect(
		groupingBy(Dish::getType,
				groupingBy(dish -> {
						if(dish.getCalories() <= 400)
								return CaloricLevel.DIET;
						else if(dish.getCalories() <= 700)
								return CaloricLevel.NORMAL;
						else
								return CaloricLevel.FAT;
				})
		)
);
```

이렇게 다수준 그룹화를 하게 되면 다음과 같이 두 수준의 맵이 만들어진다.

<img width="536" alt="스크린샷 2025-05-04 오전 2 29 40" src="https://github.com/user-attachments/assets/53f1d42d-bbaf-479d-bddc-f55a72d50fe8" />

이 다수준 그룹화 연산은 다양한 수준으로 확장할 수 있다. 즉 n 수준 그룹화의 결과는 n 수준 트리 구조로 표현되는 n수준 맵이 된다.

<img width="539" alt="스크린샷 2025-05-04 오전 2 29 54" src="https://github.com/user-attachments/assets/ebce30ca-d35d-4db0-8878-e0f78b3cdb43" />

좀 더 이해하기 쉽도록 풀어서 설명하자면, 하나의 groupingBy 연산 자체를 버킷(물건을 담을 수 있는 양동이) 개념으로 생각하면 쉽다.

첫 번째 groupingBy 는 각 키의 버킷을 만들고, 각각의 버킷을 서브스트림 컬렉터로 채워가기를 반복하며 n수준 그룹화를 달성한다.

<br>

### 6.3.3 서브 그룹으로 데이터 수집

첫 번째 groupingBy 로 넘겨주는 컬렉터의 형식에는 제한이 없다. 예를 들어 groupingBy 컬렉터에 두 번째 인수로 counting 컬렉터를 전달해서 메뉴에서 요리의 수를 종류별로 계산할 수도 있고, 요리의 종류를 분류하는 컬렉터로 메뉴에서 가장 높은 칼로리를 가진 요리를 찾는 프로그램도 구현할 수 있다.

```java
Map<Dish.Type, Long> typesCount = menu.stream().collect(
										groupingBy(Dish::getType, counting()));

-> 결과 : {MEAT=3, FISH=2, OTHER=4}

Map<Dish.Type, Optional<Dish>> mostCaloricByType = 
		menu.stream()
				.collect(groupingBy(Dish::getType,
														maxBy(comparingInt(Dish::getCalories))));
														
-> 결과 : {FISH=Optional[salmon], OTHER=Optional[pizza], MEAT=Optional[pork]}
```

### 컬렉터 결과를 다른 형식에 적용하기

**`Collectors.collectingAndThen`** 메서드를 활용하면, 컬렉터가 반환한 결과를 다른 형식으로 활용할 수 있다.

바로 예시를 봐보자

```java
Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
												.collect(groupingBy(Dish::getType, <- 분류 함수
																 collectingAndThen(
																		 maxBy(comparingInt(Dish::getCalories)),
																 Optional::get)));   <- 변환 함수																 
```

이 collectingAndThen 은 적용할 컬렉터와 변환 함수를 인수로 받아, 다른 컬렉터를 반환한다. 반환되는 컬렉터는 기존 컬렉터의 래퍼 역할을 하며 collect의 마지막 과정에서 변환 함수로 자신이 반환하는 값을 매핑한다.

이 예시에서는 maxBy로 만들어진 컬렉터가 감싸지는 대상 컬렉터이며, 변환 함수 Optional::get 을 통해 Optional에 포함된 값을 추출하여 반환한다.

대략적인 개요는 아래의 그림을 참고하자

<img width="533" alt="스크린샷 2025-05-04 오전 2 30 24" src="https://github.com/user-attachments/assets/35eb2fd2-9664-4e6e-b0f9-15a0e5591249" />

- 컬렉터는 점선으로 표시되어 있으며 groupingBy는 가장 바깥쪽에 위치하면서 요리의 종류에 따라 메뉴 스트림을 세 개의 서브스트림으로 그룹화한다.
- groupingBy 컬렉터는 collectingAndThen 컬렉터를 감싼다. 따라서 두 번째 컬렉터는 그룹화된 세 개의 서브스트림에 적용된다.
- collectingAndThen 컬렉터는 세 번째 컬렉터 maxBy를 감싼다.
- 리듀싱 컬렉터가 서브스트림에 대해 연산을 수행한 결과에 collectingAndThen 에서 정의한 변환 함수 ( = Optional::get) 이 적용된다.
