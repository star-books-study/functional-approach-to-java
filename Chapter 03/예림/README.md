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