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
- 내부 반복