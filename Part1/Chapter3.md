# 3장 람다표현식

## 3.1 람다란 무엇인가?

**람다표현식**  

- 메소드로 전달할 수 있는 익명함수를 단순화한 것
- 특징
    - 익명 : 메서드와 달리 이름을 가지지 않는다
    - 함수 : 특정 클래스에 종속되지 않아서 함수라고 부른다. 파라미터 리스트, 바디, 반환타입, 가능한 예외 리스트를 가진다.
    - 전달 : 람다를 메소드의 인자로 전달하거나 변수에 할당할 수 있다.
    - 간결성 : 익명클래스처럼 자질구레한 코드를 구현할 필요가 없다.

- 람다 표현식의 구조
    - 파라미터 리스트, 화살표, 바디로 이루어져 있다.
    
    ```java
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())；
    ```
    
- 표현식 스타일
    
    ```java
    (parameters) -> expression
    ```
    
- 블록 스타일
    
    ```java
    (parameters) -> { statements; }
    ```
    

## 3.2 어디에, 어떻게 람다를 사용할까?

람다는 함수형 인터페이스라는 맥락에서 사용할 수 있다.

**함수형 인터페이스**

- 하나의 추상 메소드를 지정하는 인터페이스
    - 참고 : 디폴트 메소드를 포함하고 있더라도 추상메소드가 하나이기만 하면 함수형 인터페이스이다.
- 대표적인 함수형 인터페이스로 Predicate, Comparator 등이 있다.
    
    ```java
    public interface Predicate<T> {
    	boolean test (T t);
    }
    ```
    
- 람다표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.
    
    ```java
    Runnable r1 = () -> System.out.printin("Hello World 1")；
    
    Runnable r2 = new RunnableO { 
    	public void run() {
    		System.out.println("Hello World 2")；
    	}
    }；
    
    process(rl)；
    ```
    

**함수 디스크립터**

- 람다 표현식의 시그니처를 서술하는 메소드를 함수 디스크립터라고 한다. 함수형 인터페이스에서 추상메소드를 가리킨다고 보면 된다.
- Runnable 인터페이스의 run 메소드가 이에 해당한다.

**@FuntionalInterface**

- 함수형 인터페이스라는 것을 알리는 어노테이션이다. 이것이 사용된 인터페이스가 함수형 인터페이스가 아니면 컴파일 에러가 발생한다.

## 3.3 람다 활용 : 실행 어라운드 패턴

람다와 동작파라미터화를 이용해서 간결하고 유연한 코드를 만들 수 있다. 실행 어라운드 패턴 예제에서 알아보자.

**실행 어라운드 패턴**

- 자원을 사용하는 경우 자원을 열고 자원을 사용하고 자원을 반납하는 과정이 필요하다. 자원을 사용하는 부분이 자원을 설정하고 정리하는 과정으로 둘러싸는 형태를 실행 어라운드 패턴이라고 한다.

**예제에 람다 적용하기**

- 예시는 BufferedReader를 이용하여서 필요한 작업을 수행하는 processFile 메소드이다. 여기서는 한줄을 읽고 리턴하는데, 만약 상황에 따라서 두 줄을 읽고 리턴해야한다면 람다와 동작 파라미터화를 적용해볼 수 있다.

```java
public String processFile() throws lOException {
	try (BufferedReader br =
		new BufferedReader(new FileReader("data.txt"))) { // 자원을 설정하고 정리한다
			return br.readLine(); // 자원을 사용한다
	}
}
```

1. 동작파라미터화
    - 설정과 정리는 중복되니 processFile에서 자원을 활용하는 부분에 동작파라미터화를 적용한다.
    - 두 줄을 읽고 리턴하려면 BufferedReader를 입력 받아서 String을 리턴해야한다.
    
    ```java
    String result = procesFile((BufferedReader br) -> br.readLine() + br.readLine());
    ```
    
2. 함수형 인터페이스 정의
    - 람다의 시그니처와 가능한 예외에 맞게 함수형 인터페이스를 작성하고 processFile 메소드에 함수형 인터페이스 타입의 인자를 추가한다.
    
    ```java
    // 새로 작성
    @FunctionalInterface
    public interface BufferedReaderProcessor {
    	String process(BufferedReader b) throws lOException;
    }
    
    public String processFile(BufferedReaderProcessor p) throws lOException { // 인자 추가
    ```
    
