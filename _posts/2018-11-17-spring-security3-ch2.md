---
layout: post
title: "SpringSecurity3 Ch2 - start"
description: "Spring Security 3"
date: 2018-11-17
tags: [spring,security]
comments: true
share: true
---

## 보안개념
### 인증
- 사용자가 주장하는 본인이 맞는지 확인하는 절차 

#### 크리덴션 기반 인증
- 사용자가 입력한 비밀번호를 기반으로 저장된 비밀번호가 일치하는지 확인
- 크리덴션은 위치에 상관없이 확인 가능

#### 이중 인증
- atm 인출, 카드에 담긴 정보와 사용자의 비밀번호를 조합하여 접근 가능여부 판단

#### 하드웨어 인증
- 자동차 열쇠

#### Java
- Principal (java.security.Principal)
  - 자바 표준 보안 인증 개념
- Spring Security는 위 Principal을 확장해 사용
  - 어디든 매핑이 가능하지만, 단일 사용자라고 가정

### 권한부여
- 두 작업의 묶음
  - 인증된 주체를 하나 이상의 권한 (역할) 에 맵핑
  - 보호된 리소스에 대한 권한 체크 (코드에서 명시적인 처리 혹은 설정기반)

## 보안 적용
### 가장 기초적인 적용
- 모든 path로 접근시 인증요청
  - [link](https://github.com/dec7/study/commit/f133142429157501df62419caaa33f2fb2f7637d)

### 부족한 부분
- 현재는 id/pw가 하드코딩 되어 있음
  - db기반 인증 프로바이더 사용필요
- 익명 사용자 접근 불가
  - 익명, 인증 사용자, 관리자의 역할 정의 필요
- 너무 평범한 로그인 폼

## 아키텍쳐
### 흐름
- 사용자가 페이지를 요청
  - 인증관리자가 요청을 가로챔
    - 사용자에게 크리덴셜 요청
    - 사용자가 크리덴션 제공
    - 인증관리자는 사용자 크리덴션 저장소에서 알고인는 크리덴션과 비교해 검증
- 인증된 사용자가 자원을 요청
  - 접근결정관리자가 요청을 가로챔
    - 시스템 리로스 권한부여 선언을 통해 사용자가 권한을 가지고 있는지 검증
    - 적절한 자원이 있는 경우 자원 제공

### 스프링 시큐리티
#### 기본 설정
- 자동 설정시 10개의 서블릿 필터를 설정
  - o.s.s.web.context.SecurityContextPersistenceFilter
    - SecurityContextRepository에서 SecurityContext를 로드하고 저장
    - SecurityContext는 사용자의 보호되고 인증된 세션을 나타냄
  - o.s.s.web.authentication.logout.LogoutFilter
    - 로그아웃 url (/j_spring_security_logout) 로 지정된 가상 url에 대해 요청 감시하고 요청시 로그아웃
  - o.s.s.web.authentication.UsernamePasswordAuthenticationFilter
    - 사용자명, 비밀번호로 이뤄진 폼 기반 인증에 사용하는 가상 url (/j_sprint_security_check) 요청을 감시하고 요청시 인증 진행
  - o.s.s.web.authentication.ui.DefaultLoginPageGeneratingFilter
    - 폼 기반, OpenID 기반 인증에 사용하는 가상 url (/spring_security_login) 요청을 감시하고 폼로그인 수행시 필요한 html 생성
  - o.s.s.web.authentication.www.BasicAuthenticationFilter
    - http 기본 인증 헤더를 감시하고 처리
  - o.s.s.web.savedrequest.RequestCacheAwareFilter
    - 로그인 성공 이후 인증 요청에 의해 가로채진 사용자 원래 요청을 재구성하는데 사용됨
  - o.s.s.web.servetapi.SecurityContextHolderAwareRequestFilter
    - HttpServletRequest를 SecurityContextHolderAwareRequestWrapper로 감싼다
  - o.s.s.web.authentication.AnonymousAuthenticationFilter
    - 이 필터가 호출되는 시점까지 사용자가 인증을 받지 못하면, 인증토큰에서 사용자는 익명사용자로 나타남
  - o.s.s.web.session.SessionManagementFilter
    - 인증된 주체를 트래킹 처리
  - o.s.s.web.access.ExceptionTranslationFilter
    - 보호된 요청을 처리하는 동안 발생할 수 있는 기대한 예외와 기본 라우팅과 위임 처리
  - o.s.s.web.access.intercepter.FilterSecurityInterceptor
    - 권한 부여와 관련된 결정을 AccessDecisionManager 에게 위임해 권한부여 결정 및 접근제어결정을 쉽게 만들어줌

#### 커스텀
- 위는 기본 설정에 대한 필터
- 필요시 추가 및 생성 가능

#### auto-config가 하는 일
- 기본 기능
  - http 기본인증
  - 폼 로그인 인증
  - 로그아웃

#### 사용자가 인증 받는 방법 
- 폼 로그인 예제로 전체 아키텍처는 동일

- 사용자 요청
  - AbstractAuthenticationProcessingFilter로 요청됨
    - o.s.s.core.Authentication 생성 (요청시 받은 사용자의 크리덴션을 나타냄)
    - AuthenticationManager로 검증 요청
      - AuthenticationProvier가 인증 크리덴션 저장소를 통해 요청된 크리덴션을 검증
        - 유효하지 않은 경우, 예외 던짐
        - 유요한 경우, 요청된 Authentication에 Principal 정보를 채움
          - o.s.s.core.GrantedAuthority 와 1:n 구조
  - 사용자가 인증됨
  
##### 주요 컴포넌트
- AbstractAuthenticationProcessingFilter
  - 웹 기반 인증 요청에 사용됨
  - 폼Post, SSO, 기타 사용자가 제공한 크리덴션을 포함한 요청을 처리
  - 책임 사슬에 따라, 사용자 크리덴션을 전달하기 위해 Authentication 생성
- AuthenticationManager
  - 사용자 크리덴션 검증
    - 인증 실패시 예외
    - 인증 성공시 Authentication 객체를 채움
- AuthenticationProvider
  - AuthenticationManager에 크리덴션 검증 기능을 제공
  - 구현체는 db 와 같은 크리덴셜 저장소를 바탕으로 크리덴션을 검증

##### Authentication
- 자주사용될 인터페이스
- 사용자 식별자, 크리덴션, 사용자에게 주어진 하나 이상의 권한을 저장
```java
Object getPrincipal()  // 주체에 대한 고유 식별자 반환 (사용자명)
Object getCredenticals()  // 주체의 크리덴션 반환
List<GrantedAuthority> getAuthorities()  // 인증 저장소에 의해 결정된 주체의 권한 집합 반환
Object getDetails()  // 주체에 대한 인증 프로바이더 의존적인 상세 정보를 반환
```

##### 폼 기반 인증 상세 플로우
- AbstractAuthenticationProcessingFilter
  - UsernamePasswordAuthenticationFilter 로 요청
- UsernamePasswordAuthenticationFilter 
  - HttpServetRequest에서 사용자명, 비밀번호 획득
  - UsernamePasswordAuthencationToken 생성
  - AuthencationManager 인터페이스로 크리덴션 검증
- ProviderManger (AuthencationManager 인터페이스로 구현됨)
  - 반복하여 모든 ProviderManager 를 검사
    - 이 프로바이더에서 UsernamePasswordAuthencationToken 을 지원하는지
      - 인증 실패시, 예외 던짐
      - 인증 성공시, Authentication 객체 반환

#### spring_security_login 이란
- DefaultLoginPageGeneratingFilter 에 지정된 기본 로그인 페이지

```html
<form name='f' action='/JBCPPets/j_spring_security_check' method='POST'>
  User:<input text='text' name='j_username' value='' />
  Password:<input type='password' name='j_password' />
  <input name='submit' type='submit' />
  <input name='reset' type='reset' />
</form>
```

- ```j``` 로 시작하는 값은 폼 필드 이름은 UsernamePasswordAuthenticationFilter 자체에서 지정됨
  - java EE servlet 2.x 명세를 따름 (SRV.12.5.3)  

#### 사용자의 크리덴션이 검증되는 곳

``` xml
<authentication-manager alias="authenticationManager">
    <authentication-provider>
      <user-service>
        <user authorities="ROLE_USER" name="guest" password="guest"/>
      </user-service>
    </authentication-provider>
  </authentication-manager>
```

- AuthencationManager 구현체는 하나 이상의 AuthenticationProvider 구현체 지원
- 기본적으로 o.s.s.web.authentication.dao.DaoAuthenciationProvider 를 사용됨
  - AuthenticationProvider를 구현한 가벼운 래퍼고, o.s.s.core.userdetails.UserDetailService 에 작업 위임
    - o.s.s.core.userdetils.memory.InMemoryDaoImpl 구현체로 설정됨
    - 위 서비스는 UserDetails 를 반환

##### Authencation, UserDetails 차이
- Authenciation
  - 주체의 신원, 비밀번호, 인증요청과 관련된 컨텍스트에 대한 정보 저장
  - 이 곳에는 사용자에 대한 인증 후 정보 (UserDetails 일수도) 가 들어 있을 수 있음
  - 인증 과정에서 특별히 필요한 정보를 지원하기 위한 용도를 제외하고는 확장하지 않는게 일반적
- UserDetails
  - 이름, 이메일, 전화번호 등 주체의 프로필을 저장하기 위한 용도
  - 업무상 요구조건을 만족하기 위해 확장하는게 일반적

##### 인증 기능 제공 흐름
- UsernamePasswordAuthenticationToken 이 인증결정을 위해 AuthenticationManager로 제공됨
- AuthenticationManger
  - DaoAuthenticationProvider 로 위임
    - InMemoryDaoImpl에서 UserDetails를 조회
    - Authentication 객체에서 얻은 크리덴셜을 UserDetails에서 얻은 크리덴셜과 비교해 검증

##### 인증시 예외가 발생하는 지검
- 인증과 관련된 예외는 o.s.s.core.AuthenticationException 을 상속함
  - authentication
    - 인증 요청과 관련한 Authentication 객체 보관
  - extraInformation
    - 특정 예외에 관련된 부가 정보 포함
    - BadCredenticalsException
      - 사용자명이 없거나 비밀번호 불일치 / UserDetails
    - LockedException
      - 사용자 계정에 락이 걸린 경우 / UserDetails
    - UsernameNotFoundException
      - 사용자명을 인증저장소에서 찾을 수 없거나, GrandedAuthority가 없는 경우 / 사용자 명을 포함한 문자열
- 인증 예외시
  - 적절한 페이지로 리다이렉트 되거나
  - 403 상태코드 반환


### 요청이 권한부여를 받는 방법
#### 흐름
- FilterSecurityInterceptor
  - 기본 스프링 시큐리티 필터 체인에서 제일 마지막 단계
  - 인증 완료된 상태로 접근
  - ```java List<GrantedAuthority> getAuthrities()``` 이 메소드 정보를 활용해 특정요청을 거부/승인 여부를 결정
- 권한부여는 이진결정으로 모호한 영역은 없음

#### o.s.s.access.AccessDecisionManager
- 권한부여 결정 책임
- 메소드
  - supports
    - 구현체가 현재 요청을 지원하는지 여부를 알려주는 두개의 메소드로 이뤄짐
  - decide
    - 요청 컨텍스트와 보안 설정을 바탕으로 접근을 승인할지 여부를 결정
    - 접근거부시 예외가 발생
- o.s.s.access.AccessDenidedException
  - 권한부여시 가장 흔하게 발생하며, 필터 체인에서 특별히 처리해야할 예외
- 기본 AccessDecisionManager
  - AccessDecisionVoter 와 Vote 취합을 통해 접근승인 방식 제공
- Voter
  - 권한 부여 과정중 다음 중 하나 / 전체를 판단하는 대상
    - 보호된 리소스에 대한 요청 컨텍스트
    - (존재시) 사용자가 제시한 크리덴셜
    - 접근하려는 보호된 리소스
    - 시스템 설정 매개변수롸 리소스 자체
- 요청된 리소스에 대한 access 선언을 voter에게 전달하는 책임
  - ```xml <intercept-url pattern="/*" access="ROLE_USER" /> ```

#### Voter
- 보터는 자신의 정보로 사용자가 리소스에 접근 가능한지 판단
- 3개 중 한가지
  - ACCESS_GRANTED 승인
    - 리소스에 대한 접근권한을 줄것을 권장
  - ACCESS_DENIDED 거부
    - 리로스에 대한 접근권한을 거부할 것을 권장
  - ACCESS_ABSTAIN 보류
    - 리소스에 대한 접근권한을 보류할 것을 권장 (의사결정 안내림)
      - 보터가 결정적인 정보가 없는 경우
      - 보터가 이 타입의 요청에 대한 결정을 내릴수 없는 경우

#### 흐름
- AbstractSecurityInterceptor
  - DefaultFilterInvocationSecurityMetadataSource 를 통해 ConfigAttribute 획득
    - ConfigAttribute는 요청 url 패턴
- AffirmativeBased 로 decide 호출
  - 순환문을 통해 모든 Voter 검사
    - 익명사용자, remember me 인증 사용자를 처리하기 위한 AuthenticatedVoter로 제공함
    - RoleVoter의 사용자 GrantedAuthority가 
      - ROLE_ 접두사로 시작하지 않는 경우 -> ACCESS_ABSTAIN
      - ConfigAttribute 와 불일치 -> ACCESS_DENY
      - ACCESS_GRANTED

#### 접근결정의 취합방식 설정
- <http> 엘리먼트에서 access-decision-manager-ref 어트리뷰트는 AccessDecisionManager 를 구현한 스프링 빈에 대한 래퍼런스를 설정할 수 있도록 함
- 구현체
  - AffirmativeBased
    - 보터가 접근 승인시, 이전에 거부된 내용과 상관없이 접근이 바로 승인됨
  - ConsensusBased
    - 다수 표가 AccessDecisionManager의 결정에 영향을 줌
    - 동률, 무효표에 대한 처리는 설정 가능
  - UnanimousBased
    - 모든 보터가 만장일치로 접근을 승인해야 함. 아니면 거부됨
    - [link](https://github.com/dec7/study/commit/f132d63dc59bfdcaffccfae90ee0e3e0c6f54f2f)
    - o.s.s.access.vote.RoleVoter
      - 사용자가 선언된 역할에 부합하는 GrantedAuthority를 가졌는지 확인
      - 이 구현체는 역할이 콤마로 구분될거라고 예상
        - ```access="ROLE_USER,ROLE_ADMIN"```
    - o.s.s.access.vote.AuthenticatedVoter
      - 와일드카드와 매칭되는 특수 선언을 지원
        - IS_AUTHENTICATED_FULLY
          - 새로운 사용자명과 비밀번호가 입력된 경우 접근 승인
        - IS_AUTHENTICATED_REMEMBERED
          - remember me 기능을 사용해 인증한 경우 접근 승인
        - IS_AUTHENTICATED_ANONYMOUSLY
          - 익명 사용자인 경우 접근 승인
        - ```access="IS_AUTHENTICATED_ANONYMOUSLY"```

#### 스프링 표현식 언어를 사용한 접근 설정
- SpEL을 사용해 설정
  - ```xml <http auto-config="true" use-expressions="true" >```

```xml
<http auto-config="true" use-expression="true">
  <intercept-url pattern="/*" access="hasRole('ROLE_USER')" />
</http>
```