---
layout: post
title: "Effective Java 3e Ch3"
description: "Effective Java"
date: 2019-02-24
tags: [java,pattern]
comments: true
share: true
---

## 모든 객체의 공통 메서드
- Object는 객체를 만들 수 있는 구체 클래스지만, 기본적으로 상속해서 사용하도록 설계됨
  - final 이 아닌 메소드인 equals, hashCode, toString, clone, finalize 모두 재정의 염두해놓고 설계됨
  - 재정의시 지켜야하는 일반 규약이 명확히 정의됨

### 10. equals는 일반 규약을 지켜 재정의
- 여러 문제가 발생할 수 있고, 피하기 위해서는 재정의 하지 않는 것
  - 재정의 하지 않는 경우, 자기 자신만이 같게됨
- 조건
  - 1. 각 인스턴스가 본질적으로 고유
  - 2. 인스턴스의 논리적 동치성을 검사할 일이 없음
  - 3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 맞음
  - 4. 클래스가 private이거나 package-private 이고 equals를 호출할 일이 없음
- equals는 동치성을 구현
  - 반사성
    - ```java x.equqls(x) == true ```
  - 대칭성
    - ```java x.equals(y) == y.equals(x) == true ```
  - 추이성
    - ```java x.equals(y) == y.equqls(z) == x.equqls(z) == true ```
  - 일관성
    - ```java x.equals(y) ``` 결과는 항상 동등
  - null 아님
    - ```java x.equqls(null) == false ```

### 11. equals를 재정의하려면 hashCode를 재정의 하라
- 규약
  - equqls가 변경되지 않은 경우, hashCode는 동일
  - equqls가 같다고 판단한 경우, hashCode는 동일
  - equals가 다르더라도 hashCode가 다를 필요는 없음, 단 다른 객체에 대해서 다른 값을 반환해야 hash table의 성능이 좋아짐
- 좋은 해시 함수
  - 주어진 서로다른 인스턴스들을 32비트 정수 범위에 균일하게 분배
  - 31로 곱할 수를 정하는 이유는 31을 홀수 + 소수이기 때문
- 나쁜 방법
  - 성능을 높이기 위해 해시코드를 계산할 때 핵심 필드를 생략해서는 안됨
    - 해시 품질이 나빠져 해시테이블 성능이 크게 나빠짐

### 12. toString을 항상 재정의하라
- toString을 잘 구현한 클래스 장점
  - 디버깅이 편함
  - 그 객체가 가진 주요 정보를 모두 반환하는게 좋음
  - 포멧에 따른 장단이 존재하나, 의도는 명확히 밝혀야함

### 13. clone 재정의는 주의해서 진행
- 아쉽게도 의도를 제대로 이루지 못함 
  - Cloneable은 복제해도 되는 클래스임을 명시하는 용도로 mixin interface
  - 가장 큰 문제: clone 메소드가 선언된 곳이 Cloneable이 아닌 Object, 이마저도 protected
    - Cloneable을 구현한 것만으로 외부에서 clone을 호출할 수 없음 
    - [link](https://github.com/dec7/study/commit/f935d48bc3c5243a92cf971ef31eb70dbf696731)
    - cloneable 아키텍쳐는 '가변객체를 참조하는 필드는 final'로 선언하라는 용법과 충돌하며 final을 지워야 할 수도 있음
- Cloneable 인터페이스 역할 
  - Object의 protected 메소드 clone 동작방식 결정
- clone 메소드는 사실상 생성자와 같은 효과 
  - clone 은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불편식을 보장해야함
- 주의
  - 상속용 클래스는 cloneable 구현해서는 안되며, 동작하지 못하도록 구현해둠 (final)
  - 적절한 동기화 필요
  - Cloneable을 구현하는 모든 클래스는 clone 을 재정의 해야함
    - public, 반환타입은 자신으로, super.clone() 호출 후 이후에 적절한 수정 처리

### 14. comparable 을 구현할지 고려
- 알파벳, 숫자, 연대 등 순서가 명확한 값 클래스 작성시 반드시 구현
- 규약
  - 순서를 비교, 주어진객체보다 작으면 음수, 같은면 0, 크면 양의 정수 반환
    - 비교할 수 없는 경우 ClassCastException을 던진다
  - ```java sgn(x,compareTo(y)) == -sgn(y.compareTo(x)) ```
  - 추이성, 반사성, 대칭성 보장
  - 필수는 아니지만, compareTo 메소드로 수행한 동치성 테스트 결과가 equals와 같아야 함
- [link](https://github.com/dec7/study/commit/41efaaa012f3c9a449becadc3b455dada5e8ad3f)
