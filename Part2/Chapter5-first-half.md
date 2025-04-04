
# 5장 : 스트림 활용

스트림을 이용하면 외부반복을 내부반복으로 바꿀 수 있다. 스트림은 내부적으로 반복을 어떻게 처리할지 결정하여 최적화를 이룰 수 있다. 그리고 코드를 병렬로 처리할지 여부도 결정한다. 이처럼 스트림은 내부 반복을 통해서 외부 반복으로 불가능한 것을 가능하게 한다.

```java
List<Dish> vegetarianDishes = new ArrayList<>();
for(Dish d: menu) {
	if(d.isVegetarian()) {
		vegetarianDishes.add(d);
	}
}

mport static java.util.stream.Collectors.toList;
List<Dish> vegetarianDishes = menu.stream()
	.filter(Dish::isVegetarian)
	.collect(toList());
```

이번 장에서는 스트림과 관련된 다음 개념을 알아본다.

- 자바8과 자바9에서 추가된 연산
- 필터링, 슬라이싱, 검색, 매칭, 리듀싱 등 데이터 처리 질의 표현
- 숫자 스트림, 파일과 배열로부터 만든 스트림, 무한스트림

# 5.1. 필터링

스트림 API는 요소를 선택하는 방법을 제공한다

## 5.1.1 프레디케이트로 필터링

**`filter` 메서드**

- 프레디케이트를 인수로 받아서 프레디케이트를 만족하는 모든 요소를 스트림으로 반환한다.
- 아래 코드는 배송중인 주문상품만을 필터링하고 싶을 때 `filter`메소드를 이용한 예시이다. `Order`의 `isShipping()`은 배송중 상태이면 true를 반환한다.

```java
List<Order> shippingOrders = orders.stream()
				.filter(Order::isShipping)               // 여기
				.collect(toList());
```

## 5.1.2 고유요소 필터링

`distinct` 메서드

- 고유요소로 이루어진 스트림을 반환한다.
- 고유요소 여부는 스트림에서 만든 객체의 `hashCode()`와 `equals()`를 이용해서 판단한다.
- 다음 코드는 명단에서 길이가 5인 이름을 중복없이 조회하기 위해서 `distinct`를 사용한다.

```java
List<String> names = Arrays.asList("Mario", "Luigi", "Peach", "Mushroom", "Peach", "Peach", "Mushroom");
        names.stream()
                .filter(name -> name.length() == 5)
                .distinct()                          // 여기
                .forEach(System.out::println);
```

# 5.2. 스트림 슬라이싱

스트림 API는 요소를 선택하거나 스킵하는 방법을 제공한다.

## 5.2.1 프레디케이트를 이용한 슬라이싱

자바9에서 `takeWhile`과 `dropWhile`을 제공한다.

`takeWhile`  활용

- 우리가 **필터링할 요소가 정렬**되어 있고 **특정 지점 이후부터는 조건을 만족하지 않는** 상황이 있다. 이런 경우에는 조건을 처음으로 만족하지 않는 경우에서 일찍 멈추는 것이 더 효율적이다. 큰 컬렉션의 경우에는 이러한 방식이 성능을 더 높여줄 것이다. 이러한 방식으로 필터링을 수행하는 것이 `takeWhile`이다.
- 다음과 같이 특별한 메뉴 리스트를 가지고 있다고 하자.

```java
List<Dish> specialMenu = Arrays.asList(
		new Dish("seasonal fruit", true, 120, Dish.Type.OTHER),
		new Dish("prawns", false, 300, Dish.Type.FISH),
		new Dish("rice", true, 350, Dish.Type.OTHER),
		new Dish("chicken", false, 400, Dish.Type.MEAT),
		new Dish("french fries", true, 530, Dish.Type.OTHER));
```

- 이 메뉴 리스트에서 칼로리가 320보다 적은 메뉴들만 필터링하고 싶다고 하자. 그렇다면 `filter`를 사용한 방법을 먼저 떠올릴 수 있다.

```java
List<Dish> filteredMenu = specialMenu.stream()
		.filter(dish -> dish.getCalories() < 320)
		.collect(toList());
```

