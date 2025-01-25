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
