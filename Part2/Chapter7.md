# 7장 병렬 데이터 처리와 성능

<aside>

## 학습 내용

1. 병렬 스트림으로 데이터를 병렬 처리하는 방법
2. 병렬 스트림의 성능을 분석하는 방법
3. ForkJoin 프레임워크
4. Spliterator로 스트림 데이터를 쪼개는 방법

</aside>

# 7.1 병렬 스트림

컬렉션에서 `parallelStream` 메소드를 호출하면 병렬 스트림을 만들 수 있다.

병렬 스트림이란?
- 주어진 별도의 스레드에서 실행할 수 있도록 스트림을 작은 청크로 분할한 것이다.
- 멀티코어 환경에서 각 코어가 청크를 실행할 수 있다.

예제를 통해서 알아보자. 다음은 1부터 n까지의 합계를 스트림으로 계산하는 메소드이다.

```java
public long sequentialSum(long n) {
	return Stream.iterate(1L, i -> i + 1)
		.limit(n) 
		.reduce(0L, Long:: sum);
}
```

이때 n값이 커지면 병렬로 처리하는 것이 좋을텐데 어떻게 처리해야할까? 이때 고려해야할 사항들은 어떻게 정할까? 병렬스트림이 이 질문들을 해결할 수 있다.

## 7.1.1 순차 스트림에서 병렬 스트림으로 변환하기

순차 스트림에 `parallel` 메소드를 호출하면 병렬 스트림을 만들 수 있다.

```java
public long parallelSum(long n) {
	return Stream.iterate(1L, i -> i + 1)
		.limit(n)
		.parallel() 
		.reduce(0L, Long::sum);
}
```

위의 코드도 앞선 코드와 마찬가지로 스트림에서 대해서 리듀싱연산을 수행한다. 그런데 리듀싱 연산을 병렬로 수행한다는 점이 앞선 코드와의 차이점이다.

스트림을 작은 청크로 나누어서 리듀싱을 수행하고, 부분 결과를 다시 리듀싱해서 최종 결과를 얻는다. 

`parallel`을 호출하면 스트림 내부적으로는 병렬로 수행된다는 불리언 플래그가 켜진다. 만약 병렬 스트림에서 `sequential`을 호출하면 다시 순차 스트림으로 변환된다. 이 두 메소드를 통해서 스트림 계산 과정에서 스트림을 병렬로 수행할 수도, 순차적으로 수행할 수도 있다.

스트림에서 마지막으로 호출된 스트림 처리 방식이 전체 파이프라인을 좌우한다. 다음 코드에서는 parallel이 마지막에 호출되므로 전체적으로는 병렬로 수행된다.

```java
stream.parallel()
	.filter(...)
	.sequential()
	.map()
	.parallel()
	.reduce();
```

### 병렬 스트림에서 사용하는 스레드 풀 설정

parllel 메소드에서 병렬 작업을 수행하는 스레드는 어디서 온 것이며, 어떻게 커스터마이즈 할 수 있을까?

병렬 스트림은 ForkJoinPool이라는 스레드풀을 이용한다. 이 스레드풀은 기본적으로 프로세스 개수만큼의 스레드를 가지고 있다.

(프로세서 수는 Runtime.getRuntime().availableProcessors() 가 반환하는 값과 동일하다.)

ForkJoinPool에서 사용하는 스레드의 수를 원하는대로 수정하려면 전역적으로 설정해야한다. 이렇게 설정하고 나면 다른 병렬 스트림에도 동일하게 적용된다.

```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "12");
```

## 7.1.2 스트림 성능 측정

앞선 예제에서 병렬화를 적용하면 순차방식이나 반복형식보다 빠를 것이라 추측할 수 있다.
그러나 소프트웨어 공학에서 추측은 위험하다. 성능을 측정하는 방법은 오로지 측정뿐이다!

### JMH를 이용한 성능 측정 방법

이제부터 자바 마이크로 하니스(JMH)라는 라이브러리를 이용해서 작은 벤치마크를 구현해보자

여기서 잠깐 짚고 두 가지를 짚고 넘어가자
1. JMH
- 어노테이션 기반으로 간단하게 JVM을 대상으로 벤치마크 프로그램을 구현할 수 있게 해주는 라이브러리이다. 출력된 결과에서 Score가 낮을 수록 성능이 좋다고 해석한다.
  
2. JVM 테스트는 어렵다
- 핫스팟(HotSpot, 자주 실행되는 코드를 의미)이 바이트코드를 최적화하는데 필요한 준비 시간, 가비지 컬렉터로 인한 오버헤드 등과 같은 여러 요소를 고려해야 하기 때문에 테스트가 쉽지 않다.

