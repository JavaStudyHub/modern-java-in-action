## 6.4 분할

분할은 **분할 함수**(predicate)를 이용해서 Boolean을 키로 가지는 두 개의 그룹으로 된 Map으로 특수한 그룹화 기능이다.

아래는 메뉴를 채식 요리와 채식이 아닌 요리로 **분할**하는 코드이다.

```java
Map<Boolean, List<Dish>> partitionedMenu =
    menu.stream().collect(partitioningBy(Dish::isVegetarian));
```

결과는 다음과 같이 두 개의 분류로 **분할**되어서 나온다.

```java
{false=[pork, beef, chicken, prawns, salmon], 
true=[french fries, rice, season fruit, pizza]}
```
</br>

### 6.4.1 분할의 장점

분할의 장점은 분할 함수가 반환하는 **참, 거짓 두 가지 요소의 스트림 리스트를 모두 유지**한다는 것이다. </br>
또, `partitioningBy`가 반환하는 맵이 참과 거짓 두 가지 키만 포함하므로 **더 간결하고 효과적**이라는 장점도 가지고 있다.

> [!TIP] 
> `partitioningBy` 메서드도 다른 Collector 인터페이스들과 마찬가지로 다른 Collector를 두 번째 인수로 전달할 수 있도록 오버로드되어 있다.

- **채식 요리와 채식이 아닌 요리로 분할 → 요리 종류로 그룹화**

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType = menu.stream()
    .collect(
        partitioningBy(Dish::isVegetarian,          // ⟵ 분할 함수
        groupingBy(Dish::getType))                  // ⟵ 두 번째 컬렉터
    );
```

**실행 결과**

```java
{false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]}, 
 true={OTHER=[french fries, rice, season fruit, pizza]}}
```

</br>

- **채식 요리와 채식이 아닌 요리로 분할 → 각 그룹에서 가장 칼로리가 높은 요리 → Optional.get()으로 반환**

```java
Map<Boolean, Dish> mostCaloricPartitionedByVegetarian =
    menu.stream().collect(
        partitioningBy(Dish::isVegetarian,
            collectingAndThen(
                maxBy(comparingInt(Dish::getCalories)),
                Optional::get
            )
        )
    );
