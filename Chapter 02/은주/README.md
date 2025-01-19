# Chapter 02. 함수형 자바
- 람다 표현식 : 자바에서 **함수형 프로그래밍 접근 방식의 핵심 요소**

## 2.1. 자바 람다란?
- 람다는 어떠한 객체에도 속하지 않는 익명 메서드와 비슷하다
```java
() -> System.out.println("hi");
```

### 2.2.1. 람다 문법
```
(<parameters>) -> { <body> }
```
- 매개변수
  - 메서드 인수와는 다르게 컴파일러가 매개변수의 타입을 추론할 수 있는 경우, **매개변수의 타입을 생략할 수 있다**
  - 묵시적 타입이 지정된 매개변수, 명시적 타입 지정된 **매개변수를 혼용하는 것은 허용되지 않는다**
  - 매개변수가 하나인 경우는 괄호를 생략할 수 있지만, 매개변수가 없거나 둘 이상이면 괄호를 사용해야 한다
- 화살표
  - 매개변수와 람다 바디를 구분하기 위해 사용한다
- 바디
  - 단일 표현식 또는 코드블럭으로 구성된다
  - 한줄로 작성된 표현식의 계산된 결과는 **암시적으로 return 문 없이 반환된다**
  - 중괄호로 감싸져 있으며 값을 반환할 경우 명시적으로 return 문을 사용해야 한다

### 2.1.2. 함수형 인터페이스
- 람다 문법은 기존의 인터페이스를 기반으로 하위 호환성을 유지하고 있다
- 람다 표현식은 **특화된 인터페이스의 하위 타입**으로 표현되며, 이를 `함수형 인터페이스` 라고 한다.
- 자바 인터페이스 바디 구성 
  - 메서드 시그니처
    - **인터페이스에 반드시 구현되어야 하는 추상메소드 시그니처** 포함됨
  - 디폴트 메소드
    - 인터페이스를 구현하는 **모든 클래스가 이를 재정의 가능하지만 필수는 아님**
  - 정적 메소드
    - 클래스 레벨에서 **필수적으로 구현되어야 하는 메서드. 하지만 상속 불가하며 오버라이딩 불가**
  - 상수
- 인터페이스를 '함수형 인터페이스' 로 만드는 조건
  - `SAM (Single Abstract Method)` : **추상메서드를 1개** 가진 인터페이스의 특성
  - 디폴트 메서드, 정적 메서드는 추상 메서드가 아니기에 여러개 존재해도 무방
  - SAM은 람다의 기능을 보완하는 데 자주 사용됨
    - ❓어떤 기능을 보완하는데 사용되지? 어떻게 보완?
- 단 하나의 추상메서드를 가진 모든 인터페이스는 자동으로 함수형 인터페이스가 된다
- @FunctionalInterface 는 꼭 필요하진 않지만 **컴파일러와 다른 어노테이션 기반 도구에게 해당 인터페이스가 함수형 인터페이스임을 알려주고 SAM 요구사항을 반드시 따라야 한다는 것을 알려준다**
- 해당 마커 어노테이션을 달면 **코드와 인터페이스의 의도를 명확히 표현하며, 추후에 의도하지 않은 변경으로부터 코드를 보호하는 역할을 한다**
- **인터페이스가 SAM 요구사항을 충족한다면, 해당 인터페이스는 람다로 표현할 수 있다.**

### 2.1.3. 람다와 외부 변수
- 람다도 '순수함수와 참조 투명성' 이라는 핵심 개념을 따르지만, **유연성을 위해 어느정도의 불순성을 허용**한다
- `캡처` 를 통해 람다가 정의된 생성스코프 내의 상수, 변수를 획득할 수 있다
```java
public class LambdaCapture {

    public static void main(String... args) {
        Runnable r = capture();
        r.run();
    }

    private static Runnable capture() {
        var theAnswer = 42;

        Runnable printAnswer = () -> System.out.println("the answer is " + theAnswer);

        return printAnswer;
    }
}
```
- 캡처 람다와 논캡처 람다의 주요 차이점은 **JVM 최적화 전략**에 있다
  - JVM 은 실제 사용 패턴에 기반하여 다양한 전략으로 람다를 최적화한다
  - ❓ JVM 최적화 전략이 어떻게 다른지?
- 변수가 캡처되지 않은 경우 람다는 내부적으로 간단한 정적 메서드가 될 수 있고, 익명 클래스와 같은 대안적인 접근 방식의 성능을 능가할 수 있다
  - ❓내부적으로 간단한 정적 메서드가 될 수 있다? 
- 변수를 캡처하는 상황에서 JVM 은 **추가적인 객체 할당이 발생하며, 성능 및 가비지 컬렉터 시간에 영향**을 줄 수 있다
- 변수 캡처를 피해야 하는 또 다른 이유 중 하나는 해당 변수가 `Effectively final` (**변수가 선언된 후 그 값이 변경되지 않음**) 이어야 한다는 필요성이 있기 때문이다

