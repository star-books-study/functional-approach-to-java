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
- 레코드는 getter의 접두사 get 없이 컴포넌트 이름으로 된 접근자 메서드를 사용할 수 있다.
  ```java
  var username = user.username();
  ```
- 접근자 메서드를 재정의할 수도 있으나, 레코드는 불변한 데이터를 보관하도록 설계되었기 때문에 추천하지는 않는다.
- 레코드는 표준 생성자도 간결하게 디자인된 형태로 제공된다. 생성자의 인수를 생략할 수 있고, 자동으로 컴포넌트가 매핑되어서 직접 `this.username = username`과 같은 코드를 쓸 필요가 없다. 다만 할당되기 전에 사용자가 직접 정의하는 것은 가능하다.
  ```java
  public User {
    username = username.toLowercase();
  }
  ```