```

**실행 결과**

```java
{false=pork, true=pizza}
```

</br>

### 6.4.2 숫자를 소수와 비소수로 분할하기

정수 n을 인수로 받아 2 ~ n까지의 자연수를 소수와 비소수로 나누는 프로그램을 구현해보자.

1. **주어진 수가 소수인지 아닌지 판단하는 프레디케이트 구현**

```java
public boolean isPrime(int candidate) {
    return IntStream.range(2, candidate)
                    .noneMatch(i -> candidate % i == 0);
}
```

다음처럼 소수의 대상을 주어진 수의 제곱근 이하의 수로 제한하면 성능이 개선된다.

```java
public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                    .noneMatch(i -> candidate % i == 0);
}
```

</br>

2. **구현한 프레디케이트를 분할 함수로 하여 그룹 분할**

```java
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n).boxed()
        .collect(
            partitioningBy(candidate -> isPrime(candidate)));
}
```
</br>

**Collectors 클래스의 정적 팩토리 메서드 표**

| **팩토리 메서드** | **반환 형식** | 활용 예 | **사용 예제** |
| --- | --- | --- | --- |
| toList | `List<T>` | List<Dish> dishes = menuStream.collect(toList()); | 스트림의 모든 항목을 리스트로 수집 |
| toSet | `Set<T>` | Set<Dish> dishes = menuStream.collect(toSet()); | 스트림의 모든 항목을 중복이 없는 집합으로 수집 |
| toCollection | `Collection<T>` | Collection<Dish> dishes = menuStream.collect(toCollection(ArrayList::new)); | 스트림의 모든 항목을 발행자가 제공하는 컬렉션으로 수집 |
| counting | `Long` | long howManyDishes = menuStream.collect(counting()); | 스트림의 항목 수 계산 |
| summingInt | `Integer` | int totalCalories = menuStream.collect(summingInt(Dish::getCalories)); | 스트림의 항목에서 정수 필드값을 더함 |
| averagingInt | `Double` | double avgCalories = menuStream.collect(averagingInt(Dish::getCalories)); | 스트림 항목의 정수 필드의 평균값 계산 |
| summarizingInt | `IntSummaryStatistics` | IntSummaryStatistics menuStatistics = menuStream.collect(summarizingInt(Dish::getCalories)); | 스트림 내 항목의 최댓값, 최솟값, 합계, 평균 등의 정수 정보 통계 수집 |
| joining | `String` | String shortMenu = menuStream.map(Dish::getName).collect(joining(", ")); | 스트림의 각 항목에 toString 메서드를 호출한 결과를 문자열로 연결 |
| maxBy | `Optional<T>` | Optional<Dish> fattest = menuStream.collect(maxBy(comparingInt(Dish::getCalories))); | 주어진 비교자(Comparator)를 이용해서 스트림의 최댓값 요소를 Optional로 감싼 값을 반환. 스트림에 요소가 없을 때는 Optional.empty() 반환 |
| minBy | `Optional<T>` | Optional<Dish> lightest = menuStream.collect(minBy(comparingInt(Dish::getCalories))); | 주어진 비교자(Comparator)를 이용해서 스트림의 최솟값 요소를 Optional로 감싼 값을 반환. 스트림에 요소가 없을 때는 Optional.empty() 반환 |
| reducing | `T (리듀스 결과 값)` | int totalCalories = menuStream.collect(reducing(0, Dish::getCalories, Integer::sum)); | 누적자를 초기값으로 설정한 후 BinaryOperator로 스트림의 각 요소를 반복적으로 누적자와 합쳐 스트림을 하나의 값으로 리듀싱 |
| collectingAndThen | `변환된 결과의 타입` | int howManyDishes = menuStream.collect(collectingAndThen(toList(), List::size)); | 다른 컬렉터를 감싸고 그 결과에 변환 함수 적용 |
| groupingBy | `Map<K, List<T>>` | Map<Dish.Type, List<Dish>> dishesByType = menuStream.collect(groupingBy(Dish::getType)); | 하나의 필드값을 기준으로 스트림의 항목을 그룹화하며 기준 필드값을 결과 맵의 키로 사용 |
| partitioningBy | `Map<Boolean, List<T>>` | Map<Boolean, List<Dish>> vegetarianDishes = menuStream.collect(partitioningBy(Dish::isVegetarian)); | 프레디케이트를 스트림의 각 항목에 적용한 결과로 항목 분할 |

</br>

## 6.5 Collector 인터페이스

> 💡 Collector 인터페이스는 리듀싱 연산(즉, 컬렉터)을 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.
> 

Collector 인터페이스는 다섯 개의 메서드로 정의되어 있다.

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    Function<A, R> finisher();
    BinaryOperator<A> combiner();
    Set<Characteristics> characteristics();
}
```

- **T** : 수집될 스트림 항목의 제너릭 형식
- **A** : 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체의 형식
- **R** : 수집 연산 결과 객체의 형식(항상 그런 것은 아니지만 보통 컬렉션 형식)

앞에서 자주 본 `toList`는 Stream<T>의 모든 요소를 List<T>로 수집하는 ToListCollector<T>라는 구현 클래스를 활용한다.

```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
```
</br>

### 6.5.1 Collector 인터페이스의 메서드 살펴보기

***supplier 메서드: 새로운 결과 컨테이너 만들기***

> 💡 빈 결과로 이루어진 Supplier를 반환
> 
- 수집 과정에서 **빈 누적자 인스턴스**를 만드는 파라미터가 없는 메서드

ex) ToListCollector의 경우, 누적자를 반환하기 때문에 빈 누적자가 비어있는 스트림의 수집 과정의 결과가 될 수 있다.

```java
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
}

// 생성자 참조 전달
public Supplier<List<T>> supplier() {
    return ArrayList::new;
}
```
</br>

***accumulator 메서드 : 결과 컨테이너에 요소 추가하기***

> 💡 리듀싱 연산을 수행하는 함수(BiConsumer)를 반환하는 메서드
> 
- 스트림에서 n번째 요소를 탐색할 때 n-1개의 항목을 수집한 누적자와 n번째 요소를 함수(BiConsumer)에 적용

ex) ToListCollector의 경우, n번째 요소를 탐색할 때, n-1까지의 요소를 수집한 누적자(`list`)와 n번째 요소(`item`)을 함수의 매개변수로 갖고, 누적자에 n번째 요소를 추가하는 연산을 수행한다.

```java
public BiConsumer<List<T>, T> accumulator() {
		return (list, item) -> list.add(item);
}

// 메서드 참조 전달
public BiConsumer<List<T>, T> accumulator() {
		return List::add;
}
```
</br>

***finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용하기***

