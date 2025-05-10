# 8장 컬렉션 API 개선

8장에서는 자바8과 자바9에서 컬렉션을 더 편하게 사용할 수 있도록 새로 추가된 API에 대해서 알아본다.

# 8.1 컬렉션 팩토리

자바 9에서는 컬렉션 객체를 쉽게 만들 수 있는 방법들을 제공한다.

우선 왜 이러한 기능이 필요한지 짚고 넘어가자.

친구들을 저장하는 작은 리스트는 만드는 상황이라고 하자. 이러한 리스트는 여러 방법으로 만들 수 있다.

첫번째 방법은 다음과 같이 리스트를 만드는 것이다. 그런데 작은 리스트를 만들기 위해서 많은 코드가 필요하다.

```java
List<String> friends = new ArrayList<>()；
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```


`Arrays.asList()` 를 이용하면 코드를 더 간결하게 만들 수 있다.

```java
List<String> friends
= Arrays.asList("Raphael", "Olivia", "Thibaut");

friends.set(0, "Richard");  // 문제가 없다
friends.add("Thibaut");     // UnsupportedOperationException이 발생한다
```

이때 리스트는 요소의 값을 변경할 수는 있지만 요소를 추가하거나 삭제할 수는 없다. 요소를 추가하려고 한다면 `UnsupportedOperationException`이 발생할 것이다. 그 이유는 내부적으로 고정된 크기의 배열로 구현되었기 때문이다. 

여기서 문제는 리스트가 변경된다는 점이다. 만약 변경되게 만들고 싶지 않다면 어떻게 해야할까?

이번에는 작은 Set을 만드는 상황을 보자. 안타깝게도 Arrays.asSet()이란 건 존재하지 않는다. 

우리는 리스트를 인수로 받는 `HashSet`의 생성자를 이용하거나 스트림 API를 이용해야 한다. 하지만 두 방법 모두 깔끔한 방법은 아니며 내부적으로 객체를 할당한다는 비효율성을 지니고 있다.

```java
Set<String> friends
= new HashSet<>(Arrays.asList("Raphael", "Olivia", Thibaut”));

Set〈String〉 friends
= Stream.of("Raphael", "Olivia", "Thibaut")
.collect(Collectors.toSet());
```

맵의 경우는 자바9 이전까지 깔끔하다고 할 만한 방법이 없었다. 더 깔끔한 방법은 없을까?

자바9에서는 새로운 메소드들을 제공함으로써 이러한 문제들을 해결한다.

## 8.1.1 List 팩토리

List.of 팩토리 메소드를 이용해서 리스트를 만들 수 있다.

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut”);
```

이 메소드가 반환한 리스트는  immutable하다. 

- 새로운 요소를 추가하면 `UnsupportedOperationException`이 발생한다.
- `set()`으로도 바꿀 수 없다

이렇게 되면 의도치 않게 리스트가 변경되는 것을 방지할 수 있는 효과가 있다. 만약 리스트를 변경해야한다면 새로운 리스트를 만들면 된다. 

[!TIP]

### List.of 의 오버로딩과 가변인수에 대한 이야기

- List.of()는 다양한 오버로드 버전을 가지고 있다. 그 덕분에 요소가 10개 이상일 때만 가변인수 메소드가 사용된다. 왜 가변인수를 받는 메소드 하나만 있는 게 아니고, 여러 요소를 인자로 받는 메소드들이 오버로드되어 있을까?
- 그 이유는 성능 때문이다. 가변인수를 사용하면 추가 배열을 할당해야한다. 이때 배열을 할당하고 가비지 컬렉션하는 비용이 발생한다. 그래서 오버로드 버전을 통해서 요소가 10개 이하인 경우에는 비용을 줄이고 있다.

```java
static <E> List<E> of(E e1, E e2, E e3, E e4)
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5)

