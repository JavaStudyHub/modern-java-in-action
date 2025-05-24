# Chapter 11. null 대신 Optional 클래스

Java에는 null 이라는 개념이 존재한다. “값이 없음” 또는 “정의되지 않음” 을 의미하는데, 그로 인해 NullPointerException 이라는, 모든 자바 개발자를 괴롭히는 예외가 생겨나버렸다.

우선 null 때문에 발생하는 문제들은 어떤 것들이 있는지 알아보자.

## 11.1 값이 없는 상황을 어떻게 처리할 수 있을까?

- 자동차와 자동차 보험을 갖고 있는 사람 객체를 중첩 구조로 구현한 형태
    
    ```java
    public class Person {
    		private Car car;
    		public Car getCar() {
    				return car;
    		}
    }
    
    public class Car {
    		private Insurance insurance;
    		public Insurance getInsurance() {
    				return insurance;
    		}
    }
    
    public class Insurance {
    		private String name;
    		public String getName() {
    				return name;
    		}
    }
    ```
    

이 때 다음의 코드에서는 어떤 문제가 발생할까?

```java
person.getCar().getInsurance().getName();
```

모든 사람이 차를 소유하는가? 그렇지 않다. 차를 소유하지 않은 사람도 매우 많다. 

그런 경우에 getCar 를 호출하면 어떤 일이 벌어질까?

대부분의 프로그래머는 null 참조를 반환하는 방식으로 자동차를 소유하고 있지 않음을 표현할 것이다.

그럼 이후의 코드들은

```java
null.getInsurance().getName();
```

과 같은 형태로 동작하게 된다. 즉 런타임에 NullPointerException 이 발생하게 되는 것이다.

<br>

### 11.1.1 보수적인 자세로 NullPointerException 줄이기

1. **null safe 시도 1 : 깊은 의심**

```java
public String getCarInsuranceName(Person person) {
	if(person != null) {
		Car car = person.getCar();
		if(car != null) {
			Insurance insurance = car.getInsurance();
			if(insurance != null) {
				return insurance.getName();
			}
		}
	}
	return "Unknown";
}
```

위 코드는 변수를 참조할 때마다 null 을 확인하며, 중간 과정에서 하나라도 null 참조가 있다면 “Unknown” 이라는 문자열을 반환한다. 

이와 같은 반복 패턴의 코드를 **`“깊은 의심”`** 이라고 부른다.

다만 위 코드의 경우 중첩된 if가 추가되면서 코드 들여쓰기 수준이 증가하게 되고, 그에 따라 구조가 엉망이 되며 가독성이 떨어지는 문제가 생긴다.

1. **null safe 시도 2 : 너무 많은 출구**

```java
public String getCarInsuranceName(Person person) {
	if(person == null) {
		return "Unknown";
	}
	Car car = person.getCar();
	if(car == null) {
    	return "Unknown";
	}
	Insurance insurance = car.getInsurance();
	if(insurance == null) {
		return "Unknown";
	}
	return insurance.getName();
}
```

위 코드는 `“Early Return”` 을 활용하여 중첩 if 블록을 없앤다.

다만 메서드에 너무 많은 출구가 생겨난다. 그러다보니 유지보수가 어려워진다.

애시당초 null 을 활용하여 값이 없다는 사실을 표현하는 것은 좋은 방법이 아니다. 따라서 값이 있거나 없음을 표현할 수 있는 좋은 방법이 필요하다.

<br>

### 11.1.2 null 때문에 발생하는 문제

1. **에러의 근원** : NullPointerException 은 자바에서 가장 흔히 발생하는 에러이다.
2. **코드를 어지럽힌다**. : 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 코드 가독성이 떨어진다.
3. **아무 의미가 없다.** : null 은 아무 의미도 표현하지 않는다. 특히 정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.
4. **자바 철학에 위배된다.** : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 예외가 있는데 그것이 바로 null 포인터이다.
5. **형식 시스템에 구멍을 만든다.** : null 은 무형식이며 정보를 포함하고 있지 않으므로 모든 참조 형식에 null 을 할당할 수 있다. 이런 식으로 null 이 할당되기 시작하면서 시스템의 다른 부분으로 null 이 퍼졌을 때 애초에 null 이 어떤 의미로 사용되었는 지 알 수 없다.