#### Effectively final
- JVM 은 캡처된 변수를 안전하게 사용하고, 최상의 성능을 얻기 위해 반드시 지켜야 하는 요구사항이 있다.
  - **반드시 캡처되는 변수는 Effective Final 이어야 한다.**
- `final` 키워드를 명시적으로 사용하거나, **초기화된 이후에 상태가 변경되지 않도록 유지**해야 한다
- 하지만 이 요구사항은 실제로 변수를 참조하기 위한 것이며, **자료 구조에는 해당되지 않는다**
- 변수가 Effective final 인지 확인하는 가장 간단한 방법은 해당 변수를 명시적으로 final 로 선언하는 것이다
  - final 키워드가 추가되고도 컴파일이 가능하다면 해당 키워드 없이도 컴파일이 된다
  - 그렇다면 왜 무조건 final 로 선언하지 않을까?
    - **컴파일러가 외부에서 참조되는 부분을 effectively final 로 처리해주기 때문에** 실제 불변성에는 큰 도움이 되지 않는다
    - 오히려 final 을 붙여서 코드의 가독성만 떨어지므로 항상 의도를 갖고 신중히 결정해야 한다

#### 참조를 다시 final 로 만들기
- effectively final 과 같은 람다 내 변수 사용에 대한 보호장치는 처음에 부담스러울 수 있다
- **외부 영역 변수를 캡처하는 대신 람다는 자체적으로 필요한 모든 데이터를 인수로 요청해야 한다**

### 2.1.4. 익명 클래스는 무엇인가?
- 구체적인 인터페이스를 구현해야 할 경우 람다 표현식과 익명 클래스의 차이점
```java
// 함수형 인터페이스
public interface HelloWorld {
    String sayHello(String name);
}

HelloWorld helloWorld = name -> "hello, " + name + "!";

// 익명 클래스
var helloWorld = new HelloWorld() {
    @Override
    public String sayHello(String name) {
        return "hello, " + name + "!";
    }
};
```
- 람다는 문법 설탕 (특정 구조를 더 간결하고 명확하게 표현할 수 있는 대안) 처럼 보일 수 있지만, 실제로 **가독성 외에도 생성된 바이트 코드와 런타임 처리 방식**에 차이가 있다.

```byte
// 익명 클래스
0: new #7 // class HelloWorldAnonymous$1 1️⃣
3: dup
4: invokespecial #9 // Method HelloWorldAnonymous1."<init>":()V 2️⃣
7: astore_1
8: return

// 람다
0: invokedynamic #7,0 // InvokeDynamic #0:sayHello:()LHelloWorld; 3️⃣
5: astore_1
6: return
```
- 1️⃣ : 익명 내부 클래스 HelloWorldAnonymous$1 의 새로운 객체가 주변 클래스인 HelloWorldAnonymous 내에서 생성됨
- 2️⃣ : 익명 클래스의 생성자가 호출됨. 객체 생성은 JVM 에서 2단계로 이루어짐 (인스턴스 생성 / 생성자 메서드 호출하여 초기화)
- 3️⃣ : invokedynamic 명령 코드는 전체적인 람다 생성 로직을 숨김
- 두 버전 모두 공통적으로 **astore_1 을 호출로 사용해서 참조를 로컬 변수에 저장하고, return 호출**한다.
- 익명 클래스 버전은 **익명 타입인 HelloWorldAnonymous$1 의 새로운 객체를 생성**하며, 이로 인해 3개의 명령 코드가 생성된다
  - new : 초기화되지 않은 새로운 인스턴스 생성
  - dup : 값을 스택 맨 위에 복제
  - invokespecial : 새로 생성된 객체의 생성자 메서드 호출하여 초기화 완료
- 람다 버전은 스택에 사용해야 하는 인스턴스를 생성할 필요가 없다.
  - 대신 **람다 생성 전체 작업을 단일 명령 코드인 invokedynamic 을 사용하여 JVM 에 위임한다**
- invokedynamic 명령어 
  - ❓ 명령어에 대해 더 자세히 찾아보기
  - 자바 7은 동적 언어를 지원하고, 더 유연한 메서드 호출을 가능하게 하기 위해 invokedynamic JVM 명령 코드를 도입했다
  - JVM 은 실제 대상 메서드와 동적 호출 사이트를 연결한다
  - 런타임은 첫번째 invokedynamic 호출에 '부트스트랩 메서드' 를 사용하여 실제로 호출할 메서드를 결정한다
  - invokedynamic 명령 코드는 **JVM 에서 리플렉션을 활용한 람다 생성** 기법으로 볼 수 있다