static <E> List<E> of(E... elements)
```

[!TIP] 

### 스트림과 팩토리 메소드 사이의 선택

- 데이터 처리 형식을 설정하거나 데이터를 변환할 필요가 있을 때 스트림을 이용하여 리스트를 만드는 것이 좋다. 그렇지 않은 경우에는 팩토리 메소드를 쓰자

## 8.1.2 Set 팩토리

리스트와 비슷한 방법으로 immutable한 집합을 만들 수 있다.

```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut"); 
```

만약 중복된 요소를 가지고 집합을 만들려고 하면 `IllegalArgumentException`이 발생한다. 

## 8.1.3 Map 팩토리

맵은 키와 밸류를 가지기에 만드는 것이 상대적으로 복잡하다.

자바9에서는 두가지 방식으로 immutable한 맵을 만들 수 있다.

1. `Map.of` 팩토리 메소드
    
    키와 밸류를 차례로 나열하는 방식이다.
    
    ```java
    Map<String, Integer> ageOfFriends
    = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
    ```
    
2. `Map.ofEntry()` 팩토리 메소드
    
    열개 이상의 키와 밸류 쌍을 갖는 맵은 이걸로 만드는 것이 좋다. 이 메소드는 가변 인수를 받는데, 인수는 키와 값을 감싸는 Map.Entry<K , V> 객체이다.
    
    ```java
    import static java.util.Map.entry; 
    
    Map<String, Integer> ageOfFriends
    = Map.ofEntries(entry("Raphael", 30),
    entry("Olivia", 25),
    entry("Thibaut", 26));
    ```
    

# 8.2 List와 Set의 처리

이제 리스트와 집합을 쉽게 처리할 수 있는 방법을 소개한다.

자바8에서는 List,Set 인터페이스에 다음 메소드가 추가되었다.

- `removeIf`: 프레디케이트를 만족하는 요소를 제거한다.
- `replaceAll`: 리스트에서 UnaryOperator를 이용해 요소를 바꾼다.
- `sort` : 리스트를 정렬한다.

참고로 **이 메소드들은 컬렉션 자체를 바꾼다!** 그 이유는 컬렉션을 바꾸는 동작이 에러를 유발하며 복잡하기 때문이다.

## 8.2.1 removeIf 메서드

우선 다음 코드를 보고 문제점을 찾아보자. 트랜잭션에서 참조 코드가 숫자로 시작하면 삭제하는 로직이다.

```java
for (Transaction transaction : transactions) {
	if(Character.isDigit(transaction.getReferenceCode().charAt(O))) {
		transactions.remove(transaction);
	}
}
```

이 코드를 실행하면 `ConcurrentModificationException`이 발생한다. 문제는 for-each 루프가 내부적으로 `Iterator`를 이용하기 때문에 생긴다. 위 코드는 다음과 같이 해석된다.

```java
for (Iterator<Transaction> iterator = transactions.iterator();
iterator.hasNext(); ) {
	Transaction transaction = iterator.next();
	if(Character.isDigit(transaction.getReferenceCode().charAt(0))) {
		transactions. remove( transaction);
	}
}
```

여기서 두개의 객체가 컬렉션을 관리하고 있다!

- Iterator가 소스를 질의하고 있다.
- 컬렉션 객체 자체가 요소를 삭제하고 있다.

이로 인해서 컬렉션의 상태와 반복자의 상태가 동기화되지 않는다.

 

`Iterator`의 remove()를 사용하면 문제를 해결할 수는 있지만 코드가 복잡하다.

```java
if(Character.isDigit(transaction.getReferenceCode().charAt(O))) {
	// transactions. remove( transaction);
	iterator.remove();
}
```

**`removeIf`**가 이 문제를 해결할 수 있다. 

이 메소드는 삭제할 요소를 지시하는 프레디케이트를 인수로 받는다. 

```java
transactions.removelf(transaction ->
Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

## 8.2.2 replaceAll 메서드

리스트의 각 요소를 새로운 요소로 바꿀 수 있다.

스트림 API를 이용하여 코드를 작성해보면 다음과 같다.

```java
referencecodes. stream()
.map(code -> Character.toUpperCase(code.charAt(0)) +
code.substring(1))
.collect(Collectors.toList())
.forEach(System.out: :println);
```

 새로운 문자열 컬렉션을 만들지 않고 기존 컬렉션을 바꾸게 만들려면, `ListIterator`의 set을 이용할 수 있다. 그러나 코드가 복잡하다.

```java
for (ListIterator<String> iterator = referencecodes.listlterator();
iterator.hasNext(); ) {
	String code = iterator.next();
	iterator.set(Character.tollpperCase(code.charAt(0)) + code.substring(1));
}
```

그리고 `Iterator`와 컬렉션이 동시에 참조하는 상황은 위험하다.

따라서 `replaceAll`을 이용하면 된다.

```java
referencecodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) + code.
substring(1));
```

# 8.3 Map 처리

자바8은 Map 인터페이스에 몇가지 디폴트 메서드를 추가했는데 어떤 메서드들이 있는지 알아보자

## 8.3.1 forEach 메서드

맵에서 키와 밸류를 반복하면서 작업하는 것은 귀찮은 작업이다. 반복 하는 방법 중 하나는 반복자로 `Map.Entry<K,V>`를 이용하는 것이다. 예시 코드는 다음과 같다.

```java
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
	String friend = entry,getKey();
	Integer age = entry,getValue();
	System.out.println(friend + " is " + age + " years old");
}
```

자바8에서부터 forEach가 Map 인터페이스에 추가되었으며 `BiComsumer`를 인자로 받는다.

```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " +
age + " years old"));
```

## **8.3.2** 정렬 메서드

자바8에서는 맵의 항목을 쉽게 비교하는 두 개의 유틸리티를 제공한다.

- `Entry.comparingByValue` : 밸류로 정렬
- `Entry.comparingByKey` : 키로 정렬

```java
Map<String, String> favouriteMovies
= Map.ofEntries(entry("Raphael", "Star Wars"),
	entry("Cristina", "Matrix"),
	entry("Olivia",
	"James Bond"));

favouriteMovies
	.entrySet()
	.stream()
	.sorted(Entry.comparingByKey())  
	.forEachOrdered(System.out::printin);
```

[!TIP]

### HashMap의 성능 변화

- 자바8에서 HashMap의 내부 구조를 바꾸어서 성능을 개선했다.
- 기존에는 해쉬충돌이 발생하면 충돌이 일아난 요소를 리스트에 저장했는데, 이렇게 저장하면 최악의 경우 O(n)의 반환 시간이 소요된다.
- 그런데 자바8에서는 리스트 대신에 레드블랙 트리를 사용하기 때문에 반환 성능을 O(log N)으로 높였다.

## **8.3.3 getOrDefault** 메서드

기존에는 키에 해당하는 값이 없으면 Null이 반환되므로 `NullPointerException`에 주의해야했다. 하지만 기본값을 제공하는 getOrDefault 덕분에 이런 문제에서 벗어날 수 있다.

이 메소드는 두개의 인자를 받는다.

- 첫번째 인자 : 키
- 두번째 인자 : 맵에 키가 존재하지 않을 경우 반환할 기본값

```java
Map<String, String> favouriteMovies
= Map.ofEntries(entry("Raphael", "Star Wars"),
entry("Olivia", "James Bond"));

System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix")); // James Bond 출력
System.out,println(favouriteMovies.getOrDefault("Thibaut", "Matrix")); // Matrix 출력
```

## **8.3.4** 계산 패턴

맵에 키가 존재하는지 여부에 따라 어떤 동작을 실행하고 결과를 저장해야 하는 상황이 필요한
때가 있다. 

다음의 세 가지 연산이 이런 상황에서 도움을 준다.

- `computelfAbsent` ： 제공된 키에 해당하는 값이 없거나 값이 널이면, 키를 이용해 새 값을 계산하고 맵에 추가한다
- `computelfPresent` ： 제공된 키가 존재하면 새 값을 계산하고 맵에 추가한다
- `compute` ： 제공된 키로 새 값을 계산하고 맵에 저장한다

### computelfAbsent 메서드

정보를 캐시할 때 활용할 수 있다.

파일의 각 행을 SHA-256으로 인코딩하는 상황을 가정하자. 그렇다면 다음과 같이 computelfAbsent를 이용할 수 있다. 

```java
Map<String, byte[]> dataToHash = new HashMap<>();
MessageDigest messageDigest = MessageDigest.getlnstance("SHA-256"); // 인코딩을 수행하는 객체

private byte[ ] calculateDigest(String key) { 
	return messageDigest.digest(key.getBytes(StandardCharsets.UTF_8));
}

lines.forEach(line -> dataToHash.computelfAbsent(line, this::calculateDigest));
```

여러 값을 저장하는 맵을 처리할 때도 유용하다.

`Map〈K, List〈V〉〉`에요소를 추가하려면 항목이 초기화되어 있는지 확인해야하는데 이는 장황하고 귀찮은 작업이다.

```java
String friend = "Raphael";
List<String> movies = friendsToMovies.get(friend);

// 이 부분이 있어야 한다
if(movies == null) { 
	movies = new ArrayList<>();
	friendsToMovies.put(friend, movies);
}

movies.add("Star Wars");
```

computelfAbsent는 키가 존재하지 않으면 값을 계산해 맵에 추가하고 키가 존재하면 기존 값을 반환한다. 이를 이용하면 다음과 같은 코드를 작성할 수 있다.

```java
friendsToMovies.computelfAbsent("Raphael", name -> new ArrayList<>())
	.add("Star Wars");
```

### computelfPresent 메서드

이 메서드는 현재 키와 관련된 값이 맵에 존재하며 널이 아닐 때만 새 값을 계산한다. 

주의할 점은 값을 만드는 함수가 널을 반환하면 현재 매핑을 맵에서 제거한다!

## **8.3.5** 삭제 패턴 : remove 메서드

매핑을 제거하는 작업은 `remove`를 이용하는 것이 제일 좋다.

다양한 오버로드 버전이 존재한다.

1. 제공된 키에 해당하는 맵 항목을 제거한다.
    
    ```java
    if (favouriteMovies.containsKey(key) &&
    	Objects.equals(favouriteMovies.get(key), value)) {
    	favouriteMovies.remove(key);
    }
    ```
    
2. 자바8에서는 특정한 값과 연관되었을 때만 항목을 제거하는 버전도 제공한다.
    
    ```java
    favouriteMovies.remove(key, value);
    ```
    

## **8.3.6** 교체 패턴

맵의 항목을 바꾸는 데 사용할 수 있는 두 개의 메서드가 있다.

- `replaceAII` ： BiFunction을 적용한 결과로 각 항목의 값을 교체한다.
- `replace` ： 키가 존재하면 맵의 값을 바꾼다. 키가 특정 값으로 매핑되었을 때만 값을 교체하는 오버로드 버전도 있다

```java
Map<String, String> favoriteMovies = new HashMap<>();
favoriteMovies.put("Raphael", "Star Wars");
favoriteMovies.put("Olivia", "james bond");

// 모든 밸류가 대분자로 바뀐다
favoriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

## 8.3.7 두 맵을 합치기

두개의 맵에서 값을 합치거나 바꿔야하는 경우에 사용한다.

### putAll 메서드

두 맵을 합칠 때는 `putAll` 메서드를 이용할 수 있다. **단 중복된 키가 없어야 한다!**

```java
Map<String, String> family = Map.ofEntries(
	entry("Teo", "Star Wars"), entry("Cristina"z "James Bond"));

Map〈String, String> friends = Map.ofEntries(
	entry("Raphael", "Star Wars"));
	
Map<String, String> everyone = new HashMap(family);

everyone.putAll(friends);
```

### merge 메서드

값을 좀 더 유연하게 합쳐야 한다면 이용할 수 있다. 

중복된 키를 어떻게 합칠지 결정하는 BiFunction을 인수로 받는다. 

```java
Map〈String, String> everyone = new HashMap<〉(family); 

// 중복되는 키가 있으면 합친다.
friends.forEach((k, v) ->
	everyone.merge(k, v, (moviel, movie2) -> moviel + " & " + movie2));
```

`merge` 메서드는 널값과 관련된 복잡한 상황도 처리한다

- 만약 지정된 키와 연관된 값이 없거나 널이라면, 키를 널이 아닌 값과 연관시킨다.
- 그렇지 않다면 (연관된 값이 있고 널이 아니라면), 주어진 매핑 함수의 결과로 밸류를 바꾼다. 만약 매핑 함수의 결과가 널이면 밸류를 지운다.

merge를 이용해 초기화 검사를 구현할 수도 있다

```java
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L);
```

- 위 코드에서 두번째 인수는 1이다. 처음에 키와 연관된 값이 널이므로 1로 초기화된다.
- 이후에는 키와 연관된 값이 1로 초기화 되어 있으므로 세번째 인자에 의해서 값이 증가한다.

# **8.4** 개선된 **ConcurrentHashMap**

`ConcurrentHashMap`은 동시성 친화적인 버전의 HashMap이다.

내부 자료구조의 특정 부분만 잠궈서 concurrent add와 update를 지원한다.

synchronized한 `Hashtable`보다 읽고 쓰는 속도가 빠르다. 

[!TIP]

### HashTable의 모든 메소드는 synchronized

- HashTable의 모든 메소드는 synchronized이다. 따라서 여러 스레드가 동시에 메소드를 수행할 수 없다. put을 실행 중이면 get도 동시에 실행할 수 없다.

[!TIP]

### 멀티스레딩에서 HashMap의 대안

- HashMap은 thread-safe하지 않은 자료구조이다. 따라서 멀티스레딩 환경에서 HashMap을 동기화된 블록에서 코드를 작성해야하는데 교착상태가 발생할 가능성이 있다.
- 그래서 멀티스레딩에서는 (HashTable이나) `ConcurrentHashMap`을 이용한다!

## **8.4.1** 리듀스와 검색

ConcurrentHashMap은 세가지 새로운 연산을 지원한다

- forEach : 각 키와 값의 쌍에 주어진 액션을 실행
- reduce ： 모든 키와 값의 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search : 널이 아닌 값을 반환할 때까지 키와 값의 쌍에 함수를 적용

위 세 가지 연산을 연산 대상에 따라서 네 가지 형태로 지원한다. 

- 키와 값으로 연산
    - `forEach`
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 모든 항목 출력
        map.forEach(1, (key, value) -> {
            System.out.println(key + " has stock " + value);
        });
        ```
        
    - `reduce`
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 전체 재고 합산
        int totalStock = map.reduce(1,
            (key, value) -> value,      // (K, V) → Integer로 변환
            Integer::sum                // 합산
        );
        
        System.out.println("Total stock: " + totalStock);
        ```
        
    - `search`
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 재고가 5 이상인 항목을 찾아 키 반환
        String result = map.search(1, (key, value) -> {
            if (value >= 5) return key;
            return null;
        });
        
        System.out.println("Item with enough stock: " + result);
        
        ```
        