<br>

### 11.1.3 다른 언어는 null 대신 무얼 사용하나?

- **그루비 : 안전 내비게이션 연산자 사용 - ?.**
    
    ```java
    def carInsuranceName = person?.car?.insurance?.name
    ```
    
- **하스켈 : 선택형값을 저장하는 Maybe 형식 사용**
    - Maybe 는 주어진 형식의 값을 갖거나 아니면 아무 값도 갖지 않을 수 있다.
- **스칼라 : 선택형값을 저장하는 Option[T] 구조 사용**
    - Option[T]는 T 형식의 값을 갖거나 아무 값도 갖지 않을 수 있다.
    

자바 8에서는 이들 중 `“선택형값”` 개념의 영향을 받아서 **Optional<T>** 라는 새로운 클래스를 제공한다.

<br>

## 11.2 Optional 클래스 소개

**Optional은 선택형값을 캡슐화하는 클래스이다.**

<img width="453" alt="스크린샷 2025-05-24 오후 11 26 11" src="https://github.com/user-attachments/assets/e9313a1d-5021-4ac5-8edc-fafeffdddf20" />

Optional 클래스는 값이 있을 경우 그 값을 감싼다. 반면 값이 없을 경우 Optional.empty 메서드로 빈 Optional 을 반환한다.

Optional.empty 메서드는 Optional 의 특별한 싱글톤 인스턴스를 반환하는 정적 팩토리 메서드이다. 이 Optional.empty 메서드는 값이 없는 경우에도 Optional 객체를 반환하기 때문에 NullPointerException 을 예방할 수 있다.

- **Optional 로 Person/Car/Insurance 데이터 모델 재정의**
    
    ```java
    public class Person {
    	private Optional<Car> car;
    	public Optional<Car> getCar() {
    		return car;
    	}
    }
    
    public class Car {
    	private Optional<Insurance> insurance;
    	public Optional<Insurance> getInsurance() {
    		return insurance;
    	}
    }
    
    public class Insurance {
    	private String name; <- 보험회사에는 반드시 이름이 있음
    	public String getName() {
    		return name;
    	}
    }
    ```
    

이 코드를 통해, 사람은 자동차를 소유할 수도 있고 아닐 수도 있다는 사실을 표현할 수 있으며, 자동차는 보험에 가입되어 있을 수도 있고 아닐 수도 있음을 명확히 설명할 수 있다.

반면 보험회사의 경우 name 필드에 Optional 을 적용하지 않음으로써, 반드시 이름이 존재한다는 사실을 명확히한다. 이처럼 반드시 값을 갖는 필드에 Optional 을 적용하는 것은 오히려 고쳐야 할 문제를 감추는 꼴이 된다. 예외를 처리해주는 것이 아니라 보험회사의 이름이 없는 이유가 무엇인지 밝혀내야하는 상황이기 때문이다.

> [!IMPORTANT]
> 
> **모든 null 참조를 Optional 로 대치하는 것은 바람직하지 않다.**
> 
> **Optional의 역할은 더 이해하기 쉬운 API 를 설계하도록 돕는 것이다. 즉 메서드의 시그니처만 보고도 선택형값인지 여부를 구별할 수 있다.**

<br>

## 11.3 Optional 적용 패턴

실제로 Optional 을 어떻게 활용할 수 있는지 알아보자.

<br>

### 11.3.1 Optional 객체 만들기

- **빈 Optional**
    
    ```java
    Optional<Car> optCar = Optional.empty();
    ```
    
    → Optional.empty() 메서드로 빈 Optional 객체를 얻을 수 있다.
    

- **null 이 아닌 값으로 Optional 만들기**
    
    ```java
    Optional<Car> optCar = Optional.of(car);
    ```
    
    → Optional.of() 메서드는 null 이 아닌 값을 포함하는 Optional 을 만드는 메서드이다. 즉 위 코드의 경우 car 가 null 일 경우에는 NullPointerException 이 발생한다.
    

- **null 값으로 Optional 만들기**
    
    ```java
    Optional<Car> optCar = Optional.ofNullable(car);
    ```
    
    → Optional.ofNullable() 메서드는 car 가 null 값일 경우 빈 Optional 객체를 반환하는 방식으로 동작한다.
    