3. 동작 실행
    - 람다표현식은 함수형 인터페이스의 추상메소드를 구현한다. 함수형 인터페이스의 추상메소드를 호출하면 람다로 전달된 코드를 실행할 수 있다.
    
    ```java
    public String processFile(BufferedReaderProcessor p) throws
    lOException {
    	try (BufferedReader br =
    		new BufferedReader(new FileReader("data.txt"))) {
    			return p.process(br)；// 이 부분
    	}
    }
    ```
    
4. 람다 작성
    - 람다를 이용해서 processFile이 다양한 동작을 하게 만들 수 있다.
    
    ```java
    String oneLine = processFile((BufferedReader br) -> br.readLine());
    String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
    ```
    

## 3.4 함수형 인터페이스 사용

자바8에서는 java.util.function 패키지에서 다양한 함수형 인터페이스를 제공한다.

**Predicate**

- 이 인터페이스는 T 타입을 인자로 받아서 불리언 타입을 반환하는 test 메소드를 가지고 있다.
    
    ```java
    @FunctionalInterface
    public interface Predicate<T> {
    	boolean test(T t);
    }
    ```
    

**Comsumer**

- 이 인터페이스는 T 타입을 인자로 받아서 void를 반환하는 accept 메소드를 가지고 있다. T타입 객체를 가지고 어떤 작업을 수행할 때 사용할 수 있다.
    
    ```java
    @FunctionalInterface
    public interface Consumer<T> {
    	void accept(T t);
    }
    ```
    

**Function**

- 이 인터페이스는 T타입의 인자를 받아서 R타입의 리턴값을 반환하는 apply 메소드를 가지고 있다.
    
    ```java
    @FunctionalInterface
    public interface Function<T, R> {
    	R apply(T t);
    }
    ```
    

**기본형에 특화된 함수형 인터페이스**

- 제네릭 타입은 참조형만 사용할 수 있다. 참조형은 박싱과 언박싱을 자동으로 수행하는 오토박싱을 제공한다. 그런데 오토박싱은 힙 공간에 박싱한 값을 저장하고 기본값으로 가져오는 과정에서 비용이 발생한다.
- 이런 오토박싱을 피하기 위해서 자바8은 기본형에 특화된 함수형 인터페이스를 제공한다.
- 예를 들어서 IntPredicate는 정수값을 Integer로 박싱하지 않는다.
    
    ```java
    public interface IntPredicate {
    	boolean test(int t);
    }
    ```
    

- 특정 형식을 입력받는 함수형 인터페이스는 이름 앞에 DoublePredicate, IntConsumer, LongBinaryOperator, IntFunction처럼 형식명이 붙는다. Function 인터페이스는 ToIntFunction〈T〉. IntToDoubleFunction 등의 다양한 출력 형식 파라미터를 제공한다
.
    
    
    | Predicate<T> | IntPredicate, LongPredicate, DoublePredicate |
    | --- | --- |
    | Consumer<T> | IntConsumer, LongConsumer, DoubleConsumer |
    | Function<T, R> | IntFunction〈R〉,
    | | IntToDoubleFunction,
    | | IntToLongFunction,
    | | LongFunction<R>
    | | LongToDoubleFunction,
    | | LongToIntFunction,
    | | DoubleFunction<R>
    | | DoubleToIntFunction,
    | | DoubleToLongFunction,
    | | ToIntFunction<T>
    | | ToDoubleFunction<T>,
    | | ToLongFunction<T> |

**예외, 람다, 함수형 인터페이스의 관계**

- 예외를 던지는 람다식을 만들어 사용하려면 확인된 예외를 선언하는 함수형 인터페이스를 만들거나 람다내에서 `try-catch`로 감싸야한다.
    
    ```java
    @FunctionalInterface
    public interface BufferedReaderProcessor {
    	String process(BufferedReader b) throws lOException;
    }
    
    BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
    ```
    
    ```java
    Function<BufferedReader, String> f = (BufferedReader b) -> {
    	try {
    		return b.readLine();
    	}
    	catch(IOException e) {
    		throw new RuntimeException(e);
    	}
    }
    ```
    

