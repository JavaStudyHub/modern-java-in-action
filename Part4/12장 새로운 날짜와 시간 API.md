# 12장 새로운 날짜와 시간 API

자바8 이전까지 자바에서 제공하는 날짜와 시간 관련 기능은 사용하기 불편하고 설계가 완벽하지 못했는데, 자바8에 들어서는 날짜와 시간과 관련해서 새로운 API가 추가되었다. 

우선 자바8 이전에는 어떤 불편함이 있었기에 새로운 API가 추가되었는지 알아보자. 

자바 1.0에서는 날짜와 시간 관련 기능을 **java.util.Date**가 제공했는데 모호한 설계로 인해서 유용하지 않았다.

- Date 클래스는 특정 시점을 날짜가 아닌 밀리초로 나타낸다.
- 또한 연도는 1900년으로부터의 오프셋으로 나타내고, 월 인덱스는 0부터 시작했다.
- 다음은 2017년 9월 21일을 나타내는 Date클래스를 만드는 코드인데, 코드를 보면 알 수 있듯이 Date는 직관적이지 않다.

```java
Date date = new Date(117, 8, 21);

// Thu Sep 21 00:00:00 CET 2017이 출력된다.
```

- 또한 Date 클래스의 toString으로 반환되는 문자열은 오해를 불러일으킨다. 위의 출력 결과를 보면 JVM의 기본시간대인 CET를 사용하는데 사실은 Date는 시간대 정보를 가지고 있지 않다.
- 자바 1.0의 Date 클래스는 문제가 있었지만 한동안 해결할 방법이 없었다.

그러다 자바 1.1에서는 Date의 많은 메소드를 deprecate시키고 **Calendar** 클래스를 도입했다. 그러나 여전히 설계 문제가 존재했다. 

- 달 인덱스는 0이라는 점은 그대로였다.
- DateFormat 같은 일부 기능은 Date에서만 지원되었다. 또한 DateFormat은 thread safe하지 않다는 문제를 지녔다.
- 또한 Date와 Calendar는 가변클래스이기 때문에 유지보수가 어려울 수 있다는 점도 문제였다.

그래서 많은 자바 개발자들은 Joda-Time 같은 서드파티 라이브러리를 사용했다. 결국 오라클은 더 나은 API를 제공하기 위해서 자바8에 Joda-Time의 많은 기능을 **java.time** 패키지에 추가했다.

이제 새로운 날짜와 시간 API가 추가된 배경을 알아봤으니 어떤 것이 추가되었는지 알아보자

# **12.1 LocalDate, LocalTime, Instant, Duration, Period** 클래스

`java.time` 패키지에서는 LocalDate, LocalTime, Instant, Duration, Period등 새로운 클래스를 제공한다.

## 1**2.1.1 LocalDate**와 **LocalTime** 사용

**LocalDate** 

LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체다. 

- 어떤 시간대 정보도 포함하지 않는다.
- `of`  메소드를 이용하면 인스턴스를 만들 수 있다.
- 다음과 같은 메소드를 가지고 있다.

```java
LocalDate date = LocalDate.of(2017, 9, 21); // 2017년 9월 21일
int year = date.getYear();                   // 2017 
Month month = date.getMonth();               // SEPTEMBER 
int day = date.getDayOfMonth();              // 21
DayOfWeek dow = date.getDayOfWeek();         // THURSDAY
int len = date.lengthOfMonth();              // 31
boolean leap = date.isLeapYear()；            // false (윤년 아님)
```

- `now` 팩토리 메소드는 시스템의 시계 정보를 이용해서 현재 날짜 정보를 얻는다. 다른 날짜와 시간 관련 클래스도 이와 비슷한 방법을 제공한다.

```java
LocalDate today = LocalDate.now();
```

- `get` 메소드에 `TemporalField`를 인자로 전달해서 날짜 정보를 얻는 방법도 있다.
    - `TemporalField` 는 시간 관련 객체에서 어떤 정보에 접근할지 결정하는 인터페이스이다. 열거자 `ChronoField`가 이 인터페이스를 정의하므로 이걸 이용하면 된다.
    
    ```java
    int year = date.get(ChronoField.YEAR);
    int month = date.get(ChronoField.MONTH_OF_YEAR);
    int day = date.get(ChronoField.DAY_OF_MONTH)；
    
    // 위의 코드와 동일한 기능을 수행한다.
    int year = date.getYear();
    int month = date.getMonthValue();
    int day = date.getDayOfMonth();
    ```
    

