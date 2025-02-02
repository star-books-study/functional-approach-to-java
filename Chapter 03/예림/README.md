# Chapter 03. JDK의 함수형 인터페이스

- JDK는 함수형 도구를 시작하는 데 도움이 되도록 `java.util.function` 패키지를 통해 40개 이상의 함수형 인터페이스 제공

### 3.1 네 가지 함수형 인터페이스
- Function : 인수를 받고 결과를 반환한다.
- Consumer : 인수만 받고 결과를 반환하지 않는다.
- Supplier : 인수를 받지 않고 결과만 반환한다.
- Predicate : 인수를 받아서 표현식에 대해 테스트하고 boolean 값을 결과로 반환한다.


### 3.1.1 Function
- 하나의 입력과 출력을 가진 전통적인 함수를 의미
- `Function<T, R>`의 단일 추상 메서드는 `apply`로 `T` 타입의 인수를 받아 `R` 타입의 결과를 생성
```java
@FunctionalInterface
public interface Fucntion<T, R> {
    R apply(T t);
}
```

### 3.1.2 Consumer
- 입력 파라미터를 "소비"하지만 아무것도 반환하지 않는다.
- `Consumer<T>`의 단일 추상 메서드는 `accept`로 T 타입의 인수를 필요로 한다.
```java
@FunctionalInterface
public interface Comsumer<T> {
    void accept(T t);
}
```
- 값의 소비만이 '순수한' 함수형 개념에 완벽하게 부합하지 않을 수 있지만, 비함수형 코드와 고차 함수 사이의 많은 간극을 극복하는 역할을 한다.
- `Comsumer<T>`는 `java.util.concurrent` 패키지의 자바5+ Callable<V>와 유사하다. 다만, 후자는 checked Exception을 던진다.

### 3.1.3 Supplier
- 어떠한 입력 파라미터도 받지 않지만, T 타입의 단일 값을 반환
- `Supplier`의 단일 추상 메서드는 `get`으로 정의된다.
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```
- 종종 지연 실행에 사용
    - 비용이 많이 드는 작업을 Supplier로 래핑하고 필요할 때만 get 호출
### 3.1.4 Predicate
- 단일 인수를 받아 그것을 로직에 따라 테스트하고 true 또는 false 반환
- `Predicate<T>`의 단일 추상 메서드는 test로, T 타입의 인수를 받아 boolean 타입을 변환한다.
```java
@FunctionalInterface
public interface Predicate<T> {
    booelan test(T t);
}
```
- 필터 메서드와 같이 의사 결정에 주로 사용됨

## 3.2 함수형 인터페이스 변형이 많은 이유
- 네 가지 함수형 인터페이스로도 다양한 사용 사례를 충분히 다룰 수 있지만 이보다 더 특화된 변형도 존재
- 이런 다양한 타입은 자바가 람다를 도입한느 과정에서 하위 호환성을 유지하기 위해 필수적
- 다양한 함수형 인터페이스들 간의 연결 방법이 달라서 각각의 상화에 따라 최적의 방법으로 사용해야 함

### 3.2.1 함수 아리티
- 함수의 인수 개수를 나타내는 아리티는 함수가 받아들이는 피연산자의 수를 의미
  - ex. 아리티가 1인 경우 람다는 단일 인수를 받아들이는 것을 의미
    ```java
    Function<String, String> greeterFn = name -> "Hello " + name;
    ```
- SAM에는 각 아리티에 대한 명시적인 함수형 인터페이스가 있어야 한다.
- JDK에는 더 높은 라이티를 지원하기 위해 주요 함수형 인터페이스 범주에서 인수를 받아들이는 특수한 변형이 포함되어 있다.
  | 인수가 1개인 경우 | 인수가 2개인 경우 |
  | ----------------- | ----------------- |
  | Function<T, R>    | BiFunction<T, U, R> |
  | Consumer<T>       | BiConsumer<T, U>    |
  | Predicate<T>      | BiPredicate<T, U>   |

- 자바의 기본적인 인터페이스들은 인수를 최대 2개까지 지원
- 자바의 함수형 API와 사용 사례를 살펴보면 아리티가 1개 또는 2개인 것이 가장 일반적
- 더 높은 아리티를 추가하는 것은 다음 코드처럼 매우 간단
```java
@FunctionalInterface
public interface TriFunction<T, U, V, R> {
    R accept(T t, U u, V v);
}
```
- 하지만 이런 방식을 사용하지 않는 것을 권한
  - 정적 및 기본 메서드를 사용하는 게 최상위 하휘호환성 & 이해하기 쉬운 사용 패턴을 보장
- 함수형 연산자의 개념은 가장 일반적으로 사용되는 두 아리티를 간소화하여 동일한 제네릭 타입을 가진 함수형 인터페이스를 제공
```java
@FunctionalInterface
interface BinaryOperation<T> extends BiFunction<T, T, T> {
    // ...
}
```
- 사용 가능한 Operator 함수형 인터페이스는 다음과 같다.
  | 연산자 | 슈퍼 인터페이스 |
  | ----------------- | ----------------- |
  | UnaryOperator<T, R>    | Function<T, U, R> |
  | BinaryOperator<T>       | BiFunction<T, T, T>    |

- 하지만 연산자 타입과 해당 상위 인터페이스는 상휘 호환 불가
- 예를 들어 어떤 메서드 시그니처가 UnaryOperator<String>을 요구하는 경우 Function<String, String>과 호환되지 않을 수 있다. 그러나 그 반대의 경우는 가능하다.
```java
UnaryOperator<String> unaryOp = String::toUpperCase;
Function<String, String> func = String::toUpperCase;