- 키로 연산
    - `forEachKey`
        - 상품 이름과 재고 수량이 있는 맵에서 모든 항목을 출력
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 모든 키를 대문자로 출력
        map.forEachKey(1, key -> 
            System.out.println("Key: " + key.toUpperCase()));
        ```
        
    - `reduceKeys`
        - 모든 키를 콤마로 구분된 문자열로 결합.
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 키들을 연결하여 하나의 문자열로 만듦
        String result = map.reduceKeys(1, (s1, s2) -> s1 + ", " + s2);
        
        System.out.println("All keys: " + result);
        ```
        
    - `searchKeys`
        - 이름이 긴 키 찾기
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 길이가 6 이상인 첫 번째 키 찾기
        String foundKey = map.searchKeys(1, key -> {
            if (key.length() >= 6) {
                return key;
            }
            return null;
        });
        
        System.out.println("Found long key: " + foundKey);
        
        ```
        
- 값으로 연산
    - `forEachValue`
        - 각 상품의 재고 수량만 출력
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 모든 재고 수량을 출력
        map.forEachValue(1, value -> 
            System.out.println("Stock: " + value));
        
        ```
        
    - `reduceValues`
        - 재고 수량 중 최대값 계산
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 값 중 최댓값 찾기
        int maxValue = map.reduceValues(1, Integer::max);
        
        System.out.println("Max stock: " + maxValue);
        ```
        
    - `searchValues`
        - 4개 이상인 재고 수량이 있는 첫 항목 찾기
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 4 이상인 첫 번째 재고 수량 찾기
        Integer result = map.searchValues(1, value -> {
            if (value >= 4) return value;
            return null;
        });
        
        System.out.println("Sufficient stock: " + result);
        
        ```
        
