---
layout: post
title: "Effective Java 3e Ch4"
description: "Effective Java"
date: 2019-03-02
tags: [java,pattern]
comments: true
share: true
---

## 클래스와 인터페이스
### 15. 클래스와 멤버의 접근 권한 최소화
- 잘 설계된 컴포넌트
  - 얼마나 캡슐화가 잘 되었는지 ?
    - 클래스 내부 데이터, 내부 구현 정보를 외부로부터 얼마나 잘 숨겼는지 ?
    - 구현과 API를 얼마나 깔끔하게 분리했는지 ?
- 원칙
  - 모든 클래스와 멤버의 접근성을 가능한 좁힌다.
  - private
    - 멤버를 선언한 톱레벨 클래스에서만 접근
  - package-private
    - 멤버가 소속된 패키지 안의 모든 클래스에 접근
  - protected
    - 멤버를 선언한 클래스 하위 클래스에서 접근
  - public
    - 모든곳에서 접근
- 접근수준
  - 같은 패키지의 다른 클래스가 접근해야하는 경우, package-private으로 변경
  - protected로 바꾸는 순간 공개API되므로 주의
  - 테스트 목적인 경우 package-private까지는 허용 가능
  - 길이가 0이 아닌 배열은 변경 가능하니 주의해야함
- [link](https://github.com/dec7/study/commit/5958be008cfcff0b77877ccd1a5d3eded53ddec3)

### 16. public 클래스 public 필드가 아닌 접근자를 사용

### 17. 변경 가능성을 최소화하라
- 불편클래스
  - String, 기본타입의 박싱된 클래스, BigInteger, BigDecimal
  - 구현이 쉽고, 오류 가능성이 낮음
- 만들기 위한 5가지 규칙
  - 객체 상태를 변경하는 메소드를 제공하지 않음
  - 클래스를 확장하지 못하도록 함
  - 모든 필드를 final로 선언
  - 모든 필드를 private으로 선언
  - 자신외에는 내부의 가변 컴포넌트에 접근할 수 없도록 함
- 특징
  - 스레드 안전하여 따로 동기화 불필요
  - 안심하고 공유 가능
  - 적극적으로 인스턴스 재활용
  - 불변객체끼리는 내부 데이터 공유 가능
  - 그 자체로 실패 원자성 제공
    - 실패원자성: 메소드에서 예외가 발생한 후에도 그 객체는 여전히 동일한 상태 
  - (단점) 값이 다르면 반드시 독립된 객체로 만들어야 함
- 성능
  - 원하는 객체를 만들기까지 단계가 많고, 중간단계에서 만들어진 객체가 모두 버려지는 경우 성능문제 발생 가능
    - 다단계 연산 (multistep operation) 예측하여 기본 제공
      - 가변동반클래스를 private package로 제공하여 빠른 연산을 제공
- 상속을 못하도록 구현
  - 정적 팩토리 제공
- 직렬화
  - realObject, realResolve 메소드를 반드시 제공하거나
  - ObjectOutputSteam.writeUnshared, ObjectInputStream.readUnshared 를 사용해야함
- 결론
  - 불변
  - 불변이 아니라면 변경부분 최소화
  - 특별한 이유가 없는경우 필드는 private final
  - 생성자는 불편식 설정이 완료된, 초기화가 완벽히 끝난 상태의 객체를 반환
  - 확실한 이유가 없는경우 생성자, 정적 팩토리외에는 public이 아니어야 함

### 18. 상속보다 컴포지션을 사용
- 단점
  - 패키지 경계를 넘어 다른 패키지의 구체 클래스를 상속하는건 위험
  - 메소드 호출과 다르게 상속은 캡슐화를 꺠뜨림
  - 상위 클래스가 메소드를 추가할 경우, 재정의 필요
- 래퍼 클래스
  - 래퍼 클래스가 콜백 프레임웍과는 어울리지 않음
- 상속은 반드시 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 쓰여야 함 (is-a관계)

### 19. 상속을 고려해 설계-문서화, 그렇지 않은경우 상속 금지
- 상속용으로 만든 클래스는 배포전에 하위 클래스를 최소 3개를 만들어 검증
- 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메소드를 호출해서는 안됨
  - private, final, static메소드는 재정의 불가능, 생성자에서 안심하고 써도 됨
- 주의
  - clone, readObject 모두 직접적, 간접적이든 재정의 가능메소드를 호출해서는 안됨
- [link](https://github.com/dec7/study/commit/61d34f38ebfda84c55554a7c2dcf2ef2104c7a02)

### 20. 추상 클래스보다 인터페이스를 우선하라
- 차이
  - 둘의 가장 큰 차이는 추상 클래스가 정의한 타입을 구현한 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 한다는 것
  - 반면 인터페이스는 선언한 메소드를 모두 정의하면 어떤 다른 클래스를 상속했든 같은 타입으로 취급됨
- 인터페이스 장점
  - 기존 클래스에도 쉽게 인터페이스 구현 가능
  - 믹스인 정의에 적절
    - 믹스인: 클래스가 구현할 수 있는 타입, 원래 주된 타입외에도 특정 선택적 행위를 제공한다고 선언하는 효과
  - 계층구조가 없는 타입 프레임워크를 만들 수 있음 
    - [link](https://github.com/dec7/study/commit/8722a109ca0f7612c49c9ef06e5267fb3af7a0e2)
  - 기능을 향상 시키는 안전하고 강력한 수단

### 21. 인터페이스는 구현하는 쪽을 생각하라
- 디폴트 메소드
  - 재정의하지 않은 모든 클래스에서 디폴트 구현이 사용됨
    - jdk8이 전에 구현된 구현체에 대해 문제가 발생할 수 있음
  - 컴파일에 성공하더라도 런타임 오류가 날 수도 있음
  - 디폴트 메소드는 인터페이스로부터 메소드를 제거하거나 수정하려는 용도가 아님

### 22. 인터페이스는 타입을 정의하는 용도로만 사용하라
- 상수 인터페이스는 안티 패턴
  - 클래스 내부에서 사용하는 상수는 내부 구현에 해당함
  - 상수 인터페이스를 구현하는 것은 내부 구현을 API로 노출하는 행위
- 상수를 공개할 목적이라면 
  - 특정 클래스나 그 인터페이스 자체에 추가해야함

### 23. 태그 달린 클래스보다 클래스 계층구조 활용하라
- 태그 달린 클래스는 장황하고 오류를 내기 쉽고 비효율적
  - 생성자가 태그 필드를 설정하고 적절한 의미였는지 컴파일러가 도와줄수 없음
  - 태그 달린 클래스는 클래스 계층구조를 어설프게 흉내낸 아류일뿐

### 24. 멤버 클래스는 되도록 static으로 만들라.
- 중첩클래스  
  - 다른 클래스 안에 정의된 클래스로 자신을 감싼 바깥클래스에서만 사용되어야 함
  - 종류
    - 정적 멤버 클래스
    - 멤버 클래스
    - 익명 클래스
    - 지역 클래스
    - 정적 멤버 클래스를 제외한 나머지는 내부 클래스에 해당함
- 정적 멤버 클래스
  - 바깥 클래스의 private 멤버에 접근할 수 있는것 제외하면 일반 클래스와 동일
  - 바깥 클래스와 함께 쓰일때만 유용함
- 멤버 클래스
  - 정적 멤버 클래스와 static 유무가 차이
  - 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결됨 
    - 비정적 멤버 클래스의 인스턴스 메소드에서 정규화된 this를 사용해 바깥 인스턴스의 메소드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있음
    - 바깥 인스턴스 없이 생성할 수 없음
  - 어댑터를 정의할 때 자주 쓰임
    - 어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 뷰로 사용
    - Map 인터페이스의 keySet, entrySet, values 처럼 자신의 컬렉션 뷰를 구현
    - 멤버 클래스에 바깥 인스턴스에 접근할 일이 없는 경우 무조건 static을 붙여서 정적 멤버 클래스로 사용

### 25. 톱레벨 클래스는 한 파일에 하나만 담아라
- 많아져서 문제는 없으나, 심각한 위험을 만들 수 있음
  - 한 클래스를 여러 가지로 정의할 수 있고, 그중 어느 것을 사용할지는 어떤 소스파일을 먼저 컴파일하느냐에 따라 달라짐