void acceptsUnary(UnaryOperator<String> unaryOp) { ... };
void acceptsFunction(Function<String, String> func) { ... };

acceptsUnary(unaryOp); // OK
acceptsUnary(func); // 컴파일 오류

acceptsFunction(func); // OK
acceptsFunction(unaryOp); // OK
```
- 이 예제에서는 가장 일반적인 타입, 즉 Function<String, String>을 선택하는 것이 호환성을 높이는 방법임

### 3.2.2 원시 타입
- 지금까지 만난 대부분의 함수형 인터페이스는 제네릭 타입 정의를 가지고 있었지만 항상 그렇지 않음
- 원시 타입은 아직까지 제네릭 타입으로 사용될 수없음. 그래서 원시 타입에 특화된 함수형 인터페이스가 존재
- 객체 래퍼 타입에 대해서는 어떤 제네릭 함수형 인터페이스든 사용할 수 있으며 오토박싱이 나머지를 처리하도록 담당할 수 있다. 그러나 오토박싱은 성능에 영향을 미칠 수 있다.
  - 오토박싱 : 기본값 -> 객체 기반 상대 타입
  - 언박싱 : 객체 기반 상대 타입 -> 기본값
  - 
- 그래서 JDK에서 제공하는 많은 함수형 인터페이스가 오토박싱을 피하기 위해 원시 타입을 사용함
- 원시 함수형 인터페이스는 int, long, double과 같은 숫자형 원시 타입 중심으로 제공되지만 **모든 원시 타입을 사용할 수 있는 것은 아님**
   -> 원시 함수형 인터페이스를 통해 오토박싱을 사용할 때 발생하는 불필요한 오버헤드를 줄일 수 있음

### 3.2.3 함수형 인터페이스 브리징
- 함수형 인터페이스는  이름 그대로 인터페이스이며, 람다 표현식은 이러한 인터페이스들의 구체적인 구현체이다.
```java
interface LikePredicate<T> {
    boolean test(T value);
}

LikePredicate<String> isNull = value -> value == null;
Predicate<String> wontCompile = isNull;

// 에러:
// 호환되지 않는 타입: LikePredicate<String> 은 
// java.util.function.Predicate<String> 으로 변환될 수 없음

