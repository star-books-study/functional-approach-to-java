# Chapter 03. JDK의 함수형 인터페이스

- JDK는 함수형 도구를 시작하는 데 도움이 되도록 `java.util.function` 패키지를 통해 40개 이상의 함수형 인터페이스 제공

### 3.1 네 가지 함수형 인터페이스
- Function : 인수를 받고 결과를 반환한다.
- Consumer : 인수만 받고 결과를 반환하지 않는다.
- Supplier : 인수를 받지 않고 결과만 반환한다.
- Predicate : 인수를 받아서 표현식에 대해 테스트하고 boolean 값을 결과로 반환한다.


### 3.1.1 Function
- 