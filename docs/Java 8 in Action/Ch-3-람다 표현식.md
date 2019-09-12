# Ch 3. 람다 표현식

Java 8 in Action :: Contents

### 람다란

---

메서드로 전달할 수 있는 익명 함수를 단순화한 것

- 특징 : 익명, 함수, 전달, 간결
- 구성 : Parameter 리스트, 화살표, Body

    // 기본 문법
    (parameters) -> expression
    
    (parameters) -> {
    	statements;
    	...
    }

- 예제 : boolean, 객체 생성/소비/선택/추출, 두 객체 조합/비교 등

- expression vs statements

람다는 어디에서 사용하는가? *함수형 인터페이스*의 문맥에서 사용한다.

### 함수형 인터페이스란

---

오직 하나의 추상 메서드만 갖고있는 인터페이스 (뒤에서 나올 default method는 예외)

- 람다 표현식으로 '함수형 인터페이스의 추상 메서드 구현'을 직접 전달할 수 있으므로, **전체 표현식을 함수형 인터페이스의 인스턴스로 취급**(기술적으로 따지면 함수형 인터페이스를 concrete 구현한 클래스의 인스턴스)할 수 있다.
- @FunctionalInterface 어노테이션을 붙여, 컴파일 에러 확인가능

Function Descriptor )

- 람다식의 시그니처
- rough하게 표현하면, " (?) → ? "

활용 예제 : 실행 어라운드 패턴 (execute around pattern)

4단계의 과정 ... 전후처리 과정 중복을 줄인다.

@FunctionalInterface

자바는 이미 Comparable, Runnable, Callable 등의 다양한 함수형 인터페이스를 포함하고 있다.

자바8에선 java.util.function 패키지로 Predicate, Consumer, Function 등을 추가로 제공한다.

    @FunctionalInterface
    public interface Predicate<T> {
    	boolean test(T t);
    }

    @FunctionalInterface
    public interface Consumer<T> {
    	void accept(T t);
    }

    @FunctionalInterface
    public interface Function<T, R> {
    	R apply(T t);
    }

자바의 모든 형식은 기본형 or 참조형인데, 여기서 T와 같은 제네릭 파라미터에는 참조형만 사용 가능하다.

※ autoboxing : boxing과 unboxing을 자동으로 수행

boxing : 기본형 → 참조형

unboxing : 참조형 → 기본형

당연히 boxing한 값은 기본형을 감싸는 래퍼이며 Heap에 저장된다. 변환과정도 물론 비용이 소모되어, boxing한 값은 메모리를 더 소비하고 기본형을 가져올때도 탐색과정이 필요하다.

따라서, 이때 자바8은 기본형을 IO로 사용하는 상황에서 autoboxing을 피할수있도록 *특별한 버전의 함수형 인터페이스*를 제공한다.

    // boxing이 일어나지 않음
    IntPredicate evenNumbers = (int i) -> i % 2 == 0;
    // boxing이 일어남
    Predicate<Integer> evenNumbers = (Integer i) -> i % 2 == 0;

