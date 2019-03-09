---
layout: post
title: "Effective Java 3e Ch12"
description: "Effective Java"
date: 2019-03-09
tags: [java,pattern]
comments: true
share: true
---

## 12. 직렬화
- 객체 직렬화
  - 자바가 객체를 바이트 스트림으로 인코딩 하고, 바이트 스트림으로 부터 다시 객체를 재구성하는 매커니즘

### 85. 자바 직렬화의 대안을 찾으라
- 보안 취약함
- 해결방법
  - 1. 쓰지 않는것
  - 2. 레거시 제외, 신뢰할 수 없는 데이터는 절대 역직렬화하지 않는 것
  - 3. 화이트리스트 방식 클래스 직렬화 허용
- 대안
  - json
    - text형식, 사람 읽을 수 있음
    - 오직 데이터 표현
  - protobuf
    - 이진 형식, 효율기반
    - 문서를 위한 스키마(타입) 제공하고 올바로 쓰도록 강요
    - pbtxt

### 86. Serializable 을 구현할지는 신중히 결정
- 구현 후 릴리즈 후에는 수정 어려움
- 보안 취약함
  - 객체는 생성자로 만들어져야 함, 직렬화는 언어의 기본 매커니즘을 우회하는 것
- 신버전 릴리즈시 테스트 증가
- 구현 여부는 신중함 필요
- 상속용으로 설계된 클래스나 인터페이스도 Serializable 구현/확장 하면 안됨
- 내부 클래스는 직렬화를 구현하지말아야 함

### 87. 커스텀 직렬화 형태를 고려해봐라

### 88. readObject 메소드는 방어적으로 작성하라

### 89. 인스턴스 수 통제가 필요한 경우 readResolve보다 열거타입을 사용

### 90. 직렬화된 인스턴스 보다, 직렬화 프록시 사용을 고려