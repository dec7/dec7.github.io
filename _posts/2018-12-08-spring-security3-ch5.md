---
layout: post
title: "SpringSecurity3 Ch5"
description: "Spring Security 3"
date: 2018-12-08
tags: [spring,security]
comments: true
share: true
---

## 미세 접근 제어

### 애플리케이션 기능과 보안에 대해 재고민
#### 애플리케이션 보안 기획
- 보안팀, 엔지니어링팀 회의
#### 사용자 역할 기획
- 사용자 계층의 역할은 GrangedAuthority에 맵핑됨
- 계층 (계층 / 설명 / 역할)
  - 손님 
    - 모르는 사용자나 인증되지 않은 사용자 / 없음
  - 고객
    - 계정을 설정 / ROLE_CUSTOMER, ROLE_USER
  - 구매완료한 제품이 있는 고객
    - 한번 이상 구매를 마친 사용자 / ROLE_PURCHASER, ROLE_USER
  - 관리자
    - 사용자계정 관리 책임이 있는 사용자 / ROLE_ADMIN, ROLE_USER
  - 공급자
    - 제품의 재고를 관리할 수 있는 공급자 / ROLE_SUPPLIER, ROLE_USER
#### 페이지 레벨 보안 기획
- 페이지 레벨요소별 보안 기획
- 사용자가 페이지 이동시 동이한 사용자경험을 느끼도록 사이트전반에 대한 페이지 기능 기획 필요

### 다양한 미세 권한 부여방법
- 미세권한 부여는 특정 사용자의 요청 컨텍스트에 따라 애플리케이션 기능을 사용할 수 있게 하는 것
  - 2장 스프링 시큐리티 시작, 3장 사용자 경험 개선, 4장 크리덴션 안전하게 저장 부분은 페이지 전체에 대한 접근 제한
  - 미세 권한 부여는 페이지의 특정 영역을 선택적으로 접근할 수 있게 함
- 방법
  - 1) 스프링 스큐리티 jsp 태그 라이브러리 활용
  - 2) mvc컨트롤러에서 군한을 체크후 접근결정을 내리고 접근결정 결과를 모델 데이터에 반영, 복잡하지만 mvc설계에 적합

#### tag library 활용한 조건적 콘텐츠 렌더링
- 권한부여 규칙에 따라 페이지 영역을 조건적으로 렌더링하는 기능을 가장 많이 사용
- ```xml <authorize>``` 태그 활용

##### url 접근 규칙을 바탕
- 적절한 경우만 MyAccount 링크가 표시되도록 설정
```xml
// context
<intercept-url url="/account/*" access="hasRole('ROLE_USER') and fullyAuthenticated" />

// jsp
<sec:authorize url="/account/home">
// <sec:authorize url="/account/home" method="GET"> HTTP 메소드 권한부여도 체크 가능
  <li><a href="/account/home">My Account</a></li>
</sec:authorize>
```

##### Spring EL 표현식 기반
- authorize 태그를 SpringEL 표현식과 연동

#### 컨트롤러 로직을 사용한 조건적 컨텐츠 렌더링
- LogIn 링크를 보여줄지 판단하는 Boolean 변수가 모델데이터에 있다고 가정
- 컨트롤러에서는 링크 노출여부를 판단후 모델데이터에 추가
```java
protected Authentication getAuthentication() {
  return SecurityContextHolder.getContext().getAuthentication();
}

@ModelAttribute("showLoginLink")
public boolean getShowLoginLink() {
  for (GrantedAuthority authority : getAuthentication().getAuthorities()) {
    if (authority.getAuthority().equals("ROLE_USER")) {
      return false;
    }
  }
  return true;
}
```
#### 페이지내 권한부여를 설정하는 가장 좋은 방법
- 장점
  - 페이지를 단일 url로 분리할 수 있는 경우 분명히 시별 가능
- 단점
  - 하지만 일반적으로 어플리케이션 복잡도 때문에 간단하게 적용하기 어려움
  - 렌더링 조건이 훨씬 많을 수 있음
  - 역할 멤버십을 벗어난 복잡한 조건 지원 안함
  - authorize 태그가 모든 페이지에서 참조되어야 하고 역할 규칙이 서로다른 페이지로 나뉘져 위험 요소가 됨
  - 컴파일 시에 선언된 규칙의 정확성을 검증하기 어려움
- 따라서 컨트롤러 방식으로 사용하는게 나을 수 있음

### 비지니스 티어 보호
- 메소드를 보호할 수 있는 방법
  - 사전 권한부여
    - 허용된 메소드가 실행되기 전에 특정 제약조건을 항상 충족시켜줌
    - 만약, 선언된 조건을 충족하지 못하면 메소드 호출 실패
  - 사후 권한부여
    - 메소드가 반환된 후에도 선언된 제약조건을 만족시키도록 보장
    - 잘 사용되지 않지만 복잡하게 연관된 비지니스 티어 메소드 주변에 보안 레이어를 추가로 둘 수 있는 기능 제공
  - 특징
    - 사전/사후 조건은 런타임시 검사하여 메소드 실행과 관련된 조건이 항상 참이 되도록 선언할 수 있음

