# Ch 10. null 대신 Optional

Java 8 in Action :: Contents

### null 때문에 발생하는 문제

- 에러의 근원 : NPE는 자바에서 가장 흔히 발생하는 문제
- 코드 복잡도 증가 : 여기저기서 null 확인코드를 추가하며 코드가 어지러워진다
- 아무 의미가 없음 : 특히 정적형식언어에서 '값이없음'을 표현하는 방법으로 적절하지않다
- 자바 철학 위배 : 자바는 개발자로부터 모든 포인터를 숨겼으나, 예외가 바로 null 포인터
- 형식 시스템에 구멍을 만듬 : 시스템의 다른부분으로 null이 퍼지면서, 애초에 null이 어떤 의미로 사용되었는지 알기어려워짐

⇒ 다른 언어에서 null 레퍼런스를 해결하는 방식

- 그루비 : 안전 내비게이션 연산자

    ( ?. )

- 하스켈/스칼라 : 선택형 값(Optional Value) - 값을 갖거나 갖지않을 수도 있는 구조

    ( Maybe/Option[T] )

## Optional 클래스

---

자바8은 하스켈과 스칼라의 영향을 받아 java.util.Optional<T>라는 선택형값을 캡슐화하는 클래스를 제공한다.

Optional 클래스는 값이 있으면 값을 감싸고, 없으면 Optional.empty()로 싱글턴 인스턴스를 반환한다. Optional.empty()와 null 레퍼런스는 의미상으론 비슷하지만 실제론 차이가 있으며, 이에 대해 곧 언급된다.

대신 null을 참조하려하면 NPE가 발생하지만, Optional.empty()는 Optional 객체이므로 다양하게 활용할 수 있다.

Optional<T>를 사용하면, 이는 값이 없을수도있음을 명시적으로 보여주며 의미를 더 명확하게 한다. 교재 예시의 경우, Car 객체는 Insurance를 가질수도 or 안가질수도 있는 것이다. 반면 Insurance 객체는 String타입의 name 필드를 무조건 갖도록 강제시키는 것이다.

## Optional 적용 패턴

---

### Optional 객체 만들기

- Optional.empty() : 본 정적 팩토리 메서드로 빈 Optional 객체 반환
- Optional.of(T) : null이 무조건 아닌 값을 포함하는 Optional 객체 반환 (T가 null이면 에러 발생)
- Optional.ofNullable(T) : T는 null일수도 아닐수도. null이면 빈 Optional 객체 반환

### Map으로 Optional의 값을 추출 및 변환

Optional을 요소가 1개인 스트림으로 생각해보자

    String name = null;
    if(insurance != null) {
    	name = insurance.getName();
    }

⇒

    Optional<Insurance> optInsurance = Optional.ofNullable(insurance);
    Optional<String> name = optInsurance.map(Insurance::getName);

### flatMap으로 Optional 객체 연결

'예제에서' map 연산의 결과는 Optinal<Optional<Car>> 형식이다. 이때 flatMap을 이용해 이차원을 일차원으로 평준화한다

    public String getCarInsuranceName(Optional<Person> person) {
    		return person.flatMap(Person::getCar)
    									.flatMap(Car::getInsurance)
    									.map(Insurance::getName)
    									.orElse("Unknown");
    }

- [ ]  도메인 모델에서의 Optional 사용

사실 Optional 클래스는 선택형 반환값을 지원하는 용도로만 설계되었다.

Optional 클래스를 도메인 모델의 값으로, 즉 필드 형식으로 사용할 것을 고려하지 않았고, 따라서 Serializable 인터페이스를 구현하지 않는다.

교재에선 그럼에도 불구하고 Optional을 사용해 도메인 모델을 구성하는 것이 바람직하다고 생각하고 있으나, 우테코에서 피드백 등을 통해 접하는 분위기는 권장하지 않는 것 같다. return 용도로만.

### 디폴트 액션과 Optional 언랩

- get() : 값을 읽는 가장 간단한 메서드면서 동시에 위험한 메서드 (값이 없으면 예외를 발생시키며, 값이 반드시 있다고 보장할수있는 상황이 아니면 가급적 자제. 결국 중첩된 null 확인 코드를 넣는것과 크게 다르지 않다)
- orElse() : 값이 없을때 디폴트값 제공 가능
- orElseGet() : orElse의 게으른 버전. 값이 없을때만 Supplier 실행
- orElseThrow() : 비었을때 예외를 발생시키는건 get과 비슷하나, 발생시킬 예외 선택가능
- ifPresent() : 값이 존재할때, 인수로 넘긴 동작 실행가능

### 두 Optional 합치기

Optional을 사용했지만 null 확인코드와 비슷해진 코드와, 이를 개선해서 Optional을 언랩하지 않고 두 Optional을 합치는 예제

### filter로 특정값 거르기

역시, if문과 get()을 이용해 기존과 비슷한 형태의 코드보다, 이를 filter를 이용해 개선하는 예제

→ 교재 참고

## Optional을 활용한 실용 예제

---

- 잠재적으로 null이 될 수 있는 대상을 Optional로 감싸기

    기존 자바 API를 바꿀순 없으므로, 우리가 받는 부분에서 간단히 감싸서 사용한다.

    예를 들어, `Object value = map.get("key");` 보단

     `Optional<Object> value = Optional.ofNullable(map.get("key"));`

- 예외와 Optional

    `Integer.partInt(String)` 은 실패시 NumberFormatException을 발생시킨다. 이에 따라 거추장스러운 try/catch가 생기는데, 간단히 다음과 같은 유틸리티 메서드를 만들어 감추고 Optional<Integer>를 반환하도록 한다.

        public static Optional<Integer> stringToInt(String s) {
        		try {
        				return Optional.of(Integer.parseInt(s));
        		} catch (NumberFormatException e) {
        				return Optional.empty();
        		}
        }

- 기본형 Optional은 사용을 자제한다

    스트림처럼 Optional도 OptionalInt/Long/Double 등이 존재한다. 하지만 어짜피 이들은 요소가 1개이므로 성능을 향상시킬 수 없다. 또 이들은 map, flatMap, filter와 같은 유용한 메서드들을 제공하지 않는다. 기본형 특화 Optional들은 일단 Optional과 혼용할수도없어 더욱 사용이 권장되지 않는다.

⇒ Optional을 사용하면서, 자바8 이전에 null 체크하던것처럼 코딩하지 않도록 주의해야할 것 같다.

Optional 안쓰느니만 못하고, 함수형 사고로 바꿔야한다.