JMH를 설치해보자. 메이븐을 사용하는 환경에서는 다음과 같이 pom.xml 파일을 작성한다.
- 첫번째 라이브러리 : 핵심 JMH 구현을 포함
- 두번째 라이브러리 : 자바 아카이브(JAR) 파일을 만드는 데 도움을 주는 어노테이션 프로세서를 포함

```xml
<dependency>
    <groupld>org.openjdk.jmh</groupld>
    <artifactld>jmh-core〈/artifactld〉
    <version>1.17.4</version>
</dependency>
<dependency>
    <groupld>org.openjdk.j mh</groupld>
    <artifactld〉jmh-generator-annprocess</artifactld〉    <version>1.17.4</version></dependency>
```

그리고 플러그인으로 maven-shade-plug이 필요하다. 이 플러그인은 자바 아카이브 파일로 벤치마크를 편리하게 실행하게 만들어 준다.

우리가 사용할 벤치마크는 다음과 같다. 
- 벤치마크에 영향을 주지 않도록 힙의 크기를 4GB로 설정하고, 벤치마크가 끝날때마다 가비지 컬렉션이 이루어지도록 했다.
- 하지만 벤치마크의 결과는 정확하지 않을 수 있다.

```java
@BenchmarkMode(Mode. AverageTime) // 벤치마크 대상 메서드를 실행하는 데 걸린 평균 시간 측정
@OutputTimeUnit(TimeUnit.MILLISECONDS) // 벤치마크 결과를 밀리초 단위로 출력
@Fork(2, jvmArgs={"-Xms4G", "-Xmx4G"})
public class ParallelStreamBenchmark {    
	private static final long N= 10_000_000L; // 4Gb의 힙 공간을 제공한 환경에서 두 번 벤치마크를 수행해 결과의 신뢰성 확보    
	
	@Benchmark  // 벤치마크대상메서드    
	public long sequentialSum() {        
		return Stream.iterate(1L, i -> i + 1).limit(N)        
		.reduce( 0L, Long::sum);    
}    
	
	@TearDown(Level.Invocation) // 매 번 벤치마크를 실행한 다음에는 가비지컬렉터 동작 시도    
	public void tearDown() {        System.gc();    }
	
}
```

클래스를 컴파일하면 benchmarks.jar라는 파일이 만들어진다. 이 파일을 다음 명령으로 실행한다.

```
java -jar ./target/benchmarks.jar ParallelStreamBenchmark
```

이렇게 실행하면 JMH 명령은 핫스팟이 코드를 최적화할 수 있도록 20번을 실행해서 벤치마크를 준비하고 20번을 더 실행해서 최종 결과를 계산한다.

이제 “인텔 i7-4600U 2.1GHz 쿼드 코어” 환경에서 성능을 측정한 결과를 보자!

### 순차 스트림의 성능 측정 결과

순차 스트림 방식의 결과는 Score는 121정도가 나온다.

### 전통적인 for 루프의 성능 측정 결과

그렇다면 다음과 같은 for 루프 방식은 어떨까?

```java
@Benchmarkpublic long iterativeSum() {    
	long result = 0;    
	for (long i = 1L; i <= N; i++) {        
		result += i;    
	}    
	return result;
}
```

결과는 다음과 같다. Score가 3정도 나오므로 순차 스트림 방식에 비해서 더 빠르다.
순차 스트림에 비해서 저수준에서 동작하며 오토박싱이 없기 때문으로 추정해볼 수 있다.

### 병렬 스트림의 성능 측정 결과

```java
public long parallelSum(long n) {    
	return Stream.iterate(1L, i -> i + 1)        
		.limit(n)        
		.parallel()
    .reduce(0L, Long::sum);
}
```

이번에는 기대가 큰 병렬 스트림의 성능을 보자. Score가 600정도인데, 놀랍게도 순차 스트림보다 성능이 5배 안좋게 나왔다.

### 문제점 2가지

문제는 두 가지다.
1. 반복의 결과로 박싱된 객체가 만들어지는데 합을 구하려면 언박싱을 해야한다.
2. 반복작업은 병렬로 수행할 수 있는 독립작업으로 나누기 어렵다.

두번째 문제에 특히 주목해야 한다. 우리는 병렬로 실행할 수 있는 스트림 모델이 필요하다.