- Map.Entry로 연산
    - `forEachEntry`
        - 상품 이름과 재고 수량이 있는 맵에서 모든 항목을 출력함
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 모든 Map.Entry를 출력
        map.forEachEntry(1, entry -> 
            System.out.println(entry.getKey() + " = " + entry.getValue()));
        ```
        
    - `reduceEntries`
        - 총 재고 수량을 계산
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 모든 값을 합산
        int total = map.reduceEntries(1,
            entry -> entry.getValue(),            // 변환: Map.Entry → Integer
            Integer::sum);                        // 병합: Integer끼리 합산
        
        System.out.println("Total stock: " + total);
        
        ```
        
    - `searchEntries`
        - 재고가 충분한 첫 번째 상품 찾기
        
        ```java
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("apple", 3);
        map.put("banana", 5);
        map.put("orange", 2);
        
        // 재고 수가 5 이상인 항목을 검색
        String result = map.searchEntries(1, entry -> {
            if (entry.getValue() >= 5) {
                return entry.getKey(); // 조건 만족 시 키 반환
            }
            return null;
        });
        
        System.out.println("Item with sufficient stock: " + result);
        
        ```
        
    

위의 연산은 ConcurrentHashMap을 잠그지 않고 연산을 수행한다. 따라서 연산에 제공하는 함수는 계산 중에 객체, 값, 순서에 의존해서는 안된다.

