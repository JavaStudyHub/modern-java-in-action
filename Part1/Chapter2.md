# Chapter 2. 동작 파라미터화 코드 전달하기

> [!IMPORTANT]
> **동작 파라미터화** = 아직은 어떻게 실행할 것인지 결정하지 않은 코드 블록


**요약**

동작 파라미터를 사용하는 이유?

- 변화하는 요구사항에 유연하게 대응

람다 표현식을 사용하는 이유?

- 동작 파라미터를 추가하는 과정에서 쓸데없는 코드를 간결하게 만들어줌

## 2.1 변화하는 요구사항에 대응하기

책에서는 동작 파라미터를 사용하는 이유를 효과적으로 이해시키기 위해 하나의 예제를 기준으로 코드를 점차 개선하면서 유연한 코드로 만드는 것을 보여준다.

첫번째 요구사항부터 세번째 요구사항까지 저자가 어떤 식으로 리팩토링을 시도하는지 살펴보자.

---

### ***요구사항 1. 농장 재고목록 애플리케이션에 리스트에서 녹색 사과만 필터링하는 기능을 추가하자***

```java
enum Color {RED, GREEN}

public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (**GREEN.equals(apple.getColor())) {  // 녹색 사과만 필터링**
            result.add(apple);
        }
    }
    return result;
}
```

---

### ***요구사항 2. 녹색 사과 말고 빨간 사과도 필터링해야 한다.***

코드를 어떻게 고쳐야 할까?

크게 고민하지 않은 사람이라면 메서드를 그대로 복사해서 `filterRedApples` 라는 새로운 메서드를 만들고, if문의 조건을 빨간 사과로 바꾸는 방법을 선택할 것이다.

하지만, 이런 방법은 전혀 객체 지향적 이지 않고 후에 요구사항이 더욱 다양하게 변경될 때마다 개발 비용이 기하급수적으로 증가하게 된다.

> [!TIP]
>  **거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.**


`filterRedApples` 에서는 GREEN이라는 상수가 RED로 바뀌고 나머지 코드는 모두 중복될 것이니 이 부분을 파라미터로 추상화시키면 된다.

**색을 파라미터화**한 코드

```java
public static List<Apple> filterAppelsByColor(List<Apple> inventory, Color color){
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (color.getColor().equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```

해당 메서드를 호출하는 코드

```java
List<Apple> greenApples = filterApplesByColor(inventory, GREEN);
List<Apple> redApples = filterApplesByColor(inventory, RED);
```

---

### ***요구사항 3. 색 이외에도 가벼운 사과와 무거운 사과를 구분해야 한다. 150그램 이상인 사과를 무거운 사과라고 한다.***

