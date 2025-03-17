# Chapter 06. 스트림을 이용한 데이터 처리
- 함수형 언어는 선언적 접근법을 선호함 (<-> 명령형 접근법 : 루프에 의존)

## 6.1 반복을 통한 데이터 처리
### 6.1.1 외부 반복
- for-loop를 사용하여 1970년 이전에 출간된 책의 제목 순으로 정렬하여 세 편의 과학 소설을 찾아야 한다고 가정하자.
```java
record Book(String title, int year, Genre genre) {
    // 바디 생략
}

// 데이터 준비
List<Book> books = ...; // 정렬되지 않은 책들의 컬렉션 // 다음 단계에서 정렬될 수 있도록 변경 가능해야 함

Collections.sort(books, Comaparator.comparing(Book::title)); // 출력될 때 알파벳 순으로 나열되도록 처음에 정렬되어야 함

// FOR-LOOP

List<String> result = new ArrayList<>();

for (var book : books) {
    if (book.year() >= 1970) {
        continue;
    }

    if (book.genre() != Genre.SCIENCE_FICTION) {
        continue;
    }

    var title = book.title();
    result.add(title);

    if (result.size() == 3) {
        break;
    }
}
```
- 코드 양이 많고 **데이터 처리보다는 순회를 관리하는 데 더 많은 코드 라인이 소요됨**
- 또 연속적인 순회 과정이 필요해서 병렬 데이터 처리가 필요한 경우 전체 루프 구조를 재작성해야 함 (ConcurrentModificationException 발생 가능)

### 6.1.2 내부 반복
- 내부 반복은 외부 반복과 반대로 데이터 소스 자체가 '어떻게 수행되는지'를 담당하도록 함.
- 반복자 사용 대신 스스로 반복을 수행하는 파이프라인을 사전에 구성함.
- 스트림이 바로 내부 반복을 가진 데이터 파이프라인의 예시임.

## 6.2 함수형 데이터 파이프라인으로써의 스트림
- 스트림은 마지막 연산이 파이프라인에 연결된 이후에 스트림은 각 요소가 하나씩 순차적으로 파이프라인을 통해 처리됨
- '느긋한 순차 데이터 파이프라인'으로 설명할 수 있음
- 이 파이프라인은 순차 데이터 순회를 위한 고수준 추상화
- 연속된 고차 함수로 구성되어 함수적인 방식으로 각 요소 처리
- 소스 -> 스트림 -> 중간 연산 -> 중간 연산 -> ... -> 중간 연산 -> 종료 연산 -> 결과
  (1) 기존 데이터 소스에서 스트림을 생성함. 연속적인 요소를 제공할 수 있다면 스트림으로 변환할 수 있음
  (2) 중간 연산이라 불리는 것들은 java.util.stream.Stream<T>의 메서드로 제공되는 고차 함수들로, 파이프라인을 통과하는 요소들을 대상으로 하여 필터링, 매핑, 정렬 등 여러 작업을 수행. 각 연산을 새로운 스트림을 반환함
  (3) 스트림 대신 결과를 반환하는 마지막 종료 연산은 스트림 파이프라인 설계를 완료하고 실제 데이터 처리를 시작함
```java
List<Book> books = ...;

List<String> result =
    books.stream() // 생성 : Stream<Book>
    .filter(book -> book.year() < 1920) // 중간 : Stream<Book>
    .filter(book -> book.genre() == Genre.SCIENCE_FICTION) // 중간 : Stream<Book>
    .map(Book::title) // 중간 : Stream<String>
    .sorted() // 중간 : Stream<String>
    .limit(3L) //중간 : Stream<String>
    .collect(Collectors.toList()); // 종료 : List<String>
```

### 6.2.1 스트림의 특성
- 스트림은 특정한 동작과 예상치가 내장된 함수형 API
- 다른 방식으로 직접 생성해야 할 수많은 사전 정의된 컴포넌트와 보장된 속성들을 제공

#### 느긋한 계산법
- 스트림에서 중간 연산을 수행할 때 즉각적으로 실행되지 않음