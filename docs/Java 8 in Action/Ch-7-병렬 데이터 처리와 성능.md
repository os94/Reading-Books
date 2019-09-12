# Ch 7. 병렬 데이터 처리와 성능

Java 8 in Action :: Contents

자바7이 등장하기 전에는 데이터 컬렉션을 병렬로 처리하기가 어려웠다.
데이터를 서브파티로 분할하고, 각각의 스레드로 할당해 race condition이 발생하지 않도록 적절한 동기화를 추가한뒤, 다시 부분 결과를 합치는 등.
자바7은 더 쉽게 병렬화를 수행하며 에러를 줄일 수 있도록, 
Fork/Join framework 기능을 제공한다.
이번장에서는 이러한 내용과, 또 이들과 자바8 병렬 스트림의 관계에 대해 알아본다. (병렬 스트림은 내부적으로 ForkJoinPool을 사용)

# 병렬 스트림

---

병렬 스트림은 스트림 요소를 각 스레드에서 처리할 수 있도록, 여러 청크로 분할한다. 이를 통해 모든 멀티코어 프로세서가 각 청크를 처리할 수 있다.

- parallel/sequential을 호출하여, 병렬/순차 스트림으로 바꿀 수 있다

    .parallel()을 호출하면, 내부적으로 이후 연산이 병렬로 수행되야함을 의미하는 boolean Flag가 설정된다

- 두 메서드가 모두 호출된다면, 둘 중에 최종적으로 호출된 메서드가 전체 파이프라인에 영향을 미친다 (← ?)

### 스트림 성능측정 실험

1. 순차 스트림 : 97 ms
2. 고전 for 루프 : 2 ms
3. 병렬 스트림 : 164 ms ← ?

    ⇒ 이유

    1. 박싱/언박싱의 오버헤드
    2. iterate는 본질적으로 순차적. 병렬실행을 위한 독립적인 청크로 분할하기 어려우므로, 각 합계가 다른 스레드에서 수행되었지만 결국 순차방식과 큰 차이없고, 스레드를 할당하는 오버헤드만 발생함

⇒ 병렬 스트림이 무조건 좋은게 아니다. 상황에 맞게 적절히 선택해야 한다. 병렬화를 이용하면 스트림을 재귀적으로 분할해야 하고, 각 서브 스트림을 서로 다른 스레드로 할당해서 각 결과를 하나로 합쳐야한다. 멀티코어 간의 데이터 이동은 생각보다 비쌀 수 있다.

### 병렬 스트림을 잘못 사용하는 예제 : 공유된 가변 상태

Race Condition으로, total += value 결과값이 올바르게 나오지 않는다.

### 효과적인 병렬 스트림 사용

- 병렬/순차 무엇이 더 빠를지 확신이 서지 않는다면, 직접 벤치마크로 측정하라
- 박싱을 주의하라. 되도록 기본형 특화 스트림을 사용한다
- 순차보다 병렬 스트림에서 성능이 떨어지는 연산이 있다

    ex) limit, findFirst

- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라

    전체 연산 비용을 NQ로 예상할 수 있다. (처리해야할 요수 수 N, 하나의 요소를 처리하는 데 드는 비용 Q) 이때 Q가 높아진다면, 병렬 스트림으로 성능을 개선할 가능성이 있다는 것

- 소량의 데이터에선 병렬 스트림이 도움 되지 않는다
- 스트림을 구성하는 자료구조가 적절한지 확인하라
- 스트림의 특성과 파이프라인의 중간연산이 스트림의 특성을 어떻게 바꾸는지에 따라, 분해 과정의 성능이 달라질 수 있다

    SIZED vs filter : 분할되는 스트림의 길이 절반 vs 예측불가

- 최종 연산의 병합 과정 비용을 살펴보라. ex) Collector의 combiner

