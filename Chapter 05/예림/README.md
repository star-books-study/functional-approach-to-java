# Chapter 05. 레코드

## 5.1 데이터 집계 유형
- 데이터 집계 : 다양한 소스로부터 데이터 수집 & 의도한 목적에 맞도록 표현하는 과정
  - like 튜플
### 튜플
- 튜플에는 두 가지 종류가 있다.

```java
apple = ("apple", "green")
banana = ("banana", "yellow")
cherry = ("cherry", "red")


fruits = [applie, banana, cherry]

for fruit in fruits:
  print "The", fruit[0], "is", fruit[1]
```

```java
typealias Fruit = (name : String , color: String)

let fruits: [Fruit] = [
  (name: "apple", color: "green"),
  (name: "banana", color: "yellow"),
  (name: "cherry", color: "red"),
]

for fruit in fruits:
  println("The \(fruit.name) is \(fruit.color))")
```

### 간단한 POJO
- 전통적인 'user' 클래스 -> 불변의 POJO로 개선한 뒤에 레코드로 만들어보자!
> 전통적인 'user' 클래스 코드는 생략 
### 불변 POJO 만들기
- User POJO를 불변하게 만들면 더 이상 setter 메서드가 필요하지 않음

```java
public final class User {
  private final String username;
  ...

  // public User() {} 코드 사라짐
  // 필드는 반드시 객체가 생성될 때 선언되어야 하므로, 완전히 '통과' 생성자만 사용 가능

  public User(String username,
              boolean active,
              LocalDateTime lastLogin) {
    this.username = username; // setter 메서드 없이 필드들이 선언됨
    this.active = active;
    this.lastLogin = lastLogin;
  }

  public String getUsername() { // getter 메서드는 변경 불가능한 상태로 유지된다.
    return this.username;
  }

  public boolean isActive() {
    return this.active;
  }

  public LocalDateTime getLastLogin() { // getter 메서드는 변경 불가능한 상태로 유지된다.
    return this.lastLogin;
  }

  @Override
  public int hashCode() { // 부가적으로 생성된 equals, toString 메서드는 변경되지 않는다.
    // 변경되지 않음
  } 

  @Override
  public boolean eqauls(Object obj) { // 부가적으로 생성된 equals, toString 메서드는 변경되지 않는다.
    // 변경되지 않음
  } 

  @Override
  public String toString() { // 부가적으로 생성된 equals, toString 메서드는 변경되지 않는다.
    // 변경되지 않음
  } 
  
}
```

### POJO를 레코드로 만들기
```java
public record User(String username, boolean active, LocalDateTime lastLogin) {
  // 바디 생략
}
```
- User 레코드는 불변 POJO와 동일한 기능을 갖추고 있음
- 어떻게 하면 이렇게 적은 양의 코드로 많은 일을 처리할 수 있을까?


## 5.2 도움을 주기 위한 레코드
- 레코드는 인덱스 대신 이름을 통해 데이터에 접근할 수 있음

```java
public record User(String username, boolean active, LocalDateTime lastLogin) {
  // 바디 생략
}
```

- 레코드의 일반적인 구문은 두 가지로 나뉜다.
   - 헤더 : 다른 유형과 동일한 속성 정의
   - 바디 : 추가 생성자와 메서드 지원
  ```java
  // 헤더
  [visibility] record [Name][<optional generic types>]([data components]) {
    // 바디
  }
  ```
- 컴파일러에 의해 한 줄이 코드가 앞서 봤던 간단한 불변 User 타입과 유사한 클래스로 변환됨

### 5.2.1 내부 동작
- javap 명령어를 이용해 .class 파일을 디스어셈블하여 바이트 코드로 POJO의 레코드 버전과 User 타입의 실제 차이를 확인해보면, 접근자 메서드의 차이만 있을 뿐 결과적으로 두 클래스는 기능적으로 동일하다.

### 5.2.2 레코드의 특징
#### 컴포넌트 접근자
- 레코드는 getter의 접두사 get 없이 컴포넌트 이름으로 된 접근자 메서드를 사용할 수 있다.
  ```java
  var username = user.username();
  ```
- 접근자 메서드를 재정의할 수도 있으나, 레코드는 불변한 데이터를 보관하도록 설계되었기 때문에 추천하지는 않는다.

#### 표준, 간결, 사용자 정의 생성자
- 레코드는 표준 생성자도 간결하게 디자인된 형태로 제공된다. 생성자의 인수를 생략할 수 있고, 자동으로 컴포넌트가 매핑되어서 직접 `this.username = username`과 같은 코드를 쓸 필요가 없다. 다만 할당되기 전에 사용자가 직접 정의하는 것은 가능하다.
  ```java
  public User {
    username = username.toLowercase();
  }
  ```
