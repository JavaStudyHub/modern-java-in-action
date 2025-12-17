# Chapter 10 (10.1 ~ 10.2)
## 개요

### DSL(Domain-Specific Language)이란?

> 특정 도메인을 대상으로 한 특수 프로그래밍 언어
> 
- 애플리케이션의 **비즈니스 로직을 표현**하는데 사용한다.
- 일반 프로그래밍 언어보다 간결하고 이해하기 쉽다.
- 예) 메이븐, 앤트 등의 빌드 도구에서 사용한다.

### DSL과 자바의 관계

- 과거 자바는 완고함, 장황함 등의 특성 때문에 기술 배경이 없는 사람들이 사용하기에 부적절하다고 간주된다.
- 하지만 요즘 자바에 **람다 표현식**을 지원하면서 코드가 간결해지고 이해하기 쉬워진다.
    
    → **DSL 표현력 상승**
    

### 외부적 DSL vs 내부적 DSL

- **외부적 DSL** : SQL, HTML처럼 텍스트 기반 독립 DSL
    
    → 별도의 파서와 해석기가 필요
    
- **내부적 DSL** : 자바 코드로 구현, 기존 언어 문법을 이용한 DSL
    
    → 자바 API 및 메서드를 조합해 동작
    

```java
menu.stream()
    .filter(d -> d.getCalories() < 400)
    .map(Dish::getName)
    .forEach(System.out::println);
```

위 코드는 자바의 스트림 API 기반의 **내부적 DSL**

- **플루언트 스타일(fluent style)**로 메서드 체인을 통해 루프를 제거하고 복잡함을 해소한다.
- `*SELECT name FROM menu WHERE clorie < 400`* (**외부적 DSL**) **과 유사한 의미를 전달한다.

</br>

## 10.1 도메인 전용 언어

- DSL은 특정 비즈니스 도메인의 문제를 해결하기 위해 만든 언어
- 예) 회계 전용 소프트웨어 애플리케이션의 비즈니스 도메인 ⇒ 통장 입출금 내역서, 계좌 통합
- 자바에서는 **도메인을 표현할 수 있는 클래스와 메서드의 집합**을 DSL이라고 볼 수 있음
    - 엄밀히 말하면, DSL = 특정 비즈니스 **도메인을 인터페이스로 만든 API**

**DSL의 특징**

1. **의사 소통의 왕** : 코드의 의도가 명확히 전달되어야 하고, 비개발자도 이해할 수 있어야 한다.
2. **한 번 코드를 구현하지만 여러 번 읽는다** : 항상 가독성을 고려해서 구현해야 한다.

</br>

### 10.1.1 DSL의 장점과 단점

**장점**

- **간결함** : 비즈니스 로직을 캡슐화하여 반복을 제거하고 코드를 간결하게 한다.
- **가독성** : 도메인 용어 사용 → 비 도메인 전문가도 코드를 쉽게 이해할 수 있다.
- **유지보수** : 빈번히 바뀌는 비즈니스 로직 관리에 용이하다.
- **높은 수준의 추상화** : 같은 추상화 수준에서 동작하므로 도메인의 문제와 직접적으로 관련되지 않은 세부 사항을 숨김
- **집중** : 비즈니스 도메인의 규칙을 표현하고 있는 언어이므로 개발자는 특정 코드에만 집중하여 생산성이 증가
- **관심사 분리** : 인프라 관련 로직과 비즈니스 관련 로직 분리에 용이하다.

**단점**

- **DSL 설계의 어려움** : 제한적인 언어에 도메인 지식을 담는 것이 까다롭다.
- **개발 비용** : 코드에 DSL을 추가하는 작업은 초기 프로젝트에 많은 비용과 시간이 소모되는 작업이다.
- **추가 우회 계층** : DSL은 도메인 모델을 감싸는 추가적인 계층이므로 계층을 최대한 작게 만들어 성능 문제를 회피해야 한다.
- **새로 배워야 하는 언어** : DSL을 프로젝트에 추가하는 것은 팀이 배워야 하는 언어가 한 개 더 늘어난다는 부담이 따른다.
- **호스팅 언어 한계** : 자바 같은 장황한 프로그래밍 언어로 만든 DSL은 문법의 제약을 받고 오히려 읽기가 어려워지는 문제가 있다. (`람다 표현식`은 이 문제를 해결해주었다.)

</br>

### 10.1.2 JVM에서 이용할 수 있는 다른 DSL 해결책

**✅ 내부 DSL**

역사적으로 자바는 장황하고 유연성이 떨어지는 문법 때문에 읽기 쉽고, 간단하고, 표현력 있는 DSL을 만드는 데 한계가 있었다.