#### 비지니스 메소드 보안 기본
- [link](https://github.com/dec7/study/commit/d0b42fec74c2f067c19fa9ff731b931e288c9ccc)

#### 메소드 보안의 여러가지 방식
##### JSR-250 호환표준 규칙
- jsr-250 호환 런타임 환경에서 사용할 수 있는 보안관련 어노테이션을 정의
  - 스프링은 2.x부터 jsr-250 호환성 유지
  - jsr-250은 스프링 네이티브 어노테이션 만큼은 아니지만, 자바EE, Apache Tuscany 같은 프레임웍에서 함꼐 지원
- [link](https://github.com/dec7/study/commit/fe90ae0228cfac98a3a07328a6de3d302f712aee)
- 클래스 레벨 어노테이션
  - 메소드 레벨처럼 동일하게 적용
  - 메서드 레벨 어노테이션이 항상 오버라이드됨

##### Spring @Secure 어노테이션
- @RolesAllowed와 동일, 그러므로 @Secured 어노테이션을 사용할 이유 없음
```xml
<global-method-security secured-annotations="enabled" />
```

##### 관점지향 프로그래밍을 사용한 메소드 보안 규칙
- aop 사용하여 메소드나 메소드 집합에 포인트컷을 설정하고 멤버십을 체크하는 어드바이스를 사용

##### 메소드 보안 동작원리
- 접근방식은 웹 방식과 동일
  - AccessDecisionManager는 접근판단을 AccessDecisionVoter에 위임
  - AccessDecisionManager는 Voter의 결정의 취합하여 메소드 호출 여부를 결정
- 흐름
  - 사용자가 메소드 호출
    - 스프링 Aop가 요청을 가로챔
      - o.s.s.access.intercept.aopalliance.MethodSecurityInterceptor
    - 접근이 허용되는지 판단
      - o.s.s.access.AccessDecisionManager
        - o.s.s.access.prepost.PrePostAnnotationSecurityMetadataSource 로 부터 ConfigAttribute를 가져옴
        - AccessDecisionVoter를 검사
          - o.s.s.access.prepost.PreInvocationAuthorizationAdviceVoter 
            - o.s.s.access.prepost.PreInvocationAuthorizationAttribute 에서 접근결정 가져옴
    - 접근 거부시
      - AccessDeniedException
    - 접근 허용시
      - 메소드 호출
- 호출대상
  - global-method-security 이 정의되면 
    - o.s.beans.factory.config.BeanPostProcessor가 등록되어 AOP설정 검사 개입여부 확인

### 고급 메소드 보안
#### 빈 데코레이터를 사용한 메소드 보안 규칙
```xml
<bean id="userService" class="...">
  <security:intercept-methods>
    <security:protect access="ROLE_USER" method="changePassword"/>
    <!--<security:protect access="ROLE_USER" method="set*"/>-->
  </security:intercept-methods>
</bean>
```

#### 메소드 매개변수를 포함하는 메소드 보안 규칙
- 자신의 비밀번호를 수정할 수 있도록 보안설정하기 위해
```java
@PreAuthorize("#username == principal.username and hasRole('ROLE_USER')")
public void changePassword(String username, String password);
```

### 역할 기반 필터링을 통한 메소드 데이터 보호
- 보안 트리밍, 보안 프루닝
  - 컬렉션이나 배열에 보안 기반 필터링 적용
  - @PreFilter, @PostFilter
  - 런타임시 주체의 크리덴셜을 사용해 특정 객체에서 선택적으로 맴버를 제거하는 역할을 함

#### PostFilter 로 역할기반 데이터 필터링
- [link](https://github.com/dec7/study/commit/9dc5ffdb6bc380c6fee230edca50958fc73fb167)
- @PostFilter는 사전 조건이 아닌 메소드 동작을 규정
  - orm과 바인딩되는 경우 저장까지 갈수 있으므로 조심

#### @PreFilter를 사용한 사전 필터링
- 현재 보안 컨텍스트에서 메소드에 전달된 컬렉션 요소를 필터링 하기 위해 사용
  - 컬렉션만 지원, 배열 지원 안함
  - 하나 이상의 인자를 가지는 경우, 필터를 적용할 메소드 매개변수를 지정하는데 사용할 추가선택 어트리뷰트 filterTarget을 가지고 있음
  - 전달된 원본 컬렉션이 영구 수정됨
- 사용하는 이유
  - 데이터 레이어에서 쿼리 조회시 전달된 매개변수에 대한 보안 검사를 할 수 있음
  - 비지니스 티어에서 반환할 정보를 여러 데이터 소스로부터 수집하는 경우가 있고, 이 경우 보안 트리밍이 필요할 수 있음
- [link](https://github.com/dec7/study/commit/9dc5ffdb6bc380c6fee230edca50958fc73fb167)

