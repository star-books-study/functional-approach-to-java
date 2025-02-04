# Chapter 04. 불변성
- 함수형 접근 방식에서는 `불변성` 이 데이터를 처리하는 주요 방법이며 다양한 개념의 전제조건이다

## 4.1. 객체 지향 프로그래밍의 가변성과 자료 구조
- 객체 지향 언어인 자바에서는 **객체의 상태를 가변 형태로 캡슐화**한다
- 객체 지향 자바 코드에서 가변 상태를 다루는 가장 일반적인 형태로는 JavaBean, POJO 가 있다
  - 컴포넌트 간의 **재사용성을 향상**시키기 위해 상태값을 캡슐화하도록 설계된 일반적인 자바 객체들이다
- POJO는 설계에 대해 어떠한 제한도 없다.
  - POJO : Plain Old Java Object / 환경과 기술에 종속되지 않고 필요에 따라 재활용될 수 있는 방식으로 설계된 오브젝트
  - 비즈니스 로직 상태를 **'단지' 캡슐화 하는 것이 목적**이다
  - 가변 상태를 가진 객체지향적인 컨텍스트에서 더 유연하게 작동하도록 필드에 getter/setter 를 제공한다
- 반면 JavaBean 은 내부 검사와 재사용성을 가능하게 해주며, 이를 위해 특정 규칙을 따르도록 되어 있다

| | POJO | JavaBean |
|---|------|---|
| 일반 제한 |자바 규칙에 따름|JavaBean API 사양에 따름|
| 직렬화 |선택적|java.io.Serializable 상속해야함|
| 필드 가시성 |제한 없음|private 만 가능|
| 필드 접근 |제한 없음|getter, setter 를 통해서만 접근 가능|
| 생성자 |제한 없음|인수가 없는 생성자가 존재해야 함|

- 가변 상태는 **복잡성과 불확실성을 유발**한다
  - 가변 상태를 공유하는 것은 공유된 상태에 액세스하는 컴포넌트의 수명을 포함하여 복잡성을 증가시킨다
  - 특시 동시성 프로그래밍은 공유된 상태의 복잡성에 영향을 받고, 많은 문제가 가변성에서 발생한다

## 4.2. 함수형 프로그래밍의 불변성
- 불변성의 기본 원칙은 단순하다
  - **자료 구조는 생성 후에 변경할 수 없어야 한다**
  - 이 원칙은 함수형 프로그래밍 언어에서 기본으로 지원한다
- 불변 자료 구조는 데이터에 대한 지속적인 뷰를 제공하지만 직접 데이터를 변경할 수는 없다
- 자료구조를 변경하려면 **의도한 변경사항을 반영한 새로운 복사본을 생성해야 한다**
- 데이터를 직접 변경할 수 없으며 오버헤드를 발생시킨다는 점이 이상할 수 있지만, 불변성은 함수형 접근 방식을 넘어 자바에서는 충분한 가치가 있다
- **예측 가능성**
  - 자료구조를 참조하는 한, 생성된 시점과 `동일한` 상태임을 알 수 있다
- **유효성**
  - 초기화한 후, 자료 구조는 `완전한` 상태가 된다
  - 단, 한번의 검증만 필요하며 유효 상태로 유지된다
- **숨겨진 사이드 이펙트 없음**
  - 불변 자료 구조는 항상 그대로이기 때문에, 사이드 이펙트가 발생하지 않는다
- **스레드 안전성**
  - 사이드 이펙트가 발생하지 않는 불변 자료 구조는 `스레드 경계를 자유롭게 이동`할 수 있다
  - 프로그램에 대한 추론이 보다 간단해지며, 예기치 않은 변경 또는 경합조건 없이 프로그램을 이해할 수 있다
- **캐시 가능성 및 최적화**
  - 자료구조 생성 직후부터 변경되지 않기에 불변 자료 구조를 신뢰하고 `캐싱`할 수 있다
  - **메모이제이션과 같은 최적화 기술은 불변 자료구조에서만 가능하다**
- **변경 추적**
  - 모든 변경이 새로운 자료구조를 생성한다면, 이전 참조를 저장함으로써 이전 상태를 `추적`할 수 있다
- 이 모든 이점은 **선택한 프로그래밍 패러다임과는 독립적이다**

## 4.3. 자바 불변성 상태
- 불변성은 코드를 더 안전하고 오류를 적게 발생시키도록 만들어준다

### 4.3.1. java.lang.String
- 문자열은 어디에서나 사용되기에 String 타입에는 **높은 최적화와 안전성이 필요**하다.
  - 최적화 방법 중에는 불변성이 있다
- 문자열은 + 연산자로 연결할 때마다 메모리의 힙 영역에 새로운 String 인스턴스가 생성되어 메모리를 차지하게 된다
  - 물론 JVM 에서는 필요 없는 인스턴스를 GC 를 통해 정리하지만, 끊임없이 생성되는 **String 객체로 인한 메모리 오버헤드는 실제 런타임에 부담이 될 수 있다**