하지만, 람다 표현식이 등장하면서 이 문제가 어느정도 해결되었다.

먼저 기존 자바 코드를 살펴보자.

```java
List<String> numbers = Arrays.asList("one", "two", "three");
numbers.forEach(new Consumer<String>() {
		@Override
		public void accept(String s){
				System.out.println(s);
		}
});
```

현재 `신호 대비 잡음` 비율이 매우 높은 편이다.

| **용어** | **의미** |
| --- | --- |
| **신호(Signal)** | 코드가 전달하고자 하는 핵심 로직, 의도, 비즈니스 의미 |
| **잡음(Noise)** | 그 의도를 방해하거나, 핵심이 아닌 문법적 장황함 (ex. 반복되는 보일러플레이트 코드) |

위 예시 코드는 단순히 리스트 목록을 출력하려는 목적(=신호)을 갖지만,

- new Consumer<String>()
- @Override, accept() 등

많은 문법적 요소(잡음)가 포함되어 있다.

> 현준 : 저는 이 부분에서 책에서 번역의 오류(?)가 있었다고 생각합니다..  
책의 ‘`위 코드 예제에서 굵은 글씨로 표시한 부분이 코드의 잡음이다.`' 이 부분에서 설명하는 것이 잡음이 아닌 신호인 것 같습니다..
> 

이를 람다 표현식으로 바꾸면

```java
numbers.forEach(s -> System.out.println(s));
```

또는

```java
numbers.forEach(System.out::println);
```

→ 잡음이 줄어들어 신호대비 잡음 비율이 적정 수준으로 유지된다.

순수 자바로 DSL을 구현하면 다음과 같은 **장점**을 얻을 수 있다.

- 외부 DSL에 비해 추가 학습 부담이 적다.
- 별도의 DSL 도구나 컴파일러가 필요 없다.
- 기존 자바 IDE의 기능(자동완성, 자동 리팩터링)을 그대로 활용이 가능하다.
- 여러 도메인을 다뤄야 하는 상황에서 자바로 구현한 DSL은 쉽게 합칠 수 있다.

**✅ 다중 DSL**

> **JVM 위에서 실행되는 여러 언어**(Scala, Kotlin 등)를 이용해 DSL을 유연하게 구현하는 방식
> 

요즘 JVM에서 실행되는 언어는 매우 많다. 스칼라, 루비처럼 매우 유명한 언어를 포함하여 JRuby나 Jython 같은 언어도 JVM 기반이다. 

코틀린, 실론 같이 스칼라와 호환성을 유지하면서 단순하고 쉽게 배울 수 있는 언어도 있다.

**DSL은 기반 프로그래밍 언어의 영향을 받으므로 간결한 DSL을 만드는 데 새로운 언어의 특성**은 매우 중요하다.

스칼라는 DSL 개발에 필요한 커링, 임의 변환 등의 기능을 갖췄다.

예를 들어, 함수 f를 주어진 횟수만큼 반복 실행하는 유틸리티 함수를 구현한다고 하면, 스칼라에서는 다음과 같이 작성할 수 있다.

```java
def times(i: Int, f: => Unit): Unit = {
  f
  if (i > 1) times(i - 1, f)
}
```

스칼라는 꼬리 호출 최적화를 통해 times 함수 호출을 스택에 추가하지 않아 다음 코드에서 i가 아주 큰 숫자라 하더라도 스택 오버플로 문제가 발생하지 않는다.

이 함수를 통해 다음처럼 “Hello World”를 3번 출력할 수 있다.

```java
times(3, println("Hello World"))
```

times 함수를 커링하거나 두 그룹으로 인수를 놓는 것도 가능하다.

```java
def times(i: Int)(f: => Unit): Unit = {
  f
  if (i > 1) times(i - 1)(f)
}

times(3) {
  println("Hello World")
}
```

또, 스칼라는 Int를 암묵적으로 클래스처럼 확장할 수 있어, 다음과 같이도 작성 가능하다.

```java
implicit def intToTimes(i: Int) = new {
  def times(f: => Unit): Unit = {
    f
    if (i > 1) (i - 1).times(f)
  }
}

3 times {
  println("Hello World")
}
```

이처럼 문법적 잡음 없이도 반복을 표현 가능하고, 개발자가 아닌 사람도 쉽게 이해할 수 있다. 숫자 3은 자동으로 클래스 인스턴스로 변환되고, 점 표기 없이도 times 메서드를 호출할 수 있다.

하지만 자바에서는 비슷한 효과를 얻기 어렵다. 이는 **DSL 친화성이 언어마다 다름**을 명확히 보여준다. 이 접근법은 다음과 같은 **불편함**도 초래한다.