사실 iterate 연산은 이전 연산의 결과에 따라서 다음 함수의 입력이 달라지기 때문에 독립된 작업으로 나누기 어렵다.
그래서 우리 예제에서 병렬 작업 수행을 위해서 스레드들에게 작업이 할당되어 있으나 이전의 작업이 끝나야 다음 작업이 실행될 수 있다.
따라서 병렬 작업 할당에 대한 오버헤드가 증가하기만할 뿐 성능이 나아지지는 않는다.

### 해법 : 더 특화된 메서드 사용

멀티코어 프로세서에서 더 효과적으로 합계를 계산하는 방법은 특화된 메서드를 사용하는 것이다.
LongStream.rangeClosed()라는 메서드가 바로 특화된 메서드이다. 이 메서드는 iterate()에 비해 다음 두 가지 장점을 갖는다.

1. 기본형 long 타입을 사용하므로 오토박싱 오버헤드가 없다!
2. 쉽게 청크로 분할할 수 있는 숫자 범위를 생산한다!

### 언박싱 오버헤드 확인

이제 rangeClosed를 이용한 코드의 성능을 측정하여 언박싱 오버헤드가 얼마나 되는지 알아보자

```java
@Benchmarkpublic 
long rangedSum() {    
	return LongStream.rangeClosed(1, N)    
		.reduce(0L, Long::sum);
}
```

성능 측정 결과를 보면 Score가 5로 측정된다. iterate를 이용했을 때보다 성능이 더 높다.
이를 통해서 상황에 따라서 적절할 자료구조를 선택하는 것의 중요성을 알 수 있다.

### 특화된 메서드를 활용한 병렬화 성능 확인

그럼 이제 병렬화를 적용하여 성능을 측정해보자.

```java
@Benchmarkpublic 
long rangedSum() {   
	return LongStream.rangeClosed(1, N)
	   .parallel()
    .reduce(0L, Long::sum);
}
```

측정 결과를 보면 Score가 2.6 정도이며 순차 실행보다 성능이 더 뛰어나다!

### 병렬화에 대한 조언

병렬화는 공짜가 아니다. 병렬화를 이용하려면 스트림을 재귀적으로 분할하고, 각 서브스트림을 스레드에 할당해 리듀싱을 수행하고, 리듀싱 결과를 다시 모아야 한다. 이때 멀티 코어 간에 데이터 이동에 대한 오버헤드가 있다. 데이터를 이동하는데 드는 시간보다 전체 작업 시간이 더 길때 병렬화를 이용하자

그리고 병렬화가 아예 불가능한 경우가 있을 수도 있다. 병렬화를 적용할 때는 우리가 잘 적용하고 있는지 유의하자.

## 7.1.3. 병렬 스트림을 올바르게 사용하는 방법

병렬화에서 문제가 생기는 때는 공유 데이터를 바꿀 때이다.

다음 코드는 n까지의 수를 공유된 누적자에 더하는 코드이다.

```java
public long sideEffectSum(long n) {    
	Accumulator accumulator = new Accumulator();    
	LongStream.rangeClosed(1, n).forEach(accumulator::add); // 공유 데이터를 접근하는 코드    
	return accumulator.total;}
	
public class Accumulator {    
	public long total = 0; // 공유된 누적자    
	public void add(long value) { total += value; } // 데이터 레이스 발생 가능 지점
}
```

이 코드는 순차 수행되도록 구현되어 있다. 만약 병렬로 실행한다면 total을 접근하는 부분에서 데이터 레이스가 발생할 수 있다.
그렇다고 동기화를 적용하면 병렬화라는 특성이 없어진다.

이 코드에 병렬화를 적용하면 어떻게 될지 예제를 통해서 알아보자

```java
public long sideEffectParallelSum(long n) {
    Accumulator accumulator = new Accumulator();
    LongStream.rangeClosed(1z n),parallel().forEach(accumulator::add);    
    return accumulator.total;
}
  
System.out.println("SideEffeet parallel sum done in: " +measurePerf(Parallelstreams::sideEffectParallelSum, 10_000_000L) + " msecs" );
```

우리가 예상하는 결과는 50000005000000이지만 이와 다른 부정확한 결과가 나올 수 있다.

문제가 되는 지점은 앞서 말한대로 total의 값을 변경하는 부분이다. 이처럼 병렬 스트림을 사용할 때는 공유된 가변 상태를 조심해야 한다.

## 7.1.4 병렬 스트림 효과적으로 사용하기

어떤 상황에서 병렬 스트림을 사용할지에 대한 힌트를 정리해보면 다음과 같다.