Optional에서 어떻게 값을 가져올 때에는 get 메서드를 활용할 수 있다. 하지만 이 방식은 Optional 이 비어있을 경우 NPE 가 발생한다. 

즉 Optional 을 잘못 사용하게되면 결국 null 을 사용했을 때와 같은 문제를 겪게된다.

따라서 Optional로 null 에 대한 명시적인 검사를 없애는 방식을 먼저 알아보자.

<br>

### 11.3.2 맵으로 Optional의 값을 추출하고 변환하기

값을 확인하기 전 null 인지 우선적으로 체크하고 객체의 정보를 추출할 때 어떻게 Optional 로 대체할 수 있는지 확인해보자.

- **기존 코드**
    
    ```java
    String name = null;
    if(insurance != null) {
    	name = insurance.getName();
    }
    ```
    
- **Optional 활용**
    
    ```java
    Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
    Optional<String> name = optInsurance.map(Insurance::getName);
    ```
    

Optional 의 map 메서드는 스트림의 map 메서드와 개념적으로 비슷하다 

스트림의 map은 스트림의 각 요소에 제공된 함수를 적용하는 연산이었던 것처럼, Optional 의 map 은 Optional 이 값을 포함하는 경우, 해당 값을 map 의 인수로 제공된 함수를 통해 가공하여 값을 반환한다. 만약 Optional이 비어있으면 아무 일도 일어나지 않는다.

<img width="537" alt="스크린샷 2025-05-24 오후 11 28 12" src="https://github.com/user-attachments/assets/3e2e483c-bc9b-4ce4-aa95-61d7d041037a" />

그럼 아래의 코드처럼 객체를 연결하며 메서드를 호출하는 코드는 어떻게 Optional로 구현할 수 있을까?

```java
public String getCarInsuranceName(Person person) {
	return person.getCar().getInsurance().getName();
}
```

이 때는 flatMap 을 사용하면 좋다.

<br>

### 11.3.3 flatMap으로 Optional 객체 연결

우선 앞서 배운 map 을 사용하는 방식은 다음과 같다.

```java
Optional<Person> optPerson = Optional.of(person);

Optional<String> name = 
	optPerson.map(Person::getCar)
			 .map(Car::getInsurance)
			 .map(Insurance::getName);
```

**하지만 위 코드는 컴파일되지 않는다!!!**

변수 optPerson 의 형식은 Optional<Person> 이므로 map 메서드를 호출할 수 있다.

하지만 getCar 는 Optional<Car> 형식의 객체를 반환한다. 

즉 map 연산의 결과가 **`Optional<Optional<Car>>`** 형식이 되어버린다.

<img width="302" alt="스크린샷 2025-05-24 오후 11 29 13" src="https://github.com/user-attachments/assets/3018df0e-5058-475e-838a-3ece93dc6501" />

그러다보니 이러한 형태의 중첩 Optional 객체 구조가 되어버린다.

이 문제를 해결하기 위해서는 2차원의 Optional을 1차원의 Optional 로 평면화 해야한다.

이 때 사용할 수 있는 것이 **`Optional 의 flatMap 메서드`** 이다. 스트림에서 보았던 flatMap 메서드가 Optional 에도 동일하게 존재하며, 마찬가지로 Optional 을 평면화 해주는 동작을 한다고 생각하면 된다.

<img width="546" alt="스크린샷 2025-05-24 오후 11 29 26" src="https://github.com/user-attachments/assets/0ed2b98f-4ead-48e5-b4e6-34759082dfbb" />

- **스트림에서의 정사각형**
    - 삼각형 2개를 감싸고 있는 스트림
- **Optional 에서의 정사각형**
    - 삼각형을 감싸고 있는 Optional

- **Optional 로 자동차의 보험회사 이름 찾기**
    
    ```java
    public String getCarInsuranceName(Optional<Person> person) {
    	return person.flatMap(Person::getCar)
    				       .flatMap(Car::getInsurance)
    				       .map(Insurance::getName)
    				       .orElse("Unknown"); <- 결과 Optional이 비어있으면 기본값 사용
    }
    ```
    

이처럼 Optional을 적절히 사용한다면 앞서 보았던 예제들처럼 null 을 확인하기 위해 조건 분기문을 추가함으로써 코드를 복잡하게 만들지 않으면서 동일한 동작을 하는 코드를 작성할 수 있다.