1. 새로운 언어를 배우거나 팀 내에 해당 언어에 능숙한 사람이 있어야 한다.
2. 여러 언어를 혼용하면 빌드 과정이 복잡해진다.
3. 대부분 JVM 언어가 자바와의 호환성을 주장하지만, 실제로는 완벽하지 않다.
    - 예) 스칼라와 자바 컬렉션이 상호 변환되지 않아 APi 호환을 위해 추가 작업이 필요하다.

**✅ 외부 DSL**

외부 DSL을 구현하기 위해서는 **새로운 문법과 구문을 가진 언어를 직접 설계**해야 한다.

새 언어를 파싱하고 결과를 분석하고, 실행 코드를 만드는 것은 매우 큰 작업이다.

ANTLR 같은 **자바 기반 파서 생성기**를 활용하면 도움이 될 수 있다. 하지만 논리적으로 명확한 언어를 새로 만드는 일은 결코 간단하지 않다. 외부 DSL은 쉽게 초기 목적을 벗어나 제어 범위를 넘기기도 한다는 점도 문제다.

외부 DSL의 가장 큰 장점은 **무한한 유연성**이다.

- **필요한 특성을 완벽하게 제공**하는 언어를 설계할 수 있다.
- **비즈니스 문제를 정확하게 표현하고 해결할 수 있는 표현력 높은 언어**를 만들 수 있다.

자바로 개발된 **인프라 구조 코드**와 외부 DSL로 구현된 **비즈니스 코드**를 명확하게 분리해준다는 장점도 있다.

→ **관심사 분리**에 유리

하지만 이러한 **관심사 분리**로 인해 **DSL과 호스트 언어 사이에 인공 계층**이 생기므로 이는 양날의 검으로 작용할 수 있다.

</br>

## 10.2 최신 자바 API의 작은 DSL

**네이티브 자바 API와 DSL**

- 자바 8 이전에도 이미 한 개의 추상 메서드를 가진 인터페이스가 존재했지만, 익명 내부 클래스를 사용하면 코드가 장황해졌다.
- 자바 8부터 **람다식**과 **메서드 참조**가 도입되며 DSL의 형태로 API를 사용할 수 있게 되었다.
- 대표적 예시로 Comparator 인터페이스에 새 메서드가 추가되었고, 그 활용법이 DSL 스타일로 바뀌었다.

**Comparator 예제**

- 기존 방식 (내부 클래스 사용)

```java
Collections.sort(persons, new Comparator<Person>() {
    public int compare(Person p1, Person p2) {
        return p1.getAge() - p2.getAge();
    }
});
```

- 자바 8 방식 (람다 표현식)

```java
Collections.sort(people, (p1, p2) -> p1.getAge() - p2.getAge());
```

**Comparator 유틸리티 메서드 활용**

- comparing 메서드 사용

```java
Collections.sort(persons, comparing(p -> p.getAge()));
```

- 메서드 참조 활용

```java
Collections.sort(persons, comparing(Person::getAge));
```

- `reverse` 정렬도 가능 : 비교 기준을 역순으로 바꿔 정렬

```java
Collections.sort(persons, comparing(Person::getAge).reverse());
```

- 다중 비교자 구성 (`thenComparing`)

```java
Collections.sort(persons, 
    comparing(Person::getAge)
        .thenComparing(Person::getName));
```

**List 인터페이스의 sort 메서드 사용**

```java
persons.sort(
    comparing(Person::getAge)
        .thenComparing(Person::getName));
```

- 더 간결하고 자연스럽게 정렬이 가능

> 💡 람다와 메서드 참조 ⇒ DSL의 가독성, 재사용성, 결합성을 높일 수 있다!
>

</br>

### 10.2.1 스트림 API는 컬렉션을 조작하는 DSL

- **Stream 인터페이스**는 네이티브 자바 API에 내부 DSL을 적용한 좋은 사례
- 컬렉션의 **필터, 정렬, 변환, 그룹화, 조작** 등 다양한 작업을 선언적으로 처리할 수 있는 강력한 DSL

**Ex) 로그 파일에서 “ERROR”로 시작하는 첫 40행을 수집하는 작업을 수행하는 예제**

✅ **기존 코드 (절차형 코드)**

```java
List<String> errors = new ArrayList<>();
int errorCount = 0;
BufferedReader bufferedReader = new BufferedReader(new FileReader(fileName));
String line = bufferedReader.readLine();
while (errorCount < 40 && line != null) {
    if (line.startsWith("ERROR")) {
        errors.add(line);
        errorCount++;
    }
    line = bufferedReader.readLine();
}
```

**문제점**