**LocalTime**

**13:45:20** 같은 시간을 표현한다.

- `of` 메소드를 통해서 인스턴스를 생성할 수 있는데 두 가지 오버로드 버전이 존재한다.
    - of(시간, 분)
    - of(시간, 분, 초)
- 다음과 같은 getter 메소드를 제공한다.
    
    ```java
    LocalTime time = LocalTime.of(13, 45, 20); // 13:45:20
    int hour = time.getHour();      // 13
    int minute = time.getMinute();  // 45
    int second = time.getSecond()；  // 20
    ```
    

**문자열로부터 LocalDate와 LocalTime 생성하기**

`parse` 메소드를 이용하면 시간 날짜 관련 문자열을 LocalDate나 LocalTime으로 바꿀 수 있다.

```java
LocalDate date = LocalDate.parse("2017-09-21");
LocalTime time = LocalTime.parse("13:45:20");
```

- parse 메소드에는 `DateTimeFormatter`를 전달할 수도 있다.
    - DateTimeFormatter는 날짜 객체나 시간 객체의 형식을 지정한다.
    - 자세한 사용법은 12.2.2절 참고
- 문자열을 LocalDate나 LocalTime으로 파싱할 수 없을 때 parse 메서드는 `DateTimeParseException`을 일으킨다

## **12.1.2** 날짜와 시간 조합

**LocalDateTime**

**LocalDate**와 **LocalTime**을 쌍으로 갖는 복합 클래스로 날짜와 시간을 모두 표현할 수 있다.

- `of` 메소드로 인스턴스를 만들 수 있다.
    - 날짜와 시간 정보를 나열하거나 날짜 객체와 시간 객체를 전달할 수 있다.

```java
// 2017-09-21T13:45:20
LocalDateTime dt1 = LocalDateTime.of(2017, Month.SEPTEMBER, 21, 13, 45, 20);
LocalDateTime dt2 = LocalDateTime.of(date, time);
```

- LocalDate의 `atTime` 메서드에 시간을 제공하거나 LocalTime의 `atDate` 메서드에 날짜를 제공해서 LocalDateTime을 만드는 방법도 있다.

```java
LocalDateTime dt3 = date.atTime(13, 45, 20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date)；
```

- `toLocalDate`나 `toLocalTime` 메서드로 LocalDate나 LocalTime 인스턴스를 추출할 수 있다

```java
LocalDate date = dt1.toLocalDate();
LocalTime time = dt1.toLocalTime();
```

## **12.1.3 Instant** 클래스**：**기계의 날짜와 시간

**Instant**

java.time.Instant ****클래스에서는 기계적인 관점에서 시간을 표현한다. 즉 유닉스 에포크 시간을 기준으로 특정 저점까지의 시간을 초로 나타낸다.

- **`ofEpochSecond`** 팩토리 메서드 에 초를 넘겨줘서 인스턴스를 만들 수 있다.
    - 오버로드된 **ofEpochSecond** 메서드 버전에서는 두 번째 인수를 이용해서 나노초 단위로 시간을 보정할 수 있다. (두 번째 인수는 0 ~ 999,999,999 사이의 값)
    
    ```java
    // 모두 같은 시간을 나타내고 있다!
    Instant.ofEpochSecond(3);
    Instant.ofEpochSecond(3, 0);
    Instant.ofEpochSecond(2, 1_000_000_000); // 2초 이후의 1억나노초
    Instant.ofEpochSecond(4, -1_000_000_000);
    ```
    
- 정적 팩토리 메서드 `now`는 현재 시간을 알려준다.
- Instant는 사람이 읽을 수 있는 정보를 제공하지 않는다. 만약 다음과 같이 코드를 작성한다면 `UnsupportedTemporalTypeException`이 발생한다.
    
    ```java
    int day = Instant.now().get(ChronoField.DAY_0F_M0NTH);
    ```
    

## **12.1.4 Duration**과 **Period** 정의

지금까지 살펴본 클래스는 Temporal 인터페이스를 구현한다. 이 인터페이스는 시간을 모델링하는 객체에서 어떻게 값을 읽고 조작하는지를 정의한다.

**Duration**

이번에는 두 시간 객체 사이의 지속 시간인 duration을 만들어보자. 

