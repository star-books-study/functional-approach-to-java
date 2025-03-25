# Chapter 06. 스트림을 이용한 데이터 처리

- 스트림 API 는 데이터 처리에 대한 `선언적` 이고, `지연 평가` 된 접근법을 제공한다

## 6.1. 반복을 통한 데이터 처리

### 6.1.1. 외부 반복
- 외부 반복 : 무엇을 하는가 (데이터 선택, 변형, 수집) 와 어떻게 수행하는가(ex. for 문을 통한 요소 반복) 이 혼용되는 것
- for문으로 반복하는 것의 큰 문제 중 하나는 `연속적`인 순회 과정이 필요하다는 것이다
  - 병렬 데이터 처리가 필요할 경우 전체 루프 구조 재작성 필요

### 6.1.2. 내부 반복
- 내부 반복에서는, 반복자를 사용하는 대신, 데이터 처리 로직이 스스로 **반복을 수행하는 파이프라인을 사전에 구성**한다
- **'무엇을 하고 싶은지' 에 집중할 수 있으며**, '어떻게 수행되는지' 에 대한 반복적인 작업은 걱정하지 않아도 된다
- `스트림`은 이러한 내부 반복을 가진 `데이터 파이프라인`의 예시이다

## 6.2. 함수형 데이터 파이프라인으로써의 스트림
- 선언적 접근법
  - 간단하고 명료한 **다단계 데이터 파이프라인** 구축
- 조합성
  - 데이터 처리를 위한 고차함수로 이루어진 연산들을 **조합**하여 사용할 수 있다
- 지연처리
  - **마지막 연산이 파이프라인에 연결된 이후** 각 요소가 하나식 순차적으로 파이프라인을 통해 처리되므로 필요한 연산 수를 줄일 수 있음
- 성능 최적화
  - 연산 종류에 따라 **순회방식을 자동 최적화**
- 병렬 데이터 처리
  - **내장된 병렬 처리** 기능을 가짐
- 스트림은 데이터 처리 기능을 제공하는 방식으로, `느긋한 순차 데이터 파이프라인` 이라고 말할 수 있다
1. 스트림 생성
   - **연속적**인 요소를 제공할 수 있다면 스트림으로 변환 가능
2. 중간 연산 수행
   - Stream 의 메소드로 제공되는 **고차함수**로, 파이프라인 통과하는 요소를 대상으로 필터링, 매핑, 정렬 등의 작업 수행
   - 각 연산은 새로운 스트림 반환, 이를 통해 원하는 만큼의 다양한 중간 연산과 연결될 수 있다
3. 종료 연산 수행
    - **종료 연산은 파이프라인 설계를 완료**하고 실제 데이터 처리를 시작한다
- 스트림은 `어떻게` 실행되는지 복잡한 설명 없이 `무엇이` 발생하는지를 선언적으로 나타낸다

### 6.2.1. 스트림의 특성
#### 느긋한 계산법
- 스트림의 중간 연산의 호출은 **파이프라인을 단순히 '확장'**하고, 새롭게 지연평가된 스트림을 반환한다
- 파이프라인은 모든 연산을 모아두고, **종료 연산 호출 전까지는 아무런 작업도 시작하지 않는다**
- 스트림 요소의 흐름은 '깊이 우선 방식' 을 따르며, 이를 통해 **필요한 CPU 주기, 메모리 사용량, 스택 깊이를 줄일 수 있다**

#### (대부분) 상태 및 간섭 없음
- 스트림은 데이터 소스와 요소들이 분리되어 있어 연산들은 소스 데이터에 영향을 주지 않는다
- 스트림은 **간섭하지 않고 통과하는 파이프라인이다**

#### 최적화
- 스트림은 성능 향상을 위해 여러 전략을 활용한다
  - 연산 융합
  - 불필요한 연산 제거
  - 단축 파이프라인 경로
- 파이프라인은 각 호출마다 새로운 스택 프레임을 필요로 한다

#### 보일러플레이트 최소화
- 스트림은 데이터처리를 **하나의 메서드 호출 체인으로 단순화**하여, 데이터 처리 로직의 복잡성을 명료하고 표현력 있게 풀어낸다
- 스트림 파이프라인은 필요한 만큼의 시각적 부담과 인지적 부담을 최소화한다

#### 재사용 불가능
- **스트림 파이프라인은 단 한번만 사용**할 수 있어, 종료 연산이 호출된 후 정확히 한 번만 전달된다
- 스트림은 **소스 데이터를 변경하거나 영향을 주지 않기에** 항상 동일한 소스 데이터로부터 다른 스트림을 생성할 수 있다

#### 쉬운 병렬화
- 처음부터 병렬 실행을 효율적으로 지원하도록 설계되었으며, Fork/Join 프레임워크를 기반으로 한다
- 단순히 parallel 메서드를 호출하는 것만으로 가능하지만, **스트림 원본은 충분한 데이터를 포함해야 하며 연산이 여러 스레드의 오버헤드를 수용할 만큼의 비용을 감당할 수 있어야 한다**

#### 예외 처리의 한계
- 람다 표현식과 스트림 연산 로직은, 예외를 처리하기 위해 특별한 고려 사항이나 문법 설탕을 제공하지 않으므로 **try-catch 보다 더 간결하게 예외 처리를 다룰 수는 없다**