- 그런데 메뉴 리스트를 잘 보면 칼로리에 대해서 오름차순으로 정렬되어 있다.  그러니 `takeWhile`을 쓸 수 있다.

```java
List<Dish> filteredMenu = specialMenu.stream()
		.takeWhile(dish -> dish.getCalories() < 320)
		.collect(toList());
```

`dropWhile`  활용

- 프레디케이트를 만족할 때까지 요소를 버리다가 프레디케이트가 만족하지 않으면 이후 요소를 반환하는 연산이다.
- 무한스트림에서도 동작을 한다.
- 앞서 본 메뉴리스트에서 320 칼로리보다 큰 메뉴를 찾는다면 다음과 같이 `dropWhile`을 이용한다.

```java
List<Dish> filteredMenu = specialMenu.stream() 
		.dropWhile(dish -> dish.getCalories() < 320) // 여기
		.collect(toList());
```

## 5.2.2 스트림 축소

`limit` 

- 주어진 길이 이하의 스트림을 반환하는 메소드이다.
- 정렬되지 않은 스트림에도 `limit`을 적용할 수 있다. 이때 결과값은 정렬되지 않은 상태이다.
- 다음은 최대 3개의 요리를 반환한다.

```java
List<Dish> dishes = specialMenu.stream()
		.filter(dish -> dish.getCalories() > 300)
		.limit(3)           // 여기
		.collect(toList());
```

## 5.2.3 요소 건너뛰기

`skip`

- 처음 n개의 요소를 제외한 스트림을 반환한다.
- 스트림의 요소개 n개 이하라면 빈 스트림을 반환한다.
- 처음 2개의 요리를 건너뛰고 나머지 요리를 보여준다.

```java
List<Dish> dishes = menu.stream()
		.filter(d -> d.getCalories() > 300)
		.skip(2)           // 여기
		.collect(toList());
```

# 5.3 매핑

스트림 API에서는 특정 객체에서 특정 값을 선택하는 연산을 지원한다.

## 5.3.1 스트림의 각 요소에 함수 적용하기

`map` 메소드

- 함수를 인자로 받아서 스트림의 각 요소에 함수를 적용한 결과가 새로운 요소로 변환된다.
- map은 여러개를 나열해서 연결할 수 있다.
- 다음 코드는 주문 리스트에서 주문자의 이름을 모두 찾아서 반환한다.

```java
List<String> names = orders.stream()
	.map(Order::getOrderer())
	.map(Orderer::getName())
	.collect(toList());
```

## 5.3.2 스트림 평면화

`flatMap` 메소드

- 스트림의 각 값을 스트림으로 만든 다음에 모든 스트림을 하나의 스트림을 연결하는 기능을 수행한다. 스트림으로부터 스트림의 스트림이 발생하는 경우에 **스트림의 스트림을 하나의 스트림으로 평면화**하는 역할을 한다! 이 의미는 예시를 통해서 알아보자
- 예를 들어서 [”Hello”, “World”] 라는 리스트가 있다고 하자. 문자열 배열에 나타나는 모든 문자를 요소로 갖는 리스트 ["H", "e", "l", "o", "W", "r", "d"]를 만들어야 한다고 하자.
    - 각 문자열을 문자로 변환한 후에 distinct를 적용하면 되지 않을까 생각할 수 있다. 하지만 이는 원하는 결과를 가져다 주지 못한다. 왜 그럴까?
    - 이유는 `map`을 사용한 결과 `String`은 `String[]` 배열이 된다. 따라서 결과 스트림은 `Stream<String[]>`이다!
        
        ```java
        words.stream()
        	.map(word -> word.split(""))
        	.distinct()
        	.collect(toList());
        ```
        
        ![image.png](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20%E1%84%92%E1%85%AA%E1%86%AF%E1%84%8B%E1%85%AD%E1%86%BC%201cb2dcb5a74a80eab30ac1391104b0e4/image.png)
        
    - 우리는 문자열 스트림을 결과로 얻고 싶다. `Arrays.stream()` 을 이용하면 문자열 배열을 문자열 스트림으로 만들 수 있다. 그렇다면 map에서 Arrays.stream()을 이용하면 해결이 되느냐? 그건 아니다. 여전히 결과는 우리가 원하는 `Stream<String>`이 아니라 `Stream<Stream<String>>`으로 변화했을 뿐이다.
        
        ```java
        words.stream()
        	.map(word -> word.split(""))
        	.map(Arrays::stream)
        	.distinct()
        	.collect(toList());
        ```
        
    - 이제 `flatMap`이 등장할 차례이다. `flatMap`을 이용하면 **배열의 각 요소를 스트림으로 변환하지 않고, 스트림의 콘텐츠로 변환**한다! **하나의 평면화된 스트림을 제공**하는 것이다!
        
        ```java
        words.stream()
        	.map(word -> word.split(""))
        	.flatMap(Arrays::stream)
        	.distinct()
        	.collect(toList());
        ```
        
        ![image.png](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20%E1%84%92%E1%85%AA%E1%86%AF%E1%84%8B%E1%85%AD%E1%86%BC%201cb2dcb5a74a80eab30ac1391104b0e4/image%201.png)
        