위의 색과 마찬가지로 무게의 기준도 얼마든지 바뀔 수 있기 때문에 다양한 무게에 대응할 수 있도록 무게 정보 파라미터도 추가해주면 된다.

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight){
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getWeight() > weight) {
            result.add(apple);
        }
    }
    return result;
}
```

**동작 파라미터**의 개념을 모른다면, 위와 같이 다른 메서드를 선언하는 수 밖에 없을 것이다.

하지만 위 구현 코드 또한 전체 사과 목록을 조회하며, 특정 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 **중복**된다.

이는 소프트웨어 공학의 **DRY(Don’t Repeat Yourself = 같은 것을 반복하지 말 것)** 원칙을 어기는 것이다.

<details><summary>참고) 소프트웨어 공학의 3대 원칙</summary>
  
  1. **KISS - Keep It Simple, Stupid!**
      > **단순함을 유지하라**
        - 설계를 최대한 단순하게 하여 이해하기 쉽고 유지보수하기 쉽게 만든다
        - 복잡한 구조보다는 직관적이고 명확한 구조가 더 바람직하다
        - 지나치게 복잡한 설계는 오히려 오류를 유발하고 유지보수를 어렵게 만든다
          
  2. **YAGNI (You Aren’t Gonna Need It)**

       > **필요하지 않은 기능은 구현하지 말라**
       >
        - 미래에 필요할지도 모른다는 이유로 미리 기능을 구현하지 않는다
        - 실제로 필요해졌을 때 구현하는 것이 효율적이며, 불필요한 복잡성을 줄일 수 있다
          
  3. **DRY (Don’t Repeat Yourself)**
     
       > **중복을 피하라**
       >
        - 동일한 코드를 여러 곳에 반복해서 작성하지 않고, 하나의 정의로 관리한다
        - 중복이 많아질수록 유지보수가 어려워지고, 오류 발생 시 일관성 있게 수정하기 어렵다
        - 공통된 로직은 함수, 모듈, 클래스 등으로 분리하여 최대한 재사용 가능하게 만든다
</p>
        </details>

</br>
여기서, ***‘`filter` 라는 메서드를 만들고 색과 무게 중 어떤 기준으로 사과를 필터링할지 구분하는 플래그를 추가해주면 되지 않나?’***

라는 의문이 생긴다.

하지만 다음 코드를 보면 이 방법을 절대 사용하지 말아야 할 이유를 알 수 있다.

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color, 
                                       int weight, boolean flag){
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ((flag && apple.getColor().equals(color)) || 
                (!flag && apple.getWeight() > weight)) {
            result.add(apple);
        }
    }
    return result;
}
```

위 코드를 사용하려면 다음과 같이 사용해야 한다.

```java
List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> redApples = filterApples(inventory, null, 150, false);
```

이 코드의 문제점은 다음과 같다.

- 인자가 어떤 의미를 가지는지 직관적이지 않다.
    - (이 코드만 보고 true와 false가 무엇을 의미하는지 알 수 없음)
- 요구사항 변경에 유연하게 대응이 불가능하다.
    - 사과의 크기, 모양, 원산지 등으로 사과를 필터링하고 싶다면? ⇒ X
    - 녹색 사과 중 무거운 사과를 필터링하고 싶다면? ⇒ X

이렇게 지금까지는 문자열, 정수, 불리언 등의 값으로 `filterApples` 메서드를 파라미터화 했다.

문제가 바뀌지 않고 잘 정의되어 있는 상황에서는 잘 동작하겠지만, 어떤 기준으로 사과를 필터링할 것인지 효과적으로 전달할 수 있다면 더 좋을 것이다.

이를 위해 나온 것이 바로 **동작 파라미터화**이다.

---

## 2.2 동작 파라미터화

동작 파라미터는 어떤 동작, 즉 코드 블록을 한번더 추상화 시키는 개념이라고 보면 된다.

앞에서 본 사과 필터링 예제를 조금더 추상화 해보자.

공통적으로 요구사항 모두 사과의 어떤 속성(무게, 색 등등)에 기초해서 boolean값을 반환(사과가 150그램 이상이 맞는지? 녹색이 맞는지?)하도록 구현되었다.

이때, 참 또는 거짓을 반환하는 함수를 **프레디케이트**라고 한다.

**선택 조건을 결정하는 인터페이스**

```java
public interface ApplePredicate {
    boolean test(Apple apple);
}
```

다음 예제처럼 다양한 선택 조건을 대표하는 ApplePredicate가 나올 수 있다.

```java
// 무거운 사과만 필터링
public class AppleHeavyWeightPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

// 녹색 사과만 필터링
public class AppleGreenColorPredicate implements ApplePredicate {
    public boolean test(Apple apple) {
        return GREEN.equals(apple.getColor());
    }
}
```

### _**전략 디자인 패턴(strategy design pattern)**_

> 각 알고리즘(전략)을 캡슐화하는 알고리즘 그룹을 정의해둔 다음 런타임에 알고리즘을 선택하는 기법
>

<img width="454" alt="스크린샷_2025-03-29_오전_1 11 13" src="https://github.com/user-attachments/assets/e4683ba4-eaac-4d07-b7a6-51c03cc50002" />