- Duration클래스의 정적 팩토리 between을 이용하면 두 객체 사이의 duration을 만들 수 있다.
    - 다음과 같이 LocalTime, LocalDateTime, Instant를 이용하여서 만들 수 있다.

```java
Duration d1 = Duration.between(time1, time2); 
Duration d2 = Duration.between(dateTime1, dateTime2); 
Duration d3 = Duration.between(instant1, instant2); 
```

- LocalDateTime과 Instant는 혼합해서 사용할 수 없다.
- 또한 LocalDateTime으로 Duration을 만들 수 없다.

**Period**

시간의 양을 년, 월, 일을 나타낼 때는 Period 클래스를 이용한다.

- 팩토리 메서드 **between**을 이용하면 두 **LocalDate**의 차이를 확인할 수 있다

```java
Period tenDays = Period.between(LocalDate.of(2017, 9, 11 ),
LocalDate.of(2017, 9, 21));
```

**Duration과 Period의 다양한 생성 방법**

Duration과 Period 클래스는 자신의 인스턴스를 만들 수 있도록 다양한 팩토리 메서드를 제공한다

```java
Duration threeMinutes = Duration.ofMinutes(3);
Duration threeMinutes = Duration.of(3, ChronoUnit.MINUTES);

Period tenDays = Period.ofDays(10);
Period threeWeeks = Period.ofWeeks(3);
Period twoYearsSixMonthsOneDay = Period.of(2, 6, 1)；
```

![image.png](12%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%201f52dcb5a74a80bca059f19b8a22dcec/image.png)

# **12-2** 날짜 조정,파싱,포매팅

**`withAttribute`**메서드로 기존의 LocalDate를 수정한 버전을 만들 수 있다

- 기존 객체는 바꾸지 않고 속성을 바꾼 새로운 객체가 생성된다.

```java
LocalDate datel = LocalDate.of(2017, 9, 21); 
LocalDate date2 = datel .withYear(2011); 
LocalDate date3 = date2.withDay0fMonth(25); 
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR, 2)； 
```

 `with` 메서드는 첫번째 인수로 **TemporalField**를 갖는 메서드로, 이를 사용하면 좀 더 범용적으로 메서드를 활용할 수 있다

`Temporal` 인터페이스는 with 메서드와 get 메서드를 정의한다.

- **get**과 **with** 메서드로 **Temporal** 객체의 필드값을 읽거나 고칠 수 있다.
- 만약 해당 시간 객체가 필드를 지원하지 않으면 **`UnsupportedTemporalTypeException`이 발생한다.**

선언형으로 LocalDate를 사용하는 방법도 있다. 

- **`plus`, `minus`**메서드도 **Temporal** 인터페이스에 정의되어 있다. 메서드의 인수에 숫자와 **`TemporalUnit`**을 활용할 수 있다. **TemporalUnit의 구현체로 `ChronoUnit`** 열거형을 이용하면 된다.

```java
LocalDate datel = LocalDate.of(2017, 9, 21); // 2017-09-21
LocalDate date2 = datel .plusWeeks(1);       // 2017-09-28
LocalDate date3 = date2.minusYears(6);       // 2011-09-28
LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS)； // 2012-03-28
```

날짜와 시간을 표현하는 모든 클래스는 서로 비슷한 메서드를 제공한다

![image.png](12%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%201f52dcb5a74a80bca059f19b8a22dcec/image%201.png)

## **12.2.1 TemporalAdjusters** 사용하기

**TemporalAdjuster** 

때로는 다음 주 일요일, 돌아오는 평일, 어떤 달의 마지막 날 등 좀 더 복잡한 날짜 조정 기능이 필요하다. 이때는 오버로드된 버전의 **`with`** 메서드에 **`TemporalAdjuster`** 를 전달하는 방법으로 문제를 해결할 수 있다

- TemporalAdjuster는 ****좀 더 다양한 동작을 수행할 수 있도록 하는 기능을 제공한다.
- **`TemporalAdjusters`**에서 정의하는 정적 팩토리 메서드로 이를 생성할 수 있다

```java
import static java.time.temporal.TemporalAdjusters.*;

LocalDate datel = LocalDate.of(2014, 3, 18); // 2014-03-18
LocalDate date2 = datel.with(nextOrSame(DayOfWeek.SUNDAY)); // 이 이후 가까운 일요일은 2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth())；   // 이달의 마지막 날은 2014-03-31
```