> 💡 스트림 탐색이 끝나고 누적자 객체를 최종 결과로 변환하면서 누적 과정을 끝낼 때 호출할 함수(Function)을 반환
> 

ex) ToListCollector의 경우, 누적자 객체(List<T>)가 이미 최종 결과이다. 따라서 항등 함수를 반환한다.

```java
public Function<List<T>, List<T>> finisher() {
		return Function.identity();   // <- 항등함수
}
```

위 3가지 메서드로도 다음과 같이 간단하게 순차 리듀싱 과정의 논리적 순서를 파악해볼 수 있다.

<img width="350" alt="스크린샷_2025-05-04_오후_5 48 29" src="https://github.com/user-attachments/assets/17d94607-80fe-4148-a17c-6b9c5fbe1c1f" alt = "순차 리듀싱 과정의 논리적 순서"/>

실제로 리듀싱 과정은 중간 연산과 파이프라인을 구성하게끔 해주는 게으른 특성 및 병렬 실행을 고려해야 하므로 훨씬 복잡하다.

</br>

***combiner 메서드 : 두 결과 컨테이너 병합***

> 💡 리듀싱 연산에서 사용할 함수(BinaryOperator)를 반환
> 
- 스트림의 서로 다른 서브파트를 **병렬로 처리**할 때 누적자가 이 결과를 어떻게 처리할지 정의한다.

ex) ToListCollector의 경우, 스트림의 두번째 서브 파트에서 수집한 항목 리스트를 첫번째 서브 파트 결과 리스트의 뒤에 추가한다.

```java
public BinaryOperator<List<T>> combiner() {
		return (list1, list2) -> {
				list1.addAll(list2);
				return list1;
		}
}
```

병렬화 리듀싱 과정에서 combiner 메서드는 다음과 같이 동작한다.

<img width="360" alt="스크린샷_2025-05-04_오후_5 54 47" src="https://github.com/user-attachments/assets/50c9d48a-3e5c-414b-96b8-95bcfe2b8279" />

1. 스트림을 2개의 서브 파트로 분할
2. 각 서브파트마다 위 그림에서 보았던 순차 리듀싱 과정을 병렬로 처리
3. 병렬 처리된 순차 리듀싱 과정의 결과를 combiner 메서드가 반환하는 함수로 머지(merge)

</br>

***Characteristics 메서드***

> 💡 스트림을 병렬로 리듀스할 것인지, 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지에 대한 힌트(Characteristics 형식의 불변 집합)를 제공
> 
- 앞의 4가지 메서드는 **collect 메서드에서 실행하는 함수를 반환**하는 반면,
- characteristics 메서드는 collect 메서드가 병렬화 같은 최적화를 이용해서 리듀싱 연산을 수행할 것인지 결정하도록 돕는 힌트 특성 집합을 제공한다.

**Characteristics = 3가지 열거형**

- **UNORDERED**
    - 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않음
- **CONCURRENT**
    - 여러 스레드에서 동시에 누적 작업(accumulator) 가능
    - 단, UNORDERED가 함께 설정되어 있거나 데이터 소스가 정렬되어 있지 않아 순서가 무의미한 경우에만 병렬 처리 가능
- **IDENTITY_FINISH**
    - finisher()가 별도 변환 없이 누적자 자체를 결과로 반환
    - 즉, 중간 누적자 객체를 최종 결과 객체로 바로 사용 가능

ex) ToListCollector의 경우, 

- **IDENTITY_FINSIH** : 스트림의 요소를 누적하는 데 사용한 리스트가 최종 결과 형식이므로 추가 변환 필요 X
- **UNORDERED** : 리스트의 순서는 상관 X
- **CONCURRENT** : 요소의 순서가 무의미한 데이터 소스인 경우에만 병렬 처리 가능

</br>

### 6.5.2 응용하기

지금까지 배운 다섯 가지 메서드를 이용해서 ToListCollector를 구현해보자.

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.Collector;
import static java.util.stream.Collector.Characteristics.*;

public class ToListCollector<T> implements Collector<T, List<T>, List<T>> {

    @Override
    public Supplier<List<T>> supplier() {
        return ArrayList::new;
    }

    @Override
    public BiConsumer<List<T>, T> accumulator() {
        return List::add;
    }

    @Override
    public Function<List<T>, List<T>> finisher() {
        return Function.identity();
    }

