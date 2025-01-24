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
