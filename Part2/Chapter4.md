# 스트림 소개

SQL 쿼리문과 컬렉션을 비교해보자.

```sql
SELECT name FROM dishes WHERE calorie < 400;
```

이렇게 sql 쿼리를 호출하면, `dished`라는 테이블에서 `calorie` 라는 속성으로 필터링 되어서 결과값이 반환된다.

하지만, 컬렉션은 어떠한가? 우리가 반복문과 누적자를 이용해서 직접 어떻게 필터링해야 하는지를 구현해줘야 한다. 또, 이들을 사용해서 필터링을 구현해주었다고 하더라도 많은 요소가 포함된 커다란 컬렉션이라면, 서버 성능의 저하를 초래한다.

병렬로 처리하면 되지 않느냐고 물을 수도 있다. 하지만 병렬 처리를 위해선 멀티코어 아키텍처를 확용해야 하는데 경험해보면 알 수 있듯이 단순 반복 처리 코드에 비해 복잡하고 어려울 뿐더러 디버깅도 쉽지 않다…

이와 같은 문제를 해결하기 위해 나온  기술이 바로 **스트림** 이다.

# 4.1 스트림이란 무엇인가?

**자바 8 이전 코드 (필터링 → 정렬 → 매핑)**

```java
//필터링
List<Dish> lowCaloricDishes = new ArrayList<>();
for (Dish dish : menu) { // 누적자로 요소 필터링
    if (dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}

//정렬
Collections.sort(lowCaloricDishes, new Comparator<Dish>() { // 익명 클래스로 요리 정렬
    public int compare(Dish dish1, Dish dish2) {
        return Integer.compare(dish1.getCalories(), dish2.getCalories());
    }
});

//매핑
List<String> lowCaloricDishesName = new ArrayList<>();
for (Dish dish : lowCaloricDishes) {
    lowCaloricDishesName.add(dish.getName()); // 정렬된 리스트를 처리하면서 요리 이름 선택
}

```

위 코드에서는 로직 상으로는 전혀 문제가 없다.

하지만 `lowCaloricDishes` 변수의 역할을 살펴보면, 필터링과 정렬의 결과를 잠시 담아두기 위한 중간 변수, 즉 가비지 변수 라는 것을 알 수 있다. 

**자바 8 이후 코드 (stream 적용)**

```java
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.toList;

List<String> lowCaloricDishesName =
    menu.**stream**()
        .filter(d -> d.getCalories() < 400)               // 400칼로리 이하의 요리 선택
        .sorted(comparing(Dish::getCalories))             // 칼로리로 요리 정렬
        .map(Dish::getName)                               // 요리명 추출
        .collect(toList());                               // 모든 요리명을 리스트에 저장
```

이런 streamAPI는 소프트웨어공학적으로 다양한 이득을 가져다 준다.

1. **선언형** : 더 간결하고 가독성이 좋아진다
    - 동작을 어떻게 구현할지 지정할 필요 X → “저칼로리의 요리만 선택하라” 같은 동작을 선언하여 수행
    - 선언형 코드는 3장에서 배운 동작 파라미터화와 함께 활용하면 변하는 요구사항에 쉽게 대응이 가능
2. **조립할 수 있음** : 유연성이 좋아진다
    - filter, sorted, map, collect 같은 **고수준 빌딩 블록 연산**을 하나의 파이프라인 안에 명확하게 사용이 가능
    - **고수준 빌딩 블록**은 특정 스레딩 모델에 제한되지 않고 자유롭게 어떤 상황에서든 사용이 가능 (내부적으로 단일 스레드 모델에 사용할 수 있지만 멀티코어 아키텍처를 최대한 투명하게 활용할 수 있게 구현되어 있음)

> [!NOTE]
> 
> **빌딩 블록 방식**
>
> 하나의 컴퓨터 시스템이나 대형 소프트웨어를 설계하는 과정에서 이들 제품을 구성하는 각각의 구성 요소를 서로 독립된 모듈로써 구성하는 방법
>
> <img width="398" alt="스크린샷_2025-04-02_오후_9 01 48" src="https://github.com/user-attachments/assets/9a000e39-a336-44dd-902e-0112ba396e50" />
    
    
3. **병렬화** : 성능이 좋아진다
    
    위에서 본 코드에서 `parallelStream()`을 사용하면 손쉽게 컬렉션을 병렬처리 할 수 있다.
    
    ```java
    //병렬 처리 stream
    List<String> lowCaloricDishesName =
        menu.**parallelStream**()
            .filter(d -> d.getCalories() < 400)
            .sorted(comparing(Dish::getCalories))
            .map(Dish::getName)
            .collect(toList());
    ```
    