또한 Optional을 인수로 받거나, Optional을 반환하는 메서드를 정의한다면 결과적으로 이 메서드를 사용하는 모든 사람에게 이 메서드가 빈 값을 받거나 빈 결과를 반환할 수 있음을 잘 문서화해서 제공하는 것과 같다.

<br>

- **Optional을 이용한 Person/Car/Insurance 참조 체인**
    
    <img width="501" alt="스크린샷 2025-05-24 오후 11 30 14" src="https://github.com/user-attachments/assets/df6a82a8-930c-4df6-8eea-05b7479b02d1" />

    - Person 클래스의 getCar 메서드는 Optional< Car > 를 반환한다. 따라서 flatMap 을 통해 평면화를 시킴으로써 결과로 Optional<Optional< Car >> 가 아닌, Optional< Car > 를 생성한다.
    - Car 클래스의 getInsurance 메서드는 Optional< Insurance > 를 반환한다. 따라서 flatMap 을 통해 평면화 시킴으로써 결과로 Optional<Optional< Insurance >> 가 아닌 Optional< Insurance > 를 생성한다.
    - Insurance 클래스의 getName 메서드는 String 을 반환한다. 따라서 별도로 평면화를 할 필요가 없어 map 메서드를 사용하면 된다.
    - 3단계까지 마무리 되면 호출 체인의 결과로 Optional< String > 이 반환된다. Optional 이 비어있을 수도 있고 값이 저장되어있을 수도 있다. 이 때 Optional이 비어있으면 기본값을 제공하고, 비어있지 않으면 저장된 값을 반환하는 메서드로 `orElse 메서드`를 사용할 수 있다.

<br>

> [!IMPORTANT]
> **도메인 모델에 Optional을 사용했을 때 데이터를 직렬화할 수 없는 이유**
> 
> 자바에서는 Optional의 용도를 선택형 반환값을 지원하는 것이라고 명확하게 못박아둔다.
> 
> Optional 클래스는 필드 형식으로 사용하는 경우를 가정하지 않는데, 그렇기 때문에 Serializable 인터페이스를 구현하지 않는다.
> 따라서 도메인 모델에 Optional을 사용한다면 직렬화를 사용하는 도구나 프레임워크에서 문제가 생길 수 있다.
> 
> 그럼에도 불구하고 Optional을 사용해 도메인 모델을 구성하는 것은 null 을 예방하는 차원에서 바람직 하기 때문에 직렬화 모델이 필요하다면 다음과 같이 Optional 로 값을 반환받을 수 있는 메서드를 추가하는 방식이 권장된다.
> 
> ```java
> public class Person {
> 		private Car car;
> 		public Optional<Car> getCarAsOptional() {
> 			return Optional.ofNullable(car);
> 		}
> }
> ```

<br>

### 11.3.4 Optional 스트림 조작

자바 9에서는 Optional을 포함하는 스트림을 쉽게 처리할 수 있도록 Optional에 stream() 메서드를 추가했다. Optional 스트림을 값을 가진 스트림으로 변환할 때 이 기능이 유용하게 사용된다.

- **사람 목록을 이용해 가입한 보험 회사 이름 찾기**
    
    ```java
    public Set<String> getCarInsuranceNames(List<Person> persons) {
    	return persons.stream()
    				  .map(Person::getCar)
    				  .map(optCar -> optCar.flatMap(Car::getInsurance))
              .map(optIns -> optIns.map(Insurance::getName))
              .flatMap(Optional::stream)
              .collect(toSet());
    ```
    
    - **flatMap(Optional::stream)** 을 통해 **Stream<Optional< String >>** **→ Stream<Stream< String >> →** **Stream< String >** 으로 변환
    
    이 처럼 Insurance::getName 까지의 단계를 거치고 나면 결과로 Stream<Optional< String >> 을 얻게되는데, 이 때는 사람이 차를 갖고 있지 않거나, 차가 보험에 가입되어있지 않아 결과가 비어있을 수 있다. 따라서 최종 결과를 얻기 위해서는 빈 Optional 을 제거하고 값을 언랩해야한다.
    
    만약 Optional.stream() 을 사용하지 않는 경우에는 다음처럼 filter, map 을 순차적으로 이용하면 된다.
    
    ```java
    Stream<Optional<String>> stream = ...
    Set<String> result = stream.filter(Optional::isPresent)
                               .map(Optional::get)
                               .collect(toSet());
    ```
    
    이 때는 isPresent 를 통해 비어있지 않은, 값이 저장되어있는 Optional 요소들만 추출하고, 그 Optional 을 unwrap 함으로써 값을 얻어낼 수 있다. 
    
    하지만 앞서 봤듯이 Optional 클래스의 stream() 메서드를 이용하면 한 번의 연산으로 같은 결과를 얻어낼 수 있다.
    
    Optional 의 stream() 메서드는 각 Optional 이 비어있는지 아닌지에 따라 Optional 을 0개 이상의 항목을 포함하는 스트림으로 변환한다. 즉 비어있는 Optional 의 경우 포함시키지 않고 건너뛰는 동작까지 한 번에 수행해주는 것이다.
    