- 추가 생성자도 정의 가능하지만 모든 사용자 정의 새성자는 첫 번째 구문에서 표준 생성자를 명시적으로 호출해야 한다.

#### 객체 식별과 설명
- hashCode와 equals, toString() 메서드의 표준 구현은 기본적으로 제공해주고, 마찬가지로 재정의 가능

#### 제네릭
- 제네릭도 지원한다.
```java
Ccontainer<String> stringContainer = new Container<>("hello, String!", "a String container");

String content = stringContainer.content();
```

#### 어노테이션
- 기존 대상 외 레코드에서 더 세밀한 어노테이션을 제어하기 위해 `ElementyType.RECORD_COMPONENT`가 새롭게 도입되었다.
  
#### 리플렉션
- 자바 16에서는 java.lang.Class에 getRecordComponents 메서드를 추가했다. 이 메서드는 다른 타입의 클래스에 대해서는 null을 반환하고, 레코드 기반 타입이면 RecordCompnent 객체의 배열을 반환한다.

### 5.2.3 누락된 기능

#### 추가적인 상태
바디에 필드가 추가될 수 없지만, 새로운 메서드를 추가함으로써 derived state(파생 상태)는 만들 수 있음.

#### 상속
- 레코드는 인터페이스를 구현하고 default 메서드를 통해 공통 기능을 공유할 수 있다.

#### 컴포넌트의 기본값과 컴팩트 생성자
- 자바는 기본값을 지원하지 않지만 다음과 같이 정적 팩토리 메서드를 활용하여 사용자과 원하는 조합을 쉽게 만들 수 있다.
```java
public record Origin(int x, int y) {
  public Origin() {
    this(0, 0);
  }
}
public record Rectangle(Origin origin, int width, int height) {
  public static Rectangle atX(int x, int width, int height) {
    return new Ractangle(x, 0, width, height);
  }

  public static Rectangle atY(int y, int width, int height) {
    return new Ractangle(0, y, width, height);
  }

  // ...
}

var xOnlyRectangle = Ractangle.atX(23, 300, 400);
// => Ractangle=Origin[x=23, y=0], width=300, height=400]

```
#### 단계별 생성
- 변경 가능한 자료 구조 대신 빌더 패턴을 사용하면 좋다.
- 사용할거면 정적 중첩 클래스로 직접 패치하는 것이 좋다.
```java
public record User(long id, String username, boolean active, Optional<LocalDateTime> lastLogin) {
  public static final class Builder {
    // ...
  }
}

var builder = new User.Builder("bed");
```
- 빌더는 레코드 인스턴스를 더 다양하고 유연하게 생성할 수 있게 한다.

## 5.3 사용 사례와 일반적인 관행

### 5.3.1 레코드 유효성 검사 및 데이터 정제
- 레코드는 따로 인수가 필요없는 **컴팩트 생성자**라는 걸 지원한다고 앞서 언급했는데, 이 컴팩트 생성자에 유효성 검사나 데이터 정제 로직을 넣으면 딱임

```java
public record NeedsValidation(int x, int y) {
  if (x < y) {
    throw new IllegalArgumentException("x는 y 이상이어야 합니다.");
  }
}
```
- 아니면 데이터 범위를 초과하는 값을 정규화할 수도 있음. (60분이 넘는 초는 분을 더해준다던가)

- 빈 검증 API (@NotNull, @Positive)를 이용할 수도 있음. 레코드는 JavaBean이 아니지만 프로젝트 종속성을 추가해서 사용할 수 있음. (다만 검증이 자동으로 수행되는 것은 아니고 종종 바이트 코드 수정 필요)

### 5.3.2 불변성 강화
- 레코드 컴포넌트도 표준 생성자를 이용하면 컴포넌트의 불변 복사본을 만들 수 있음(얕은 불변성이지만)
```java
public record IncreaseImmutability(List<Stiring> values) {
  public IncreaseImmutability {
    values = Collections.unmodifiableList(values); // 변경 불가능한 뷰 생성
  }
} 
```

### 5.3.3 변형된 복사본 생성
- 변형된 복사본을 생성하는 방법 세가지
  - wither 메서드 : `withXXX` 형태로 명명하고 기존 인스턴스를 변경하는 대신 새 인스턴스를 반환함
  - 빌더 패턴
  - 도구 지원 빌더 : 컴포넌트와 레코드 복제에 필요한 코드 사이에 응집도로 인해 리팩토링이 어려워지는 것을 방지해 `@RecordBuilder` 애너테이션 지원


### 5.3.4 명목상 튜플로써의 로컬 레코드