[자바8의 대표적인 함수형 인터페이스](https://www.notion.so/06b6c95192cd4f46bb7ff9fc83b4310b)

ㄴ 물론 필요한 함수형 인터페이스를 직접 만들수도 있다.

- 예외, 람다, 함수형 인터페이스의 관계

예외를 던지는 람다 표현식을 만들려면, 예외를 선언하는 함수형 인터페이스를 직접 정의하거나 or 람다는 try/catch 블록으로 감싸야 한다.

### 형식검사, 형식추론, 제약

---

- 형식 검사

람다가 사용되는 context를 이용해 람다의 형식(type)을 추론할 수 있다.

    List<Apple> heavierThan150g = filter(inventory, (Apple a) -> a.getWeight > 150);

람다 표현식이 사용될때 실제로 어떤일이 일어나는지 지켜보자

1. filter 메서드의 선언을 확인한다
2. filter 메서드는 2번째 파라미터로 Predicate<Apple> 형식을 기대한다
3. Predicate<Apple>은 test라는 한개의 추상메서드를 정의하는 함수형 인터페이스다
4. test 메서드는 Apple을 받아 boolean을 반환하면 함수 descriptor를 묘사한다
5. filter 메서드로 전달된 인수는 이와같은 요구사항을 만족해야 한다

: 검사과정을 따라가다가, 함수 descriptor는 Apple → boolean인데 람다의 시그니처도 동일하므로 형식검사 완료.

- 람다의 바디에 일반 표현식이 있으면 void를 반환하는 함수 descriptor와 호환된다

    Predicate<String> p = s -> list.add(s);
    Consumer<String> b = s -> list.add(s);

list.add()는 Consumer 콘텍스트(T → void)가 기대하는 void대신 boolean을 반환하지만, 유효한 코드이다. (즉, 저런 형태의 어느정도 융통성은 가지고 있다는 것) 하지만 가급적 자제하라는것같다.

- 형식 추론

    Comparator<Apple> c = (Apple a1, Apple a2) -> 생략;
    Comparator<Apple> c = (a1, a2) -> 생략;

상황에 따라 명시적으로 형식을 표현하는것이 좋을때도 있고, 형식을 배제하는것이 좋을때도 있다. 상황에 따라 어느 방식이 가독성을 향상시키는지 스스로 판단하자.

(심지어 형식추론대상 파라미터가 1개일땐, "a → @@@"와 같이 괄호까지도 생략할 수 있다)

- 지역 변수의 사용과 제약

지역변수는 스택에 저장되므로 날아갈수있다. 그래서 복사본을 저장하므로 값의 변조가 일어나면 안된다. final 또는 final처럼 사용되야함.

### 메서드 레퍼런스

---

method reference는 특정 람다식을 축약한 느낌

람다식과 메서드 레퍼런스 - 역시 상황에 따라 가독성이 더 좋은방식을 선택할것

[단축 표현 예제](https://www.notion.so/467f90ddb2374b069bb66cd6a989c8cb)

- 만드는 방법

기존 메서드 이용

1. 정적 메서드 레퍼런스 (ex. Integer::parseInt)
2. 다양한 형식의 인스턴스 메서드 레퍼런스 (ex. String::length)
3. 기존 객체의 인스턴스 메서드 레퍼런스 (ex.apple::getCost)

생성자 이용

4. 생성자 레퍼런스 (ex. Apple::new)

- 응용 예제

    static Map<String, Function<Integer, Fruit>> map = new HashMap<>();
    static {
    	map.put("apple", Apple::new);
    	map.put("orange", Orange::new);
    }
    
    public static Fruit giveMeFruit(String fruitName, Integer weight) {
    	return map.get(fruitName)
    						.apply(weight);
    }

- 이제까지의 동작 파라미터화, 익명클래스, 람다표현식, 메서드레퍼런스를 총동원한

    **P.115 [사과 예제] 따라가며 이해하기 ★**

### 람다식을 조합할수있는 유틸리티 메서드

---

몇몇 함수형 인터페이스들은 간단한 몇개의 람다식을 조합해 좀더 복잡한 람다식을 만들수있는 유틸리티 메서드를 제공한다.

물론 이들은 default method이므로 (추상 메서드가 아니므로) 함수형 인터페이스의 정의에 어긋나진 않을지 걱정하지 않아도 된다.

예제 )

    // Comparator 조합
    inventory.sort(comparing(Apple::getWeight)
    					.reversed()
    					.thenComparing(Apple::getCountry));
    // Predicate 조합
    Predicate<Apple> redApple = a -> "red".equals(a.getColor);
    Preficate<Apple> redAndHeavyOrGreen =
    		redApple.and(a -> a.getWeight() > 150)
    						.or(a-> "green".equals(a.getColor)));
    // Function 조합
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    Function<Integer, Integer> h = f.compose(g);
    int result = h.apply(1); // 3