<br>

### 11.3.5 디폴트 액션과 Optional 언랩

Optional 클래스에는 orElse 를 통해, 빈 Optional 일 경우 기본값을 반환하도록 하면서 Optional 을 읽을 수 있었다.

이 외에는 어떤 방식으로 Optional 을 읽을 수 있을까?

1. **get()**
    - 값을 읽는 가장 간단한 메서드이면서 동시에 가장 안전하지 않은 메서드
    - Optional 이 비어있지 않다면 저장된 값을 반환한다.
    - Optional 이 비어있다면 NoSuchElementException 을 발생시킨다.
    - 사실상 중첩된 null 확인 코드를 넣는 상황과 크게 다르지 않다.
    
2. **orElse(T other)**
    - Optional 이 비어있는 경우 기본값을 제공한다.
    
3. **orElseGet(Supplier<? extends T> other)**
    - orElse 메서드의 Lazy 버전
    - Optional 이 비어있는 경우 Supplier가 실행된다.

> [!NOTE]
> **orElse vs orElseGet**
> - **orElse(T other)**
    - 기본값 계산 : 무조건 실행
    - Optional 이 비어있든 비어있지 않든 other 에 해당하는 값을 미리 계산해둔다.
> - **orElseGet(Supplier<? extends T> supplier)**
    - 기본값 계산 : Optional 이 비어있을 때만 실행
    - Optional 이 비어있는 경우에만 기본값을 계산. 즉 지연 실행(Lazy execution)
> ```java
> public String getDefault() {
>    System.out.println("getDefault() 호출됨");
>    return "default";
>}
> 
> Optional<String> opt = Optional.of("actual");
> String result = opt.orElse(getDefault());
> // 출력: getDefault() 호출됨
> 
> String result = opt.orElseGet(() -> getDefault());
> // 출력 없음
> ```
> → 따라서 기본값 생성에 큰 비용이 든다면 orElseGet() 을 사용하는 것이 좋다.

<br>

1. **orElseThrow(Supplier<? extends X> exceptionSupplier)**
    - Optional 이 비어있을 때 예외를 발생시킨다. get() 과 유사한 듯 보이지만 그와 다르게 발생시킬 예외의 종류를 선택할 수 있다.

2. **ifPresent(Consumer<? super T> consumer)**
    - Optional 이 비어있지 않으면 인수로 넘겨준 동작을 실행할 수 있다.
    - Optional 이 비어있다면 아무 일도 일어나지 않는다.

3. **ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) → 자바 9 추가**
    - Optional 이 비어있다면 인수로 넘겨준 Runnable 을 실행한다.

<br>

### 11.3.6 두 Optional 합치기

**가장 저렴한 보험료를 제공하는 보험회사 찾기**

- **Optional 활용 X**
    
    ```java
    public Insurance findCheapestInsurance(Person person, Car car) {
    	// 다양한 보험회사가 제공하는 서비스 조회
    	//모든 결과 데이터 비교
    	return cheapestCompany;
    }
    ```
    
- **Optional 활용 O**
    - 만약 인수로 전달한 값 중 하나라도 빈 Optional 이라면 빈 Optional<Insurance> 를 반환
    
    ```java
    public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
    	if(person.isPresent() && car.isPresent()) {
    		return Optional.of(findCheapestInsurance(person.get(), car.get()));
    	} else{
    		return Optional.empty();
    	}
    }
    ```
    