한편 위의 연산에 병렬성 기준값(threshold)을 지정해야한다. 연산의 첫번째 인자가 바로 그것이다.

- 맵의 크기가 기준값보다 작으면 순차적으로 수행되고, 그 이상이면 병렬적으로 수행된다.
- 기준값이 1이면 항상 병렬화되고, Long.MAX_VALUE를 기준값으로 지정하면 한 개의 스레드로만 연산이 수행된다.

[!TIP]

### ConcurrentHashMap의 기본형 특화 reduce

int, long, double 등의 기본값에는 전용 each reduce 연산이 제공되므로 `reduceValuesToInt`, `reduceKeysToLong` 등을 이용하면 박싱작업을 할 필요가 없다!

## **8.4.2** Counting

ConcurrentHashMap 클래스는 맵의 매핑 개수를 반환하는 mappingCount 메서드를 제공한다.

이 메소드는 size 메서드와 동일한 역할을 하지만, 반환 타입이 long이라서 int를 반환하는 size 메서드보다 더 큰 크기를 다룰 수 있다. 

**그러니 `size` 메서드 대신에 `mappingCount`를 사용하자!**

## **8.4.3** Set view

ConcurrentHashMap 클래스는 ConcurrentHashMap을 집합 뷰로 반환하는 `keySet`이라는 새

메서드를 제공한다.

**맵을 바꾸면 집합이 바뀌고, 집합을 바꾸면 맵이 바뀐다!**

`newKeySet`이라는 새 메서드를 이용하면 기존 맵과 독립된 집합을 만들 수 있다.

참고자료:

[https://hello-backend.tistory.com/110](https://hello-backend.tistory.com/110)