1. 순차 스트림이 좋을지 병렬 스트림이 좋을지 확신이 서지 않으면 적절한 벤치마크를 이용해서 직접 성능을 측정해보아라
2. 박싱이 성능을 저해할 수 있음을 기억하라. 되도록이면 기본형에 특화된 스트림을 사용하는 것이 좋다.
3. 순차스트림보다 병렬스트림이 성능이 더 떨어지는 경우가 존재한다. 예를 들어서 병렬스트림에서 findFirst는 사용하지 않는 것이 좋다.
4. 스트림에서 수행하는 전체 파이프라인의 연산 비용을 고려하라. 처리해야할 요소 수가 N개이고, 한 요소를 처리하는데 드는 비용이 Q일때, 전체 파이프라인의 비용은 N * Q이다. Q가 클수록 병렬스트림으로 성능을 개선할 여지가 있다.
5. 소량의 데이터에서는 병렬화 비용으로 인해서 성능이 더 나빠진다.
6. 스트림을 구성하는 자료구조가 적절한지 확인하라. 예를 들어서 ArrayList는 LinkedList와 달리 분할에 용이하다.
7. 스트림의 특성과 스트림의 중간연산이 스트림의 특성을 어떻게 바꾸는지에 따라서 분해과정의 성능의 달라진다.
8. 최종 결과를 합치는데 드는 비용이 병렬화를 하는데 얻는 이익보다 크지 않은지 확인하라.

# 7.2 포크조인 프레임워크

포크/조인 프레임워크는 작업을 재귀적으로 분할할 다음에 각 서브태스크의 결과를 모아서 결과를 만들어낸다. 이 프레임워크는 ForkJoinPool의 스레드에 서브태스크를 할당해준다. ExecutorService라는 인터페이스를 구현한다. 

## 7.2.1 RecursiveTask 활용

스레드 풀을 이용하려면 RecursiveTask<R>의 서브 클래스를 만들어야 한다.
- R은 병렬화된 태스크가 생성하는 결과 형식이다. 결과가 없을 때는 RecursiveAction 형식이다.

RecursiveTask를 정의하려면 추상메서드 compute를 구현해야한다.

```java
protected abstract R compute();
```

compute 메서드는 태스크를 서브태스크로 분할하고 더이상 분할할 수 없으면 서브태스크의 결과를 생산하는 알고리즘을 정의한다. 의사코드는 다음과 같다.

```java
if(태스크가 충분히 작아서 분할할 수 없다면){
	순차적으로 태스크를 계산한다
}
else{
	태스크를 두 개의 서브태스크로 분할한다
	태스크가 다시 서브태스크로 분할되도록 이 알고리즘을 재귀적으로 호출한다
	모든 서브태스크의 연산 결과가 완료될까지 기다린다
	각 서브태스크의 결과를 합친다 
}
```

이 알고리즘은 분할정복(divide and conquer) 알고리즘의 병렬화 버전이라고 할 수 있다. 

이제 특정 범위의 숫자를 더하는 문제를 통해서 포크조인 프레임워크 사용방법을 알아보자.

1. 클래스는 RecursiveTask를 구현해야한다.

```java
public class ForkJoinSumCalculator
extends java.util.concurrent.RecursiveTask<Long> {
}
```

2. compute를 오버라이드해야한다. 의사코드에 따라서 compute를 작성하면 된다. 

```java
@Override
protected Long compute() { 
	int length = end - start;
	if (length <= THRESHOLD) {
		return computeSequentially();
	}
	ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
	leftTask.fork();
	ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
	Long rightResult = rightTask.compute();
	Long leftResult = leftTask.join();
	return leftResult + rightResult; 
}

// 이건 compute에서 더이상 분할할 수 없을 때 작업을 수행하는 메서드이다 
private long computeSequentially() {
	long sum = 0;
	for (int i = start; i < end; i++) {
		sum += numbers[i];
	}
	return sum;
}
```

3. 추가적으로 클래스를 구현하기 위한 작업을 수행한다.
- compute 메서드에서 코드를 작성하면서 저장이 필요한 데이터도 필드로 추가한다.

```java
private final long[] numbers; // 더해야할 숫자를 저장
private final int start; // 서브태스크에서 처리할 요소의 시작
private final int end;   // 서브태스크에서 처리할 요소의 끝
public static final long THRESHOLD = 10_000; // 분할을 멈출 기준
```

- compute를 작성하면서 필요했던 생성자도 추가한다.

```java
public ForkJoinSumCalculator(long[] numbers) {
 this(numbers, 0, numbers.length); // 메인 태스크의 서브태스크를 재귀적으로 만들 때 사용할 비공개 생성자
}

private ForkJoinSumCalculator(long[] numbers, int start, int end) {
	this.numbers = numbers;
	this.start = start;
	this.end = end;
}
```

