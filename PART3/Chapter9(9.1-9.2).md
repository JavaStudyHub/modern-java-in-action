# Chapter 9. 리팩터링, 테스팅, 디버깅

프로젝트를 진행하면서 시작부터 모든 것을 설계하고 효율적으로 구현하는 것은 쉽지 않다. 따라서 우리는 가독성과 유연성을 높이기 위한 리팩토링 과정을 필수적으로 거쳐야한다.

이 때 람다 표현식을 사용해 어떤 식으로 리팩토링을 할 수 있는지 알아보자.

<br>

## 9.1 가독성과 유연성을 개선하는 리팩토링

람다 표현식은 앞서 배웠던 내용들처럼, 동작 파라미터화의 형식을 지원한다. 이를 통해 람다 표현식을 이용한 코드는 더 큰 유연성을 갖출 수 있다.

<br>

### 9.1.1 코드 가독성 개선

> **코드 가독성이란 ‘어떤 코드를 다른 사람도 쉽게 이해할 수 있음’ 을 의미한다.**
> 

→ 즉 코드 가독성을 개선하겠다는 의미는, 우리가 구현한 코드를 다른 사람이 쉽게 이해하고 유지보수할 수 있게 만드는 것을 의미한다.

그럼 이제 몇 가지 예제를 살펴보면 어떻게 하면 람다, 메서드 참조, 스트림을 활용해 코드 가독성을 개선할 수 있을지 살펴보자

<br>

### 9.1.2 익명 클래스를 람다 표현식으로 리팩토링하기

> **하나의 추상 메서드를 구현하는 익명 클래스는 람다 표현식으로 리팩토링할 수 있다.**
> 

익명 클래스는 인수로 클래스를 선언함과 동시에 인스턴스를 만들어서 넘겨줄 수 있다. 그러다보니 코드가 장황해지고 에러를 발생시키기 쉽다는 단점을 갖는다.

이 때 람다식을 이용하면 더 간결하고 가독성이 좋은 코드를 구현할 수 있다.

- **Runnable 객체를 만드는 익명 클래스**
    
    ```java
    Runnable r1 = new Runnable() {
    		public void run() {
    				System.out.println("Hello");
    		}
    };
    ```
    
- **Runnable 객체를 만드는 람다 표현식**
    
    ```java
    Runnable r2 = () -> System.out.println("Hello");
    ```
    
<br>

> [!IMPORTANT]
>
> **모든 익명 클래스를 람다 표현식으로 변환할 수 있는 것은 아니다!**
>
> 1. **익명 클래스에서 사용한 this와 super는 람다 표현식에서 다른 의미를 갖는다.**
>    - 익명 클래스에서의 `this` : 익명 클래스 자신을 가리킴
>    - 람다에서의 `this` : 람다를 감싸는 클래스를 가리킴
>
> 2. **익명 클래스는 자신을 감싸고 있는 클래스(혹은 메서드)가 지니는 변수를 가릴 수 있다. (섀도 변수 - shadow variable)**
>    - **익명 클래스**는 **자신을 감싸고 있는 클래스(혹은 메서드)와 다른 스코프를 지닌다.**  
>      익명 클래스 안에서 같은 이름의 변수를 새로 선언하면, 바깥의 변수는 가려지고(섀도잉), 익명 클래스 안에서는 새로 선언한 변수만 보인다.
>
>      ```
>      int a = 10;
>
>      Runnable r1 = new Runnable() {
>          public void run() {
>              int a = 2;
>              System.out.println(a);
>          }
>      };
>
>      r1.run(); 
>      // 2 출력
>      ```
>
>    - **람다 표현식**은 **자신을 감싸고 있는 클래스(혹은 메서드)와 같은 스코프를 공유**한다.  
>      따라서 람다 내부에서는 같은 이름의 변수를 새로 선언할 수 없다.
>
>      ```
>      int a = 10;
>
>      Runnable r2 = () -> {
>          int a = 2;
>          System.out.println(a);
>      };
>
>      r2.run();
>      // 컴파일 에러
>      ```
>
> 3. **익명 클래스를 람다 표현식으로 바꾸면 콘텍스트 오버로딩에 따른 모호함이 초래될 수 있다.**
>    - 익명 클래스는 인스턴스화 할 때 명시적으로 형식이 정해지지만, 람다의 형식은 콘텍스트에 따라 달라진다.
>
>      ```
>      // Runnable과 동일한 시그니처를 갖는 함수형 인터페이스 Task
>
>      interface Task {
>          public void execute();
>      }
>
>      public static void doSomething(Runnable r) {
>          r.run();
>      }
>
>      public static void doSomething(Task t) {
>          t.execute();
>      }
>      ```
>
>    - **익명 클래스 사용**
>
>      ```
>      doSomething(new Task() {
>          public void execute() {
>              System.out.println("Danger danger!!");
>          }
>      });
>      ```
>      → 인스턴스화 할 때 명시적으로 형식 지정
>
>    - **람다 표현식 사용**
>
>      ```
>      doSomething(() -> System.out.println("Danger danger!!"));
>      ```
>      → 이 경우, Runnable과 Task 모두 대상이 될 수 있어 doSomething(Runnable)과 doSomething(Task) 중 어느 것을 호출할지 모호해진다.
>
>      → 이럴 때에는 람다식에 대해 명시적 형변환을 해줌으로써 모호함을 해소할 수 있다.
>
>      ```
>      doSomething((Task)() -> System.out.println("Danger danger!!"));
>      ```

    
<br>

