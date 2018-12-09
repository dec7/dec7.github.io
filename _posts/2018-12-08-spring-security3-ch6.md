---
layout: post
title: "SpringSecurity3 Ch6"
description: "Spring Security 3"
date: 2018-12-08
tags: [spring,security]
comments: true
share: true
---

## 고급 설정과 확장

### 커스텀 보안 필터 작성
#### 서블릿 필터 레벨에서 IP 필터링
- 관리자 권한 사용자의 보안 강화
  - ROLE_ADMIN 역할 사용자가 지정된 IP에서만 접근할 수 있도록 필터링
##### 커스텀 서블릿 필터 적용
- [link](https://github.com/dec7/study/commit/48deeab1f7c6616f74f913b96ddbc1a9965c940d)

### 커스텀 AuthenticationProvider 작성
- 요구사항이 스프링 표준 시큐리티 기능을 넘어서는 경우라면 개발자는 AuthenticationProvider를 구현해야함

#### 커스텀 AuthenticationProvider 
- 간단한 싱글사인온 구현
  - 요구사항
    - 사용자명, 비밀번호가 http헤더로 각각 j_username, j_password로 전송
    - 특정 알고리즘으로 인코딩한 j_signature 헤더가 있다고 가정
- 흐름
  - 보통 AuthenticationProvider는 서블필 필터 체인에서 앞의 서블릿 필터가 채워넣은 AuthenticationToken에서 특정 부분 검사
    - FilterSecurityInterceptor
      - 앞에서 요청으로부터 Authentication 토큰 생성
      - 이 인터셉터에서 AuthenticationManager가 토큰을 지원하는 Provider를 찾아서 처리 위힘
      - AuthenticationProvider가 인증결정
  - 따라서, AuthenticationToken 제공자와 이를 사용하는 AuthenticationProvider 가 단절되어 있음

##### 커스텀 인증 토큰
- 사용자가 제시한 토큰을 저장

##### 요청 헤더를 처리하는 서블릿 필터
- 요청 헤더를 분석 + 헤더정보를 새로만든 토큰으로 반환하는 서블릿 필터
- o.s.s.web.authentication.AbstractAuthenticationProcessingFilter
  - 인증을 담당하는 각종 필터 (OpenID, CAS, Form기반 사용자인증) 에서 주로 상속하여 사용
- 필터에서 헤더를 분석해서 토큰을 생성하고 인증요청 시도
- [link](https://github.com/dec7/study/commit/f3be120d982da6c26a6901d3f507c15be10f7d53)

##### Custom AuthenticationProvider 사용시 고려사항
- 사용자의 요청 정보를 Authentication 에 채우는 일은 보통 서블릿 필터 체인 내에 속하는 컴포넌트가 담당
  - 사용자가 제시한 크리덴션의 유효성을 검증하기 위한 데이터가 필요하지 여부에 따라 인증 컴포넌트가 확장되거나 아닐 수 있음
- 유효한 Authentication 토큰을 기반으로 사용자를 인증하는 책임은 AuthenticationProvider 구현체가 담당
- 인증되지 않은 세션이 감지될때 AuthenticationEntryPoint가 필요한 경우도 있음

### 세션관리와 동시성
- 활성화된 세션을 통해 동일 사용자가 보호된 어플리케이션에 로그인 하는 것 감지
- 2가지 방식으로 설정
    - 세션 고정 보호
    - 동시성 제어

#### 세션 고정 보호 설정
- security 네임스페이스 방식 설정은 세션 고정보호는 기본 설정됨
```xml
<!--명시적 설정-->
<http auto-config="true" use-expressions="true">
    <session-management session-fixation-protection="migrateSession" />
</http>
```
- 악의적 사용자로 테스트하기 전까지는 알기 어려움

##### 세션 고정 공격 이해
- 악의적 사용자가 시스템의 인정되지 않은 사용자의 세션을 훔치려는 시도
    - 다양한 기법 존재, 해커는 사용자에 대한 고유세션 식별자 (JSESSIONID 같은) 를 갖게 됨
- 해커가 세션에 접근하는 게 문제, 하지만 (보안설계가 제대로 되었다면) 민감한 영역엔 접근 불가
- 문제는 사용자가 인증받은 후에도 동일 세션 식별자가 사용되는 경우
- 흐름
    - 사용자가 웹 어플리케이션에 접근해 새로운 세션 생성
        - 악의적 사용자가 중간에서 JSESSIONID를 훔침 
    - 사용자가 웹 어플리케이션에 로그인
        - 악의적 사용자는 훔친 값을 사용하여 JSESSIONID 쿠키 구성
            - 해커가 사용자 로그인

##### SpringSecurity로 세션 고정 공격 예방
- 인증 전에 사용자가 가지고 있던 세션이 인증 후에는 사용하지 않도록 처리
- 스프링 시큐리티의 세션 공격 예방 방식
    - 인증 후 명시적으로 새로운 세션 생성, 기존 세션 무효화
- 흐름
    - 사용자가 웹 어플리케이션에 접근해 새로운 세션 새성
        - 악의적 사용자가 JSESSIONID를 훔침
    - 사용자가 웹 어플리케이션에 로그인
        - o.s.s.web.session.SessionManagerFilter
            - 사용자가 새로 인증되었는지 확인
                - 아니오) 필터 체인으로 요청 계속됨
                - 예) o.s.s.web.authentication.session.SessionAuthenticationStrategy
                    - o.s.s.web.authentication.session.SessionFixationProtectionStrategy
                    - 새로운 세션을 생성하도록 Servlet container가 요청을 받음
        - 악의적 사용자가 JSESSIONID로 쿠키 구성후 로그인 시도시
            - 접근 거부

