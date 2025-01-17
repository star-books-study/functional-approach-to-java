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
- 람다는 유연성을 위해 어느 정도 불순성을 허용
- 캡처를 통해 람다가 정의된 스코프 내의 상수와 변수 획득 가능
- 다음과 같이 원래의 스코프가 더 이상 존재하지 않더라도 이러한 변수 사용 가능
```java
void capture() {
    var theAnswer = 42; // theAnswer는 capture 메서드의 스코프 내 선언됨

    Runnable printAnswer =
      () -> System.out.println("the answer is " + theAnswer); // 람다 표현식 printAnswer는 그 변수를 바디 내에서 캡처함
    
    run(printAnswer); // 이 람다 표현식은 다른 메서드오 스코프에서 실행될 수 있지만 변수 theAnswer에도 여전히 접근 가능
}

capture();
// 출력 :
// the answer is 42
```

- 캡처 람다와 논캡처 람다의 주요 차이점은 JVM 최적화 전략에 있다.
- JVM은 실제 사용 패턴에 기반하여 다양한 전략으로 람다 최적화
- 변수가 캡처되지 않은 경우 람다는 내부적으로 간단한 정적 메서드가 될 수 있으며, 익명 클래스와 같은 대안적인 접근 방식의 성능 능가 가능
- 그러나 변수 캡처가 성능에 미치는 영향은 명확하지 않음
- 최소한의 객체 할당 또는 최고의 성능이 필요한 경우에는 불필요한 캡처 사용을 피하는 것이 좋다.
- 변수 캡처를 피해야 하는 또 다른 이유 중 하나는 해당 변수가 'Effectively final'이어야 한다는 필요성이 있기 때문

#### Effectively final
- JVM은 캡처된 변수를 안전하게 사용 / 최상의 선응을 얻기 위해 반드시 지켜야 하는 본질적인 요구사항이 있음 : 캡처되는 변수는 반드시 Effectively final이어야 한다.
- 캡처된 어떤 변수든 초기화된 이후에 값이 한 번도 변경되지 않았다면 Effectively final이라고 할 수 있다.
  - final 키워드를 명시적으로 사용하거나 쵝화된 이후에 상태가 변경되지 않도록 유지해야 함
- 실제로 변수를 참조하기 위한 것이며 자료구조에는 해당되지 않는다. (ex. List<String>에 대한 참조가 final로 선언되어도 여전히 새 항목 추가 가능)
```java
final List<String> wordList = new ArrayList<>();

// 컴파일 성공
Runnable addItemInLambda = () -> wordList.add("adding is find");

// 컴파일 실패
wordList = List.of("assigning", "another", "List", "is", "not");
```
- 컴파일러가 외부에서 참조되는 부분을 effectively final로 처리해주기 때문에 실제 불변성에는 큰 도움이 되지 않는다.
- final과 같은 수정자를 추가하는 것은 항상 의도를 갖고 신중하게 결정해야 함.

#### 참조를 다시 final로 만들기
