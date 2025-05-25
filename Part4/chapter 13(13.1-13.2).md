# Chapter 13. 디폴트 메서드 (13.1 - 13.2)

## 13.0 개요

### 13.0.1 자바 8의 변화

- 자바 8 이전에는 인터페이스에 메서드를 추가하면 모든 구현 클래스가 그 메서드를 **반드시 구현**해야 했다.
- 인터페이스에 메서드를 추가하면 기존 구현 클래스를 모두 수정해야 했기 때문에 API 진화에 큰 제약이 생기는 문제가 발생한다.
    - ex) 자바 8 API에서 List 인터페이스에 `sort` 메서드가 추가되었는데 기존의 List 인터페이스를 구현해둔 Guava, Apache Common 같은 컬렉션 프레임워크가 모두 List 인터페이스를 상속한 모든 클래스를 수정해야 한다.
- 자바 8에서는 이러한 문제를 해결하기 위해 **기본 구현을 포함하는 인터페이스**를 정의하는 두 가지 방법을 제공한다.
    1. 인터페이스 내부에 **정적 메서드(static method)** 사용
    2. 인터페이스의 기본 구현을 제공할 수 있도록 **디폴트 메서드(default method)** 사용
- 결과적으로 기존 구현을 변경하지 않고도 새로운 기능을 인터페이스에 추가할 수 있게 되었다.

### 13.0.2 대표적인 사용 사례

**디폴트 메서드(default method)**

1. List 인터페이스의 sort

```java
default void sort(Comparator<? super E> c) {
    Collections.sort(this, c);
}
```

1. Collection 인터페이스의 stream 

```java
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
```

**정적 메서드(static method)**

- Comparator의 naturalOrder (자연순서 - 알파벳 순서로 요소를 정렬)

```java
List<Integer> numbers = Arrays.asList(3, 5, 1, 2, 6);
numbers.sort(**Comparator.naturalOrder()**);
```

### 13.0.3 디폴트 메서드의 구조적 장점

- **Ex) 인터페이스에 메서드를 추가하는 상황**

<img width="345" alt="스크린샷_2025-05-25_오후_4 32 34" src="https://github.com/user-attachments/assets/e3505580-c134-46d7-aa15-00e18904b6b0" />

1. 기존에 인터페이스를 구현한 클래스는 그대로 유지
2. 새로운 메서드는 디폴트 구현을 통해 자동 상속
3. 라이브러리 설계자 입장에서는 기존 API와의 **호환성 유지**와 **기능 확장**을 동시에 달성 가능

> [!TIP]
> **정적 메서드 vs 인터페이스**
> - **`Collections`가 `Collection` 객체를 활용하는 유틸리티 클래스**인 것처럼 자바에는 인터페이스의 인스턴스를 활용할 수 있는 다양한 정적 메서드를 정의하는 유틸리티 클래스를 활용한다.
> - 자바 8에서는 인터페이스에 직접 정적 메서드를 선언할 수 있으므로 유틸리티 클래스를 없애고 직접 인터페이스 내부에 정적 메서드를 구현할 수 있게 되었다.
> - 그럼에도 불구하고 과거 버전과의 호환성을 위해 자바 API에는 여전히 유틸리티 클래스를 남겨두었다. (즉, 사용자가 원하면 커스텀하여 선언하고 아니면 기존 유틸리티 클래스 사용 가능)

</br>

## 13.1 변화하는 API

### 13.1.1 API 버전 1

- Resizable 인터페이스 초기 버전

```java
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
}
```

- 사용자 구현 (Ellipse 클래스)

```java
public class Ellipse implements Resiable {
		...
}
```

- (자신이 만든 Ellipse를 포함한) 다양한 Resiable 모양을 처리하는 게임

```java
public class Game {
  public static void main(String... args) {
      List<Resiable> resizableShapes = 
          Arrays.asList(new Square(), new Rectangle(), new Ellipse());
      Utils.paint(resiableShapes);
  }  
}

public class Utils {
  public static void paint(List<Resizable> l) {
      l.forEach(r -> {
          r.setAbsoluteSize(42, 42);
          r.draw();
      });
  }
}
```

### 13.1.2 API 버전 2

- **추가 요구사항** : Resiable을 구현하는 Square와 Rectangle 구현을 개선 요청
    
    <img width="310" alt="스크린샷_2025-05-25_오후_5 26 54" src="https://github.com/user-attachments/assets/35a1d32a-a99d-4c86-91df-fe4a0ffcfb79" />

    - Resizable에 메서드를 추가하며 API가 변경됨 → 인터페이스 수정 후 애플리케이션을 재컴파일하면 에러 발생

```java
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    void setRelativeSize(int wFactor, int hFactor); // API 버전 2에 추가된 새로운 메서드
}
```

- **사용자가 겪는 문제**
    - Resizable 인터페이스를 구현하는 모든 클래스는 `setRelativeSize()`를 구현해야 하지만 라이브러리 사용자가 직접 구현한 Ellipse는 `setRelatvieSize()`를 구현하지 않고 있다.
    - 이 상황에서는 **바이너리 호환성**은 유지된다.

> [!TIP]
> **바이너리 호환성**이란?
> → 새로 추가된 메서드를 호출하지만 않으면 새로운 메서드 구현이 없이도 기존 클래스 파일 구현이 잘 동작하도록 하는 것


**☝🏻 그렇다면 문제는?**

