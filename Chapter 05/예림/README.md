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
    // 생략
}

  // setter 사라짐
}