## 3.5 형식 검사,형식 추론,제약

형식검사

- 할당문, 함수호출, 형변환 등 람다가 사용되는 컨텍스트를 통해서 람다의 형식을 추론할 수 있다. 컨텍스트에서 예상할 수 있는 람다의 형식을 `대상형식`이라고 한다.
- 다음 예시 코드의 형식 검사 과정을 살펴보자
    
    ```java
    filter(inventory, (Apple apple) -> apple.getWeight() > 150)
    ```
    
    1. 우선 콘텍스트를 알아내기 위해서 filter의 선언을 본다. filter는 두번째 인자로 Predicate<Apple>을 대상형식으로 기대한다.
        
        ```java
        filter(List〈Apple〉inventory, Predicate<Apple> p);
        ```
        
    2. Predicate<Apple>은 test라는 메소드를 가지고 있는 함수형 인터페이스이다. test는 Apple을 인자로 받아서 boolean을 반환하는 함수디스크립터이다.
        
        ```java
        @FunctionalInterface
        public interface Predicate<Apple> {
        	boolean test(Apple t);
        }
        ```
        
    3. 주어진 람다가 함수디스크립터에 부합하는지 본다. 주어진 람다는 올바른 코드임을 알 수 있다.

- 같은 람다 표현식이더라도 호환되는 추상메소드를 갖는 함수형 인터페이스로 사용될 수 있다.
    
    ```java
    @FunctionalInterface
    public interface PrivilegedAction<T> {
    	T perform();
    }
    
    // 같은 람다를 다른 함수형 인터페이스에 할당했다.
    Callable<Integer> c = () -> 42;
    PrivilegedAction<Integer> p = () -> 42;
    ```
    

형식추론

- 자바 컴파일러는 람다가 사용된 컨텍스트를 통해서 람다와 관련된 함수형 인터페이스를 추론할 수 있고, 람다의 시그니처도 추론할 수 있다.
- 람다의 파라미터에 접근할 수 있으므로 람다의 파라미터에서 형식을 생략해도 된다.
    
    ```java
    filter(inventory, apple -> GREEN.equals(apple.getColor())); // apple의 형식이 없다
    ```
    

지연변수 사용

- 람다 내부에서 외부에서 정의된 변수인 `자유변수` 를 사용할 수 있다. 또한 이를 `람다 캡처링`이라고 한다.
- 자유변수에는 제약이 있다. 인스턴스 변수와 정적변수를 캡처할 수 있지만, 지역 변수는 final이거나 사실상 final이어야 한다.
    
    ```java
    int portNumber = 1337;
    Runnable r = () -> System.out.println(portNumber);
    // 아래 코드를 추가하면 portNumber는 람다에서 참조할 수 없다.
    // portNumber = 1444 
    ```
    
- 지역변수에 제약을 거는 이유는 지역변수가 스택에 생성되기 때문이다. 만약 지역변수를 할당한 스레드가 사라진 상태에서 람다를 사용하는 스레드가 지역변수에 접근하려하면 자바는 자유변수의 복사본을 제공한다. 복사본이 유지되기 위해서는 지역변수는 변하면 안된다. 이것이 제약을 거는 이유이다.

## 3.6 메소드 참조

**메소드 참조**

- 메소드 참조는 하나의 메소드를 사용하는 람다의 축약된 형태라고 보면 된다.
- 구분자(::) 뒤에 메소드명을 작성하고, 구분자 앞에는 클래스이름이나 인스턴스 이름을 작성한다. 메소드 호출은 아니므로 괄호를 넣지 않는다.
    
    ```java
    // 아래 람다표현식을 메소드 참조로 표현
    Apple::getWeight
    
    // 람다 표현식
    (Apple apple) -> apple.getWeight()
    ```
    

메소드 참조를 만드는 방법

- 정적 메소드 참조
    
    ```java
    Integer::parselnt
    ```
    
- 인자의 인스턴스 메소드 참조
    
    ```java
    (String s) -> s.toUpperCase() // 인자로 들어온 String 객체 s의 인스턴스 메소드 호출
    
    String::toUpperCase
    ```
    
