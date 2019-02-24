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