##### 세션 고정 공격 시물레이션
- 세션 고정 보호 비활성화
```xml
<session-management session-fixation-protection="none" />
```

##### 세션 고정 보호 옵션 비교
- session-fixation-protection 옵션
    - none
        - 세션 고정보호 비활성화
    - migrateSession
        - 사용자 인증 후 새로운 세션이 할당되고 기존 세션의 모든 속성값이 새로운 세션으로 이동
        - 사용자 인증 후 사용자 세션에 대한 중요 속성값 (클릭정보, 장바구니 등) 유지 필요시
    - newSession
        - 사용자 인증 후 새로운 세션이 생성

#### 동시 세션 제어를 통한 사용자 보안 강화
- 고정 세션 보호 이후 , 추가 보안 강화 기능
- 단일 사용자가 동시에 고정된 수의 활성화된 세션을 초과해 가질 수 없게 해줌
- [link](https://github.com/dec7/study/commit/0169010e56a144ab3e294bdb7e536c18c158fb92)

##### 동시 세션 제어 설정
- [link](https://github.com/dec7/study/commit/d6b419932b1832f88d7ced265e71cdf6baa9a4ff)

##### 동시 세션 제어 이해
- 세션 절도의 경우, 동시 세션 제어를 적용시 선의의 사용자가 로그인한 상황에서 해커가 훔친 세션을 사용할 수 있는 가능성이 낮아짐
    - 동시 세션 제어는 o.s.s.core.session.SessionRegistry를 사용해 활성화된 HTTP 세션 목록과 해당 세션을 부여받은 인증된 사용자를 관리
    - 세션 상태를 트래킹하기 위해 HttpSessionEventPublisher가 세션 라이프사이클을 기준으로 실시간 업데이트됨
- 흐름
    - 웹 애플리케이션 컨테이너
        - o.s.s.web.session.HttpSessionEventPublisher
            - 세션이 만료되는 경우 이벤트를 보냄
        - o.s.s.web.session.HttpSessionDestroyedEvent
            - ApplicationListener를 통해 통보받음
        - o.s.s.core.session.SessionRegistryImpl
            - 새로운 세션이 생성되면 세션 추가
            - o.s.s.web.authentication.session.ConcurrentSessionControlStrategy
- ConcurrentSessionControlStrategy 
    - 새로운 세션을 트래킹하고, 동시세션제어가 실제 동작하게 하는 기능 제공
    - 사용자가 보호된 사이트에 접근할 때마다 SessionRegistry를 가처 활성화된 세션을 체크하기 위해 SessionManagementFilter가 사용됨
    - 만약 SessionRegistry에 없는 경우 세션 바로 만료

###### 만료 세션 리다이렉트 설정
- [link](https://github.com/dec7/study/commit/7e162d5ca3f75d1f3af0e2c9b23a63747f555c50)

#### 동시 세션 제어의 다른 장점
- SessionRegistry에서 활화성화된 세션을 트래킹해줌
    - 시스템에서 인증된 사용자에 대해 어떤 활동이 일어나고 있는지 런타임시 정보 획득 가능

##### 활성화된 사용자 수 표시
- [link](https://github.com/dec7/study/commit/0dfbda659ffc42bede69fa0ab2c2f8a0620e53c2)

##### 모든 사용자에 대한 정보 보여주기
- SessionRegistry API
    - getAllPrincipals()
        - 활성화된 세션을 가지고 있는 Princial 객체 (보텅 UserDetails)
    - getAllSessions()
        - 각 Principal 이 가지고 있는 세션 정보를 가지고 있는 SessionInformation, 만료된 세션 정보도 담고 있음
- [link](https://github.com/dec7/study/commit/32194f86be653527ff71c2a5ac4606878af92f8a)

### 예외 처리에 대한 이해와 설정
- o.s.s.web.access.ExceptionTransactionFilter
    - 표준 스프링 시큐리티 필터 체인의 마지막 단계
    - 인증, 권한부여 과정에서 발생하는 예외를 검사하고 이 예외에 적절하게 반응하는 일 담당
- 흐름
- ExceptionTransactionFilter
    - AuthenticationException 인지 ?
        - 예) 사후 인증을 위해 요청을 저장
            - o.s.s.web.AuthenticationEntryPoint
                - 사용자를 로그인 페이지나 다른 로그인 매커니즘으로 보냄
                    - (로그인 매커니즘은 인증방식에 의존함)
        - 아니오) AccessDeniedException 인지 ?
            - 아니오) 필터 체인을 따라 위로 예외를 올려보냄 (버블링)
            - 예) 익명사용자인가 ?
                - 예) 사후 인증을 위해 요청을 저장 ...
                - 아니오) o.s.s.web.access.AccessDeniedHandler 
                    - error 페이지 정의되어 있을시, errorPage로 리다이렉트 =
                    - 아닐 경우, HTTP 403 에러 반환
