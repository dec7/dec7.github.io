---
layout: post
title: "Effective Java 3e Ch5"
description: "Effective Java"
date: 2019-03-03
tags: [java,pattern]
comments: true
share: true
---

## 제네릭
### 26. 로 타입을 사용하지 말라
- 용어
  - 제네릭 클래스, 제네릭 인터페이스
    - 클래스와 인터페이스 선언에 타입 매개변수가 쓰인 경우
    - List<E> -> 제네릭 타입
      - 매개변수화 타입
  - 로 타입
    - 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때
    - List<E>의 로 타입: List
- 에러는 가능한 빨리 찾는 게 좋음
  - 컴파일 타임 > 런타임
- List, List<Object> 차이
  - List는 로타입을 사용한것, List<Obejct>는 모든 타입을 허용하겠다는 것을 컴파일러에게 전달할것
  - 따라서 List를 사용하면 타입 안전성을 잃게 됨
- 비 한정 와일드 카드 타입
  - 제네릭 타입을 쓰고 싶지만, 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않을 때
  - Set<?>
  - 비 한정 와일드 카드 타입을 쓰면, null 이외에 어떤 원소도 넣을 수 없음
- 예외
  - class 리터널에는 로 타입으로 써야 함
  - 런타임에는 제네릭 타입 정보가 지워지므로 instanceof 연산자는 비한정 와일드카드 타입이외에 매개변수화 타입에 적용 불가

### 27. 비검사 경고를 제거하라
- 할 수 있는한 모든 비검사 경호를 제거
- 경고를 제거할 수 없지만 타입이 안전하다고 확신할 경우 아래 어노테이션을 달아 경고를 숨겨라
  - ```java @SuppressWarings("unchecked") ```
  - 클래스, 개별지역변수까지 선언가능하나 가능한 범위를 좁히자
  - 남길경우, 안전한 이유를 주석으로 남겨라

### 28. 배열보다 리스트를 사용하라 
- 배열과 제네릭 차이
  - 1. 배열은 공변이라 sub가 super의 하위타입일 경우 Sub[]는 Super[]의 하위타입이 됨
    - 반면 제네릭은 불공변이라 서로 다른 타입
  - 2. 배열은 실체화 됨
    - 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인
      - 따라서 Long 배열에 String을 넣으려고 하면 ArrayStoreException 발생
      - 반면, 제네릭은 타입정보가 런타임에 소거됨
    - 제네릭 배열은 안전하지 못하기 떄문에 컴파일 되지 않음
- 제네릭은 비실체화 타입.
  - 실체화되지 않아서 런타임에서는 컴파일타임보다 정보를 적게 가짐

### 29. 이왕이면 제네릭 타입으로 만들라.
- [link](https://github.com/dec7/study/commit/dca70e10686df4b90a55ba828ae2496cd55f928a)

### 30 이왕이면 제네릭 메소드로 만들어라
- 제네릭 메소드
  - [link](https://github.com/dec7/study/commit/aaa97d723096e00dd06cfd758458168fa40c9f5c)
- 제네릭 싱글톤
  - [link](https://github.com/dec7/study/commit/6e23bd733bf0c1c9679a360d272eb7e5c3426d9e)
- 재귀적 타입 한정
  - 상대적으로 드물지만, 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용범위를 한정
  - 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰임
  - [link](https://github.com/dec7/study/commit/3205bcda12e7290d144778c4766d09186525c5b7)

### 31 한정적 와일드 카드를 사용해 API 유연성을 높여라
- 매개변수화 타입은 불공변, 때에따라 유연한 무언가가 필요
- producer-extends / consumer-super
  - 생산자 , 소비자 
  - stack에서 push는 생상자, pop은 소비자
- 반환타입에는 한정적 와일드카드 타입을 사용하면 안됨
  - 클라코드도 와일드카드 타입을 써야하기 때문
  - [link](https://github.com/dec7/study/commit/863b8b0a1082882f69a7b6fb90d45efefebb60b3)
- 타입 매개변수와 와일드카드의 용법
  - 비한정적 타입 매개변수
    - ```java public static<E> void swap(List<E> list, int i, int j); ```
  - 비한정적 와일드카드
    - ```java public static void swap(List<?> list, int i, int j); ```
  - public API인 경우 두번째가 나음
    - 신경쓸 타입이 없음
    - 메서드 선언에 타입 매개변수가 한번만 나올 경우 와일드 카드로 대체
    - List<?> 는 null외에는 어떤 값도 넣을 수 없음
    - private helper method로 처리할 수 있음

### 32 제네릭과 가변인수를 함께 쓸때 신중하라
- 가변인수는 메소드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주는데, 구현방식에 허점이 있음
  - 가변인수 메소드를 호출하면, 가변인수를 담기 위한 배열이 자동으로 만들어짐
  - 그런데 내부로 감춰야할 배열이, 클라에 노출됨
  - 그 결과 varargs 매개변수에 제네릭, 매개변수화 타입 포함시 컴파일 에러 남
  - 매개변수화 타입의 변수가 타입이 다른 객체 참조시 힙 오염이 발생함

```java
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = Lists.of(42);
  Object[] objects = stringLists;
  objects[0] = intList;
  String s = stringLists[0].get(0); // ClassCastException
}
```

- 다만 제네릭 가면인수는 안전하고 매우 유용하기 때문에 적용됨
```java
static <T> T[] toArray(T... args) {
  return args;
}
```
- 위 에제처럼, 가변인수의 배열이 밖으로 노출되면, 타입 안전성을 깰 수 있고 힙 오염을 호출쪽 콜스택으로 전이하는 결과를 만든다.


### 33 타입 안전 이종 컨테이너를 고려하라
- 타입 안전 이종 컨테이너
  - 매개변수화된 키를 함게 제공하여 타입을 보장
  - [link](https://github.com/dec7/study/commit/b1f4b6b8ccea9282b0b1b46889d9d11918dad525)

- 제약
  - 1. 악의적인 클라이언트에 의해 잘못된 타입을 전달하면 타입 안전성이 깨짐
  - 2. 실체화 불가 타입에는 사용 불가
    - 이를 해결하기 위해 수퍼 타입토큰으로 시도 할 수 있음 (닐 개프터)
      - ParameterizedTypeReference

- 한정적 타입토큰을 받는 메소드에서 어떻게 형변환을 할까 ?
```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
  Class<?> annotationType = null; //비한정적 타입 토큰
  try {
    annotationType = Class.forName(annotationTypeName);
  } catch (Exception e) {
    throw new IllegalArgumentException(e);
  }
  return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```