그러면 다음과 같이 코드를 완성할 수 있다.

```java
public class ForkJoinSumCalculator
extends java.util.concurrent.RecursiveTask<Long> {
	private final long[] numbers; // 더해야할 숫자를 저장
	private final int start; // 서브태스크에서 처리할 요소의 시작
	private final int end;   // 서브태스크에서 처리할 요소의 끝
	public static final long THRESHOLD = 10_000; // 분할을 멈출 기준
	
	public ForkJoinSumCalculator(long[] numbers) {
	 this(numbers, 0, numbers.length); // 메인 태스크의 서브태스크를 재귀적으로 만들 때 사용할 비공개 생성자
	}
	
	private ForkJoinSumCalculator(long[] numbers, int start, int end) {
		this.numbers = numbers;
		this.start = start;
		this.end = end;
	}
	
	@Override
	protected Long compute() { 
		int length = end - start;
		if (length <= THRESHOLD) {
			return computeSequentially();
		}
		ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
		leftTask.fork();
		ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
		Long rightResult = rightTask.compute();
		Long leftResult = leftTask.join();
		return leftResult + rightResult; 
	}
	
	private long computeSequentially() {
		long sum = 0;
		for (int i = start; i < end; i++) {
			sum += numbers[i];
		}
		return sum;
	}
}
```

이제 완성된 코드를 다음과 같이 사용할 수 있다. 
- 태스크를 생성한 후에 ForkJoinPool의 invoke메소드를 호출한다. 
- invoke의 반환값은 ForkJoinSumCalculator에서 정의한 태스크의 결과이다.

```java
public static long forkJoinSum(long n) {
	long[] numbers = LongStream.rangeClosed(1, n).toArray();
	ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
	return new ForkJoinPool().invoke(task);
}
```

ForkJoinPool에 대해서 다시 짚고 넘어가자.

- 일반적으로 한 애플리케이션에서는 ForkJoinPool을 싱글턴으로 관리하여서 필요한 곳에서 가져다 쓴다.
- ForkJoinPool에서 기본생성자는 JVM에서 이용할 수 있는 모든 프로세서가 자유롭게 풀에 접근할 수 있음을 의미한다.
- ForkJoinPool은 Runtime.availableProcessors의 반환값으로 스레드의 수를 결정한다. 여기서 사용할 수 있는 프로세스는 실제 프로세스 뿐 아니라 하이퍼스레딩과 관련된 가상 프로세서도 포함한다.

여기서 잠깐 하이퍼스레딩이란?
- 하이퍼스레딩(Hyper-Threading)은 **Intel**에서 만든 기술로, 하나의 물리적인 CPU 코어가 **두 개의 논리적인(가상) 코어처럼 동작하도록 하는 기술이다.**
- CPU는 어떤 작업을 처리할 때, 일부 자원(IU, ALU, 레지스터 등)이 **빈 시간**이 생길 수 있다. 하이퍼스레딩은 이 **유휴 자원을 활용해서 다른 스레드도 동시에 처리**하려는 기법이다. 그래서 특히 I/O가 많은 경우나 병렬성이 높은 작업에서 성능이 좋아질 수 있다

## 7.2.2. 포크/조인 프레임워크를 제대로 사용하는 방법

1. join 메소드를 호출하면 태스크를 할당받은 스레드가 작업을 마칠 때까지 호출자가 블록된다. 따라서 join 메소드는 두 서브태스크가 모두 시작된 후에 join을 호출해야 한다.
2. RecursiveTask 내에서는 invoke를 사용하지 않아야 한다.
3. 두 작업 모두 fork하기보다는 한 작업은 fork를 사용하고 다른 작업은 compute하는 것이 좋다. 그렇게 하면 풀에 있는 스레드를 효율적으로 사용할 수 있다.
4. 포크조인프레임워크에서 스택트레이스를 이용한 디버깅은 도움이 되지 않는다.
5. 멀티코어에서 포크조인 프레인워크 방식이 무조건 더 빠르지는 않다. 각 서브태스크의 실행 시간이 서브태스크를 포킹하는데 드는 시간보다 길어야 한다. 
6. 순차버전과 병렬 버전을 비교할 때는 컴파일러 최적화와 같은 다른 요인을 고려해야 한다. 앞서 벤치마킹에서 한 것처럼 JIT 컴파일러가 최적화할 때까지 여러번 프로그램을 실행해야 할 수 있다.
 