[스트림 소스와 분해성](https://www.notion.so/a5d9a38f79e94784bafded3c0d582cb3)

# Fork / Join Framework

---

포크/조인 프레임워크는 (병렬화할 수 있는) 작업을 *재귀적으로* 작은 작업으로 분할한 다음, subTask 각 결과를 다시 합칠 수 있도록 설계되었다.

포크/조인 프레임워크는 subTask를 스레드 풀(ForkJoinPool)의 작업자 스레드에 분산 할당하는 ExceutorService 인터페이스를 구현한다.

스레드 풀을 이용하려면 RecursiveTask<R>의 서브 클래스를 만들어야 하고, RecursiveTask를 정의하려면 추상 메서드 compute를 구현해야 한다.

compute 메서드는 테스크를 서브테스크로 분할하는 로직과 더 이상 분할할 수 없을 때 각 서브테스크의 결과를 생산할 알고리즘을 정의한다.

    if (테스크가 충분히 작거나 더 이상 분할할 수 없으면) {
    	순차적으로 테스크 계산
    } else {
    	테스크를 두 서브테스크로 분할
    	테스크가 다시 서브테스크로 분할되도록 이 메스드를 재귀적으로 호출
    	모든 서브테스크의 연산이 완료될 때까지 기다림
    	각 서브테스크의 결과를 합친다
    }

사실 divide-and-conquer 알고리즘의 병렬화 버전이다.

### Fork/Join Framework를 이용해 범위의 숫자를 더하는 예제
 - ForkJoinSumCalculator

    P.240 - 241 (차분히 읽어보면 이해된다)
    public class ForkJoinSumCalculator extends java.util.concurrent.RecursiveTask<Long> {
    	...
    }

이렇게 만든 클래스의 인스턴스를 스레드 풀로 던져 이용한다.

    public static long forkJoinSum(long n) {
    	long[] numbers = LongStream.rangeClosed(1, n).toArray();
    	ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
    	return new ForkJoinPool().invoke(task);
    }

일반적으로 둘 이상의 ForkJoinPool을 만들지않고, 한번만 인스턴스화해 싱글턴으로 만들어, 필요한 곳들에서 가져다 쓴다.

### 효과적인 Fork/Join Framework 사용

- join 메서드를 테스크에 호출하면, 테스크가 생산하는 결과가 준비될 때까지 호출자를 블록시킨다. 따라서 두 서브테스크가 모두 시작된 다음에 join을 호출한다
- RecursiveTask 내에서 ForkJoinPool의 invoke 메서드를 사용하지 않는다
- 서브테스크에 fork 메서드를 호출해 ForkJoinPool의 일정을 조절한다. 왼쪽/오른쪽 작업 모두에 fork를 호출하는게 자연스러워보이지만, 한쪽에는 fork보다 compute를 호출하는것이 효율적이다. 그러면 둘 중 한곳은 같은 스레드를 재사용할 수 있으므로, 풀에서 불필요한 테스크를 할당하는 오버헤드를 피할 수 있기 때문
- 포크/조인 프레임워크를 이용하는 병렬 계산은 디버깅하기 어렵다. 스레드가 달라서 stack trace로 확인하기 어렵다
- 역시 마찬가지로, 무조건 포크/조인 프레임워크를 사용하는것이 순차보다 빠를거란 생각을 버린다

### Work Stealing (작업 훔치기)

이론적으로는, 코어 개수만큼 분할하면 모든 cpu 코어를 이용해 크기가 같은 각 테스크들이 같은 시간에 끝날 것 같다. 하지만 현실에선 다른 많은 변수들로 그렇지않다.

이때 포크/조인 프레임워크에선 작업훔치기 기법으로 ForkJoinPool의 모든 스레드를 거의 공정하게 분할한다.

각 스레드는 자신의 할일이 먼저 끝나면 idle 상태로 바뀌는것이 아니라, 다른 바쁜 스레드의 tail에서 작업을 훔쳐와 도와준다.

# Spliterator : 자동으로 스트림을 분할

---

우린 위의 코드처럼 직접 분할 로직을 개발하지않고도, 병렬 스트림을 이용할 수 있었다.

자바8은 `Spliterator`라는 인터페이스를 제공한다.

> "분할할 수 있는 반복자"라는 의미. Iterator처럼 소스의 요소 탐색 기능을 제공한다는 점은 같지만, 병렬 작업에 특화되어 있다.
자바8은 Collection 프레임워크의 모든 자료구조에 사용할 수 있는 디폴드 Spliterator 구현을 제공한다.

    public interface Spliterator<T> {
    	boolean tryAdvance(Consumer<? super T> action);
    	Spliterator<T> trySplit();
    	long estimateSize();
    	int characteristics();
    }

- tryAdvance : 요소를 하나씩 순차적으로 소비하며, 탐색할 요소가 남아있으면 true를 반환
- trySplit : (자신이 반환한) 요소를 분할해 두번째 Spliterator를 생성하는 메서드
- estimateSize : 탐색할 요소 수 정보 제공
- characteristics : Spliterator의 특성 집합을 포함하는 int 반환

    P.252 - 254 WordCounterSpliterator 예제코드