- 람다와 익명 내부 클래스 간의 또 다른 중요한 차이점은 각각의 `스코프` 이다
  - 내부클래스는 **자체 스코프를 생성**하고, 해당 범위 내의 로컬 변수를 외부로부터 감춘다
  - 반면, **람다는 자신이 속한 스코프 범위 내에 존재**한다.
    - 변수는 동일한 이름으로 재선언될 수 없으며, 정적이지 않은 경우 this 는 람다가 생성된 인스턴스를 참조한다

## 2.2. 람다의 실전 활용
- 람다는 타입이 지정되며 익명성을 허용하고 간결하다

### 2.2.1. 람다 생성
- 람다 표현식을 만들려면 `단일 함수형 인터페이스`를 표현해야 한다
- 새 인스턴스를 생성하려면 왼쪽에 **타입이 정의되어야 한다**
```java
Predicate<String> isNull = value -> value == null;

// 컴파일 실패
var isNull = (String value) -> value == null;
```
- 여전히 자바 컴파일러는 **참조에 구체적인 타입을 요구하며, 메서드 시그니처만으로는 충분하지 않다**
- 기존의 정적 타입 시스템을 사용함으로써, 람다는 자바에 완벽하게 적합하며 컴파일 시간 안전성을 제공받는다
```java
LikePredicate<String> isNull = value -> value == null;

Predicate<String> wontCompile = isNull;
// 에러:
// 호환되지 않는 타입: LikePredicate<String> 은 
// java.util.function.Predicate<String> 으로 변환될 수 없음
```
- 메서드 인수와 리턴타입으로써 임의로 생성된 람다는 타입 호환성이 없다

### 2.2.2. 람다 호출
- 자바에서는 람다의 SAM 을 명시적으로 호출해야 한다.
```java
Function<String, String> helloWorld = name -> "hello" + name;
// var helloWorld = name -> "hello" + name; (X)
```

### 2.2.3. 메서드 참조
- `메소드 참조` : 람다 표현식을 생성하는 새로운 방법으로, **새로운 연산자인 :: 을 사용하여 기존의 메서드를 참조하는 간결한 문법 설탕**
  - 코드의 가독성, 이해성을 저해하지 않으면서 코드를 **간소화**할 수 있다
```java
// LAMBDAS
String[] lambdas =
    customers.stream()
            .filter(customer -> customer.isActive())
            .map(customer -> customer.getName())
            .map(name -> name.toUpperCase())
            .peek(name -> System.out.println(name))
            .toArray(count -> new String[count]);

// METHOD-REFERENCES
String[] methodsRefs =
    customers.stream()
            .filter(Customer::isActive)
            .map(Customer::getName)
            .map(String::toUpperCase) 
            .peek(System.out::println)
            .toArray(String[]::new);
```
- 4가지 메서드 참조 방법
  - `정적 메서드 참조`
    - 특정 타입의 정적 메서드를 가리킴
    - **ClassName::staticMethodName** 모양으로 사용됨
    ```java
    public class MethodReferencesStatic {
        public static void main(String... args) {

            // LAMBDA
            Function<Integer, String> asLambda = i -> Integer.toHexString(i);

            // STATIC METHOD REFERENCE
            Function<Integer, String> asRef = Integer::tohexString;
        }
    }
    ```
  - `바운드 비정적 메서드 참조`
    - 이미 존재하는 객체의 비정적 메서드를 참조할 경우 사용됨
    - 현재 인스턴스에서 메서드를 this:: 로 참조하거나, 상위 구현을 super:: 로 참조할 수도 있음
    ```java
    public class MethodReferencesBoundNonStatic {

        public static void main(String... args) {

            boundNonStatic();
            boundNonStaticNoIntermediate();
        }

        private static void boundNonStatic() {
            // 기존 객체를 기반으로 한 람다
            var now = LocalDate.now();
            Predicate<LocalDate> isAfterNowAsLambda = $ -> $.isAfter(now);

            // 바운드 비정적 메서드 참조
            Predicate<LocalDate> isAfterNowAsRef = now::isAfter;
        }

        private static void boundNonStaticNoIntermediate() {
            // 반환값 바인딩
            Predicate<LocalDate> isAfterNowAsRef = LocalDate.now()::isAfter;

            // 정적 필드 바인딩
            Function<Object, String> castToStr = String.class::cast;
        }
    }
    ```
    - 바운드 메서드 참조는 변수, **현재 인스턴스 또는 super 에 이미 존재하는 메서드를 사용하는 방법**
    - **objectName::instanceMethodName** 모양으로 사용됨
  - `언바운드 비정적 메서드 참조`
    - **특정 객체에 바운딩되지 않는 대신 (언바운드는 메소드 호출 시 객체 결정되지만, 바운드는 이미 특정 객체에 바인딩되어 있음), 타입의 인스턴스 메서드를 참조**한다
    ```java
    public class String {
        public String toLowerCase() {
            ...
        }
    }

    public class MethodReferencesUnboundNonStatic {

        public static void main(String... args) {
            Function<String, String> toLowerCaseLambda = str -> str.toLowerCase();
            Function<String, String> toLowerCaseRef = String::toLowerCase;
        }
    }
    ```
    - **ClassName::instanceMethodName** 모양으로 사용됨
    - ClassName 은 참조된 인스턴스 메서드가 정의된 인스턴스 유형을 나타낸다
  
  - `생성자 참조`
    - new 키워드를 통해 생성자를 참조한다
    ```java
    public class MethodReferencesConstructor {
        public static void main(String... args) {

            Function<String, Locale> newLocaleLambda = language -> new Locale(language);

            Function<String, Locale> newLocaleRef = Locale::new;
        }
    }
    ```
    - **ClassName::new** 로 사용됨