### 9.1.3 람다 표현식을 메서드 참조로 리팩토링하기

> **람다 표현식을 대신해 메서드 참조를 이용할 경우, 메서드 명을 통해 코드의 의도를 명확하게 알릴 수 있어 가독성을 높일 수 있다.**
> 

- **칼로리 수준으로 요리를 그룹화하는 코드**
    - **람다 표현식**
        
        ```java
        Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream()
        		collect(
        				groupingBy(dish -> {
        						if(dish.getCalories() <= 400)
                                    return CaloricLevel.DIET;
        						else if(dish.getCalories() <= 700)
                                    return CaloricLevel.NORMAL;
        						else 
                                    return CaloricLevel.FAT;
        }));
        ```
        
    - **메서드 참조**
        
        > **람다 표현식을 별도의 메서드로 추출한 후 groupingBy 에 인수로 전달하는 형태**
        > 
        
        ```java
        Map<CaloricLevel, List<Dish>> dishedByCaloricLevel = menu.stream().collect(**groupingBy(Dish::getCaloricLevel**));
        ```
        
        ```java
        public class Dish {
        		...
        		public CaloricLevel getCaloricLevel() {
        				if(dish.getCalories() <= 400)
                            return CaloricLevel.DIET;
                        else if(dish.getCalories() <= 700)
                            return CaloricLevel.NORMAL;
                        else 
                            return CaloricLevel.FAT;
        		}
        }
        ```
        

<br>

**`comparing`, `maxBy` 와 같은 정적 헬퍼 메서드**를 활용하는 것도 좋다.

```java
// 1. 람다 표현식
inventory.sort(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 2. 메서드 참조
inventory.sort(comparing(Apple::getWeight));
```

<br>

**`sum`**, **`maximum` 등 자주 사용하는 리듀싱 연산도 메서드 참조와 함께 사용할 수 있는 내장 헬퍼 메서드를 제공한다.**

```java
// 1. 람다 표현식
int totalCalories = menu.stream().map(Dish::getCalories)
		.resuce(0, (c1, c2) -> c1 + c2);
		
// 2. 메서드 참조
int totalCalories = menu.stream().collect(summingInt(Dish::getCalories));	
```

이러한 예시 코드들을 확인해보면 메서드 참조를 통해 실제 로직 부분이 더 간결해지고 코드의 의미를 해석하기에 용이해졌음을 알 수 있다.

<br>

### 9.1.4 명령형 데이터 처리를 스트림으로 리팩토링하기

> **이론적으로는 반복자(for, while) 를 이용한 기존의 모든 컬렉션 처리 코드를, 스트림 API로 바꿔야한다.**
> 

→ 파이프라인의 의도를 더 명확하게 보여줄 뿐더러, 강력한 최적화 및 멀티코어 아키텍처를 활용할 수 있는 지름길을 제공하기 때문이다. 

