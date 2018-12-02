---
layout: post
title: "SpringSecurity3 Ch4"
description: "Spring Security 3"
date: 2018-12-01
tags: [spring,security]
comments: true
share: true
---

## 크리덴셜 안전하게 저장
### db를 사용한 인증적용
#### db 적용
- [link](https://github.com/dec7/study/commit/53135354308d714469234550a59cdb26a53229f2)

#### 동작원리
- o.s.s.authentication.UsernamePasswordAuthenticationToken
  - o.s.s.authentication.AuthenticationManager
    - o.s.s.authentication.dao.DaoAuthenticationProvider
      - InMemoryDaoImpl에서 UserDetails 조회
        - jdbc의 경우 JdbcDaoImpl에서 조회
      - Authentication의 크리덴션과 UserDetails의 크리덴션 검증

#### 기본으로 제공되는 jdbc기반 사용자 관리

```java
void createUser(UserDetails user)
void updateUser(final UserDetails user)
void deleteUser(String username)
boolean userExists(String username)
void changePassword(String oldPassword, String newPassword)
```

- [link](https://github.com/dec7/study/commit/5f76d99d3d3ce634712587048d3104ffaf247397)

#### JdbcDaoImpl 고급설정
##### 그룹기반 접근제어
- GrantedAuthority를 그룹 이라는 논리적인 집합으로 분류
  - 사용자와 GrantedAuthority 선언 사이에 간접 레벨 제공
  - 이 기능으로 사용자는 하나 이상의 그룹에 속하고, 그룹의 멤버쉽에 GrantedAuthority 선언 참조
- [link](https://github.com/dec7/study/commit/046a3d26140e7b0471fc85870ba46b7648b8f98e)

##### db인증시 레거시 또는 커스텀 스키마 사용
- JdbcDaoImp 에서는 레거시 스키마 매핑 가능
- sql
  - usersByUsernameQuery
    - 사용자명과 일치하는 하나 이상의 사용자 반환, 첫번째 사용자 사용됨
    - 컬럼: 사용자명, 비밀번호, 활성화 여부
  - authoritiesByUsernameQuery
    - 사용자에게 직접 부여된 하나 이상의 승인된 권한 반환
    - gbac가 적용되지 않은 경우 사용됨
    - 컬럼: 사용자명, 허용권한 
  - groupAuthoritiesByUsernameQuery
    - 그룹 멤사십을 통해 사용자에게 승인된 권한과 그룹의 상세 정보 반환
    - gbac가 적용된 경우 사용
    - 컬럼: 그룹 키, 그룹명, 허용권한
- [link](https://github.com/dec7/study/commit/86ef608b2575bc350ccc43040c890922fd6fd678)

### 보안 비밀번호 설정
- 비밀번호 보안은 인증된 주체에 대한 신뢰 / 권위를 인정해 주는 가장 핵심적인 영역
  - 악의적인 사용자가 비밀번호를 악용하는 것이 사실상 불가능할 정도로 안정한 방식을 비밀번호를 저장해야함
  - 규칙
    - 단순테스트로 저장하면 안됨
    - 사용자가 제공한 비밀번호와 저장된 비밀번호를 반드시 비교
    - 비멀번호는 요청시 사용자가 요청시 바로 제공되어서는 안됨

- 방식  
  - 사용자가 비밀번호를 사용해 로그인
    - AuthenticationManager 는 비밀번호 인코더로 인코딩
    - 결과를 저장

- PasswordEncoder
  - 스프링시큐리티에서 제공하는 구현체
  - PlainTextPasswordEncoder
  - Md4PasswordEncoder
  - Md5PasswordEncoder
  - ShaPasswordEncoder
  - LdapShaPasswordEncoder
    - ldap 인증 저장소와 연동된 LDAP SHA와 LDAP SSHA 알고리즘 구현체

#### 비밀번호 인코딩 설정
1. sql 스크립트 실행 -> db에 저장할 비밀번호 암호화
2. DaoAuthenticationProvider가 PasswordEncoder와 연동되도록 설정

- [link](https://github.com/dec7/study/commit/3a4778653f5d7617b3cbdffe186817ec108a9500)

#### salt 적용
- 암호화 해쉬를 적용하더라도 동일한 값에 대해서는 동일한 해시를 반환함
  - 레인보우 테이블을 사용하여 유추가능
- salt 
  - 평문이 암호화 적용되기 전에 일반 텍스트가 첨부되는 또 다른 일반 텍스트 영역
    - 사용 권장
      - 사용자 관련 데이터 일부에서 텍스트를 생성하는 알고리즘 사용 (사용자가 생성된 시점의 타임스템프)
      - 랜덤 텍스트 생성, 사용자의 비밀번호 레코드와 함께 저장

##### 방법
- 사용자가 비밀번호 제시
  - AuthenticationManger가 솥트 소스를 조회
  - 솔트된 비밀번호 계산
  - 암호화한 후 db와 비교

##### Slat interface
- o.s.s.authentication.dao.saltSource
  - SystemWideSaltSource
    - 모든 비밀번호에 적용되는 단일 static솔트
  - ReflectionSaltSource
    - 사용자의 비밀번호에 대한 솔트값을 가져오기 위해 UserDetails 객체의 bean속성을 사용

##### 솔트 비밀번호 설정
- [link](https://github.com/dec7/study/commit/67ac7bf85b7c3a33a942a3c3383d40da2ae5c61d)
- 현재 SaltSource는 UserDetails에 의존하여 솔트값을 생성
  - db에 salt를 저장하는 공간을 만들지 않았기 때문에 임시로 설정


#### 커스텀 솔트 적용
- 솔트가 변경 가능하기 때문에 악의 적인 사용자는 자신의 이름을 변경하며, 암호화된 비밀번호 구성을 찾으려 할 수 있음
- [link](https://github.com/dec7/study/commit/0813c6b54f1289bd724ba9f4413f50b78606c484)

### RememberMe 기능 DB로 이전
- 현재 remember me 기능은 사용자의 세션이 만료되는 시점까지만 동작
  - 재구동시 만료됨
- remember me 스키마 생성 필요
- [link](https://github.com/dec7/study/commit/5206352ad207ced081f4377ad9e2807d948d6285)
- 안전 ?
  - TokenBasedRememberMeServices 
    - 쿠키를 안전하게 인코딩하기 위해 사용자의 정보를 MD5 해싱하여 적용
  - PersistentTokenBasedRememberMeServices
    - 영속 토큰에 대한 처리를 구현하고 검증 메소드로 토큰 보안을 처리
    - 이때 tampering 에 대한 처리 방식이 조금 달라짐
      - 사용자가 사이트에 상호작용, 인증받을 때 series 컬럼내 고유 토큰을 사용하는 방식으로 사용자마다 고유한 식별자를 생성
      - 이 시리즈와 토큰 조합은 쿠키에 저장되고 영속 토큰 저장소와 대조를 통해 사용자를 인증
      - 시리즈, 토큰 모두 랜덤 생성
    - 물론 완벽하게 안전하지 않기 때문에 (mitm)
      - ip주소를 확인하거나, 민감한 영역은 사용자의 재인증을 통해 병행하는게 좋음

### ssl 적용
- [link](https://github.com/dec7/study/commit/c46daee4356bfeb25ea760d14982c1f3461f3ee3)
- key 발급
```bash
$ keytool -genkeypair -alias tlsserver -keyalg RAS -validity 365 -keystore tomcat.keystore -storetyle JKS
```

- tomcat conf 설정
```xml
<Connector 
  port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
  maxThreads="150" SSLEnabled="true" secure="true" sslProtocol="TLS" 
  keystoreFile="conf/tomcat.keystore"
keystorePass="password"
</Connector>
```
- https 요구시 보호되지 않은 url에서 https로 리다이렉트 처리
- 세션쿠키 생성시 secure인디케이터 활성화시 쿠키를 안전하게 전송할 수 있음

#### 흐름
- 사용자 요청
  - o.s.s.web.access.channel.ChannelProcessingFilter
    - FilterInvocationSecureMetadataSource 에서 적용가능한 속성을 가저옴
  - o.s.s.web.access.channel.ChannelDecisionManager
    - ChannelDecisionManagerImpl 구현
    - 작업이 필요한지 판단하기 위해 ChannelProcessor 리스트를 가져옴
      - SecureChannelProcessor 구현
      - 요청이 안전하지 않은 경우 요청을 전달
        - ChannelEntryPoint
          - RetryWithHttpEntryPoint 에 의해 리다이트 전달

- ChannelEntryPoint
  - HTTP 302 로 리다이렉트 전달
  - POST URL 은 당연히 불가능하며
    - (일반적으로 POST방식에서 보안프로토콜과 비보안 프로토콜을 서로 이동하지 않음, 브라우저는 미리 경고)