# 5.4 검색과 매칭

## **5.4.1** 프레디케이트가 적어도 한 요소와 일치하는지 확인

`anyMatch`

- 주어진 스트림에 프레디케이트를 만족시키는 원소가 하나라도 있는지 확인할 때 사용한다.
- 최종 연산에 해당한다.

```java
if(menu.stream().anyMatch(Dish::isVegetarian)) {
	System.out.println("채식주의자를 위한 메뉴 있음!!");
}
```

## **5.4.2** 프레디케이트가 모든 요소와 일치하는지 검사

`allMatch`

- 주어진 스트림에 있는 모든 원소가 프레디케이트를 모두 만족하는지 확인할 때 사용한다.

```java
boolean isHealthy = menu.stream().allMatch(dish -> dish.getCalories() < 1000);
```

`noneMatch`

- allMatch 와 반대이다. 주어진 스트림에 있는 모든 원소가 프레디케이트를 모두 만족하지 않으면 true를 리턴한다.

```java
boolean isHealthy = menu.stream().noneMatch(d -> d.getCalories() >= 1000);
```

**기억할 점 : 쇼트서킷**

anyMatch, allMatch, noneMatch 모두 쇼트 서킷 기법을 이용한다. 만약 스트림의 모든 원소에 대해서 확인하지 않은 상태에서도 결과를 반환할 수 있다면 반환한다.

## **5.4.3** 요소 검색

`findAny`

- 스트림에서 임의의 원소를 반환한다. 다른 스트림 연산과 연결해서 사용할 수 있다.

```java
Optional<Dish> dish = menu.stream()
		.filter(Dish::isVegetarian)
		.findAny();
```

<aside>
💡

### Optional이란?

`java.util.Optional<T>` 는 값의 존재나 부재 여부를 표현하는 컨테이너 클래스다

null로 인한 버그를 해결하기 위해서 고안되었다.

값이 존재하는지 확인하는 작업과 값이 없는 경우 해야할 작업을 강제할 수 있다.

주요메소드

- `isPresent()` : 값이 존재하면 true를 반환하고, 값이 없으면 false를 리턴한다.
- `ifPresent(Consumer<T> block)` : 값이 존재하면 `block`을 실행한다.
- `T get()` : 값이 존재하면 T 타입 객체를 반환하고, 값이 없으면 `NoSuchElementException`을 반환한다.
- `T orElse(T other)` : 값이 있으면 그 값을 반환하고, 값이 없으면 기본값으로 `other`를 반환한다.
</aside>

## **5.4.4** 첫 번째 요소 찾기

`findFirst`

정렬된 연속적인 데이터로부터 생성된 스트림에서 첫번째 요소를 찾을 때 사용한다.

```java
Optional<Integer> firstSquareDivisibleByThree = someNumbers.stream()
		.map(n -> n * n)
		.filter(n -〉 n % 3 == 0)
		.findFirst();
```

<aside>
💡

### findAny랑 findFirst는 언제 쓰나?