### 6.2.2. 스트림의 핵심, Spliterator
- 스트림도 자체적인 반복 인터페이스인 java.util.Spliterator<T> 사용하며, **특정 특성을 기반으로 요소 일부를 다른 Spliterator 로 분리할 수 있다**
- Spliterator 는 부분적인 시퀀스들을 `병렬`로 처리하면서 Collection API 를 `순회`할 수 있게 한다
  - Spliterator 의 특성들은 스트림이 내부적으로 `어떻게 반복을 수행`하는지, `어떠한 최적화 기능을 지원`하는지를 결정한다 
- 스트림 활용 시 보통 Spliterator 를 직접 만들 필요가 없고 백그라운드에서 메서드들이 이를 대신 처리해준다

## 6.3. 스트림 파이프라인 구축하기
- Map, Filter, Reduce
  - Map : 데이터 **변환**
  - Filter : 데이터 **선택**
  - Reduce : **결과** 도출
- 이들은 모든 데이터 처리의 기본적인 구성요소이며, 이러한 접근 방식을 통해 순수하고 독립적인 함수를 조합하여 내부 반복을 활용하여 제어문을 제거할 수 있다

### 6.3.1. 스트림 생성하기
- stream() : List, Set 과 같은 컬렉션 기반 자료구조에서 새로운 스트림 인스턴스를 생성하는 가장 간단한 방법

### 6.3.2. 실습하기
- dropWhile() : 처음으로 false 가 등장하는 시점까지의 요소를 모두 버림
  - **순서가 정해진** 스트림을 위해 설계됨
- takeWhile() : dropWhile 의 반대 개념으로, Predicate 이 false 가 될 때까지 요소를 선택함
  - **이미 정렬되어 있다는 사실을 이용해** Predicate 이 false 가 되면 반복을 중단 할 수 있다. 큰 작업에서는 성능에 직접적인 영향을 줄 수 있으므로 takeWhile() 을 사용하면 된다
- skip() : 앞에서부터 n개의 요소를 건너뛰고 나머지 요소들을 다음 스트림 연산으로 전달
- distinct() : 요소를 비교하기 위해 **모든 요소를 버퍼에 저장**해야 한다
- flatMap() : 컬렉션 또는 Optional 과 같은 컨테이너 형태의 요소를 **'펼쳐서' 새로운 다중 요소를 포함하는 새로운 스트림**으로 만든다
- mapMulti() : map 과 flatMap 의 두 연산을 하나로 압축한다는 점이다
  - flatMap 이 **새 스트림을 생성하는 오버헤드를 피할 수 있다** -> 따라서 매핑되는 요소의 수가 매우 적거나 전혀 없는 상황에서 사용이 권장된다
```java
// flatMap()
List<String> words = Arrays.asList("hello", "world");
List<String> uniqueLetters = words.stream()
    .flatMap(word -> Arrays.stream(word.split("")))
    .distinct()
    .collect(Collectors.toList());
// 결과: [h, e, l, o, w, r, d]

// mapMulti()
List<String> words = Arrays.asList("hello", "world");
List<String> uniqueLetters = words.stream()
    .mapMulti((word, consumer) -> {
        for (char c : word.toCharArray()) {
            consumer.accept(String.valueOf(c));
        }
    })
    .distinct()
    .collect(Collectors.toList());
// 결과: [h, e, l, o, w, r, d]
```

#### 스트림에서 Peek 사용
- peek 연산은 주로 `디버깅`을 지원하기 위해 설계되었다

### 6.3.3. 스트림 종료하기
#### 요소 축소
- 축소 연산 : **누적 연산자를 반복적으로 적용**하여 스트림의 요소를 **하나의 결과값**으로 만들어낸다.
  - 이전의 결과를 현재의 요소와 결합하여 새로운 결과를 만들어낸다
- 축소는 3가지로 구분된다
  - 요소 : ex. 컬렉션 타입
  - 초기값 
  - 누적 함수 : ex. 현재 요소와 이전의 결과 또는 초기값만으로 작동하는 **순수 함수**
```java
private static Integer max(Collection<Integer> numbers) {
    int result = Integer.MIN_VALUE; // 초기값

    for (var value : numbers) { // numbers: 요소
        result = Math.max(result, value); // 누적 함수
    }

    return result;
}

private static <T> T reduce(Collection<T> elements,
                            T initialValue,
                            BinaryOperator<T> accumulator) {

    T result = initialValue;
    for (T element : elements) {
        result = accumulator.apply(result, element);
    }
    return result;
}

private static Integer max(Collection<Integer> numbers) {
    return reduce(numbers,
                    Integer.MIN_VALUE,
                    Math::max);
}
```
- 스트림 API 가 제공하는 3가지 기본 reduce 연산
  -     T reduce(T identity, BinaryOperator<T> accumulator); 
  -     Optional<T> reduce(BinaryOperator<T> accumulator);
    - 스트림의 첫번째 요소가 초기값으로 사용된다     
  -     <U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
    - 스트림에는 T 타입 요소가 포함되어 있지만 최종적으로 원하는 축소 결과가 U 타입인 경우 사용
```java
Integer reduceOnly = Stream.of("apple", "orange", "banana")
                            .reduce(0, // 초기값
                                    (acc, str) -> acc + str.length(), // acc 가 계속 초기값 처럼 누적되어 사용되는 것 
                                    Integer::sum);

System.out.println("reduceOnly: " + reduceOnly);

int mapReduce = Stream.of("apple", "orange", "banana")
                       .mapToInt(String::length)
                       .reduce(0, (acc, length) -> acc + length);

System.out.println("mapReduce: " + mapReduce);
```