이러한 Optional 활용 메서드의 장점은, person 과 car 의 시그니처 만으로 두 인수 모두 아무 값도 반환하지 않을 수 있다는 정보를 명시적으로 보여주었다는 점이다. 

다만 위 코드는 사실 조건문을 통해 null 을 체크하는 코드와 크게 다를 바 없는 코드이긴 하다.

<br>

- **Optional 을 unwrap 하지 않고 두 Optional 합치기**

```java
public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
	return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```

<br>

### 11.3.7 필터로 특정값 거르기

**보험회사 이름이 CambridgeInsurance 인지 확인해야하는 상황 가정**

- **Optional 활용 X**
    
    ```java
    Insurance insurance = ...;
    
    if(insurance != null && "CambridgeInsurance".equals(insurance.getName())) {
    	System.out.println("ok");
    }
    ```
    
    - 이처럼 Optional 을 활용하지 않을 경우 Insurance 객체가 null 인지 여부를 확인한 후에 getName 메서드를 호출해야한다.

- **Optional 활용 O**
    - **filter 메서드 활용**
    
    ```java
    Optional<Insurance> optInsurance = ...;
    optInsurance.filter(insurance -> "CambridgeInsurance".equals(insurance.getName()))
    						.ifPresent(x -> System.out.println("ok"));
    ```
    
    - filter 메서드는 프레디케이트를 인수로 받는다. Optional 객체가 값을 가지면서 프레디케이트와 일치하면 filter 메서드는 그 값을 반환하고, 그렇지 않으면 빈 Optional 객체를 반환한다.
    - Optional 이 비어있다면 filter 연산은 아무런 동작도 하지 않는다.
    - Optional 이 비어있지 않아서 filter 연산을 적용했는데, 만약 프레디케이트 적용 결과가 true 라면 Optional 에는 아무 변화도 일어나지 않지만, 결과가 false 라면 값을 없애버리고 Optional 은 빈 상태가 된다.

- **Optional 클래스의 메서드**
    <img width="492" alt="스크린샷 2025-05-24 오후 11 30 49" src="https://github.com/user-attachments/assets/e08ca5a9-6b40-42b3-81fe-3dba5cd9f1c9" />
    

<br>

## 11.4 Optional을 사용한 실용 예제

### 11.4.1 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

자바 API에서는 요청한 값이 없거나 어떤 문제로 계산에 실패했음을 알리기 위해 null 을 반환한다.

예를 들어 Map 의 get() 메서드가 존재하는데, 이러한 메서드들의 시그니처를 우리가 직접 바꿀 수는 없다. 대신 get 메서드의 반환값을 Optional 로 감싸는 방식을 사용할 수 있다.

```java
Optional<Object> value = Optional.ofNullable(map.get("key"));;
```

이처럼 null이 될 수 있는 값을 Optional 로 안전하게 변환할 수 있다.

<br>

### 11.4.2 예외와 Optional 클래스

자바 API 에서는 null 을 반환하는 대신 예외를 발생시킬 때도 있다.

문자열을 정수로 바꾸는 정적 메서드 Integer.parseInt(String) 이 그 전형적인 예시이다. 이 메서드는 문자열을 정수로 바꾸지 못할 때 NumberFormatException을 발생시킨다.

이 때는 null 여부 체크가 아니라 try/catch 블록을 사용해야한다.

만약 동일하게 문자열을 정수로 변환하는 동작을 구현하면서 예외가 아닌 null 을 발생시키고 싶은데, 동시에 null safe 하게 구현하고싶다면 parseInt 메서드를 감싸는 작은 유틸리티 메서드를 구현해볼 수 있다.

```java
public static Optional<Integer> stringToInt(String s) {
	try {
		return Optional.of(Integer.parseInt(s));
	} catch (NumberFormatException e) {
		return Optional.empty();
	}
}
```

이와 같은 메서드를 포함하는, OptionalUtility 와 같은 유틸리티 클래스를 만들어 사용한다면 거추장스럽게 try/catch 문을 사용할 필요도 사라진다.

<br>

### 11.4.3 기본형 Optional을 사용하지 말아야 하는 이유

