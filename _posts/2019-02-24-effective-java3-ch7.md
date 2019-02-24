---
layout: post
title: "Effective Java 3e Ch7"
description: "Effective Java"
date: 2019-02-24
tags: [java,pattern]
comments: true
share: true
---

## 람다와 스트림

### 42. 익명 클래스보다 람다를 이용
- 이전에 함수 타입을 표현할 때 추상 메서드를 하나만 가진 인터페이스를 사용. 
- 이러한 인터페이스의 인스턴스를 함수객체라고 표현

```java
// 낡은 방식
Collections.sort(words, new Comparator<String> {
  public int compare(String s1, String s2) {
    return Integer.compare(s1.lenght(), s2.length());
  }
})
```
- 함수형 인터페이스라 부르는 이 인터페이스의 인스턴스를 람다식을 사용해 만들 수 있게됨
  - 개념은 비슷하나 코드는 훨씬 간결함
  - ```java Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length())); ```
  - 컴파일러가 타입을 추론해줌
  - 타입을 명시해야하는 코드가 더 명확할때를 제외한, 나머지는 람다의 모든 매개변수 타입 생략
  - ```java Collections.sort(words, comparingInt(String::length));```
  - ```java words.sort(comparingInt(String::length));```
  - [link](https://github.com/dec7/study/commit/f07e6a076b628c90ee5b1e10fa35cd3c0b77f42e)
- 람다
  - 람다는 이름이 없고, 문서화 안됨
  - 따라서 코드 자체가 많아지거나, 동작이 명확히 설명되지 않는 경우엔 쓰지 말아야 함
    - 람다는 한줄, 최대 3줄까지다
  - 람다는 자신을 참조할 수 없음, 람다의 this는 바깥 인스턴스를 가리킴
    - 반면 익명 클래서의 this는 인스턴스 자신을 가리킴
    - 함수 객체가 자신을 참조해야 하는 경우 반드시 익명 클래스 써야함
  - 람다를 직렬화 하는 일은 극힘 삼가야 함 (익명 클래스 포함)

### 43. 람다보다 메소드 참조를 사용해라
- 람다가 익명 클래스보다 나은 점은 간결함
  - 더 간결한 방법은 메소드 참조
  - ```java map.merge(key, 1, (count, incr) -> count + incr); ```
  - => ```java map.merge(key, 1, Integer::sum); ```
- 람다로 할 수 없는 일은, 메소드 참조도 할 수 없음
  - 람다의 구현이 복잡할 경우 메소드 참조가 좋은 대안
- 메소드와 람다가 같은 클래스에 있는 경우
  - ```java service.execute(GoshThisClassNameIsHumongous::action);```
  - => ```java service.execute(() -> action());```
  - 위의 경우는 메소드 참조보다, 람다가 더 명확함

#### 메소드 참조 유형 - 5가지
- 인스턴스 메소드를 참조하는 유형
  1. 한정적 (수신 객체를 특정)
    - 근본적으로 정적 참조와 비슷
    - 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 같다
    - ```java Instant.now()::isAfter```
    - ```java Instant then = Instant.now(); t -> then.isAfter(t);```
  2. 비 한정적 (수신 객체를 특정하지 않음)
    - 함수 객체를 적용하는 시점에 수신객체를 알려줌
    - ```java String::toLowerCase```
    - ```java str -> str.toLowerCase()```

### 44. 표준 함수형 인터페이스를 사용하라
- java.util.funciton 패키지 43개 인터페이스 존재
  - 되도록 표준 함수형 인터페이스 활용
- Operator: 반환값과 인수의 타입이 같은 함수
  - UnaryOperator
  - BinaryOperator
- Predicate: 인수를 하나 받아 boolean으로 반환
- Function: 인수와 반환타입이 다른 함수
- Supplier: 인수를 받지 않고, 값을 반환
- Consumer: 인수를 받고, 반환 값이 없음

- 기본 타입만 지원하며 박싱된 기본타입을 넣어 사용하진 말라
- Comparator 인터페이스
  - 구조적으론 ToIntBiFunction(T,U) 와 동일
  - 1. API에서 자주 사용되는에 이름이 명확함
  - 2. 구현하는 쪽에서 반드시 지켜야할 규칙을 가지고 있음
  - 3. 비교자를 변환하고 조합해주는 유용한 디폴트 메소드를 가지고 있음

- @FunctionalInterface 어노테이션을 달아야 하는 이유
  - @Override와 비슷
  - 1. 해당 클래스의 코드를 읽는 사람에게 람다용 인터페이스임을 알림
  - 2. 추상 메소드가 오직 하나마 있어야 컴파일 되게 해줌
  - 3. 유지보수 과정에서 누군가 실수로 메소드 추가를 막아줌
- 주의 사항
  - 서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메소드를 다중 정의하지 말것 
  - 다중정의는 주의해서 사용

### 45. 스트림은 주의해서 사용
- 스트림 API 의 핵심 추상 개졈
  1. 스트림은 데이터 원소의 유한/무한 스퀀스
    - 스트림 원소는 어디서든지 올 수 있음
      - 컬렉션, 배열, 파일, 정규식, 난수생성기 등
  2. 스트림 파이프라인은 이 원소로 수행하는 연산 단계
    - 소스 스트림 --> 종단 연산으로 끝남
    - 중간 연산이 있을 수 있음
    - 각 중간 연산은 스트림을 어떤 방식으로 든 변환
    - 지연 평가됨
      - 평가는 종단 연산이 호출될때 이뤄지면, 종단 연산에 쓰이지 않는 원소는 계산에 쓰이지 않음
- 스트림 API는 메서드 연쇄를 지원하는 fluent API
- 기본적으로 순차적 수행
  - 스트림 API는 어떤 계산이라도 할 수 있으나, 해야한다는 것은 아님
  - 스트림을 과하게 사용하면 읽거나, 유지보수에 어려움
- 람다에서는 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지됨
- 기존 코드의 경우 스트림을 사용하여 리팩터링 하되, 새 코드가 나아보일때만 반영필요

#### 스트림에 적합한 일
- 원소의 시퀀스를 일관되게 변환
- 원소의 시퀀스를 필터링
- 원소의 시퀀스를 하나의 연산을 사용해 결합
- 원소의 시퀀스를 컨렉션에 모은다
- 원소의 시퀀스를 특정조건에 만족하는 원소를 찾음

#### 스트림에 적합하지 않은 일
- 한 데이터가 파이프라인의 여러 단계를 통과할 때 이 데이터의 각 단계에서의 값에 동시에 접근하기는 어려움
  - 스트림 파이프라인은 일단 한 값을 다른 값에 매핑한 뒤에는 잃는 구조

### 46. 스트림에서는 부작용 없는 함수를 사용하라
#### 스트림 패러다임 핵심
- 계산을 일련의 변환으로 재구성
  - 각 변환단계는 가능한 한 이전 단계의 ㄱㄹ과를 받아 처리하는 순수 함수 여야함
  - 순수 함수는 오직 입력만이 결과에 영향을 주는 함수
  - 스트림 연산에 전달되는 함수는 모두 부작용이 없어야 함

```java
// 잘못된 예제
Map<String, String> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
  words.forEach(word -> {
    freq.merge(word.toLowerCase(), 1L, Long::sum);
  })
}
```
```java
// 적절한 예제
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
  freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```
- forEach 연산은 가장 덜 스트림 다움
  - 스트림 계산 결과를 보고할 때만 사용할것

#### collector
- java.util.stream.Collectors 클래스는 메소드를 39개 가짐
- 축소(reduction) 전략을 캡슐화한 블랙박스 객체
  - 스트림의 원소들을 객체 하나에 취합한다는 뜻
- toList(), toSet(), toCollection(collectionFactory)


### 47. 반환 타입으로 스트림보다 컬렉션이 낫다.
- 반환 타입
  - 기본 컬렉션 타입
  - for-each만 쓰이거나 반환된 원소 시퀀스가 일부 collection 메소드를 구현할 수 없을때 Iterable 인터페이스
  - 반환 원소가 기본타입이거나, 성능 민감시 배열
- 스트림
  - 스트림을 반복을 지원하지 않음
  - Stream은 Iterable 인터페이스가 정의한 추상 메소드를 모두 추함하지만, Stream은 Iterable 을 확장하지 않음
- 어댑터 구현
```java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}

public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.stream(iterable.spliterator(), false);
}
```

- 원소 시퀀스를 반환하는 공개 API 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 좋음
  - Collection 인터페이스는 Iterable의 하위 타입, Stream 메소드도 제공
  - 단지, 컬렉션을 반환하는 이유로 덩치큰 시퀀스를 메모리에 올려서는 안됨

### 48. 스트림 병렬화는 주의해서 사용해라
- 동시성 프로그래밍은 안전성, 응답성을 유지하기 위해 애써야 함
- 병렬 스트림 파이프라인에서도 동일
  - 데이터 소스가 Stream.iterate 거나, limit 를 쓰면 파이프라인 병렬화로는 성능개선을 기대할 수 없음
  - 대체로 스트림 소스가, ArrayList, HashMap, HashSet, ConcurrentHashMap, 배열, int, long 범위일 때 효율이 좋음
  - Spliterator가 나누는 작업을 담당
    - 참조지역성이 뛰어남
    - 참조지역성이 낮으면 쓰레드는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리므로 느려짐
  - 종단 연산중 reduction이 병렬화에 가장 적합
    - min, max, count, sum
    - anyMatch, allMatch, noneMatch ..