- 다양한 **TemporalAdjusters**의 팩토리 메서드로 만들 수 있는 **TemporalAdjuster를 나열하면 다음과 같다.**

![image.png](12%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%201f52dcb5a74a80bca059f19b8a22dcec/image%202.png)

필요한 기능이 정의되어 있지 않을 때는 커스텀 **TemporalAdjuster** 구현을 만들 수 있다.

- **TemporalAdjuster** 인터페이스는 다음처럼 하나의 메서드만 정의한다.

```java
@FunctionalInterface
public interface TemporalAdjuster {
 Temporal adjustInto(Temporal temporal);
}
```

- **TemporalAdjuster** 인터페이스는 Temporal 객체를 다른 Temporal 객체로 변환하는 UnaryOperator<Temporal> 형식으로 취급할 수 있다.

## 1**2.2.2** 날짜와 시간 객체 출력과 파싱

포매팅과 파싱 전용 패키지인 **java.time.format**이 새로 추가되었다

**DateTimeFormatter**

이 패키지에서 **DateTimeFormatter는** 정적 팩토리 메서드와 상수를 이용해서 손쉽게 포매터를 만들 수 있게 해준다.

- **BASIC_ISO_DATE**와 **ISO__LOCAL_DATE** 등의 상수를 미리 정의하고있다
- `format` 메소드와 `DateTimeFormatter`를 이용해서 ****날짜나 시간을 특정 형식의 문자열로 만들 수 있다
    
    ```java
    LocalDate date = LocalDate.of(2014, 3, 18);
    String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE); //  20140318
    String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE); //  2014-03-18
    ```
    
- 날짜나 시간을 표현하는 문자열을 파싱해서 날짜 객체를 다시 만들 수 있다. 특정 시점이나 간격을 표현하는 모든 클래스의 팩토리 메서드 `parse`를 이용해서 문자열을 날짜 객체로 만들 수 있다.
    
    ```java
    LocalDate datel = LocalDate.parse("20140318",
    DateTimeFormatter.BASIC_ISO_DATE); 
    
    LocalDate date2 = LocalDate.parse("2014-03-18",
    DateTimeFormatter.ISO_LOCAL_DATE);
    ```
    
- 기존의 **java.util.DateFormat** 클래스와 달리 모든 **DateTimeFormatter**는 스레드에서 안전하게 사용할 수 있는 클래스다.
- 또한  **DateTimeFormatter** 클래스는 특정 패턴으로 포매터를 만들 수 있는 정적 팩토리 메서드 `ofPattern`도 제공한다
    
    ```java
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
    LocalDate datel = LocalDate.of(2014, 3, 18);
    String formattedDate = datel.format(formatter);
    LocalDate date2 = LocalDate.parse(formattedDate, formatter)；
    ```
    
    - **ofPattern** 메서드도 **Locale**로 포매터를 만들 수 있도록 오버로드된 메서드를 제공한다
    
    ```java
    DateTimeFormatter italianFormatter =
    DateTimeFormatter.ofPattern("d. MMMM yyyy"z Locale.ITALIAN);
    LocalDate datel = LocalDate.of(2014, 3, 18);
    String formattedDate = date.format(italianFormatter); // 18. marzo 2014
    LocalDate date2 = LocalDate.parse(formattedDate, italianFormatter)；
    ```
    
    > [!TIP] **Locale formatter란?**
    > **Locale formatter**는 **지역(locale)** 에 따라 숫자, 날짜, 시간, 통화 등을 **해당 문화권의 형식에 맞게 포맷팅하거나 파싱**할 수 있도록 도와주는 기능입니다. **각 나라나 지역의 표기 방식에 맞춰 출력**할 수 있게 해줍니다.
    
    ```java
    import java.time.LocalDate;
    import java.time.format.DateTimeFormatter;
    import java.util.Locale;
    
    public class DateLocaleExample {
        public static void main(String[] args) {
            LocalDate date = LocalDate.of(2025, 5, 16);
    
            DateTimeFormatter formatterUS = DateTimeFormatter.ofPattern("MMMM d, yyyy", Locale.US);
            DateTimeFormatter formatterKR = DateTimeFormatter.ofPattern("yyyy년 M월 d일", Locale.KOREA);
    
            System.out.println("US: " + date.format(formatterUS)); // May 16, 2025
            System.out.println("KR: " + date.format(formatterKR)); // 2025년 5월 16일
        }
    }
    ```
    

**DateTimeFormatterBuilder 클래스**