- JVM 은 문자열 연결을 java.lang.`StringBuilder` 로 대체하거나, invokedynamic 명령 코드를 사용하는 등 여러 최적화 전략을 내부적으로 사용한다
  - Java 9 이후 **JVM에서 문자열 연결은 동적 메소드 호출**(바이트코드의 invokedynamic 명령어)을 통해 수행된다. 이러한 방식은 이전 구현보다 더 빠르게 작동하고 불필요한 중간 객체 생성을 줄이기에 메모리를 덜 소비하며, 바이트코드 변경 없이 향후 최적화를 위한 공간을 남겨 둔다.
    - JVM은 `StringConcatFactory.makeConcatWithConstants()` 또는 StringConcatFactory.makeConcat() (내부적으로 makeConcatWithConstants 메소드 호출)을 사용하여 문자열에 대해 invokedynamic 연결을 수행
- `문자열 리터럴`도 JVM 에서 특별한 관리를 받는데, **문자열 풀링 덕분에 동일한 리터럴은 한번만 저장되어 재사용되므로 힙메모리 공간 절약**에 도움이 된다
  - String Pool : 자바에서 문자열을 효율적으로 관리하기 위한 개념으로, **JVM 의 힙 메모리 영역에 위치하며, 문자열 리터럴을 저장하는 공간**
    - 자바에서 문자열 리터럴은 불변이기 때문에, 한 번 생성된 문자열은 변경되지 않고 재사용된다
    - 동일한 문자열 리터럴에 대해 여러 참조가 같은 객체를 가리킬 수 있다
    - 스트링 풀은 GC의 대상이 될 수 있는데, 자바 7 이전까지는 스트링 풀이 힙 메모리의 영구 세대(PermGen)에 위치했으나, 자바 7부터는 힙 메모리의 일부로 이동되었기 때문이다. 힙 메모리의 일부로 관리되기 때문에, 더 이상 사용되지 않는 문자열 객체는 가비지 컬렉터에 의해 수집될 수 있다
- 기술적 관점에서 String 타입은 "완전히" 불변하진 않다. 
- 성능 고려로 인해 String 은 hashCode 를 지연해서 계산한다. **동일한 String 은 항상 동일한 hashCode 를 생성한다**
  - hashCode() 가 처음 호출될 때까지 해시코드 계산을 미루는데, 이 과정에서 hashCode 값을 저장하는 내부 필드가 변경될 수 있다.
  - 하지만 이런 내부 구현의 변경이 String 객체의 불변성을 위반하지는 않는다. hashCode의 지연 계산은 **불변성의 핵심 특성을 훼손하지 않으면서 성능을 개선하기 위한 내부 구현 세부사항**이다

### 4.3.2. 불변 컬렉션
- 자바에서 불변성을 제공하는 3가지 방법
  - `변경 불가능한 컬렉션`
    - 불변 컬렉션 : **생성 후 그 상태가 절대로 변경되지 않음**. 컬렉션에 요소를 추가, 삭제, 변경 불가
    - 변경 불가능한 컬렉션 : **특정 컬렉션 뷰에 대해 수정 작업을 실행할 수 없음**. 이 뷰를 통해서는 데이터를 변경할 수 없지만, **원본 컬렉션 자체는 참조를 통해 변경 가능**.
  - `불변 컬렉션 팩토리 메서드`
  - `불변 복제`
- 하지만 이들은 얕은 불변성만을 갖고 있어, 요소 자체의 불변성을 보장하지 않는다
  - 얕은 불변성 : 최상위 계층에서만 불변성 유지. **자료 구조 자체의 참조는 변경되지 않지만,** 컬렉션과 같이 **참조된 자료 구조의 요소들은 변경될 수 있다**

#### 변경 불가능한 컬렉션
- Collection<T> unmodifiableCollection(Collection<? extends T> c)
- Set<T> unmodifiableSet(Set<? extends T> s)
- List<T> unmodifiableList(List<? extends T> list)
- Map<K,V> unmodifiableMap(Map<? extends K, ? extends V> m)
```java
List<String> modifiable = new ArrayList<>();
modifiable.add("blue");
modifiable.add("red");

List<String> unmodifiable = Collections.unmodifiableList(modifiable);
unmodifiable.clear();

// Exception java.lang.UnsupportedOperationException
//       at Collections$UnmodifiableCollection.clear (Collections.java:1086)
//       at (#5:1)
```
- 반환된 인스턴스를 수정할 때 UnsupportedOperationException 발생
- **'변경 불가능한 뷰' 의 명백한 단점은 기존 컬렉션에 대한 추상화에 불과하다**는 점이다
- 여전히 **기본 컬렉션은 변경될 수 있으며**, 원본에 직접적인 변경이 이루어질 경우 뷰가 추구하는 '변경 불가능한 특성' 을 우회하게 된다
- '변경 불가능한 뷰' 는 **주로 반환값으로 사용될 컬렉션에 대해 원치않는 변경을 막기 위해 사용**된다 (원본값에 대한 변경은 막을 수 없기 때문 !)
  - 원본 데이터에 대한 참조가 필요하고, 원본 데이터의 변경에 따라 불변 뷰의 값도 함께 변경되어야 하는 경우에 사용하면 좋다