Optional 도 기본형으로 특화된 OptionalInt, OptionalLong, OptionalDouble 등의 클래스를 제공한다.

예를 들어 Optional<Integer> → OptionalInt 로 대신해서 사용할 수 있다.

스트림의 경우 많은 요소를 가질 경우 기본형 특화 스트림을 이용해서 언박싱 비용을 줄임으로써 성능을 향상시킬 수 있다고 했었다. 

하지만 Optional이 가질 수 있는 최대 요소의 수는 한 개이다. 즉 기본형 특화 클래스를 통해 성능을 개선할 수 없다.

심지어 기본형 특화 Optional 은 map, flatMap, filter 등을 지원하지 않는다. 따라서 기본형 특화 Optional 을 사용하는 것은 권장되지 않는다.

<br>

### 11.4.4 응용

- **프로그램의 설정 인수로 Properties 전달 및 테스트**
    
    ```java
    Properties props = new Properties();
    props.setProperty("a", "5");
    props.setProperty("b", "true");
    props.setProperty("c", "-3");
    
    // 프로그램으로 전달
    public int readDuration(Properties props, String name) {
    	String value = props.getProperty(name);
    	if(value != null) {
    	    try {
    			int i = Integer.parseInt(value);
    			if(i > 0) {
    				return i;
    			}
    		} catch (NumberFormatException nfe) {}
    	}
    	return 0; -> 하나의 조건이라도 실패하면 0 을 반환
    }
    ```
    
    - Properties 의 값을 읽어서 각 값을 초 단위의 지속 시간으로 해석.
    - 즉 값은 양의 정수여야 하므로, 문자열이 양의 정수를 가리키면 해당 정수를 반환하고 그렇지 않다면 0을 반환하도록 한다.

위 코드는 Optional 을 사용하지 않았다. 따라서 value 값에 대한 null 체크를 하기 위해 조건문이 존재하고, Integer.parseInt 가 발생시키는 NumberFormatException 에 대한 처리를 위한 try/catch 문이 중첩되어있는 것을 볼 수 있다.

그러다보니 코드의 가독성이 나빠진 것을 볼 수 있다. 이 코드를 Optional을 활용한 코드로 바꿔보자.

- **Optional 활용 코드**
    
    ```java
    public int readDuration(Properties props, String name) {
    	return Optional.ofNullable(props.getProperty(name))
    							.flatMap(OptionalUtility::stringToInt)
    							.filter(i -> i > 0)
    							.orElse(0);
    ```
    
    1. props.getProperty(name) 메서드가 null 을 반환할 수 있으므로 ofNullable 메서드를 통해 Optional 로 감싼다.
    2. Integer.parseInt() 메서드는 NumberFormatException 을 발생시킬 수 있기 때문에 해당 예외를 내부적으로 처리하는 OptionalUtility.stringToInt() 메서드를 사용한다. 이 때 stringToInt() 메서드는 Optional<Integer> 를 반환하게 되므로 flatMap 을 통해 해당 값에 Optional 을 추가적으로 씌우지 않고 그대로 사용한다.
    3. filter 메서드를 통해 음수값들을 필터링해준다.
    4. 만약 빈 Optional 이 반환될 경우 orElse 메서드에 의해 기본값인 0이 반환된다.
    
<br>

## 11.5 마치며

- 역사적으로 프로그래밍 언어에서는 **null** 참조로 값이 없는 상황을 표현해왔다
- 자바 8 에서는 값이 있거나 없음을 표현할 수 있는 클래스 **java.util.Optional**〈**T**〉를 제공한다
- 팩토리 메서드 **Optional.empty, Optional.of, Optional.ofNullable** 등을 이용해서 **Optional** 객체를 만들 수 있다
- **Optional** 클래스는 스트림과 비슷한 연산을 수행하는 **map, flatMap, filter** 등의 메서드를 제공한다
- **Optional**로 값이 없는 상황을 적절하게 처리하도록 강제할 수 있다. 즉,  **Optional**로 예상치 못한 **null** 예외를 방지할 수 있다.
- **Optional**을 활용하면 더 *좋은* **API**를 설계할 수 있다. 즉, 사용자는 메서드의 시그니처만 보고도 **Optional**값이 사용되거나 반환되는지 예측할 수 있다.