- 코드가 장황해 의도를 한 눈에 파악하기 어려움
- 문제가 분리되지 않아 가독성과 유지보수성 모두 저하
- **같은 의무를 가진 코드**가 여러 위치에 분산되어 있음
    1. File을 읽는 목적의 코드
    2. 첫 40행을 수집하는 목적의 코드

**✅ 개선된 방식 (함수형 코드)**

```java
List<String> errors = Files.lines(Paths.get(fileName))
    .filter(line -> line.startsWith("ERROR"))
    .limit(40)
    .collect(toList());
```

- **String** : 파일에서 파싱할 행
- **Files.line()** : 파일을 열어서 Stream<String>을 반환
- **filter()** : “ERROR”로 시작하는 행을 필터링
- **collect()** : 결과 문자열을 리스트로 수집

**⇒ 선언적이기 때문에 간결하고 의도가 명확!**

</br>

### 10.2.2 데이터를 수집하는 DSL인 Collectors

- **Stream 인터페이스** : **데이터 리스트를 조작**하는 DSL
- **Collectors API** : **데이터 수집**을 수행하는 DSL

특히 **정적 팩토리 메서드**를 활용하여 필요한 Collectors를 만들고 결합하는 방식이 DSL 관점 설계의 대표사례이다.

**✅ 다중 수준 그룹화**

```java
Map<String, Map<Color, List<Car>>> carsByBrandAndColor =
    cars.stream().collect(groupingBy(Car::getBrand,
                                     groupingBy(Car::getColor)));
```

- **중첩 groupingBy**를 통해 다중 수준 그룹화 Collector 구성 가능

**✅ Comparator와의 비교**

```java
Comparator<Person> comparator =
    comparing(Person::getAge).thenComparing(Person::getName);
```

- Comparator는 플루언트 방식으로 연결해서 다중 필드 Comparator를 정의
    
    ⇒ 비교 순서가 명확해짐
    
- 이에 비해 Collectors는 중첩 구조이며, 평가 순서(즉, 연산 순서)가 명확하지 않음

**✅ 다중 수준 그룹화를 위한 Collector**

```java
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> carGroupingCollector =
    groupingBy(Car::getBrand, groupingBy(Car::getColor));
```

- 셋 이상의 컴포넌트를 조합할 때는 보통 **플루언트 형식**이 **중첩 형식**에 비해 가독성이 좋다.
- 위 Collector에서 가장 안쪽의 Collector가 첫 번째로 평가되어야 하지만 논리적으로는 최종 그룹화에 해당하는 것을 보면 플루언트 형식과 중첩 형식간의 **처리 방식 차이**가 보인다.

**✅ 해결책(?): GroupingBuilder 빌더 도입**

```java
import static java.util.stream.Collectors.groupingBy;

public class GroupingBuilder<T, D, K> {
    private final Collector<? super T, ?, Map<K, D>> collector;

    private GroupingBuilder(Collector<? super T, ?, Map<K, D>> collector) {
        this.collector = collector;
    }

    public Collector<? super T, ?, Map<K, D>> get() {
        return collector;
    }

    public <J> GroupingBuilder<T, Map<K, D>, J> after(Function<? super T, ? extends J> classifier) {
        return new GroupingBuilder<>(groupingBy(classifier, collector));
    }

    public static <T, D, K> GroupingBuilder<T, List<T>, K> groupOn(Function<? super T, ? extends K> classifier) {
        return new GroupingBuilder<>(groupingBy(classifier));
    }
}
```

```java
Collector<? super Car, ?, Map<Brand, Map<Color, List<Car>>>> 
		carGroupingCollector =
		    groupOn(Car::getColor)
		        .after(Car::getBrand)
		        .get();
```

- **groupOn → after → get()** 순서로 **플루언트 방식** 사용
- **GroupingBuilder**는 내부적으로 groupingBy를 조합해 Collector를 생성

🧐 과연 **중첩 형식 → 플루언트 형식**이 해결책일까?

GroupingBuilder를 사용하는 예제를 보면 중첩된 그룹화 수준과 그룹화 함수를 반대로 구현해야 하는 것을 알 수 있다. 이는 **중첩 형식**을 사용할 때와 비교했을 때 직관력은 떨어진다.

**💡 결론**

- 중첩 형식은 가독성이 떨어지고 순서 파악이 어렵다. 그렇다고 플루언트 형식으로 이를 바꾼다고 모두 해결되는가?
- 아니다. 플루언트 형식이 문법적으로는 가독성이 좋아질지 몰라도 그룹화 순서의 차이가 반대로 해석되기 때문에 직관적이지 않다.
- 자바 형식 시스템으로는 이러한 순서 문제를 명확하게 표현하는데 한계가 있다는 것을 보여준다.
