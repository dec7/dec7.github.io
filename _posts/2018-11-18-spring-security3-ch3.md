---
layout: post
title: "SpringSecurity3 Ch3"
description: "Spring Security 3"
date: 2018-11-18
tags: [spring,security]
comments: true
share: true
---

## 로그인 페이지 커스터마이징
### 커스텀 로그인 페이지
#### 흐름
- 사용자가 페이지를 요청
  - 접근결정 관리자가 요청을 가로챔
    - 사용자가 적절한 권한이 있는경우: 홈페이지로 이동
    - 사용자가 로그인하지 않았고, 리소스 접근에 인증이 필요한 경우: 커스텀 로그인 페이지

#### 로그인 jsp 페이지 추가
- 구현
  - [link](https://github.com/dec7/study/commit/70c4e7fffe8f76384367fbbc7aea846fa01a0148)
- 폼액션은 UsernamePasswordAuthenticationFilter 서블릿 필터에서 설정한 액션과 일치해야함
  - j_spring_security_check 가 기본설정
- 사용자명과 비밀번호를 타나내는 폼필드는 서블릿 명세와 동일해야함
  - j_username, j_password
- SpringSecurity tagLibarry
  - heder reference
    - ```xml <$@ tablib prefix="sec" uri="http://www.springframework.org/security/tags" %> ```
##### 모든 요청에 대해서 ROLE_USER 권한을 요청할 경우
  - 모든 페이지에 대해 권한을 요청하므로, login 페이지에도 권한을 요구하게 되어 무한 리다이렉트 됨
  - SpringSecurity 의 모든 url 요청은 top-bottom 방식으로 규식을 해석하게 됨

#### 로그아웃 기능
- 구현
  - [link](https://github.com/dec7/study/commit/781c2161cfb42263a93c5e63ce5c08cefb57ba86)
- 사용자의 안전한 세션을 무효화하는 사용자의 행동을 나타내는 용어
  - 로그아웃 후, 사용자는 보호되지 않은 영역으로 리다이렉트 됨
##### 동작원리
- 모든 url 요청은 서블릿 요청으로 처리되기 전에 SpringSecurity 필터체인을 통과하게 됨
- ```/j_spring_security_logout``` 에 대한 요청은 LogoutFilter가 가로챔

```xml
<http auto-config="ture" use-expressions="true">
  <logout invalidate-session="true" logout-success-url="/" logout-url="/j_spring_security_logout" />
</http>
```

##### 로그아웃 과정
1. HTTP 세션 무효과 (invalidate-session이 true인 경우)
2. SecurityContext 초기화 (사용자를 실제로 로그아웃 시키는 부분)
3. logout-success-url 에 지정된 url로 사용자를 리다이렉트

- 사용자가 로그아웃을 수행
  - LogoutFilter가 요청을 가로챔
    - 순환하여 모든 LogoutHandler를 검사
      - 세션 초기화
      - rememberMe 쿠키 초기화
      - SecurityContext 초기화
    - 로그아웃 정의 url로 리다이렉트
      - 없는 경우 /

- 주의사항
  - 모든 로그아웃 핸들러가 정상 처리되어야 하므로, 로그아웃 핸들러에서 예외를 던지면 안됨
- 로그아웃 링크 변경
  - [link](https://github.com/dec7/study/commit/0b7022cfd3a8a4149522fd5e32ea0315392cbdb2)


## remember me
### 기능
- 간단한 쿠키를 암호화하고 사용자 브라우저에 저장해 재방문 하는 사용자를 기억하는 기능
- 스프링 시큐리티에서 사용자가 remember me 쿠키를 갖고 있는 것을 확인하면 사용자명, 비밀번호를 입력하지 않고도 자동 로그인 가능

### 옵션 구현
- 구현
  - [link](https://github.com/dec7/study/commit/2d1d906a6714efdb03c8457dcb3e84001c80a6d5)

### 동작 원리
- 아래 정보를 Base64 인코딩된 문자열로 저장
  - 사용자명
  - 쿠키 만료일자/시간
  - 쿠키 만료일자/시간, 사용자명, 비밀번호에 대한 MD5 해시
  - remember me key 에 정의된 값
- md5
  - 암호화 해시 알고리즘
  - 임의의 길이를 갖는 입력 데이터를 다이제스트라 하는 압축된 고유 텍스트로 변환
  - 다이제스트는 원본 입력 데이터가 없더라도, 해시를 생성하는데 사용한 입력값과 정확히 일치하는 알려지지 않은 입력값을 확인하는데 사용됨
- o.s.s.web.authentication.rememberme.RememberMeAuthenticationFilter 
  - 쿠키 내용을 검토하고 올바른 쿠키일 때만 사용자 인증에 remember me를 사용함

#### 흐름
- 사용자가 요청함
  - RememberMeAuthenticationFilter 가 요청을 가로챔
    - RememberMeServices 를 구현한 TokenBasedRememberMeServices 가 쿠키 추출
      - 쿠키가 없는 경우: 다음 필터 체인 이동
      - 쿠키가 있는 경우: 
        - 제대로 디코인 안된경우: InvalidCookieException 예외 던짐
        - 기대한 MD5 시그니처 계산
          - 매칭되지 않는 경우: InvalidCookieException 예외 던짐
          - 매칭된 경우
            - 사용자 계정이 올바르지 않은경우 예외
            - 올바른 경우 인증토큰 생성 -> 다음 필터 이동

### 사용자 라이프 사이클
- RememberMeServices
  - 인증된 사용자의 세션 라이프 사이클 진행되는 동안 몇몇 위치에서 호출됨
- 호출 시점
  - 로그인 성공
    - 구현체 클래스가 remember me 쿠키 설정
  - 로그인 실패
    - 쿠키 존재시 쿠키 무효화
  - 사용자 로그아웃
    - 쿠키 존재시 쿠키 무효화


### 설정 디렉티브
- 속성
  - key
    - remember me 쿠키에 대한 고유키 설정
    - 키 값은 보안에서 핵심적인 역할, 추측할 수 없을 정도로 긴 문자열 사용하는게 좋음
      - 고유 이름 포함, 최초 36자 이상 임의 문자열 설정하는게 좋음
  - token-validity-seconds
    - 시간범위를 초단위 정의
    - -1로 설정시 사용자가 브라우저 닫을 때 함께 사라지는 세션쿠키에 설정됨
      - 이때 브라우저가 닫히지 않으면 토큰은 설정이 불가능한 2주 기간동안 유효

### remember me는 안전한가
#### 보안위협
- 사용자가 remember me 쿠키를 사용해 페이지 요청
  - 네트워크 스니퍼가 사용자 요청을 기록
  - 악의적인 사용자가 처음 요청을 그대로 인증 요청

#### remember me 사용자와 완전히 인증된 세션을 구별하는 권한부여 규칙
- 특정 영역에 접근시 remember me 세션을 사용한 사용자의 경우, 재인증하도록 강제하는 방법
```xml
<intercept-url pattern="/login" access="permitAll" />
<intercept-url pattern="/accout/*" access="hasRole('ROLE_USER') and fullyAuthenticated" />
<intercept-url pattern="/*" access="hasRole('ROLE_USER')" />
```
- 표현식을 사용하지 않는 경우
```xml
(access="IS_AUTHENTICATED_FULLY")
```

#### ip 주소를 인식하는 remember me 서비스 개발
- [link](https://github.com/dec7/study/commit/b3c156db54ea1fa86d136a5573fe94d40816574f)

## 비밀번호 수정 기능
- [link](https://github.com/dec7/study/commit/f95efc13e9c84194a8b3e67ed4016421efedffff)