여기서 잠깐 JIT 컴파일러란?
- 자바 프로그램은 먼저 `.java` 파일을 컴파일해서 `.class` 바이트코드가 되고, 이 바이트코드는 JVM(Java Virtual Machine)에서 실행된다. 이때 JVM은 처음엔 **인터프리터** 방식으로 코드를 한 줄씩 해석하면서 실행한다. 그런데 이러한 방식은 느리기 때문에 JIT 컴파일러를 사용한다.
- JIT 컴파일러는 자주 실행되는 코드(핫스팟)를 감지해서, 이를 **기계어로 변환**해버리고, 이후부터는 빠르게 실행하도록 하는 컴파일러이다.
- 즉, 실행 중에 코드를 **동적으로 최적화**해서 더 빠르게 만들어주는 역할을 한다.
- 처음 실행할 때는 JIT이 아직 최적화를 안 했기 때문에 **초기 실행 성능이 낮고** 시간이 지날수록 **JIT이 코드 최적화**를 하면서 성능이 점점 좋아진다. 그래서 벤치마크 결과를 정확하게 얻으려면 예열 후에 측정해야 한다!

## 7.2.3 작업 훔치기 (work stealing)

원래는 코어 개수를 넘어서지 않게 작업을 분할하는 것이 좋을 것이라 생각할 수 있다. 하지만 실제로는 코어 개수와 상관 없이 적절한 크기로 분할된 많은 태스크로 포킹하는 것이 바람직하다. 

이론적으로는 코어 개수만큼 태스크가 분할되면 모든 코어가 태스크를 실행해서 동일한 시간이 작업을 마칠 것이다. 하지만 현실에서는 각 코어에 할당된 태스크의 작업 완료 시간이 다를 수 있다. 분할 방식이 효율적이지 않았거나 입출력에서 지연이 생겼을 수 있다.

포크조인 프레임워크는 이 문제를 작업 훔치기라는 기법으로 해결한다. 이 기법은 ForkJoinPool의 스레드들에게 작업을 공평하게 분배한다. 

각 스레드는 할당된 태스크를 관리하는 이중리스트를 가지고 있다. 여기서 각 스레드는 리스트의 head로부터 다음에 수행할 작업을 가져온다. 

그런데 어느 시점에는 한 스레드가 작업을 모두 끝내서 할 일이 없게 될 수 있다. 이때 그 스레드는 유휴상태로 가는 것이 아니라 다른 스레드의 리스트의 tail에서 작업을 가져와서 수행한다. 모든 큐가 빌 때까지 스레드들은 일을 계속하게 된다. 

따라서 스레드 간의 작업 부하를 비슷하게 유지하려면 태스크를 작게 분할해야 한다!

# 7.3 Spliterator 인터페이스

자바8에서 Spliterator라는 인터페이스를 제공한다. 

- Spliterator는 분할할 수 있는 반복자라는 의미이다. Iterator처럼 소스의 요소 탐색이 가능한데, 병렬 작업에 특화되어 있다.
- 자바 8은 컬렉션 프레임워크에 포함된 모든 자료구조에 사용할 수 있는 디폴트 Spliterator 구현을 구현하다.

Spliterator는 다음과 같은 메소드를 제공한다.

- T는 Spliterator가 탐색하는 요소의 형식이다.
- tryAdvance 메서드는 Spliterator의 요소를 하니씩 소비하면서 탐색해야할 요소가 남아 있으며 참을 반환한다.
- trySplit 메서드는 Spliterator의 일부 요소를 분할해서 두번째 Spliterator를 생성하는 메서드다.
- estimateSize 메서드는 탐색해야할 요소 수 정보를 제공한다. 값이 정확하진 않을 수 있지만 제공된 값을 이용해서 Splilterator를 분할할 수 있다.

```java
public interface Spliterator<T> {
	boolean tryAdvance(Cons나mer〈? super T> action);
	Spliterator<T> trySplit();
	long estimateSize();
	int characteristics();
}
```

## 7.3.1 Spliterator의 분할 과정

스트림을 여러 스트림으로 분할하는 과정은 재귀적으로 일어난다. 

- 1단계에서 첫 번째 Spliterator에 trySplit을 호출하면 두 번째 Spliterator가 생성
된다
- 2단계에서 두 개의 Spliterator에 trySplit를 다시 호출하면 네 개의 Spliterator가 생
성된다
- 이처럼 trySplit의 결과가 null이 될 때까지 이 과정을 반복한다
- Spliterator에 호출한 모든 trySplit의 결과가 null이면 재귀 분할과정이 종료된다
- 이 분할 과정은 characteristics 메서드로 정의하는 Spliterator의 특성에 영향을 받는다