ApplePredicate ⇒ 알고리즘 그룹

AppleGreenColorPredicate & AppleGreenColorPredicate ⇒ 알고리즘(전략)

```java
public static List<Apple> filterApples(List<Apple> inventory, 
																	**ApplePredicate p**) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

ApplePredicate 객체를 인자로 받아 그 안에 구현되어 있는 `test` 메서드를 통해 Apple의 조건을 검사하도록 하는 코드이다.

즉, filterApples 메서드가 ApplePredicate이라는 다양한 동작으로 구현 가능한 알고리즘 그룹을 파라미터로 받아서 내부적으로 알고리즘(전략)의 다양한 동작을 수행할 수 있는 것이다.

이를 ***동작 파라미터화*** 라고 한다.

이렇게, 다양한 동작을 수행 가능한 동작 자체를 파라미터로 받기 때문에 요구사항 변경에도 유연하다. 요구사항이 변경된다면, 우리는 다시 ApplePredicate를 적절하게 구현하여 파라미터에 넣어주기만 하면 되는거다.

만약, 요구사항이 무거운 빨간 사과를 필터링해야 한다고 가정해보자.

<img width="423" alt="스크린샷_2025-03-29_오전_1 24 47" src="https://github.com/user-attachments/assets/63611832-6b89-4fbe-afb4-2698ba1c6f8d" />

ApplePredicate를 구현한 AppleRedAndHeavyPredicate 이라는 전략(동작)을 filterApples에 파라미터로 넣어주면 끝이다.

이렇게 매우 간편하지만, 현재 filter 메서드는 객체만 인수로 받기 때문에 항상 test 메서드를 ApplePredicate 객체로 감싸서 전달해야 한다. 위의 사진에서 굵게 표시된 코드처럼 필터링 로직에 관계없는 코드까지도 작성해줘야 하는 번거로움이 있다.

람다식을 활용하면, 이렇게 ApplePredicate 클래스를 정의하지 않고도 다양한 표현식을 메서드에 전달할 수 있다! 이는 3장에서 배워보자.

정리해보자면, 동작 파라미터화 라는 것은 동작 자체를 파라미터화 시킨 것으로, 한 메서드가 다른 동작을 수행하도록 재활용할 수 있기 때문에, 보다 유연한 API를 만들 수 있는 강점이 있다.

<img width="497" alt="스크린샷_2025-03-29_오전_2 14 54" src="https://github.com/user-attachments/assets/462b0c4e-e5f4-4907-a3f9-c06a59bebf20" />


## 2.3 복잡한 과정 간소화

`filterApples` 메서드로 새로운 동작을 전달하려면

1. ApplePredicate 인터페이스를 구현하는 여러 클래스를 정의한 다음
2. 이를 인스턴스화 시키고
3. 메서드에 파라미터로 넣어서 호출해야한다.

이렇게 앞에서 보았던 동작 파라미터 방법은 요구사항에 유연하다는 장점이 있기는 하지만, 다소 번거로운 작업이며, 비효율적이다.

자바는 클래스의 선언과 인스턴스화를 동시에 수행하여 이런 비효율성을 개선하기 시켰다. 이 과정에서 도입된 것이 **익명 클래스**이다.

---

### _**익명 클래스**_

- 말 그대로 이름이 없는 클래스
- 자바의 local class(블록 내부에 선언된 클래스)와 비슷한 개념
- 클래스 선언과 인스턴스화를 동시에 할 수 있음

⇒ 위에서 일일이 클래스들을 선언할 필요 없이 즉석에서 필요한 구현을 만들어서 사용이 가능하다!

```java
// 익명 클래스(Anonymous Class)를 사용한 ApplePredicate 구현
**List<Apple> redApples = filterApples(inventory, new ApplePredicate() {**
    public boolean test(Apple apple) {
        return "RED".equals(apple.getColor());
    }
});
```

```java
// 자바FX에서 이벤트 핸들러를 익명 클래스로 구현한 예
**button.setOnAction(new EventHandler<ActionEvent>() {**
    public void handle(ActionEvent event) {
        System.out.println("Whoooo a click!!");
    }
});
```

하지만 위의 예제들을 보면 여전히 굵은 코드처럼 여전히 반복되고 장황한 코드가 보여진다.

익명 클래스를 사용함으로써 여러 클래스를 선언하는 과정을 조금 줄일 수 있지만 여전히 객체를 만들고 명시적으로 새로운 동작을 정의하는 메서드를 구현해야 한다.

이런 클래스 선언과 메서드를 명시하는 동작을 최대한 최적화시키기 위해 자바 8부터 도입된 것이 바로 **람다 표현식**이다.

### _**람다 표현식**_

```java
List<Apple> result = filterApples(inventory, (Apple) apple) -> Red.equals(apple.getColor()));
```

- ch.3에서 자세하게 다룰 예정
<details><summary>익명 클래스 문제 예제</summary>
  > 생각보다 어려워서 가져와봤습니다..
  

    ```java
    public class MeaningOfThis {
        public final int value = 4;
    
        public void doIt() {
            int value = 6;
    
            Runnable r = new Runnable() {
                public final int value = 5;
    
                public void run() {
                    int value = 10;
                    System.out.println(this.value);
                }
            };
    
            r.run();
        }
    
        public static void main(String... args) {
            MeaningOfThis m = new MeaningOfThis();
            m.doIt();  // 이 행의 출력 결과는?
        }
    }
    ```
<details><summary>정답</summary>
<p>`this.value`는 Runnable 클래스의 로컬 변수를 의미하므로 정답은 5입니다.</p>
</details>
</details>


### _**동작 파라미터화 정리**_

<img width="483" alt="스크린샷_2025-03-29_오전_2 45 20" src="https://github.com/user-attachments/assets/59f1186e-167f-444c-b1df-39c6cf5bafde" />

지금까지의 내용을 한눈에 보여주는 그래프이다.

- int, boolean, String과 같은 값을 파라미터로 넘기는 **값 파라미터화**는 요구사항의 변화에 대한 대응이 뻣뻣하고 코드가 점점 장황해진다.
- 인터페이스를 만들어 이를 구현한 여러 클래스를 선언하여 클래스 자체를 파라미터로 넘기는 **동작 파라미터화 방식**은 요구사항에는 유연한 반면, 클래스 선언과 클래스 인스턴스화를 매번 해줘야하는 번거로움이 존재한다.
- 클래스 선언과 인스턴스화를 동시에 수행하게 해주는 익명 클래스를 활용한 **동작 파라미터화 방식**은 다양한 클래스를 매번 선언하는 방식보다는 코드가 간결해졌지만, 여전히 객체를 만들고 메서드를 명시하는 작업은 동반된다.
- 람다 표현식은 객체를 생성하지 않고 메서드 명도 명시해줄 필요가 없다는 점에서 다른 2가지 **동작 파라미터화** 방식보다 더 간결하다고 볼 수 있다.

### _**리스트 형식으로 추상화**_

요구사항에 조금더 유연하게 대처하기 위한 방법으로 형식 파라미터 T를 사용할 수 있다.

```java
public interface Predicate<T> {
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for (T e : list) {
        if (p.test(e)) {
            result.add(e);
        }
    }
    return result;
}
```

위의 코드처럼 작성하면 바나나, 오렌지, 정수, 문자열 등의 **여러 타입에 대한 리스트**에 필터 메서드를 사용할 수 있게 된다.

```java
List<Apple> redApples =
        filter(inventory, (Apple apple) -> "RED".equals(apple.getColor()));

List<Integer> evenNumbers =
        filter(numbers, (Integer i) -> i % 2 == 0);
```

가장 처음 나왔던 코드에 비해 유연성과 간결함이 훨씬 향상되었다.