복합적인 포매터를 정의해서 좀 더 세부적으로 포매터를 제어할 수 있다

- 대소문자를 구분하는 파싱, 관대한 규칙을 적용하는 파싱, 패딩, 포매터의 선택사항 등을 활용할 수 있다
- DateTimeFormatterBuilder로 이전 코드를 이렇게 작성할 수 있다.

```java
DateTimeFormatter italianFormatter = new DateTimeFormatterBuilder()
	.appendText(ChronoField.DAY_OF_MONTH)
	.appendLiteral(". ")
	.appendText(ChronoField,MONTH_OF_YEAR)
	.appendLiteral(" ")
	.appendText(ChronoField.YEAR)
	.parseCaseInsensitive()
	.toFormatter(Locale.ITALIAN)；
```

> [!TIP] DateTimeFormatterBuilder를 이용한 선택적인 포맷 구문
>  optionalStart(), optionalEnd()을 이용하면 중간 요소를 **선택적으로 허용**할 수 있다.
>  다음 코드는 시간은 있을 수도 있고, 없을 수도 있음을 의미한다.

```java
 DateTimeFormatter formatter = new DateTimeFormatterBuilder()
            .appendValue(ChronoField.YEAR, 4)
            .appendLiteral('-')
            .appendValue(ChronoField.MONTH_OF_YEAR, 2)
            .appendLiteral('-')
            .appendValue(ChronoField.DAY_OF_MONTH, 2)
            .appendLiteral(' ')
            // 시간은 있을 수도 없을 수도 있다
            .optionalStart()
            .appendValue(ChronoField.HOUR_OF_DAY, 2)
            .appendLiteral(':')
            .appendValue(ChronoField.MINUTE_OF_HOUR, 2)
            .optionalEnd()
            
            .toFormatter();
```

# **12.3** 다양한 시간대와 캘린더 활용 방법

새로운 자바에서는 기존의 java.util.TimeZone을 대체할 수 있는 java.time.Zoneld 클래스가 새롭게 등장했다. 이를 통해서 시간대를 간단하게 처리할 수 있다

## **12.3.1** 시간대 사용하기

**java.time.Zoneld 클래스**

기존의 java.util.TimeZone 을 대체할 수 있는 클래스로 불변 클래스이다.

- 이를 이용하면 서머타임(DST) 같은 복잡한 사항을 자동으로 처리할 수 있다.
- 기준 시간이 같은 지역을 묶어서 시간대 규칙 집합을 정의한다. **ZoneRules** 클래스에는 약 40개 정도의 시간대가 있다.
- **Zoneld**의 **`getRules()`**를 이용해서 해당 시간대의 규정을 획득할 수 있다.
- 다음처럼 지역 **ID**로 특정 **Zoneld**를 구분한다.

```java
Zoneld romeZone = Zoneld.of("Europe/Rome");
```

- 새로운 메서드인 **`toZoneld`**로 기존의 **TimeZone** 객체를 **Zoneld** 객체로 변환할 수 있다.

```java
Zoneld zoneld = TimeZone.getDefault().toZoneId();
```

- **Zoneld** 객체를 얻은 다음에는 **LocalDate, LocalDateTime, Instant**를 이용해서 **ZonedDateTime** 인스턴스로 변환할 수 있다. **ZonedDateTime**은 지정한 시간대에 상대적인 시점을 표현한다.

```java
LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
ZonedDateTime zdt1 = date.atStartOfDay(romeZone);

LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
ZonedDateTime zdt2 = dateTime.atZone(romeZone);

Instant instant = Instant.now();
ZonedDateTime zdt3 = instant.atZone(romeZone)；
```

- **ZonedDateTime**의 컴포넌트를 보면 **LocaleDate, LocalTime, LocalDateTime, Zoneld**의 차이를 쉽게 이해할 수 있다

![image.png](12%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%A2%E1%84%85%E1%85%A9%E1%84%8B%E1%85%AE%E1%86%AB%20%E1%84%82%E1%85%A1%E1%86%AF%E1%84%8D%E1%85%A1%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B5%E1%84%80%E1%85%A1%E1%86%AB%20API%201f52dcb5a74a80bca059f19b8a22dcec/75adbac1-bde2-4003-b702-c7c0a67c71b6.png)

- **Zoneld**를 이용해서 **LocalDateTime**을 **Instant**로 바꾸는 방법도 있다