**Spliterator** 특성

Spliterator는 characteristics라는 추상 메서드도 정의한다.  

- Characteristics 메서드는Spliterator 자체의 특성 집합을 포함하는 int를 반환한다
- Spliterator를 이용하는 프로그램은 이들 특성을 참고해서 Spliterator를 더 잘 제어하고 최적화할 수 있다
- 특성은 다음과 같다.

| 특성 | 의미 |
| --- | --- |
| ORDERED | 리스트처럼 요소에 정해진 순서가 있으므로 Spliterator는 요소를 탐색하고 분할할 때 이 순서에
유의해야 한다. |
| DISTINCT | x, y 두 요소를 방문했을 때 x.equals(y)는 항상 false를 반환한다 |
| SORTED | 탐색된 요소는 미리 정의된 정렬 순서를 따른다 |
| SIZED | 크기가 알려진 소스(예를 들면 Set)로 Spliterator를 생성했으므로 estimatedSize()는 정확한
값을 반환한다 |
| NON-NULL | 탐색하는 모든 요소는 null이 아니다 |
| IMMUTABLE | 이 Spliterator의 소스는 불변이다. 즉. 요소를 탐색하는 동안 요소를 추가하거나 삭제하거나 고칠수 없다 |
| CONCURRENT | 동기화 없이 Spliterator의 소스를 여러 스레드에서 동시에 고칠 수 있다 |
| SUBSIZED | 이 Spliterator 그리고 분할돠는 모든 Spliterator는 SIZED 특성을 갖는다 |

## **7.3.2** 커스텀 **Spliterator** 구현하기

문자열의 단어 수를 계산하는 단순한 메서드를 구현하는 문제를 통해서 Spliterator를 구현해보자.

일단 반복 형식으로 작성한 코드는 다음과 같을 것이다.

```java
public int countWordsIteratively(String s) {
	int counter = 0;
	boolean lastSpace = true;
	for (char c : s.toCharArrayO) {
		if (Character.isWhitespace(c)) {
		lastSpace = true;
		} else {
			if (lastSpace) counter++;
			lastSpace = false; 
		}
	}
	return counter;
}
```

### 함수형으로 단어 수를 세는 메서드 재구현하기

이제 스트림 방식으로 단어의 개수를 세는 프로그램을 작성해보자. 병렬화를 위해서 Spliterator도 적용해보자.

우선 String을 스트림으로 바꿔야 한다. 문자에 특화된 스트림은 없으므로 Stream<Character>를 이용한다. 

```java
Stream<Character> stream = IntStream.range(0, SENTENCE.length())
.mapToObj(SENTENCE::charAt);
```
이 스트림에 리듀싱을 이용해서 단어의 수를 구할 수 있을 것이다.

이제 스트림에서 리듀싱 시에 계산을 수행하고 중간 결과를 저장할 클래스 WordCounter를 작성해보자

지금까지 발견한 단어 수를 계산하는 int 변수와 마지막 문자가 공백이었는지 여부를 기억하는 Boolean 변수 등 두 가지 변수가 필요하다. 그런데 자바는 튜플이라는 자료가 없으니 클래스를 작성한다.

```java
class WordCounter {
	private final int counter;
	private final boolean lastSpace;
	
	public WordCounter(int counter, boolean lastSpace) {
		this.counter = counter;
	}
}
```

이 클래스에 accumulate라는 메서드를 추가한다. 

- accumulate 메서드는 (불변클래스에 해당하는) 새로운 WordCounter 클래스를 어떤 상태로 생성할것인지 정의한다
- 스트림을 탐색하면서 새로운 문자를 찾을 때마다 accumulate 메서드를 *호출*한다
- 새로운 비공백 문자를 탐색한 다음에 마지막 문자가 공백이면 counter를 증가시킨다

```java
public WordCounter accumulate(Character c)
	if (Character.isWhitespace(c)) {
		return lastSpace ? this : new WordCounter(counter, true); 
	} else { 
		return lastSpace ? new WordCounter(counter+1, false) : this;
	}
}
```

이 클래스에 두 번째 메서드 combine을 추가한다.

- 문자열 서브 스트림을 처리한 WordCounter의 결과를 합친다. 즉, combine은 WordCounter의 내부 counter값을 서로 합친다.

```java
public WordCounter combine(WordCounter wordcounter) { 
	return new WordCounter(counter + wordcounter. counter, wordcounter.lastSpace); 
}
```

마지막으로 결과를 얻기 위한 getCounter 메서드를 클래스에 추가한다.

```java
public int getCounter() { 
	return counter;
}
```