    @Override
    public BinaryOperator<List<T>> combiner() {
        return (list1, list2) -> {
            list1.addAll(list2);
            return list1;
        };
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH, CONCURRENT));
    }
}
```

</br>

위 구현은 Collectors.toList 메서드와 반환 결과가 완전히 일치하지는 않지만 사소한 최적화

(ex. 실제로 supplier는 싱글턴 `Collections.emptyList()`로 빈 리스트를 반환)를 제외하고는 대체로 비슷하다. 

우리가 구현한 Collector를 사용하고 싶다면 다음과 같이 collect의 인자로 전달하면 된다.

```java
List<Dish> dishes = menuStream.collect(new ToListCollector<Dish>());
```

</br>

기존 toList 코드는 팩토리를 사용하고 ToListCollector는 new 키워드로 인스턴스화한다는 것을 기억하자.

```java
List<Dish> dishes = menuStream.collect(toList());
```

</br>

**컬렉터 구현을 만들지 않고도 커스텀 수집 수행하기**

IDENTITY_FINISH 수집 연산에서는 finisher가 별도로 필요하지 않는다.

따라서, Collector 인터페이스를 새로 구현하지 않고도 결과를 얻을 수 있다.

Stream은 세 함수(발행-`supplier`, 누적-`accumulator`, 합침-`combiner`)를 인수로 받는 collect 메서드를 **오버로드**하여 각각의 메서드가 Collector 인터페이스의 메서드가 반환하는 함수와 같은 기능을 수행하도록 한다.

```java
List<Dish> dishes = menuStream.collect(
		ArrayList::new,  //발행
		List::add,       //누적
		List::addAll);   //합침
```

이런 방식은 좀 더 간결하고 축약되어 있지만 가독성은 떨어진다.

따라서, 적절한 클래스로 커스텀 컬렉터를 구현하여 중복을 피하고 재사용성을 높이는 것이 좋다.

또, 오버로드된 collect는 Characteristics를 전달할 수 없기 때문에 **IDENTITY_FINISH와 CONCURRENT지만 UNORDERED는 아닌 컬렉터**로만 동작해야 한다는 제약이 있다.

</br>

## 6.6 커스텀 컬렉터를 구현해서 성능 개선하기

6.4절에서 분할을 활용하여 2부터 n까지의 자연수를 소수와 비소수 그룹으로 분할하는 코드의 성능을 개선해보자.

```java
public Map<Boolean, List<Integer>> partitionPrimes(int n) {
    return IntStream.rangeClosed(2, n).boxed()
        .collect(partitioningBy(candidate -> isPrime(candidate)));
}
```

```java
public boolean isPrime(int candidate) {
    int candidateRoot = (int) Math.sqrt((double) candidate);
    return IntStream.rangeClosed(2, candidateRoot)
                    .noneMatch(i -> candidate % i == 0);
}
```
</br>

### 6.6.1 소수로만 나누기

기존 코드에서는 탐색 중인 자연수 x가 소수인지 비소수인지를 판단할 때, x를 **2부터 x의 제곱근까지의 모든 숫자**로 나누어 떨어지는지를 확인한다. 

하지만, x가 소수인지를 판단하기 위해서는 x를 **2부터 x의 제곱근까지의 소수**와 나누어 떨어지는지 확인해도 무방하다.

이를 구현하기 위해서는 isPrime 메서드에 현재까지 집계된 소수의 리스트를 전달하여야 한다.

```java
public static boolean isPrime(List<Integer> primes, int candidate){
		return primes.stream().noneMatch(i -> candidate % i == 0);
}
```

마찬가지로 대상 숫자의 제곱근보다 작은 소수만 사용하도록 코드를 최적화할 수 있다.

```java
public static boolean isPrime(List<Integer> primes, int candidate){
		int candidateRoot = (int) Math.sqrt((double) candidate);
		return primes.stream()
                             .takeWhile(i -> i <= candidateRoot)
                             .noneMatch(i -> candidate % i == 0);
}
```

이제 새로운 isPrime을 활용하여 본격적으로 커스텀 컬렉터를 구현하자.
</br>

***1단계 : Collector 클래스 시그니처 정의***

우리는 정수로 이루어진 스트림에서 누적자와 최종 결과의 형식이 `Map<Boolean, List<Integer>>`인 컬렉터를 구현한다.

```java
public class PrimeNumbersCollector 
              implements Collector<Integer,                      //스트림 요소의 형식
                                  Map<Boolean, List<Integer>>,  //누적자 형식
                                  Map<Boolean, List<Integer>>>  //수집 연산의 결과 형식
