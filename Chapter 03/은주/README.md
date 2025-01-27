# Chapter 03. JDK 의 함수형 인터페이스
## 3.1. 네 가지 함수형 인터페이스
- **Function** : 인수를 받고, 결과를 반환
- **Consumer** : 인수를 받고, 결과를 반환하지 않음
- **Supplier** : 인수를 받지 않고, 결과만 반환
- **Predicate** : 인수를 받아서 표현식에 대해 테스트하고 boolean 값을 결과로 반환

### 3.1.1. Function
- 하나의 입력과 출력을 가진 전통적인 함수
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

Function<String, Integer> stringLength = str -> str != null ? str.length() : 0;
Integer result = stringLength.apply("Hello, Function!");
```

### 3.1.2. Consumer 
- Consumer 는 입력 파라미터를 소비하지만 아무것도 반환하지 않는다
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

Consumer<String> println = str -> System.out.println(str);
println.accept("Hello, Consumer!")
```

### 3.1.3. Supplier
- Consumer 의 반대로, 어떠한 입력 파라미터도 받지 않지만 T 타입의 단일값을 반환한다
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
Supplier<Double> random = () -> Math.random();
Double result = random.get()
System.out.println("random = " + result)
```
- Supplier 는 종종 `지연 실행` 에 사용된다. 비용이 많이 드는 작업을 **Supplier 로 래핑하고 필요할 때만 get을 호출하는 경우**가 있다.

### 3.1.4. Predicate
- Predicate 은 단일 인수를 받아 그것을 로직에 따라 테스트하고, true or false 를 반환한다
```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```
- 필터 메서드와 같이 `의사결정`에 주로 사용된다

## 3.2. 함수형 인터페이스 변형이 많은 경우
- 다양한 함수형 인터페이스 타입은 자바가 람다를 도입하는 과정에서 **하위 호환성**을 유지하기 위해 필수적이다

### 3.2.1. 함수 아리티
- 함수의 인수 개수를 나타내는 `아리티` 는 함수가 받아들이는 피연산자의 수를 의미한다
  - ex. 아리티가 1이다 == 람다는 **단일 인수**를 받아들인다
- JDK 에는 더 높은 아리티를 지원하기 위해 주요 함수형 인터페이스 범주에서 인수를 받아들이는 특수한 변형이 포함되어 있다

|인수가 1개인 경우|인수가 2개인 경우|
|------|---|
|Function<T, R>|BiFunction<T, U, R>|
|Consumer<T>|BiConsumer<T, U>|
|Predicate<T>|BiPredicate<T, U>|
- 자바의 기본적인 인터페이스는 인수를 최대 2개까지 지원하고, 아리티가 1개 또는 2개인 것이 가장 일반적이다.
- 더 높은 아리티를 추가하는 것은 간단하긴 하지만, 함수형 인터페이스는 정적 및 기본 메소드를 통해 추가 기능을 많이 제공한다. <br> 따라서 이런 기능을 사용하는 것이 최상의 호환성과 이해하기 쉬운 사용 패턴을 보장한다
```java
public class ArityCompatibility {

    public static void main(String[] args) {
        UnaryOperator<String> unaryOp = String::toUpperCase;
        Function<String, String> func = String::toUpperCase;

        // 정상 동작
        acceptsUnary(unaryOp);
        acceptsFunction(func);
        acceptsFunction(unaryOp);

        // 컴파일 에러
        acceptsUnary(func);

        // The method acceptsUnary(UnaryOperator<String>) in the type ArityCompatibility is
        // not applicable for the arguments (Function<String,String>)
    }

    private static void acceptsUnary(UnaryOperator<String> unaryOp) {
        // ...
    }

    private static void acceptsFunction(Function<String, String> func) {
        // ...
    }
}
```
- 메서드 인수에 가장 일반적인 타입인 Function<String, String> 을 선택하는 것이 **호환성을 가장 높이는** 방법이다
- **메서드 시그니처의 가독성은 높아지지만 사용성을 극대화하고, 특화된 함수형 인터페이스에 인수를 제한하는 것이 아니기 때문에 권장한다**
- 람다를 생성할 때 특화된 타입을 사용하면, 코드가 더 간결해지고 개발자 본래 의도를 유지할 수 있다

### 3.2.2. 원시 타입
- **원시 타입은 아직까지 제네릭 타입으로 사용될 수 없으므로 원시타입에 대해 특화된 함수형 인터페이스가 존재한다**
- 객체 래퍼 타입에 대해서는 어떤 제네릭 함수형 인터페이스든 사용할 수 있고, 오토박싱이 나머지를 처리하도록 담당할 수 있다
- `오토박싱`은 무료로 처리되지 않기에 **성능에 영향을 미칠 수 있다. 따라서 JDK 에서 제공하는 많은 함수형 인터페이스가 오토박싱을 피하기 위해 원시 타입을 사용하는 이유이다.**
  - 오토박싱 : int -> Integer
  - 언박싱 : Integer -> int

### 3.2.3. 함수형 인터페이스 브리징
- `함수형 인터페이스`는 **이름 그대로 인터페이스**이며, `람다 표현식`은 **이러한 인터페이스들의 구체적인 구현체**이다.
```java
interface LikePredicate<T> {
    boolean test(T value);
}

