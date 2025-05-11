# Chapter 9 (9.3~9.4)

## 9.3 람다 테스팅

람다 표현식을 적용해 코드를 간결하게 만들 수 있지만, 핵심은 **정확히 작동하는 코드**를 구현하는 것이다.

따라서 단위 테스트(unit test)를 통해 람다가 의도대로 동작하는지 반드시 확인해야 한다.

- **예시 : Point 클래스**

```java
public class Point {
    private final int x;
    private final int y;

    private Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    public Point moveRightBy(int x) {
        return new Point(this.x + x, this.y);
    }
}
```

- **테스트 코드**

```java
@Test
public void testMoveRightBy() throws Exception {
    Point p1 = new Point(5, 5);
    Point p2 = p1.moveRightBy(10);
    assertEquals(15, p2.getX());
    assertEquals(5, p2.getY());
}
```

</br>

### 9.3.1 보이는 람다 표현식의 동작 테스팅

moveRightBy는 public 메서드이기 때문에 직접 테스트가 가능하다.

하지만 람다는 익명 함수이므로 테스트 코드에서 직접 이름으로 호출할 수 없다.

- **해결책 : 람다를 클래스 필드에 저장해서 재사용하고 이를 테스트**
    - Point 클래스 2개를 X 좌표 → Y 좌표 순서대로 비교하는 코드

```java
public class Point {
    public final static Comparator<Point> compareByXAndThenY =
        comparing(Point::getX).thenComparing(Point::getY);
}
```

- **테스트 코드**

```java
@Test
public void testComparingTwoPoints() throws Exception {
    Point p1 = new Point(10, 15);
    Point p2 = new Point(10, 20);
    int result = Point.compareByXAndThenY.compare(p1, p2);
    assertTrue(result < 0);
}
```

</br>

### 9.3.2 람다를 사용하는 메서드의 동작에 집중하라

> 💡 람다의 목표 : 정해진 동작을 다른 메서드에서 사용 가능하도록 하나의 조각으로 캡슐화하는 것
> 

따라서, 람다의 구현이 아닌, **람다가 사용된 메서드의 동작만 검증**하면 간접적으로 람다를 테스트 할 수 있다.

```java
public static List<Point> moveAllPointsRightBy(List<Point> points, int x) {
    return points.stream()
                 .map(p -> new Point(p.getX() + x, p.getY()))
                 .collect(toList());
}
```

- 테스트 코드
    - `p -> new Point(p.getX() + x, p.getY())` 테스트 X
    - `moveAllPointsRightBy` 메서드의 동작 자체를 테스트 O

```java
@Test
public void testMoveAllPointsRightBy() throws Exception {
    List<Point> points =
        Arrays.asList(new Point(5, 5), new Point(10, 5));
    List<Point> expectedPoints =
        Arrays.asList(new Point(15, 5), new Point(20, 5));
    List<Point> newPoints = Point.moveAllPointsRightBy(points, 10);
    assertEquals(expectedPoints, newPoints);
}
```

> ⚠️ Point 클래스에 equals() 오버라이딩이 필요!
> 

</br>

### 9.3.3 복잡한 람다를 개별 메서드로 분할하기

많은 로직을 포함하는 복잡한 람다 표현식을 테스트하고 싶다면 어떻게 해야할까?

- **복잡한 로직을 담은 람다식**

```java
List<String> result = items.stream()
    **.filter(item -> {
        if (item == null) return false;
        String trimmed = item.trim();
        return !trimmed.isEmpty() && trimmed.startsWith("A");
    })**
    .collect(Collectors.toList());
```

위 코드의 filter() 내부 람다에는 다음과 같은 복잡한 로직이 숨어 있다

1. null 체크
2. 공백 제거
3. 문자열 시작 조건 확인

하지만 람다는 익명이기 때문에 **직접 테스트가 불가능하다.**

- **해결책 : 람다를 메서드 참조로 변경**

```java
List<String> result = items.stream()
    .filter(**MyFilter::isValidItem**)
    .collect(Collectors.toList());

public class MyFilter {
    public static boolean isValidItem(String item) {
        if (item == null) return false;
        String trimmed = item.trim();
        return !trimmed.isEmpty() && trimmed.startsWith("A");
    }
}
```

- **테스트 가능!**

```java
@Test
public void testIsValidItem() {
    assertFalse(MyFilter.isValidItem(null));
    assertFalse(MyFilter.isValidItem(" "));
    assertFalse(MyFilter.isValidItem("banana"));
    assertTrue(MyFilter.isValidItem("Apple"));
}
```
</br>


### 9.3.4 고차원 함수 테스팅

> ❓**고차원 함수** : 함수를 인수로 받거나 다른 함수를 반환하는 메서드
> 

메서드가 람다를 인수로 받는다면 다른 람다로 메서드의 동작을 테스트가 가능하다.

- 프레디케이트를 인수로 받는 `filter` 메서드(2장. 동작파라미터화) 테스트