## 2.3. 자바의 함수형 프로그래밍 개념
### 2.3.1. 순수 함수와 참조 투명성
- 순수 함수의 특징
  - **어떤 종류의 사이드 이펙트도 발생시키지 않는** 함수로직을 가진다
  - **항상 같은 입력에 대해 같은 결과를 반환한다. 따라서 반복호출은 초기 결과로 대체**되며 호출이 참조 투명성을 갖게 된다
- 코드를 독립성이 보장된 형태로 만들면 예측가능성이 높아지면서 더 간결해진다
- **입력 인수에 의존하지 않는 예측 불가능한 로직이 있는지 살펴보라**
  - ex) 난수 생성기, 현재 날짜
- **사이드 이펙트와 가변 상태를 찾아보라**
  - 함수 외부 상태에 영향을 미치는지, 인스턴스나 전역 변수에 영향을 줄 수 있는지 확인
  - 입력된 인수의 내부 데이터를 변경하는지, 새로운 요소를 컬렉션에 추가하거나 객체 속성을 변경하는지 확인
  - 입출력과 같은 불순한 작업을 수행하는지 확인
- System.out.println 호출도 사이드이펙트이다.
  - **무해해 보일지라도 파일 시스템 접근이나 네트워크 요청과 같은 모든 종류의 입출력은 사이드 이펙트**이다
    - 왜냐하면 동일한 인수를 사용한 반복적 호출은 첫번째 반환 결과로 대체될 수 없기 때문이다
- 순수함수는 본질적으로 참조 투명성을 갖는다.
  - 따라서 **동일한 인수를 사용하여 이루어진 후속 호출은 모두 이전에 계산된 결과로 대체할 수 있다**
  - 이러한 상호 교환 가능성은 `메모이제이션`이라는 최적화 기법을 가능하게 한다
- 자바 컴파일러는 람다 표현식이나 메서드 호출의 자동 메모이제이션을 지원하지 않는다

### 2.3.2. 불변성
- 함수형 프로그래밍 개념에 있어 데이터 무결성과 안전한 사용성을 보장하기 위해 **불변 자료 구조**를 필요로 한다
- 자바는 불변성에 대한 지원이 비교적 제한적이어서 effective final 과 같은 구조를 강제로 사용해야 한다
- 자바 14에서는 불변 데이터 클래스인 레코드가 도입되었다

### 2.3.3. 일급 객체
- 자바 람다는 함수형 인터페이스의 구체적 구현이기 때문에 **일급 객체를 얻게 되어 변수, 인수 및 반환값으로 사용할 수 있다**

### 2.3.4. 함수 합성
- 어떤 패러다임을 선택하는지에 관계없이 작은 컴포넌트들을 조합하여 복잡한 시스템을 만드는 것은 프로그래밍의 기본 원리이다
- 함수형 프로그래밍 (FP) 에서는 두 함수가 결합되어 새로운 함수를 구현한 후 다른 함수들과 다시 결합할 수 있다
- 자바에서 **함수 합성은 연관된 타입에 의존한다**

#### 느긋한 계산법
- 자바는 여러 가지 느긋한 구조를 지원한다
  - **논리적 단축 계산 연산자**
  - **if-else 및 ? 삼항 연산자**
  - **for 및 while 루프 연산자**
- 복잡한 메서드를 계산하는 방법은 클래스의 결과와 전체 표현식에서 사용된 논리연산자에 따라 달라진다
  - 이러한 이유로 JVM 은 계산이 필요하지 않은 표현식을 버릴 수 있다

## 핵심 요약
- 함수형 인터페이스는 자바 람다의 구체적인 타입과 표현이다
- 람다표현식은 JVM 이 명령코드인 invokedynamic 을 사용하므로 단순한 문법 설탕이라고 할 수 없으며, 익명 클래스보다 더 나은 성능을 얻기 위한 여러 최적화 기술을 허용한다
- 람다에서 외부 변수를 사용하려면 effective final 이어야 한다