- 외부 객체의 인스턴스 메소드 참조
    
    ```java
    () -> expensiveTransaction.getValue() // 외부 객체 expensiveTransaction의 인스턴스 메소드 호출
    
    expensiveTransaction::getValue
    ```
    
    - private 헬퍼 메소드를 정의한 상황에서 유용하게 사용할 수 있다. 다음과 같이 private으로 선언된 isValidName 메소드를 filter 메소드에 인자로 넘기고 싶은 경우가 있다.
        
        ```java
        private boolean isValidName(String string) {
        	return Character.isUpperCase(string.charAt(0));
        }
        
        public List<String> filter(List<String> list, Predicate<String> p){
        	// 생략
        }
        ```
        
    - 이때 다음과 같이 메소드 참조를 이용해서 filter를 호출할 수 있다.
        
        ```java
        filter(words, this::isValidName);
        ```
        

![image.png](3%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%85%E1%85%A1%E1%86%B7%E1%84%83%E1%85%A1%E1%84%91%E1%85%AD%E1%84%92%E1%85%A7%E1%86%AB%E1%84%89%E1%85%B5%E1%86%A8%201c22dcb5a74a80118c28ce7d0b5d44d8/image.png)

**생성자 참조**

- ClassName::new를 이용하여서 생성자를 참조할 수 있다. 정적메소드의 메소드 참조와 유사한 방식이다.
- 예시
    - 매개변수가 없는 생성자
        - 매개변수가 없는 ()→T 시그니처를 가지며 이는 Supplier<T>의 시그니처와 같다.
            
            ```java
            Supplier<Apple> d = () -> new Apple(); 
            Apple a1 = d.get();
            
            // 위 코드를 아래와 같이 바꿀 수 있다.
            Supplier<Apple> d = Apple::new;
            Apple a1 = d.get();
            ```
            
    - 매개변수를 하나 갖는 생성자
        - 매개변수를 하나 가지는 생성자는 U→T 시그니처를 갖는다. 이는 Function<U, T>의 시그니처와 같다.
            
            ```java
            Function<Integer, Apple> c2 = (weight) -> new Apple(weight); 
            Apple a2 = c2.apply(110);
            
            // 위 코드를 아래와 같이 바꿀 수 있다.
            Function<Integer, Apple> c2 = Apple::new;
            Apple a2 = c2.apply(110);
            ```
            
    - 매개변수를 두 개 갖는 생성자
        - 매개변수를 두 개 가지는 생성자는 (U, V) → T 시그니처를 가지므로 BiFunction<U, V, T>의 시그니처와 같다.
            
            ```java
            BiFuction<String, Integer, Apple> c3 = (color, weight) -> new Apple(color, weight);
            Apple a3 = c3.apply(GREEN, 110);
            
            // 위 코드를 아래와 같이 변경할 수 있다.
            	BiFuction<String, Integer, Apple> c3 = Apple::new;
            Apple a3 = c3.apply(GREEN, 110);
            ```
            

## 3.7 람다, 메소드 참조 활용하기

사과리스트를 정렬하는 예제를 다시 보면서 람다, 메소드 참조를 적용해보자

1단계 : 코드 전달

- List는 sort()라는 API를 제공한다. 이 sort는 외부로부터 Comparator 객체를 전달 받아서 정렬을 수행한다. 전달받은 Comparator 객체가 구현한 전략에 따라 정렬이 달라지므로 sort에는 동작 파라미터화가 적용되어 있다.
    
    ```java
    public class AppleComparator implements Comparator<Apple> {
    	public int compare(Apple a1, Apple a2){
    		return a1.getWeight().compareTo(a2.getWeight());
    	}
    }
    inventory.sort(new AppleComparator());
    ```
    

2단계 : 익명 클래스 사용

- 그런데 AppleComparator는 한번 사용될 객체이므로 익명 클래스를 이용하는 것이 더 낫다.
    
    ```java
    inventory.sort(new Comparator<Apple>{
    	public int compare(Apple a1, Apple a2){
    		return a1.getWeight().compareTo(a2.getWeight());
    	}
    });
    ```
    

3단계 : 람다 표현식 이용