우선 필터링 및 추출 로직이 뒤섞인 명령형 코드를 살펴보자.

```java
List<String> dishNames = new ArrayList<>();

for(Dish dish : menu) {
		if(dish.getCalories() > 300) {
				dishNames.add(dish.getName());
		}
}
```

이 코드의 동작 방식을 알기 위해서는 전체 구현을 자세히 살펴본 이후에야 의도를 이해할 수 있다. 

심지어 병렬 처리 또한 매우 까다롭다.

이 때 스트림 API 를 이용하면 문제를 더 직접적으로 기술함과 동시에 쉽게 병렬화가 가능해진다.

```java
menu.parallelStream()
		.filter(d -> d.getCalories() > 300)
		.map(Dish::getName)
		.collect(toList());
```

다만 명령형 코드에는 break, continue, return 등 제어 흐름문이 여럿 존재할 수 있고, 그 흐름문을 모두 분석해서 같은 기능을 하는 스트림 연산으로 변환하기는 쉬운 일이 아니다. 하지만 다행히도 명령형 코드를 스트림 API로 바꾸도록 도움을 주는 몇 가지 도구가 존재한다.

<br>

### 9.1.5 코드 유연성 개선

람다 표현식을 이용해 동작 파라미터화를 쉽게 구현할 수 있으며, 이를 통해 다양한 동작을 표현할 수 있다.

- **함수형 인터페이스 적용**
    
    > **람다 표현식을 이용하기 위해서는 함수형 인터페이스가 필요하다.**
    > 

- **조건부 연기 실행**
    
    ```java
    if(logger.isLoggable(Log.FINER)) {
    		logger.finer("Problem: " + generateDiagnostic());
    }
    ```
    
    위와 같은 로깅 관련 코드가 존재한다고 할 때. 위의 코드에는 다음과 같은 문제가 존재한다.
    
    1. **logger의 상태가 isLoggable 이라는 메서드에 의해 클라이언트 코드로 노출된다.**
    2. **메세지를 로깅할 때마다 logger 객체의 상태를 매번 확인해야할까? 이는 코드를 어지럽힐 뿐이다.**

    <br>
    
    따라서 이런 경우 메세지를 로깅하기 전, logger 객체가 적절한 수준으로 설정되었는지 내부적으로 확인하는 log 메서드를 사용하는 것이 바람직하다.
    
    ```java
    logger.log(Level.FINER, "Problem: " + generateDiagnostic());
    ```
    
    다만 이 코드 역시도 문제가 존재한다. logger가 적절한 수준으로 설정되어있지 않더라도, 항상 로깅 메세지를 평가하게된다.
    
    즉 우리는 특정 조건 하에서만 메세지가 생성될 수 있도록, 메세지 생성 과정을 연기할 수 있어야한다. 이에 따라 자바 8 API 설계자는 Supplier 를 인수로 갖는 오버로드된 log 메서드를 제공한다.
    
     
    
    ```java
    public void log(Level level, Supplier<String> msgSupplier) {
    		if(logger.isLoggable(level)) {
    				log(level, **msgSupplier.get()**);
    		}
    }
    
    // 실행 형태
    logger.log(Level.FINER, () -> "Problem: " + generateDiagnostic());
    ```
    
    → 이처럼 람다 표현식을 인수로 전달하면서, 내부적으로는 logger수준을 확인하고 람다식을 실행시킬지 말지를 결정한다. 
    
    **이런 식으로 코드를 작성하게 되면, 만약 객체의 상태를 확인해야하는 것과 같은 로직이 중복되어 자주 사용되는 경우, 틀( = 메서드)을 만들어두고 실제 구현 부분에 대해서는 동작 파라미터화를 적용함으로써 유연성을 높이고 캡슐화를 강화할 수 있다!**
    

<br>

