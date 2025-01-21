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