- 정리: ExceptionTranslationFilter 가 처리하는 경우
    - 1) AuthenticationException 이 발생 && 사용자 로그인 해야하는 경우
        - AuthenticationEntryPoint 로직의 의존함
    - 2) AccessDeniedException 발생 && 사용자가 로그인 돼 있지 않은 경우
    - 3) AccessDeniedException 발생 && 사용자가 로그인 되어 있는 경우
        - error page or HTTP403

#### 접근거부 처리 설정
- "접근거부" 포워딩 URL 설정
- AccessDeniedException 에 대한 컨트롤러 추가 처리
- 접근 거부 페이지 작성
- [link](https://github.com/dec7/study/commit/2e64c830d7aaf4a76069e1198298ba72fd5956dd)

#### AccessDeniedException 원인
- AuthenticationException
    - AuthenticationProvider 제공된 크리덴셜이 유효하지 않거나, 계정 만료시
    - DaoAuthenticationProvider가 DAO 접근시 예외 발생
    - RememberMeServices, Remember me 쿠키가 템퍼링 된 경우
    - 인증 클래스에서 예외 발생
- AccessDeniedException
    - AccessDecisionManager 설정된 보터가 접근을 거부하도록 보트하는 경우
    - URL접근, 메소드 접근 레벨 권한 부여 체크 등 보팅 상황에서 발생 가능

#### AuthenticationEntryPoint 중요성
- 인증되지 않은 사용자의 요청을 처리할 때 핵심적인 역할을 함
- ExceptionTransactionFilter가 인증이 필요하다고 판단시 AuthenticationEntryPoint로 가서 다음 할일을 물음
    - 폼 기반 인증의 경우 o.s.s.web.authentication.LoginUrlAuthenticationEntryPoint 는 사용자를 로그인 폼으로 포워딩 하는 일을 책임짐
- 서드파티 인증 시스템과 연동하는 커스텀 연동시 AuthenticationEntryPoint를 개발자가 구현해야함

### 스프링 시큐리티 인프라스트럭처 빈의 수동 설정
- 시큐리티의 기능이 많지만, 모든것을 처리해줄 수는 없고 복잡한 개발환경에서 스프링 시큐리티 필터 체인과 지원 인프라 스트럭처를 처음부터 모두 설정할 수도 있음

#### 스프링 시큐리티 빈의 의존관계 설명

#### 웹 어플리케이션 재설정
##### 최소한의 서블릿 필터 집합 설정
- SecurityContextPersistenceFilter
    - SecurityContext를 설정하는데 사용됨
    - SecurityContext는 전체 요청이 진행되는 동안 요청자와 관련된 인증 정보를 트래킹하기 위해 요청 스코프에서 계속 사용됨
- UsernamePasswordAuthenticationFilter
    - 폼 전송을 처리하고 전송된 내용을 인증 저장소와 비교해 유효한 크리덴셜인지 체크하는데 사용
- AnonymousAuthenticationFilter
    - 익명사용자도 접근을 처리
- FilterSecurityInterceptor
    - 앞에서 처리한 결과인 Authentication 객체를 최종적으로 체크하는 기능을 담당
    - 특정 요청의 승인/거부 여부를 최종적으로 결정

##### 최소한의 지원 객체 집합 설정
- AffiramtiveBased
- RoleVoter
- AuthetnicationVoter
- DaoAuthenticationProvider
- AnonymousAuthenticationProvider
- ProviderManager

### 고급 스프링 시큐리티 빈 기반 설정
#### 세션 라이프사이클과 연관된 요소들의 조정
- AbstractAuthenticationProcessingFilter
    - allowSessionCreation (true)
    - 인증 실패시 (예외를 저장하기 위해 ) 새로운 세션이 생성될 수 있음
- UsernamePasswordAuthenticationFilter
    - allowSessionCreation (true)
    - 마지막으로 시도한 사용자명을 저장하도록 세션을 생성할 수 있음
- SecurityContextLogoutHandler
    - invalidateHttpSession (true)
    - HttpSession 이 무효화됨
- SecurityContextPresistenceFilter
    - forceEagerSessionCreation (false)
    - true인 경우, 필터 체인에 있는 나머지 필터들을 실행하기 전에 필터가 세션을 새엇ㅇ
- HttpSessionSecurityContextRepository
    - allowSessionCreation (true)
    - 사용자의 요청이 끝나는 시점에 SecurityContext가 존재하지 않으면 SecurityContext가 세션에 저장될 수 있음

### 다른 서비스들에 대한 수동 설정
- LogoutFilter
    - 가상 url, j_spring_security_logout 에대한 기본 설정
- SecurityContextLogoutHandler
    - 로그아웃시 기본 세션 처리동작을 조절할 수 있음
- RememberMeAuthenticationFilter
- ExceptionTranslationFilter
    - 지원 빈, AuthenticationEntryPoint, AccessDeniedHandler

#### SpEL 표현식 핸들러와 보터를 사용한 명시적 설정
- WebExpressionVoter
    - use-expression="true"

#### 메소드 보안에 대한 빈 기반 설정

### 인증 이벤트 처리
- 절차
    - o.s.s.authentication.ProviderManager
        - 인터페이스 구현체에 인증 성공/실패 통보
    - o.s.s.authentication.AuthenticationEventPublisher
        - 기본구현체
    - o.s.s.authetnication.DefaultAuthenticationEventPublisher
        - 인증 성공 통보
            - 예) ApplicationEvent 발송
            - 아니오) 제공된 예외 검사, 적절한 ApplicationEvent 에 맵핑
    - o.s.context.ApplicationEventPulbisher
        - 인증 처리 필터 , 권한부텨 인터셉터 에 보냄
    - 등록된 리스너에게 이벤트 다중 전송
        - o.s.context.ApplicationListener
- [link](https://github.com/dec7/study/commit/71ca28c108d303be91ea3f59b7a108aaa0f18be3)