- **실행 어라운드**
    
    > **매번 같은 준비, 종료 과정을 반복적으로 수행하는 로직을 재사용함으로써 코드 중복을 줄이는 기법**
    > 
    
    ```java
    public interface BufferedReaderProcessor {
    		String process(BufferedReader b) throws IOException;
    }
    
    public static String processFile(BufferedReaderProcessor p) throws IOException {
    		try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    				return p.process(br);
    		}
    }
    
    String oneLine = processFile((BufferedReader b) -> b.readLine());
    
    String twoLines = processFile((BufferedReader b) -> b.readLine() + b.readLine());
    ```
    
    앞에서 언급한 것과 같이 중복되는 로직은 재사용하되, 실제 각각의 기능들에 특화된 로직들을 람다식으로 전달받도록 함으로써 객체의 동작을 유동적으로 결정할 수 있다.
    

<br>

## 9.2 람다로 객체지향 디자인 패턴 리팩토링하기

> **디자인 패턴은 공통적인 소프트웨어 문제를 설계할 때 재사용할 수 있는, 검증된 청사진을 제공한다.**
> 

이 디자인 패턴에 람다 표현식이 더해지면 문제들을 더 쉽고 간단하게 해결할 수 있게 된다.

- **디자인 패턴 예시**
    - **전략 패턴**
    - **템플릿 메서드 패턴**
    - **옵저버 패턴**
    - **의무 체인 패턴**
    - **팩토리 패턴**

<br>

### 9.2.1 전략 패턴

> **전략 패턴이란, 한 유형의 알고리즘을 보유한 상태에서 런타임에 적절한 알고리즘을 선택하는 기법을 의미**
> 

전략 디자인 패턴을 이루는 세 가지 요소는 다음과 같다.
<img width="800" alt="스크린샷 2025-05-11 오전 12 24 57" src="https://github.com/user-attachments/assets/6c256306-aae5-435a-b014-ef3fe76e6628" />

- **알고리즘을 나타내는 인터페이스 (Strategy 인터페이스)**
- **다양한 알고리즘을 나타내는 한 개 이상의 인터페이스 구현**
    - ConcreteStrategyA
    - ConcreteStrategyB
- **전략 객체를 사용하는 한 개 이상의 클라이언트**

예시를 보며 이해해보자.

예를 들어 텍스트 입력이 특정한 조건에 맞게 포맷, 즉 형식이 맞춰져 있는지 검증해야한다고 가정해보자. 그럼 이후에 어떤 순서로 전략 패턴을 사용할 수 있을까?

1. **String 문자열을 검증하는 인터페이스 구현**
    
    ```java
    public interface ValidationStrategy {
    		boolean execute(String s);
    }
    ```
    

1. **인터페이스를 실제 구현하는 클래스를 하나 이상 정의**
    - 소문자 체크
        
        ```java
        public class IsAllLowerCase implements ValidationStrategy {
        		public boolean execute(String s) {
        				return s.matches("[a-z]+");
        		}
        }
        ```
        
    - 숫자 체크
        
        ```java
        public class IsNumeric implements ValidationStrategy {
        		public boolean execute(String s) {
        				return s.matches("\\d+");
        		}
        }
        ```
        

1. **검증 전략 활용 코드 구현**
    
    ```java
    public class Validator {
    		private final ValidationStrategy strategy;
    		
    		public Validator(ValidationStrategy v) {
    				this.strategy = v;
    		}
    		
    		public boolean validate(String s) {
    				return strategy.execute(s);
    		}
    }
    ```
    
    ```java
    Validator validator = new Validator(new IsNumeric());
    
    boolean b1 = validator.validate("aaaa"); <- false 반환
    
    validator = new Validator(new IsAllLowerCase());
    
    boolean b2 = validator.validate("bbbb"); <- true 반환
    ```
    

→ 이런식으로 다형성을 활용하는 것이 곧 전략 패턴이라고 볼 수 있다.

<br>

- **람다 표현식 사용**
    - ValidationStrategy 는 함수형 인터페이스이며, Predicate<String> 과 같은 함수 디스크립터를 갖고 있음을 알 수 있다.
        - 문자열을 넘겨주었을 때, 불리언 값을 반환하기 때문이다.
    
    → **즉 ValidationStrategy 자체가 함수형 인터페이스이기 때문에 Predicate<String> 에 알맞은 람다식을 작성하여 인수로 넘겨주면 새로운 클래스를 구현할 필요 없이 다양한 전략을 구현할 수 있게 된다.**
    
    ```java
    Validator validator = new Validator((String s) -> s.matches("[a-z]":));
    
    boolean b1 = validator.validate("aaaa"); 
    
    validator = new Validator((String s) -> s.matches("\\d+"));
    
    boolean b2 = validator.validate("bbbb");
    ```
    