LikePredicate<String> isNull = value -> value == null;
Predicate<String> wontCompile = isNull;

// 에러:
// 호환되지 않는 타입: LikePredicate<String> 은 
// java.util.function.Predicate<String> 으로 변환될 수 없음
```
- **람다 시점에서보면 두개의 SAM (Single Abstract Method) 은 동일하다.** 둘 다 문자열 인수를 boolean 결과를 반환한다.
- 자바의 타입 시스템에서는 이들 간에 아무런 연결이 없으므로 형변환이 불가하다.
- **'동일하지만 호환되지 않는' 함수형 인터페이스 간에 형변환을 하는 대신, `메소드 참조` 를 사용하면 코드를 컴파일할 수 있도록 SAM 을 참조할 수 있다**
```java
Predicate<String> thisIsFine = isNull::test;
```
- 메소드 참조를 사용하면 함수형 인터페이스를 묵시적 또는 명시적으로 형변환하는 대신, **새로운 동적 호출 지점이 생성되어 바이트코드의 invokedynamic 명령 코드를 통해 호출**된다
- 함수형 인터페이스를 메서드 참조와 연결하는 것은 코드를 다른 방식으로 리팩터링하거나 재설계할 수 없는 경우에 사용할 수 있는 방법이다
  - 레거시 코드베이스에서 함수형 접근 방식으로 전환하거나, 서드 파티 코드에서 제공하는 함수형 인터페이스를 활용할 때 유용하게 사용될 수 있다

## 3.3. 함수 합성
- 함수 합성 : 작은 함수들을 결합하여 더 크고 복잡한 작업을 처리하는 함수형 프로그래밍의 주요 접근 방식
- 새로운 키워드를 도입하거나 언어의 의미를 변경하는 대신 자바의 글루 메서드를 사용한다
  - 글루 메서드는 기본적으로 **함수형 인터페이스 자체에 직접 구현되며, 이를 통해 4가지 카테고리 함수형 인터페이스를 쉽게 합성할 수 있다**
  - 글루 메서드는 결합된 기능을 가진 새로운 인터페이스를 반환함으로써 두 함수형 인터페이스 사이의 연결고리를 만든다

```java
1️⃣
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> apply(before.apply(v));
}

2️⃣
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> after.apply(apply(t));
}
```
- 1️⃣ (compose): before 인수를 입력 (fn2) 하고 **결과를 이 함수 (fn1) 에 적용**하여 `합성된 함수`를 만든다  
  - fn1.compose(fn2) == fn1(fn2(input))
- 2️⃣ (andThen): compose 에 반대되는 함수로, **fn2 함수를 실행한 후**에 `이전 결과`에 after (fn1) 를 적용한다
  - 위와 동일한 결과가 나오려면 fn2.andThen(fn1(input))
- 즉, 두 케이스모두 fn2 함수 실행 후 fn1 함수 합성하는 것!
- 완성된 하나의 문장과 같은 순서로 메서드 호출 체인을 구성하는 것은 함수의 논리적 흐름과 유사하여, 함수형 프로그래밍 네이밍 규칙에 익숙하지 않은 사람이 쉽게 이해할 수 있는 방법이다.
- 함수 합성을 지원하는 4가지 주요 인터페이스
  - Function<T,R> : 양방향 합성 지원
  - Predicate<T> : 새로운 Predicate 을 합성하기 위한 and, or, negate 지원
  - Consumer<T> : andThen 만 지원하며, 두개의 Consumer 를 순차적으로 합성하여 값을 받아들임

## 3.4. 함수형 지원 확장
- 직접 만든 타입을 더 함수적으로 만들 수 있는 방법
  - 기존 타입을 더 함수적으로 마들기 위해, 인터페이스에 디폴트 메서드 추가
  - 함수형 인터페이스를 명시적으로 구현
  - 공통 함수형 작업을 제공하기 위해 정적 헬퍼 생성

### 3.4.1. 기본 메서드 추가
- 인터페이스의 계약을 변경하고 이를 구현하는 **모든 타입에 새로운 메서드를 추가하는 대신 기본 메서드를 사용하여 '상식적인' 구현을 제공할 수 있다**
  - 의도된 로직의 일반적인 변형을 모든 타입에 적용하여 UnsupportedOperationException 을 사용할 필요가 없다
- 기본 메서드를 추가하면 코드의 하위 호환성을 유지할 수 있다
- 아래는 JDK 가 java.util.Collection<E> 인터페이스를 구현하는 모든 타입에 Stream 기능을 추가하는 예시이다.
```java
public interface Collection<E> extends Iterable<E> {

    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
    
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}
```
- 필요에 따라 재정의해서도 사용할 수 있다.
  - 기본 메서드의 계층 구조를 통해 인터페이스에 새로운 기능을 추가한다
  - 이렇게 하면 **기존의 구현을 깨뜨리지 않고 새로운 메서드의 상식적인 변형을 제공할 수 있다**
```java
public interface Iterable<T> {

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}