```

</br>

***2단계 : 리듀싱 연산 구현***

Collector 인터페이스에 선언된 다섯 메서드 중 `supplier`와 `accumulator`를 구현한다.

- supplier 메서드 : 누적자로 사용할 맵 생성 & true, false 키와 빈 리스트 초기화

```java
public Supplier<Map<Boolean, List<Integer>>> supplier() {
    return () -> new HashMap<Boolean, List<Integer>>() {{
        put(true, new ArrayList<Integer>());
        put(false, new ArrayList<Integer>());
    }};
}
```
</br>

- accumulator 메서드 : isPrime의 결과에 따라 각각 소수 리스트와 비소수 리스트에 요소 추가

```java
public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
    return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
        acc.get(isPrime(acc.get(true), candidate))
           .add(candidate);
    };
}
```

</br>

***3단계 : 병렬 실행할 수 있는 컬렉터 만들기(가능하다면)***

병렬 수집 과정에서 두 부분 누적자를 합칠 수 있는 메서드인 `combiner`를 구현한다.

- combiner 메서드 : 두 번째 맵의 소수 리스트와 비소수 리스트의 모든 수를 첫 번째 맵에 추가

```java
public BinaryOperator<Map<Boolean, List<Integer>>> combiner() {
    return (Map<Boolean, List<Integer>> map1, Map<Boolean, List<Integer>> map2) -> {
        map1.get(true).addAll(map2.get(true));
        map1.get(false).addAll(map2.get(false));
        return map1;
    };
}
```

> [!NOTE] 
> 예제는 알고리즘 자체가 순차적이기 때문에 실제로 컬렉터가 병렬 처리될 일은 없다. 따라서, 빈 구현으로 남겨두거나 UnsupportedOperationException을 던지도록 구현해도 상관없다. 위의 구현은 단지 학습용이다.

</br>

***4단계 : finisher 메서드와 컬렉터의 characteristics 메서드***

나머지 두 메서드인 `finisher`와 `characteristics`를 구현한다.

- finisher 메서드 : accumulator의 형식과 최종 결과 형식이 같으므로 항등 함수

```java
public Function<Map<Boolean, List<Integer>>, 
                Map<Boolean, List<Integer>>> finisher() {
    return Function.identity();
}
```

</br>

- characteristics 메서드 : CONCURRENT x, UNORDERED x, IDENTITY_FINISH o

```java
public Set<Characteristics> characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH));
}
```

</br>


***5단계 : collect에 커스텀 컬렉터 전달***

이렇게 PrimeNumbersCollector의 구현을 마쳤으면, 6.4절에서 팩토리 메서드 partitioningBy를 이용했던 예제를 다음처럼 우리가 만든 커스텀 컬렉터로 교체해주면 된다.

```java
public Map<Boolean, List<Integer>> 
    partitionPrimesWithCustomCollector(int n) {
    return IntStream.rangeClosed(2, n).boxed()
                    .collect(**new PrimeNumbersCollector()**);
}
```

</br>


### 6.6.2 컬렉터 성능 비교

**partitioningBy vs 커스텀 Collector**

- 팩토리 메서드 `partitioningBy`로 만든 기본 컬렉터와 우리가 직접 구현한 `PrimeNumbersCollector`는 **기능적으로 동일**
- 하지만 커스텀 컬렉터가 같은 결과를 내면서도 **더 나은 성능**을 가질 수 있음

</br>

**벤치마킹 테스트 예제 (테스트를 10번 반복하여 백만 개의 숫자를 소수와 비소수로 분할)**

```java
public class CollectorHarness {
    public static void main(String[] args) {
        long fastest = Long.MAX_VALUE;
        for (int i = 0; i < 10; i++) {
            long start = System.nanoTime();
            partitionPrimes(1_000_000); // 백만 개의 수 분류
            long duration = (System.nanoTime() - start) / 1_000_000;
            if (duration < fastest) fastest = duration;
        }
        System.out.println("Fastest execution done in " + fastest + " msecs");
    }
}
```

- partitionPrimes를 partitionPrimesWithCustomCollector로 바꿔 실행 시 **약 32% 성능 향상**

Collector 인터페이스를 직접 구현하지 않고도 3개의 함수형 인자(supplier, accumulator, combiner)만으로도 collect가 가능하다.

하지만, 앞서 말했듯이 이 방법은 Characteristics를 명확히 표현하지 못한다.

</br>

**결론**

- 간결함을 원하면 람다 방식
- 재사용성과 명확한 제어가 필요하면 클래스로 구현