이렇듯 람다 표현식은 전략 디자인 패턴에서 발생하는 자잘한 코드를 제거할 수 있으며, 전략을 함수형 인터페이스로 구현해둔다면 실제 인터페이스 구현체들을 람다 표현식으로 대신할 수도 있다. 

<br>

### 9.2.2 템플릿 메서드 패턴

> **알고리즘의 개요(뼈대, 틀)를 제시한 다음, 알고리즘의 일부를 고칠 수 있는 유연함을 제공해야하는 경우 사용하는 기법**
> 

간단한 온라인 뱅킹 애플리케이션을 구현한다고 가정해보자.

이 때 각 은행마다 다양한 온라인 뱅킹 애플리케이션을 사용하며 동작 방법도 다 다르다. 하지만 모두 온라인 뱅킹 애플리케이션이라는 점에서 동일한 목적을 지니기 때문에 추상화를 해볼 수 있다.

```java
abstract class OnlineBanking {
		public void processCustomer(int id) {
				Customer c = Database.getCustomerById(id);
				makeCustomerHappy(c);
		}
		
		abstract void makeCustomerHappy(Customer c);
}
```

이 때 각각의 온라인 뱅킹 애플리케이션들은 이 추상 클래스를 상속함으로써 데이터베이스로부터 사용자 정보를 가져오는 동작은 공통적으로 사용하되, 실제 내부 동작 방식은 원하는 방식으로 커스텀하여 구현할 수 있다.

하지만 여기서 람다 표현식을 적용하면 더 간결하게 코드를 구현할 수 있다.

<br>

- **람다 표현식 사용**
    - makeCustomerHappy 메서드를 잘 살펴보면 어떤 함수형 인터페이스 하나와 시그니처가 동일하다는 것을 알 수 있다. 바로 **`Consumer<T> 함수형 인터페이스`**이다. 이들은 모두 특정 객체를 인수로 받아 void 를 반환한다. 그럼 이제 다음과 같은 형태의 구현을 고려해볼 수 있다.
        
        ```java
        class OnlineBankingLambda {
        		...
        		
        		public void processCustomer(int id, Consumer<Customer> makeCustomerHappy) {
        				Customer c = Database.getCustomerById(id);
        				
        				makeCustomerHappy.accept(c);
        		}
        }
        ```
        
    
    이제 각각의 서로 다른 온라인 뱅킹 애플리케이션을 개발할 때, OnlineBanking 클래스를 직접 상속 받고 makeCustomerHappy 메서드를 실제 구현하지 않고도, 람다 표현식을 전달함으로써 다양한 동작을 하도록 만들 수 있다.
    
    ```java
    OnlineBankingLambda obl = new OnlineBankingLambda();
    
    obl.processCustomer(1337, (Customer c) -> System.out.println("Hello " + c.getName());
    ```
    

이렇게 람다 표현식을 사용하면 템플릿 메서드 패턴에서 발생하는 자잘한 코드 역시도 제거할 수 있다.

<br>

### 9.2.3 옵저버 패턴

> **옵저버 패턴은, 어떤 이벤트가 발생했을 때 한 객체(주제)가 다른 객체 리스트 (옵저버)에 자동으로 알림을 보내야하는 상황에 사용된다.**
>
> 
<img width="800" alt="스크린샷 2025-05-11 오전 12 25 19" src="https://github.com/user-attachments/assets/39fdabd2-3841-4e59-b73a-caacc07c2407" />

**다양한 신문 매체(뉴욕 타임즈, 가디언) 가 뉴스 트윗을 구독하고 있으며, 특정 키워드를 포함하는 트윗이 등록되면 알림을 받고 싶어하는 상황**을 예시로 들어 옵저버 패턴에 대해 알아보자. 위와 같은 상황에 옵저버 패턴을 적용하려면 어떤 과정을 거쳐야할까?