Predicate<String> wontCompilerEither = (Predicate<String>) isNull;
// 예외 java.lang.ClasscastException : 클래스 LikePredicate을
// java.util.function.Predicate로 형변환할 수 없음
```
- 람다 시점에서보면 두개의 SAM (Single Abstract Method) 은 동일하다. 둘 다 문자열 인수를 boolean 결과를 반환한다.
- 동일하지만 호환되지 않는' 함수형 인터페이스 간에 형변환을 하는 대신, 메소드 참조 를 사용하면 코드를 컴파일할 수 있도록 SAM 을 참조할 수 있다.
```java
Predicate<String> thisIsFine = isNull::test;
```
- 메소드 참조를 사용하면 함수형 인터페이스를 묵시적 또는 명시적으로 형변환하는 대신, 새로운 동적 호출 지점이 생성되어 바이트코드의 invokedynamic 명령 코드를 통해 호출된다.

### 3.3 함수 합성
- 함수 합성은 작은 함수들을 결합하여 더 크고 복잡한 작업을 처리하는 함수형 프로그래밍의 주요 접근 방식이다.
- 새로운 키워드를 도입하거나 언어의 의미를 변경하는 대신 자바의 글루 메서드를 사용한다.
- 글루 메서드는 기본적으로 함수형 인터페이스 자체에 직접 구현되며, 이를 통해 4가지 카테고리 함수형 인터페이스를 쉽게 합성할 수 있다.
- Function<T, R>의 경우 두 가지 기본 메서드를 사용할 수 있다.
  - compose
  - andThen
- 첫 번째 compose 메서드는 before 인수를 입력하고 결과를 이 함수에 적용하여 합성된 함수를 만든다.
- 두 번째 andThen은 compose에 반대되는 메서드로 함수를 실행한 후에 이전 결과에 after를 적용한다.


## 3.4 함수형 지원 확장
- 대부분 함수형 인터페이스는 기본 메서드, 정적 헬퍼도 제공
- JDK에서 사용하는 세 가지 접근 방식과 마찬가지로 여전히 직접 만든 타입을 더 함수적으로 만들 수 있음.
  - 기존 타입을 더 함수적으로 마들기 위해, 인터페이스에 디폴트 메서드 추가
  - 함수형 인터페이스를 명시적으로 구현
  - 공통 함수형 작업을 제공하기 위해 정적 헬퍼 생성

### 3.4.1 기본 메서드 추가
- 인터페이스에 새로운 기능을 추가할 떄는 항상 새로운 메서드를 구현해야 하는데, 이런 상황에서 기본 메서드를 사용하면 시간 절약 가능
- 인터페이스의 계약을 변경하고 이를 구현하는 모든 타입에 새로운 메서드를 추가하는 대신 기본 메서드를 사용하여 '상식적인' 구현을 제공할 수 있다.
- 이렇게 하면 의도된 로직의 일반적인 변형을 모든 타입에 적용하여 UnsupportedOperationException 을 사용할 필요가 없다.
- 하위 호환성 또한 유지할 수 있음

### 3.4.2 함수형 인터페이스 명시적으로 구현하기
- 함수형 인터페이스는 람다나 메서드 참조를 통해 묵시적으로 구현될 수도 있지만, 더 높은 차수의 함수에서 사용가능하도록 명시적으로 구현하여 사용할 수도 있다.
- 함수형 인터페이스를 직접 구현하느 것은 이전에 비함수형이었던 타입을 함수형 코드에서 쉽게 사용할 수 있도록 한다.
- 일반적으로 명령은 이미 해당하는 전용 인터페이스를 갖고 있다.
- 함수형 인터페이스 간의 논리적 동등성만으로는 호환성을 만들 수 없다.
- 그러나 TextEditorCommand 를 Supplier<String> 으로 확장함으로써 격차를 메꿔줄 수 있다.
```java
public interface TextEditorCommand extends Supplier<T> {
    String execute();

    default String get() {
        return execute();
    }
}
```

#### 정적 헬퍼 생성하기
- JDK 자체에서 제공하는 함수형 인터페이스와 같이 타입을 직접 제어할 수 없는 경우 정적 메서드를 모으는 헬퍼 타입을 만들 수 있다.
- Function<T, R>와 Supplier/Consumer에 대한 컴포지터를 만들어보자.
```java
public fiinal class Compositor {

  public static <T, R> Supplier<R> compose(Supplier<T> before, Function<T, R> fn) {
      Objects.requireNonNull(before);
      Objects.requireNonNull(fn);

      return () -> { 
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

  private Compositor() {
    // 기본 생성자 생략
  }
}
```