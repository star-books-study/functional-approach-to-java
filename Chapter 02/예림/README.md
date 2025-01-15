# Chapter 02. 함수형 자바

## 2.1 자바 람다란?
- 람다 표현식은 0개 이상의 매개변수를 갖고 값을 반환할 수 있다.
- 어떠한 객체에도 속하지 않는 익명 메서드와 비슷
```java
() -> System.out.println("Hello, lambda!");
```
### 2.1.1 람다 문법
```java
(<parameters>) -> {<body>}
```
- 매개변수
  - 인수와 마찬가지로 쉼표(,)로 구분
  - 그러나 컴파이럴가 매개변수 타입을 추론할 수 있을 때 매개변수 타입 생략 가능
  - 묵시적, 명시적 매개변수 혼용 허용 X
  - 매개변수가 하나인 경우에는 괄호 생략 가능
- 화살표
  - 람다의 매개변수와 람다 body 구분
- 바디
  - 표현식 또는 코드 블럭으로 구성
  - 한 줄의 코드만으로 작성된 표현식은 중괄호 사용하지 않아도 됨
  - 계산된 결과는 암시적으로 return문 없이 반환
  - 하나 이상의 표현식으로 작성되면 자바 코드 블록 사용 & return문 필요
```java
(String input) -> {
    return input != null;
}

input -> {
    return input != null;
}

(String input) -> input != null

input -> input != null

```
- 어떤 방식을 사용할지는 문맥과 개발자의 선호도에 따라 다르다.

### 2.1.2 함수형 인터페이스
- 하휘 호환성 또한 자바의 중요한 특징 중 하나
- 람다 문법은 기존의 인터페이스를 기반으로 하위 호환성 유지
- 람다 표현식이 일급 객체로 취급받기 위해서는 기존의 객체와 비슷한 표현 방시을 갖추어야 함
- 람다 표현식은 특화된 인터페이스의 하위 타입으로 표현됨
- 이렇게 특화된 인터페이스를 `함수형 인터페이스`라고 함

- 함수형 인터페이스에는 별도의 문법이나 키워드가 없음.
- 다른 인터페이스들처럼 확장되거나 확장할 수 있으며, 클래스에 의해 상속될 수 있다.
- 그렇다면 인터페이스를 '함수형 인터페이스'로 만드는 조건은 ? ➡️  `SAM`<sub>Single Abstract Method</sub>
  - 추상 메서드 한 개를 가진 인터페이스의 특성
  - default, static 메서드는 여러 개 있어도 됨

```java
package java.util.function;

@FunctionalInterface // 필수는 아님
public interface Predicate<T> {
  boolean test (T t);

  default Predicate<T> and(Predicate<? super T> other) { // 함수 합성 지원
    // ...
  }

   default Predicate<T> negate() { // 함수 합성 지원
    // ...
  }

  ...

  static <T> Predicate<T> isEqual(Object targetRef) { // 편리하게 람다를 생성하거나 기존 람다를 래핑하는 데 사용
    // ...
  }

}
```
- 단 하나의 추상 메서드를 가진 모든 인터페이스는 자동으로 함수형 인터페이스가 되며, 모두 람다로 표현할 수 있다.
- `@FunctionalInterface`는 컴파일러와 다른 애너테이션 기반 도구에게 해당 인터페이스가 함수형 인터페이스임을 알려주고 SAM 요구사항을 반드시 따라야 한다는 것을 알려줌
- `@FunctionalInterface`는 기존 인터페이스의 하위 호환성 보장

### 2.1.3 람다와 외부 변수