예제를 하나 살펴보자.

```java
List<Dish> menu = Arrays.asList(
    new Dish("pork", false, 800, Dish.Type.MEAT),
    new Dish("beef", false, 700, Dish.Type.MEAT),
    new Dish("chicken", false, 400, Dish.Type.MEAT),
    new Dish("french fries", true, 530, Dish.Type.OTHER),
    new Dish("rice", true, 350, Dish.Type.OTHER),
    new Dish("season fruit", true, 120, Dish.Type.OTHER),
    new Dish("pizza", true, 550, Dish.Type.OTHER),
    new Dish("prawns", false, 300, Dish.Type.FISH),
    new Dish("salmon", false, 450, Dish.Type.FISH)
);

class Dish {
    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;

    public Dish(String name, boolean vegetarian, int calories, Type type) {
        this.name = name;
        this.vegetarian = vegetarian;
        this.calories = calories;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public int getCalories() {
        return calories;
    }

    public Type getType() {
        return type;
    }

    @Override
    public String toString() {
        return name;
    }

    public enum Type { MEAT, FISH, OTHER }
}
```

현재 menu에 담겨있는 요리 리스트를 Dish.Type이 같은 것끼리 그룹화하는 요구사항이 들어왔다.

```java
{
    FISH = [prawns, salmon],
    OTHER = [french fries, rice, season fruit, pizza],
    MEAT = [pork, beef, chicken]
}
```

위와 같은 결과가 나오려면 streamAPI를 어떻게 활용해야 할까? 

코드가 바로 떠오르지 않는다면, 앞으로 4,5,6 장을 열심히 학습하자.

- 정답
    
    ```java
    Map<Dish.Type, List<Dish>> dishesByType =
        menu.stream()
             .collect(Collectors.groupingBy(Dish::getType));
    
    ```
    

# 4.2 스트림 시작하기

### 스트림의 정의

> ***데이터 처리 연산**을 지원하도록 **소스**에서 추출된 **연속된 요소***
> 
- **연속된 요소**
    - 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스를 제공 (컬렉션과 동일)
 
> [!NOTE]
>  **컬렉션과 스트림의 차이**
> - 컬렉션은 자료구조이므로 시간과 공간의 복잡성과 관련된 요소 저장 및 접근 연산이 주를 이룸
>    - ex) ArrayList를 사용할지, LinkedList를 사용할지
> - 그와 달리, 스트림은 표현 계산식이 주를 이룸
>    - ex) filter, sorted, map 등등
    
    
- **소스**
    - 컬렉션, 배열, I/O 자원로부터 제공받는 데이터 소스를 소비하며 해당 데이터 제공 소스의 순서는 그대로 유지됨
        - 정렬된 컬렉션으로 스트림을 생성하면 정렬이 그대로 유지됨
- **데이터 처리 연산**
    - 함수형 프로그래밍 언어에서 **일반적으로 지원하는 연산**과 **데이터베이스와 비슷한 연산**을 모두 지원
    - ex) filter, map, reduce, find, match, sort 등등 데이터 조작 연산이 다양함

### 스트림의 특징

- **파이프라이닝**
    - 스트림 연산은 스트림 자신을 반환하여 다른 스트림 연산과 연결되는 **메서드 체이닝 구조**를 띄고 있음
    - 스트림 연산끼리 연결되어서 하나의 커다란 파이프 라인을 구성하는 형태
        - 데이터베이스 쿼리문과 비슷 (실제로 `Querydsl`과 아주 유사한 형태라고 생각합니다)
        - **게으름**, **쇼트서킷** 같은 최적화도 활용 가능 (5장에서 설명)
    
- **내부 반복**
    - 컬렉션 → 반복자를 이용해서 명시적으로 반복
    - 스트림 → 내부 반복을 지원 (4.3에서 다룰 것!)

다음의 예제를 살펴보며 정의를 구체화해보자.

```java
import static java.util.stream.Collectors.toList;

List<String> threeHighCaloricDishNames = 
    menu.stream()                           // 메뉴(요리 리스트)에서 스트림을 얻는다
        .filter(dish -> dish.getCalories() > 300)  // 고칼로리 요리를 필터링한다
        .map(Dish::getName)                // 요리명을 추출한다
        .limit(3)                          // 선착순 세 개만 선택한다
        .collect(toList());               // 결과를 다른 리스트로 저장한다

System.out.println(threeHighCaloricDishNames); // 결과는 [pork, beef, chicken]이다
```