#### 불변 컬렉션 팩토리 메소드
- List<E> of(E e1, ...)
- Set<E> of(E e1, ...)
- Map<K,V> of(K k1, V v1, ...)
```java
static <E> List<E> of(E e1) {
    return new ImmutableCollections.List12<>(e1);
}
```
- unmodifiableList()와 다른 점이자 중요한 점은 뷰가 아닌 **객체의 복사본을 따로 생성한다**는 점이다.
- Stream 의 toList() 도 복사본을 따로 생성하므로 불변 리스트를 만들 수 있다
    ```java
    default List<T> toList() {
        return (List<T>) Collections.unmodifiableList(new ArrayList<>(Arrays.asList(this.toArray())));
    }
    ```
- Arrays.asList() 는 조금 애매하다. add, remove는 불가능하나, set() 메소드로 기존 원소의 값을 변경하는 것은 가능하다
    > Arrays.asList()와 List.of()의 차이
    > - Arrays.asList()는 add, remove 등은 안되지만(UnsupportedOperationException 발생), set을 통해서 특정 인덱스에 있는 원소를 바꾸는 것은 가능하다. 
    > - 하지만 List.of()는 add, remove, set 모두 불가능하다.

#### 불변 복제
- Set<E> copyOf(Collection<? extends E> col1)
- List<E> copyOf(Collection<? extends E> col1)
    ```java
    static <E> List<E> copyOf(Collection<? extends E> coll) {
        return ImmutableCollections.listCopy(coll);
    }
    ```
- Map<K,V> copyOf(Map<? extends K, ? extneds V> map)
- copyOf 메서드는 새로운 컨테이너를 생성하여 **요소들의 참조를 독립적으로 유지**한다
```java
List<String> original = new ArrayList<>();
original.add("blue");
original.add("red");

List<String> copiedList = List.copyOf(original);

original.add("green");

System.out.println("original = " + original) // blue, red, green
System.out.println("copy = " + copiedList) // blue, red
```

### 4.3.3. 원시 타입과 원시 래퍼
- 원시 타입은 리터럴 또는 표현식을 통해 초기화되는 단순한 값을 나타낸다
- 이들은 각자 하나의 값을 표현하며 **사실상 불변의 특성을 갖는다**

### 4.3.4. 불변 수학
- java.math 패키지에는 정수와 소수점 계산을 더욱 안전하고 정확하게 처리하기 위한 java.math.BigInteger, java.math.BigDecimal 불변클래스가 제공된다
- 더 넓은 범위와 높은 정밀도로 사이드 이펙트가 없는 계산을 수행할 수 있기에 불변성을 유지해야 한다
- add, subtract 같은 메서드는 최소한 객체지향적 맥락에서는 값의 수정을 의미할 것 같지만, java.math 타입에서는 **새로운 결과를 가진 새로운 객체를 반환한다**
```java
public class ImmutableMath {
    public static void main(String[] args) {

        BigDecimal theAnswer = new BigDecimal(42);
        BigDecimal result = theAnswer.add(BigDecimal.ONE);

        System.out.println("result = " + result); // 43
        System.out.println("theAnswer = " + theAnswer); // 42
    }
}
```
- 불변 수학 타입은 일반적인 오버헤드를 동반하는 객체지만, **높은 정밀도를 달성하기 위해서는 추가 메모리가 필요하다**

### 4.3.5. 자바 시간 API (JSR-310)
- 자바 8 에서는 **불변성을 핵심 원칙으로 하는 자바 시간 API (JSR-310) 을 도입**했다
- java.util 패키지의 Date, Calendar, TimeZone 을 사용하는 대신, **java.time** 패키지를 통해 다양한 정밀도를 가진 여러 날짜 및 시간 타입을 사용할 수 있다

### 4.3.6. enum
- 자바 열거형은 상수로 구성된 특별한 타입이다

### 4.3.7. final 키워드
- final 키워드는 클래스, 메서드, 필드 또는 참조에 사용될 수 있다
  - final 클래스 : **하위 클래스화** 불가
  - final 메서드 : **오버라이딩** 불가
  - final 필드 : **재할당** 불가
  - final 변수 참조 : 필드처럼 동작하며 재할당 불가. **참조형 변수에 들어있는 참조값만 변경하지 못한다는 뜻이고 참조하는 대상의 값은 변경 가능**
    ```java
    public static void main(String[] args) {
        final Data data = new Data();
        // data = new Data();  // final 변경 불가 컴파일 에러

        // 참조 대상의 값은 변경 가능
        data.value = 10;
        System.out.println(data.value); // 10
        data.value = 20;
        System.out.println(data.value); // 20
    }
    ```
