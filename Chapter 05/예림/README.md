# Chapter 05. 레코드

## 5.1 데이터 집계 유형
- 데이터 집계 : 다양한 소스로부터 데이터 수집 & 의도한 목적에 맞도록 표현하는 과정
  - like 튜플
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