1. 요리 리스트(메뉴)라는 **데이터 소스**로부터 stream 메서드를 호출해서 스트림 얻음
2. 이때, 스트림은 **연속된 요소**로 이루어져 있음
3. 스트림에 filter, map, limit, collect로 이어지는 일련의 **데이터 처리 연산**을 적용
    1. collect를 제외한 모든 연산은 서로 **파이프라인**을 형성할 수 있도록 스트림을 반환
    2. 마지막에 collect를 호출하기 전까지는 menu에서 무엇도 선택되지 않으며 출력 결과도 없음
    3. collect는 스트림을 다른 형식(ex. 리스트)으로 변환

지금까지 글로 말한 내용을 그림으로 표현하면 다음과 같다.

<img width="520" alt="스크린샷_2025-04-04_오후_4 29 54" src="https://github.com/user-attachments/assets/3eb99f64-245b-452e-bbbb-679485cb4bb8" />


# 4.3 스트림과 컬렉션

## 공통점

> **연속된** 요소 형식의 값을 저장하는 자료구조의 인터페이스를 제공한다
> 
- **연속된**? → 순서와 상관없이 아무 값에나 접속하는 것이 아니라 순차적으로 값에 접근한다는 것을 의미

## 차이점

### 1. 데이터를 계산하는 시점

- 컬렉션
    - 현재 자료구조가 포함하는 **모든** 값을 메모리에 저장하는 자료구조
    - 컬렉션의 모든 요소는 컬렉션에 추가하기 전에 계산되어야 함
    - 생산자 중심 ⇒ 적극적으로 생성
- 스트림
    - **요청할 때만 요소를 계산**하는 고정된 자료구조
    - 사용자가 요청하는 값만 스트림에서 추출
    - 생산자 ↔ 소비자 관계

 > [!IMPORTANT]
>  💡 생산자 ↔ 소비자 관계
>  > 스트림은 딱 한번만 탐색할 수 있다!
>  - 탐색된 스트림의 요소는 소비됨
>  - 한번 탐색한 요소를 다시 탐색하려면 초기 데이터 소스에서 새로운 스트림을 생성해야 됨
>  ```java
>  List<String> title = Arrays.asList("Java8", "In", "Action");
>  Stream<String> s = title.stream();
>  s.forEach(System.out::println);    // title의 각 단어를 출력
>  s.forEach(System.out::println);    // java.lang.IllegalStateException : 스트림이 이미 소비되었거나 닫힘
>  ```


### 2. 데이터 반복 처리 방법

- **컬렉션 : 외부반복**
    - 사용자가 직접 요소를 반복해야 함 (ex. for-each)
    
    ```java
    List<String> names = new ArrayList<>();
    for (Dish dish : menu) {                      // 메뉴 리스트를 명시적으로 순차 반복한다
        names.add(dish.getName());               // 이름을 추출해서 리스트에 추가한다
    }
    
    List<String> names = new ArrayList<>();
    Iterator<Dish> iterator = menu.iterator();
    while (iterator.hasNext()) {                    // 명시적 반복
        Dish dish = iterator.next();
        names.add(dish.getName());                  // 이름을 추출해서 리스트에 추가
    }
    ```
    
- **스트림 : 내부반복**
    - 반복을 알아서 처리하고 결과 스트림 값을 어딘가에 저장해줌
    
    ```java
    List<String> names = menu.stream()              // 파이프라인을 실행
        .map(Dish::getName)                         // getName 메서드로 요리명을 추출
        .collect(Collectors.toList());              // 리스트로 수집
    ```
    

**내부반복의 장점**

1. 병렬성 처리가 쉽다
    - for-each와 같은 외부반복은 병렬성을 스스로 관리해야 한다(`synchronized` 키워드를 사용하며 신경써줘야 할 일이 많아짐)
    - 스트림 라이브러리의 내부 반복은 데이터 표현과 하드웨어를 활용한 병렬성 구현을 자동으로 선택한다
2. 데이터 처리 순서를 최적화할 수 있다
    - 스트림은 선언형 코드를 사용하기 때문에 동작의 순서들을 선언만 해두면 코드 내부에서 해당 순서대로 데이터를 처리해준다
    - 위 스트림 예시에서도 `Dish`의 이름을 통해 먼저 요리명을 추출하고 이 추출한 요소들을 리스트로 수집하는 과정을 선언해두어 스트림 내부적으로 반복을 통해 결과적으로 `names`라는 리스트가 반환되는 것을 볼 수 있다

# 4.4 스트림 연산