1. **런타임 에러(AbstractMethodError)**
    - 누군가가 Resizable을 인수로 받는 Utils.paint에서 setRelativeSize를 사용하도록 코드를 수정하고 Ellipse 객체를 인수로 전달한다면,
    - 메서드를 정의하지 않은 클래스(Ellipse)에 대해 **런타임 에러 (AbstractMethodError)** 발생
    
    ```java
    Exception in thread "main" java.lang.AbstractMethodError:
    Ellipse.setRelativeSize(II)V
    ```
    
2. **컴파일 에러**
    - setRelatvieSize()가 포함된 API를 기반으로 전체 애플리케이션을 재빌드하면,
    - Ellipse 클래스가 해당 메서드를 구현하지 않았기 때문에 **컴파일 에러 발생**
    
    ```java
    error: Ellipse is not abstract and does not override abstract method
    ```
    

**☝🏻 공개 API 버전 관리의 어려움**

- **문제점 요약**
    - API를 바꾸면 기존 버전과의 호환성 유지가 어렵다.
    - Ex) 공식 자바 컬렉션 API 같은 기존 API 수정이 어렵다.
- **기존 대안(불편함 존재)**
    - 자신의 API를 버전별로 나누어 관리 (예: v1, v2)
    - 위 방법은 다음과 같은 어려움이 존재한다.
        1. 라이브러리 관리 복잡도가 증가한다.
        2. 사용자 코드가 구버전과 신버전을 동시에 사용하게 된다. → 클래스 파일이 증가해 메모리, 로딩 시간이 증가

**✅ 디폴트 메서드로 해결**

- default 키워드를 사용하면, **인터페이스에 기본 구현 추가가 가능**하다.
- **기존 구현 클래스는 그대로 두고,** 새로운 기능은 자동으로 포함시킬 수 있다.

> [!TIP]
> **바이너리 호환성 vs 소스 호환성 vs 동작 호환성**
> 1. **바이너리 호환성**
>    - 기존 바이너리 파일(class 파일)을 다시 컴파일하지 않고도 실행 가능한 경우
>    - 즉, 예전 컴파일된 클래스 파일이 변경된 코드와 함께 문제 없이 동작할 수 있으면 바이너리 호환성이 유지된 것
>    - Ex) 인터페이스에 메서드를 추가했을 때, **추가된 메서드가 호출되지 않으면** 문제 X    
>    👉 핵심 조건: **새 메서드를 호출하지 않음**
>    
> 2. **소스 호환성** 
>    - 기존 소스 코드를 변경된 자바 API에 맞춰 다시 컴파일했을 때 문제 없이 컴파일되는 경우
>    - 인터페이스에 메서드를 추가하면 일반적으로 모든 구현 클래스가 새 메서드를 구현해야 하므로 소스 호환성은 깨짐
>    
>    👉 핵심 조건: **새로운 인터페이스 메서드를 구현하지 않아도 컴파일이 되어야 함**
>    
> 3. **동작 호환성**
>    - **프로그램의 실행 결과가 코드 변경 전후에 동일한 경우**
>    - 동일한 입력을 줬을 때 프로그램이 **같은 동작을 수행하고 같은 출력을 내야** 동작 호환성이 유지된 것
>    - Ex) 인터페이스에 메서드를 추가했더라도, 해당 메서드를 호출하지 않으면 기존 동작에는 영향이 없음
>    
>    👉 핵심 조건: **실행 결과가 바뀌지 않음**
   

</br>

## 13.2 디폴트 메서드란 무엇인가?

- 공개된 API에 새로운 메서드를 추가하면 기존 클래스가 이 메서드를 구현하지 않아 **호환성 문제가 발생**
- 자바 8에서는 이러한 문제를 해결하고 API 진화를 가능하게 하기 위해 **디폴트 메서드(default method)** 개념을 도입
- **인터페이스에 기본 구현을 포함시킬 수 있게 함**

**☝🏻 디폴트 메서드의 구조**

- default 키워드를 사용해 **인터페이스 내에 메서드의 기본 구현을 제공한다.**
- 기존 구현 클래스는 해당 메서드를 구현하지 않아도 된다.

예제

```java
public interface Sized {
    int size();

    default boolean isEmpty() {
        return size() == 0;
    }
}
```

- Sized 인터페이스를 구현하는 모든 클래스는 isEmpty의 구현도 상속받는다.
    
    → **소스 호환성 & 바이너리 호환성** 유지
    

```java
default void setRelativeSize(int wFactor, int hFactor) {
    setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);
}
```

- Resizable 인터페이스에 다음과 같은 디폴트 메서드를 제공하면 기존 클래스가 깨지는 것을 막을 수 있다.

**☝🏻 자바 8 이후의 활용 예시**

- List.sort(), Collection.stream() 등 자바 8 표준 API에 포함된 많은 메서드가 디폴트 메서드로 구현되었다.
- Predicate, Function, Comparator, UnaryOperator 같은 함수형 인터페이스에도 디폴트 메서드 포함한다.


> [!IMPORTANT]
> **추상클래스 vs 자바 8 인터페이스**
> | **항목** | **추상 클래스** | **자바 8 인터페이스** |
> | --- | --- | --- |
> | 상속 제한 | 하나만 가능 | 여러 개 구현 가능 |
> | 상태 보존 | 인스턴스 변수 가질 수 있음 | 가질 수 없음 |
> | 메서드 구현 | 바디 포함 가능 | default와 static 키워드로만 가능 |