병렬성 때문에 `findAny`와 `findFirst`가 둘 다 제공된다. 병렬 실행 환경에서는 첫번째 원소를 찾기 어려우니 `findAny`를 사용하자.

</aside>

# 5.5. 리듀싱

“메뉴의 모든 칼로리의 총합을 계산하시오”와 같은 질의를 처리하는 경우가 있다. 이 때 결과가 나올때까지 스트림의 모든 원소를 반복적으로 처리해야 한다. 이렇게 모든 스트림의 요소를 반복해서 결과를 만드는 연산을 **리듀싱 연산**이라고 한다. 함수형 프로그래밍에서는 **폴드**라고 한다.

## **5.5.1** 요소의 합

리스트의 모든 원소를 더하는 예제를 보자. 보통 다음과 같이 코드를 작성할 수 있다.

```java
int sum = 0; // 초기값
for (int x : numbers) {
	sum += x; // 연산
}
```

- 위 코드를 보면 초깃값이 있고, 리스트의 모든 요소를 조합하는 연산이 있다. 그런데 리스트의 모든 원소를 곱하는 코드를 작성해야한다면 어떨까? 위 코드를 복사해서 연산을 수정해야한다. `reduce`를 이용하면 위의 코드가 반복하지 않아도 된다.
- 다음과 같이 reduce를 이용하여서 위의 코드를 변경할 수 있다. 곱셈의 경우에는 두번째 인자의 람다식을 변경하면 된다.
    
    ```java
    numbers.stream().reduce(0, (a, b) -> a + b);
    ```
    
- 참고로 Integer는 정적 메소드 sum을 제공한다. 따라서 메소드 참조를 이용해서 모든 원소의 합을 다음과 같이 계산할 수 있다.
    
    ```java
    numbers.stream().reduce(0, Integer::sum);
    ```
    

이 예제를 통해서 reduce의 연산과정을 알아보자

- 그림으로 나타내면 다음과 같다. 먼저 초깃값 0이 있다. 첫번째 반복에서 첫 원소 4와 더해서 값이 4가 도니다. 두번째 반복에서 누적값 4와 두번째 원소 5를 더하면 9가 된다. 이런식으로 하나의 값이 될때까지 반복한다.
    
    ![image.png](5%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B7%20%E1%84%92%E1%85%AA%E1%86%AF%E1%84%8B%E1%85%AD%E1%86%BC%201cb2dcb5a74a80eab30ac1391104b0e4/image%202.png)
    

`reduce` 

- 두개의 인자를 갖는다.
    - 첫번째 인자로 **초깃값**을 줄 수 있다.
        
        <aside>
        💡
        
        초기값이 없을 수도 있다. 그런데 이 경우에는 `Optional`을 반환한다. 빈 스트림에 대해서 reduce를 호출하였을 때는 반환할 값이 없기 때문이다.
        
        </aside>
        
    - 두번째 인자는 두 요소를 인자로 받아서 하나의 요소로 만드는 **`BinaryOperator<T>`**이다.

## **5.5.2** 최댓값과 최솟값

스트림에서 최댓값을 찾을 때도 `reduce`를 이용할 수 있다.

메소드 참조를 이용해서 최댓값을 찾는 코드는 다음과 같다.

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
```

메소드 참조를 이용해서 최솟값을 찾는 코드는 다음과 같다.

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
```

- 메소드 참조를 이용하지 않고 직접 람다식을 작성해도 된다.
    
    ```java
    Optional<Integer> max = numbers.stream().reduce((a, b)-> a<b ? a : b);
    ```
    

<aside>
💡

## reduce의 장점 : 병렬화 용이

왜 기존의 반복문을 사용하지 않고 reduce를 사용하여서 모든 요소의 합을 계산해야하는가? reduce를 사용하면 병렬로 reduce를 계산할 수 있다. 기존의 반복문을 이용할 경우에는 공유변수가 존재하고 경쟁상태가 발생하기 때문에 병렬화가 어렵다는 점은 익히 알려진 사실이다. 그런데 reduce를 이용하면 이러한 골치아픈상황을 고민할 필요가 줄어든다.

</aside>