- 익명클래스를 sort에 전달하는 코드는 장황하다. 그러니 익명클래스를 람다로 대체힌다. 함수형 인터페이스에는 람다를 사용할 수 있다. sort의 인자는 Comparator 타입의 함수형 인터페이스를 인자로 받으므로 이 시그니처에 해당하는 람다를 인자로 받을 수 있다.
    
    ```java
    inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()) );
    ```
    
- 자바 컴파일러는 컨텍스트를 통해서 람다의 파라미터 리스트의 형식을 추론할 수 있다. 따라서 코드를 다음으로 수정할 수 있다.
    
    ```java
    inventory.sort((a1, a2) -> a1.getWeight().compareTo(a2.getWeight()));
    ```
    
- Comparator는 Comparable로부터 키를 추출해서 Comparator를 만드는 `comparing`을 정적 메서드로 제공한다. comparing에는 Comparable로부터 키를 추출하는 Function 함수를 전달하면 된다. 따라서 코드는 다음과 같이 수정할 수 있다.
    
    ```java
    import static java.util.Comparator.comparing;
    inventory.sort(comparing(apple -> apple.getWeight()));
    ```
    

4단계 : 메서드 참조 이용

- 메소드 참조를 이용하면 람다를 인수로 전달하는 코드를 더 깔끔하게 만들 수 있다. 코드가 짧아질 뿐만 아니라 코드의 의미도 명확해진다.
    
    ```java
    inventory.sort(comparing(Apple::getWeight));
    ```
    

## 3.8 람다 표현식을 사용할 수 있는 유용한 메소드

Comparator 조합

- `reverse`
    - Comparator 인터페이스에서 정렬 순서를 뒤집는 디폴트 메소드를 제공한다.
        
        ```java
        inventory.sort(comparing(Apple::getWeight).reversed());
        ```
        
- `thenComparing`
    - 비교자로 비교를 수행했는데 두 객체가 같다고 나올 경우는 어떻게 처리해야할까? Comparator는 두번째 비교자 함수를 지정하는 디폴트 메소드를 제공한다. 첫번째 비교자로 두 객체를 비교해서 같다면 두번째 비교자로 비교를 수행하여서 정렬한다.
        
        ```java
        inventory.sort(comparing(Apple::getWeight)
        	.reversed()
        	.thenComparing(Apple::getCountry)); // 무게가 같다면 국가로 정렬한다 
        ```
        

Predicate 조합

- 복잡한 프레디케이트를 만들 수 있도록 3개의 디폴트 메소드 `negate`, `and`, `or`를 제공한다.
- `negate`
    - 특정 프레디케이트를 반전시킬 때 사용
        
        ```java
        Predicate<Apple> notRedApple = redApple.negate()
        ```
        
- `and`
    - 두 프레디케이트를 모두 만족시키는 프레디케이트를 만들 때 사용
        
        ```java
        Predicate<Apple〉 redAndHeavyApple =
        redApple.and(apple -> apple.getWeight() > 150);
        ```
        
- `or`
    - 두 프레디케이트 둘 중 하나라도 만족하는 프레디케이트를 만들  때 사용
        
        ```java
        Predicate<Apple> redAndHeavyAppleOrGreen =
        redApple.and(apple -> apple. getWeight () > 150) 
        .or(apple -> GREEN.equals(a.getColor()))
        ```
        
         
        

Function 조합

- Function을 조합해서 파이프라인을 만들 수 있도록 `andThen`, `compose`를 디폴트 메소드로 제공한다.
- `andThen`
    - 이 메소드를 호출한 함수의 출력이 인자로 주어진 함수의 입력으로 전달된다.
        
        ```java
        Function<Integer, Integer) f = x -> x + 1;
        Function〈Integer, Integer) g = x -> x * 2;
        Function<Integer, Integer> h = f .andThen(g);
        int result = h.apply(1); // (1 + 1) * 2 = 4가 result에 저장된다.
        ```
        
- `compose`
    - 인자로 주어진 함수의 출력이 이 메소드를 호출한 함수의 입력으로 전달된다.
        
        ```java
        Function<Integer, Integer) f = x -> x + 1;
        Function〈Integer, Integer) g = x -> x * 2;
        Function<Integer, Integer> h = f.compose(g);
        int result = h.apply(1); // (1 * 2) + 1 = 3이 result에 저장된다.
        ```