1. **다양한 옵저버를 그룹화할 Observer 인터페이스가 필요.**
    - Observer 인터페이스는 새로운 트윗이 있을 때, 주제가 호출할 수 있도록 notify 라고 하는 하나의 메서드를 제공한다.
    
    ```java
    interface Observer {
    		void notify(String tweet);
    }
    ```
    

1. **트윗에 포함된 다양한 키워드에 다른 동작을 수행할 수 있는 여러 옵저버를 정의한다.**
    
    ```java
    class NYTimes implements Observer {
    		public void notify(String tweet) {
    				if(tweet != null && tweet.contains("money")) {
    						System.out.println("Breaking news in NY! " + tweet);
    				}
    		}
    }
    
    class Guardian implements Observer {
    		public void notify(String tweet) {
    				if(tweet != null && tweet.contains("queen")) {
    						System.out.println("Yet more news from London... " + tweet);
    				}
    		}
    }
    ```
    

1. **주제를 구현한다.**
    
    ```java
    interface Subject {
    		void registerObserver(Observer o);
    		void notifyObservers(String tweet);
    }
    
    class Feed implements Subject {
    		private final List<Observer> observers = new ArrayList<>();
    		
    		public void registerObserver(Observer o) {
    				this.observers.add(o);
    		}
    		
    		public void notifyObservers(String tweet) {
    				observers.forEach(o -> o.notify(tweet));
    		}
    }
    ```
    
    - **registerObserver** : 새로운 옵저버를 등록
    - **notifyObservers** : 옵저버에게 트윗의 정보를 알려준다.
    - **Feed**
        - 트윗을 받았을 때 알림을 보낼 옵저버 리스트를 유지
        - 특정 주제와 옵저버를 연결하여 사용 가능
    
    ```java
    Feed f = new Feed();
    
    f.registerObserver(new BYTimes());
    f.registerObserver(new Guardian());
    
    f.notifyObservers("The queen said her favourite book is Modern Java in Action!");
    
    // 결과
    Yet more news from London ... The queen said her favourite book ...
    ```
    

→ pub/sub 개념이라고 생각하면 될 것 같다.

<br>

- **람다 표현식 사용**
    - 현재 옵저버 패턴의 코드를 확인해보면 Observer 인터페이스를 구현하는 모든 클래스가 하나의 메서드 notify 를 구현하고있는 것을 볼 수 있다. 즉, 트윗이 도착했을 때 어떤 동작을 수행할 것인지 감싸는 코드를 구현한 것이다.
    
    → 여기에 람다를 적용하면 각각의 옵저버들을 명시적으로 인스턴스화하지 않고, 람다 표현식을 직접 전달해서 실행할 동작을 지정할 수 있다.
    
    ```java
    Feed f = new Feed();
    
    f.registerObserver((String tweet) -> {
    		if(tweet != null && tweet.contains("money")) {
    				System.out.println("Breaking new in NY! " + tweet);
    		}
    });
    
    f.registerObserver((String tweet) -> {
    		if(tweet != null && tweet.contains("queen")) {
    				System.out.println("Yet more news from London ... " + tweet);
    		}
    });
    ```

물론 항상 람다식을 사용해야하는 것은 아니다! 경우에 따라 람다식 부분에 들어가야할 로직이 길어진다면, 기존처럼 클래스 구현 방식을 고수하는 것이 바람직할 수도 있다.

<br>

### 9.2.4 의무 체인 패턴

> **의무 체인 패턴은 한 객체가 어떤 작업을 처리한 다음에 다른 객체로 결과를 전달하고, 다른 객체로 해야 할 작업을 처리한 다음에 또 다른 객체로 전달하는 방식이다.**
> 
    
<img width="800" alt="스크린샷 2025-05-11 오전 12 25 37" src="https://github.com/user-attachments/assets/17a84ab7-1c75-42d2-a256-d9a41db94943" />

일반적으로 다음으로 처리할 객체 정보를 유지하는 필드를 포함하는, 작업 처리 추상 클래스로 의무 체인 패턴을 구성한다.