이제 리듀싱 연산을 통해서 단어의 개수를 계산하는 메서드를 작성해보자

```java
private int countwords(Stream<Character> stream) {
	WordCounter wordcounter = stream.reduce(new WordCounter(0, true),
		WordCounter::accumulate,
		WordCounter::combine);
	return wordcounter.getCounter();
}
```

이제 단어의 개수를 세는 작업을 병렬로 수행해보자

### **WordCounter** 병렬로 수행하기

다음 처럼 작업을 병렬 스트림으로 처리해볼 수 있을 것이다. 

```java
System.out.printin("Found " + countwords(stream.parallel()) + " words");
```

그런데 지금 상태로 실행해보면 결과가 정확히 나오지 않을 수 있다. 왜 그럴까?

문제가 되는 지점은 바로 문자열을 임의의 위치에서 분할한다는 점이다. 이렇게 분할하면 같은 단어를 두 번 세는 경우가 있을 수 있다.

이 문제를 해결하려면 임의의 위치가 아니라 단어가 끝나는 부분에서 문자열을 분할해야한다. 그러려면 문자열을 분할하는 Spliterator가 필요하다.

구현한 Spliterator는 다음과 같다.

```java
class WordCounterSpliterator implements Spliterator<Character> {
	private final String string;
	private int currentchar = 0;
	
	public WordCounterSpliterator(String string) {
		this.string = string;
	}
	
	@Override
	public boolean tryAdvance(Consumer<? super Character> action) {
		action.accept(string. charAt(currentChar++));
		return currentChar < string.length();
	}
	
	@Override
	public Spliterator<Character> trySplit() {
		int currentSize = string.length() - currentchar;
		if (currentsize < 10){
			return null;
		}
		for( int splitPos = currentsize / 2 + currentchar; 
		splitPos < string.length(); splitPos++) {
			if(Character.isWhitespace(string.charAt(splitPos))) {
				Spliterator<Character> spliterator =
					new WordCounterSpliterator(string.substring(currentchar, splitPos)); 
				currentChar = splitPos;
				return spliterator;
			}
		}
		return null;
	}
	@Override
	public long estimateSize() {
		return string.length() - currentChar;
	}
	@Override
	public int characteristics() {
		return ORDERED + SIZED + SUBSIZED + NON-NULL + IMMUTABLE;
	}
}
```

WordCounterSpliterator를 분석해보자

- tryAdvance 메서드
    - 문자열에서 현재 인덱스에 해당하는 문자를 Consumer에 제공한 다
    음에 인덱스를 증가시킨다
    - 예제에서는 스트림을 탐색하면서 하나의 리듀싱 함수인 WordCounter의 accumulate 메
    서드만 적용한다.
    - 새로운 커서 위치가 전체 문자열 길이보다 작으면 참을 반환하며 이는 반복 탐색해야 할 문자가 남아있음을 의미한다
- trySplit 메서드
    - 반복될 자료구조를 분할하는 로직을 포함한다.
    - 우선 분할 동작을 중단할 한계를 설정해야 한다. 여기서는 아주 작은 한계값을 사용했지만 실전의 애플리케이션에서는 너무 많은 태스크를 만들지 않도록 더 높은 한계값을 설정해야 한다. 분할 과정에서 남은 문자 수가 한계값 이하면 null을 반환하여 분할을 중단시켜야 한다.
    - 분할이 필요한 상황에는 파싱해야 할 문자열 청크의 중간 위치를 기준으로 분할하도록 지시한다. 이때 단어 중간을 분할하지 않도록 빈 문자가 나올때까지 분할 위치를 이동시킨다. 분할할 위치를 찾았으면 새로운 Spliterator를 만든다
- estimatedSize 메서드
    - 탐색해야 할 요소의 개수를 반환한다.
- characteristic 메서드
    - Spliterator의 속성을 알려준다.

### **WordCounterSpliterator** 활용

이제 새로운 WordCounterSpliterator를 병렬 스트림에 사용할 수 있다. 
- StreamSupport.stream 팩토리 메서드의 첫번째 인수에 Spliterator를 전달한다.
- StreamSupport.stream 팩토리 메서드로 전달한 두 번째 불리언 인수는 병렬 스트림 생성 여부를 지시한다

```java
Spliterator<Character> spliterator = new WordCounterSpliterator(SENTENCE);
Stream<Character> stream = StreamSupport.stream(spliterator, true);

System.out.println("Found " + countwords(stream) + " words");
```

이렇게 하면 병렬화를 통해서 올바른 결과를 얻을 수 있다!