```java
List<String> names = menu.stream()                      // 요리 리스트에서 스트림 얻기
    .filter(dish -> dish.getCalories() > 300)           // 중간 연산: 고칼로리 요리 필터링
    .map(Dish::getName)                                 // 중간 연산: 요리명 추출
    .limit(3)                                           // 중간 연산: 상위 3개만 선택
    .collect(Collectors.toList());                      // 최종 연산: 스트림을 리스트로 변환
```

위 예제에서 연산을 두 그룹으로 구분할 수 있다.

- 중간 연산
    
    > filter, map, limit
    > 
    - 서로 연결되어 파이프라인을 형성
- 최종 연산
    
    > collect
    > 
    - 파이프라인을 실행(소비)한 다음 닫음

<img width="349" alt="스크린샷_2025-04-04_오후_6 11 32" src="https://github.com/user-attachments/assets/a33c35cb-5198-4e8a-91ec-0f3e18f9adc1" />


### 중간 연산

- 연산을 수행해서 결과로 stream을 반환한다
- **단말 연산을 스트림 파이프라인에 실행하기 전까지 아무 연산도 수행하지 않는다**

두번째 특징이 이해가 되지 않을 수 있다.

중간 연산 중간에 콘솔 출력으로 어떤 과정을 거쳐서 연산이 이루어지는지 살펴보자.

```java
List<String> names = 
    menu.stream()
        .filter(dish -> {
            System.out.println("filtering:" + dish.getName()); // 필터링한 요리명을 출력
            return dish.getCalories() > 300;
        })
        .map(dish -> {
            System.out.println("mapping:" + dish.getName());   // 추출한 요리명을 출력
            return dish.getName();
        })
        .limit(3)
        .collect(Collectors.toList());

System.out.println(names);
```

결과

```
filtering:pork
mapping:pork
filtering:beef
mapping:beef
filtering:chicken
mapping:chicken
[pork, beef, chicken]
```

결과를 살펴보면, 각 메서드 별로 단계가 나누어지는 것이 아니라 각 요소 별로 단계가 나누어지는 것을 볼 수 있다. 즉, 연산 순서의 기준이 메서드가 아닌 단계인 것이다.

이와 같은 방식으로 연산을 수행하면, 2 가지 최적화 효과를 얻을 수 있다.

1. filter 연산에서 나오는 결과가 여러 가지라도 limit로 인해 필요한 만큼만 요소가 선택되어서 연산된다
    - 300칼로리가 넘는 요리는 여러 개지만 오직 처음 3개만 선택된 것을 볼 수 있음
    - limit 연산 & **쇼트서킷**이라는 기법을 이용 (5장에서 자세히 설명)
2. 서로 다른 중간 연산이라도 한 과정으로 병합될 수 있다
    - filter와 map은 다른 연산이지만 하나의 요소에 대해서 하나의 과정으로 병합되어서 연산됨
    - **루프 퓨전** 기법을 이용

### 최종 연산

- 스트림 파이프라인에서 결과를 도출
- List, Integer, void 등 스트림 이외의 결과가 반환됨
    
    ex) forEach는 void를 반환하는 최종 연산 
    
    ```java
    menu.stream().forEach(System.out::println);
    ```
    

### 스트림 구조

- 질의를 수행할 (컬렉션 같은) **데이터 소스**
- 스트림 파이프라인을 구성할 **중간 연산** 연결
- 스트림 파이프라인을 실행(소비)하고 결과를 만들 **최종 연산**

참고 ) 스트림 파이프라인의 개념은 빌더 패턴과 유사하다 

```java
User user = new User.builder() // 데이터 소스
            .name("홍길동")     // 중간 연산
            .age(25)
            .email("hong@example.com")
            .build();         // 최종 연산
```

**중간 연산**

| 연산 | 형식 | 반환 형식 | 연산의 인수 | 함수 디스크립터 |
| --- | --- | --- | --- | --- |
| filter | 중간 연산 | Stream<T> | Predicate<T> | T → boolean |
| map | 중간 연산 | Stream<R> | Function<T, R> | T → R |
| limit | 중간 연산 | Stream<T> | - | - |
| sorted | 중간 연산 | Stream<T> | Comparator<T> | (T, T) → int |
| distinct | 중간 연산 | Stream<T> | - | - |

**최종 연산**

| 연산 | 형식 | 반환 형식 | 목적 |
| --- | --- | --- | --- |
| forEach | 최종 연산 | void | 스트림의 각 요소를 소비하면서 람다를 적용한다 |
| count | 최종 연산 | long (generic) | 스트림의 요소 개수를 반환한다 |
| collect | 최종 연산 | - | 스트림을 리듀스해서 리스트, 맵, 정수 형식의 컬렉션을 만든다. 자세한 내용은 6장 참조 |