public interface Collection<E> extends Iterable<E> {

    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }
}

public class ArrayList<E> extends AbstractList<E> implements List<E>, ... {

    @Override
    public Spliterator<E> spliterator() {
        return new ArrayListSpliterator(0, -1, 0);
    }
}

```

### 3.4.2. 함수형 인터페이스 명시적으로 구현하기
- 함수형 인터페이스는 람다나 메서드 참조를 통해 묵시적으로 구현될 수도 있지만, **더 높은 차수의 함수에서 사용가능하도록 명시적으로 구현하여 사용**할 수도 있다.
- 함수형 인터페이스 간의 논리적 동등성만으로는 호환성을 만들 수 없으므로 TextEditorCommand 를 Supplier<String> 으로 확장함으로써 격차를 메꿔줄 수 있다.
```java
public interface TextEditorCommand extends Supplier<T> {
    String execute();

    default String get() {
        return execute();
    }
}
```
- **함수형 인터페이스의 SAM 은 실제 작업을 수행하는 메서드를 호출하는 간단한 기본메서드이다**
- 이렇게 함으로써 모든 커맨드들이 Supplier<String> 를 인수로 받는 어떤 고차함수와도 호환성을 갖추게 되고, 이 경우 **메서드 참조를 브릿지로 사용하지 않아도 된다**
  - ex) Supplier<String> supplier = command::execute;
- 하나 이상의 함수형 인터페이스를 구현하는 것은, 타입에 함수형 시작점을 제공해줄 수 있으며 **함수형 인터페이스에서 사용 가능한 모든 추가 디폴트 메서드도 포함된다**

### 3.4.3. 정적 헬퍼 생성하기
- JDK 자체에서 제공하는 함수형 인터페이스와 같이 타입을 직접 제어할 수 없는 경우, 정적 메서드를 모으는 헬퍼 타입을 만들 수 있다
```java
1️⃣
default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
    Objects.requireNonNull(before);
    return (V v) -> {
        T result = before.apply(v);
        return apply(result);
    }
}
// 타입 체인 : V -> T -> R

2️⃣
default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
    Objects.requireNonNull(after);
    return (T t) -> {
        R result = apply(t);
        return after.apply(result);
    }
}
// 타입 체인 : T -> R -> V
```
- Function<T,R> 과 Supplier/Consumer 에 대한 컴포지터를 만들어보자
```java
public class CompositorUsage {

    public static void main(String... args) {

        // 단일 문자열 함수
        Function<String, String> removeLowerCaseA = str -> str.replace("a", "");
        Function<String, String> upperCase = String::toUpperCase;

        // 합성된 문자열 함수
        Function<String, String> stringOperations = removeLowerCaseA.andThen(upperCase);

        // 문자열 함수와 Consumer 의 합성 (문자열 함수 -> Consumer 실행)
        Consumer<String> composedConsumer = Compositor.compose(stringOperations, System.out::println);

        composedConsumer.accept("abcd"); // BCD 출력

        // ACCEPTIF
        Consumer<String> acceptIfNotNull = Compositor.acceptIf(str -> str != null, System.out::println);

        acceptIfNotNull.accept("this is printed");
        acceptIfNotNull.accept(null);
    }

    static class Compositor {

        static <T, R> Supplier<R> compose(Supplier<T> before, Function<T, R> fn) {
            Objects.requireNonNull(before);
            Objects.requireNonNull(fn);

            return () -> { // Supplier 는 인수를 받을 수 없음
                T result = before.get();
                return fn.apply(result);
            };
        }

        static <T, R> Consumer<T> compose(Function<T, R> fn, Consumer<R> after) {
            Objects.requireNonNull(fn);
            Objects.requireNonNull(after);

            return (T t) -> { 
                R result = fn.apply(t);
                after.accept(result);
            };
        }

        static <T> Consumer<T> acceptIf(Predicate<T> predicate, Consumer<T> consumer) {
            Objects.requireNonNull(predicate);
            Objects.requireNonNull(consumer);

            return (T t) -> {
                // 특정 수준의 논리와 결정을 추가할 수 있음
                if (!predicate.test(t)) {
                    return;
                }
                consumer.accept(t);
            };
        }

        private Compositor() {
        }
    }
}
```

## 핵심 요약
- 특화된 함수형 인터페이스 변형은 최대 2개의 인수를 지원한다.
  - 그러나 **메서드 시그니처는 최대한 호환성을 극대화하기 위해 해당하는 슈퍼 인터페이스를 사용하는 것이 좋다**
- **동일하지만 호환되지 않는 함수형 인터페이스 사이의 격차를 극복하기 위해** SAM 의 `메서드 참조`를 사용할 수 있다
- 인터페이스에 기본 메서드를 사용하여 구현을 변경하지 않고도 함수형 사용 사례를 다룰 수 있다
- 일반적인, 누락된 함수형 작업은 정적 메서드가 있는 헬퍼타입에 누적될 수 있다