```java
Instant instant = Instant.now();
LocalDateTime timeFromlnstant = LocalDateTime.ofInstant(instant, romeZone);
```

## **12.3.2 UTC/Greenwich** 기준의 고정 오프셋

때로는 **UTC(** 협정세계시)**/GMT**(그리니치 표준시)를 기준으로 시간대를 표현하기도 한다

**ZoneOffset** 

**Zoneld**의 서브클래스인 **`ZoneOffset`** 클래스로 런던의 그리니치 0도 자오선과 시간값의 차이를 표현할 수 있다

- 예를 들어 ‘뉴욕은 런던보다 5시간 느리다’라고 표현할 수 있다
- **하지만 ZoneOffset**으로는 서머타임을 제대로 처리할 수 없으므로 권장하지 않는다.

```java
ZoneOffset newYorkOffset = ZoneOffset.of("-05:00");
```

**OffsetDateTime**

**ISO-8601** 캘린더 시스템에서 정의하는 **UTC/GMT**와 오프셋으로 날짜와 시간을 표현하는 **`OffsetDateTime`**을 만드는 방법도 있다.

- 이렇게 하면 "2014-03-18T13:45 -05:00”으로 저장된다.

```java
LocalDateTime dateTime = LocalDateTime.of(2014, Month.MARCH, 18, 13, 45);
OffsetDateTime dateTimelnNewYork = OffsetDateTime.of(dateTime, newYorkOffset);
```

자바8에서는 추가로 4개의 캘린더 시스템을 제공한다

- **`ThaiBuddhistDate`, `MinguoDate`, `JapaneseDate`, `HijrahDate`** 4개의 클래스가 각각의 캘린더 시스템을 대표한다
- 위의 4개의 클래스와 **`LocalDate`** 클래스는 **`ChronoLocalDate`** 인터페이스를 구현한다.
    - **ChronoLocalDate**는 임의의 연대기에서 특정 날짜를 표현할 수 있는 기능을 제공하는 인터페이스다.
- **LocalDate**를 이용해서 이들 4개의 클래스 중 하나의 인스턴스를 만들 수 있다

```java
LocalDate date = LocalDate.of(2014, Month.MARCH, 18);
JapaneseDate japaneseDate = JapaneseDate.from(date);
```

- 특정 **Locale**과 **Locale**에 대한 날짜 인스턴스로 캘린더 시스템을 만드는 방법도 있다
    - **`Chronology`**는 캘린더 시스템을 의미하며 정적 팩토리 메서드 **`ofLocale`**을 이용해서 **Chronology** 의 인스턴스를 획득할 수 있다
    
    ```java
    Chronology japaneseChronology = Chronology.ofLocale(Locale.JAPAN);
    ChronoLocalDate now = japaneseChronology.dateNow();
    ```
    

> [!TIP] **ChronoLocalDate**보다는 **LocalDate**
> 날짜와 시간 **API**의 설계자는 **ChronoLocalDate**보다는 **LocalDate**를 사용하라고 권고한다
> 프로그램의 입출력을 지역화하는 상황을 제외하고는 모든 데이터 저장, 조작,비즈니스 규칙 해석 등의 작업에서 **LocalDate**를 사용해야 한다!

### 이슬람력

자바8에 추가된 새로운 캘린더 *중* **HijrahDate**(이슬람력)가 가장 복잡한데, 이슬람력에는 변형이 있기 때문이다.

**`withVariant`** 메서드로 원하는 변형 방법을 선택할 수 있다.

다음 코드는 현재 이슬람 라마단 기간의 시작과 끝을 **ISO** 날짜로 출력하는 예제다

- `IsoChronology`는 우리가 일상에서 쓰는 ISO 그레고리력(양력)을 의미한다.

```java
HijrahDate ramadanDate = HijrahDate.now()
	.with(ChronoField. DAY_OF_MONTH, 1)  // 현재 날짜에서 이슬람력의 "1일"로 설정
	.with(ChronoField.MONTH_OF_YEAR, 9); // 이슬람력의 9월(Ramadan) 로 설정

System.out.printin("Ramadan starts on " +
	IsoChronology.INSTANCE.date(ramadanDate) + // 양력(LocalDate) 으로 변환
	" and ends on " +
	IsoChronology.INSTANCE.date(ramadanDate.with(TemporalAdjusters.lastDayOfMonth()))); // 라마단 달의 마지막 날(30일 또는 29일)
```