```java
@Test
public void testFilter() throws Exception {
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    List<Integer> even = filter(numbers, i -> i % 2 == 0);
    List<Integer> smallerThanThree = filter(numbers, i -> i < 3);
    
    assertEquals(Arrays.asList(2, 4), even);
    assertEquals(Arrays.asList(1, 2), smallerThanThree);
}
```

</br>

## 9.4 디버깅

**디버깅의 기본 요소**

- **스택 트레이스(Stack Trace)**
    
    예외 발생 시 어디서 멈췄는지 호출된 메서드 목록과 위치(파일:라인)를 보여주는 정보
    
- **로깅(Logging)**
    
    프로그램 실행 중 주요 지점의 상태나 값을 기록하여 문제 발생 위치를 파악
    
</br>

### 9.4.1 스택 트레이스 확인

- 예외 발생 시 가장 먼저 확인해야 할 것 : **어디서 멈췄는가, 왜 멈췄는가**
- **스택 프레임(Stack Frame)**에는 다음과 같은 정보가 저장된다.
    - 메서드 호출 위치
    - 호출할 때의 인수값
    - 지역 변수 등
- 프로그램이 멈추면 스택 프레임을 얻어 문제가 발생한 원인은 추적할 수 있다.

**❗️ 람다 표현식과 스택 트레이스**

람다는 익명이기 때문에 스택 트레이스에 **의미 있는 메서드명**이 나타나지 않는다.

```java
import java.util.*;

public class Debugging {
    public static void main(String[] args) {
        List<Point> points = Arrays.asList(new Point(12, 2), null);
        points.stream().map(p -> p.getX()).forEach(System.out::println);
    }
}
```

```
Exception in thread "main" java.lang.NullPointerException
    at Debugging.lambda$main$0(Debugging.java:6)
    at Debugging$$Lambda$5/284720968.apply(Unknown Source)
    . . .
```

- lambda$main$0 : 람다 표현식이 이름이 없으므로 컴파일러가 람다를 참조하는 이름을 임의로 생성한 것!
- 디버깅이 어려움

메서드 참조로 바꿔도 마찬가지로 스택 트레이스에 이상한 정보가 출력된다.

```java
points.stream().map(Point::getX).forEach(System.out::println);
```

```
Exception in thread "main" java.lang.NullPointerException
		at Debugging$$Lambda$5/284720968. apply (Unknown Source)
```

**✅ 메서드 참조를 사용하는 클래스와 같은 곳에 선언되어 있는 메서드를 참조**

```java
public class Debugging {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3);
        numbers.stream()
               .map(Debugging::divideByZero)
               .forEach(System.out::println);
    }

    public static int divideByZero(int n) {
        return n / 0;
    }
}
```

```java
Exception in thread "main" java.lang.ArithmeticException: / by zero
    at Debugging.divideByZero(Debugging.java:10)  ← 메서드명 명확히 표시됨
    ...
```

결론 : 람다 표현식과 관련한 스택 트레이스는 이해하기 어려울 수 있다는 점을 염두에 두자.

</br>

### 9.4.2 정보 로깅

스트림 내부 파이프라인의 연산 결과를 로깅하거나 출력해서 디버깅할 수 있다.

```java
List<Integer> numbers = Arrays.asList(2, 3, 4, 5);

numbers.stream()
       .map(x -> x + 17)
       .filter(x -> x % 2 == 0)
       .limit(3)
       .forEach(System.out::println);
       
       
//출력 결과
20
22
```

하지만, 이 `forEach()`는 호출 순간 전체 스트림이 소비되기 때문에 파이프라인의 중간 연산(map, filter, limit)이 어떤 결과를 도출하는지 확인하기 어렵다.

**✅ peek을 이용한 중간값 추적**

`peek()`은 **스트림 요소를 소비하지 않으면서 중간 상태를 출력**하는 디버깅용 연산이다.

- 각 파이프라인 단계별로 요소의 상태를 확인하고 확인한 요소를 파이프라인의 다음 연산으로 그대로 전달한다.
    
    <img width="381" alt="스크린샷_2025-05-11_오후_6 20 16" src="https://github.com/user-attachments/assets/5ebf514c-574f-4d33-a3d9-b0947982e4aa" />


```java
List<Integer> result = numbers.stream()
    .peek(x -> System.out.println("from stream: " + x))
    .map(x -> x + 17)
    .peek(x -> System.out.println("after map: " + x))
    .filter(x -> x % 2 == 0)
    .peek(x -> System.out.println("after filter: " + x))
    .limit(3)
    .peek(x -> System.out.println("after limit: " + x))
    .collect(toList());
```

**출력 결과**

```
from stream: 2
after map: 19
from stream: 3
after map: 20
after filter: 20
after limit: 20
from stream: 4
after map: 21
from stream: 5
after map: 22
after filter: 22
after limit: 22
```