**작업 처리 객체가 자신의 작업을 끝냈으면 다음 작업 처리 객체로 결과를 전달**한다.

```java
public abstract class ProcessingObject<T> {
		protected ProcessingObject<T> successor;
		
		public void setSuccessor(ProcessingObject<T> successor) {
				this.successor = successor;
		}
		
		public T handle(T input) {
				T r = handleWork(input);
				if(successor != null) {
						return successor.handle(r); <- 실제 체인 부분
				}
				return r;
		}
		
		abstract protected T handleWork(T input);
}
```

각각의 작업 처리 객체는 이 ProcessingObject 클래스를 상속받아 handleWork 메서드를 구현함으로써 다양한 종류의 작업 처리를 할 수 있다.

```java
public class HeaderTextProcessing extends ProcessingObject<String> {
		public String handleWork(String text) {
				return "From Raoul, Mario and Alan: " + text;
		}
}

public class SpellCheckerProcessing extends ProcessingObject<String> {
		public String handleWork(String text) {
				return text.replaceAll("labda", "lambda");
		}
}
```

이 두 작업 처리 객체를 연결해서 작업 체인을 만들 수 있다.

```java
ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObject<String> p2 = new SpellChecketProcessing();

p1.setSuccessor(p2);
String result = p1.handle("Aren't labdas is Awesome?!");
System.out.println(result);

// 결과
From Raoul, Mario and Alan: Aren't lambdas is Awesome?!
```

<br>

- **람다 표현식 사용**
    - 이 형태는 메서드 체이닝 과 상당히 유사함을 알 수 있다.
    - 작업 처리 객체들을 **`Function<String, String>`**, 더 정확히는 **`UnaryOperator<String>`** 형식의 인스턴스로 표현할 수 있다.
    - **`andThen`** 메서드로 함수를 조합해서 체인을 만들 수 있다.
    
    ```java
    UnaryOperator<String> headerProcessing = (String text) -> "From Raoul, Mario and Alan: " + text; <- 첫 번째 작업 처리 객체
    
    UnaryOperator<String> spellCheckerProcessing = (String text) -> text.replaceAll("labda", "lambda"); <- 두 번째 작업 처리 객체
    
    Function<String, String> pipeline = headerProcessing.andThen(spellCheckerProcessing); <- 동작 체인으로 함수를 조합
    
    String result = pipeline.apply("Aren't labdas is Awesome?!");
    ```
    
<br>

### 9.2.5 팩토리 패턴

> **팩토리 디자인 패턴은 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들 수 있도록 하는 패턴이다.**
> 

다양한 종류의 객체를 만들어야하는 상황이라고 할 때 Factory 클래스를 적용해보자.

```java
public class ProductFactory {
		public static Product createProduct(String name) {
				switch(name) {
						case "loan" : return new Loan();
						case "stock" : return new Stock();
						case "bond" : return new Bond();
						default: throw new RuntimeException("No such product " + name);
				}
		}
}
```

이러한 코드를 통해 생성자와 설정을 외부로 노출하지 않으면서도, 클라이언트는 단순하게 상품을 생산할 수 있게 된다.

실제로 코드를 사용하는 사람은 내부 로직을 모르고도 객체를 마음대로 생성할 수 있다.

```java
Product p = ProductFactory.createProduct("loan");
```

<br>

- **람다 표현식 사용**
    
    > **생성자도 메서드 참조처럼 접근이 가능하다.**
    > 
    
    ```java
    final static Map<String, Supplier<Product>> map = new HashMap<>();
    
    static {
    		map.put("loan", Loan::new);
    		map.put("stock", Stock::new);
    		map.put("bond", Bond::new);
    }
    
    public static Product createProduct(String name) {
    		Supplier<Product> p = map.get(name);
    		if(p != null) return p.get();
    		throw new IllegalArgumentException("No such product " + name);
    }
    ```
    
    → 이처럼 팩토리 메서드 패턴이 수행하던 작업을 자바 8의 새로운 기능을 활용해 간결하게 만들었다.
    
    다만 생성자의 파라미터 수가 많은 경우, 위와 같은 형식으로 코드를 작성하는 것은 어려울 수 있으니 주의하자
