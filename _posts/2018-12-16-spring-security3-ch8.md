---
layout: post
title: "SpringSecurity3 Ch7"
description: "Spring Security 3"
date: 2018-12-08
tags: [spring,security]
comments: true
share: true
---

## OpenId 에 대한 개방
- OpenId
  - 신뢰성 있는 단일 프로바이더를 통해 사용자들이 자신의 신원을 관리할 수 있도록 해줌
  - 사용자들은 신뢰성 있는 OpenID 프로바이더를 통해 비밀번호, 개인정보 저장을 요청
  - 선택적으로 개인정보를 공개할 수 있음

### OpenID 라는 약속의 땅
- 웹에 있는 사용자가 자신의 개인데이터 및 정보를 신뢰된 프로바이더에게 집중 관리할 수 있게 해줌
- 사용자가 사용하려는 사이트에서 이러한 신뢰받은 프로바이더를 대리인으로 사용해 사용자에 대한 신뢰성을 담보
- 장점
  - OpenID 프로바이더에서 OpenID와 로그인 연동을 하려는 사이트와 호환되는 공개 OpenID 프로토콜만 구현하면 됨
  - OpenID 스펙도 공개되어, 다양한 프로바이더들이 제공하고 있음
  - 사용자는 많은 프로바이더 중에 선택할 수 있음

#### 관계도
- 사용자가 OpenID를 적용한 어플리케이션에 식별제 제공
  - OpenID를 Url로 구성 -> 응답
    - OpenID 프로바이더로 리졸브
    - 사용자를 OpenID 프로바이더로 리다이렉트
  - OpenID 프로바이더
    - 사용자가 OpenID 프로바이더 웹 사이트에 로그인
      - 실패시: 재시도
      - 성공시
        - OpenID를 사용한 어플리케이션에서 사용자를 로그인하게 함
- 특징
  - 사용자가 고유한 이름의 식별자(uri) 형태로 크리덴셜 제공
    - OpenID 프로바이더에 의해 사용자에게 제공됨
  - 맹복적인 신뢰는 안됨
    - 사용자가 그럴듯해 보이는 OpenID를 제시했더라도 추가적인 형태의 신원확인 과정이 필요
  - OpenID를 사용하는 어플리케이션은 사용자를 OpenID 프로바이더에게 리다이렉트 하고 사용자는 OpenID 프로바이더에게 제공
    - 이렇게 되면 OpenID 프로바이더가 접근결정을 내림
    - 접근결정 후, 프로바이더는 사용자를 원래 사이트로 리다이렉트 함

### 스프링 시큐리티에서 OpenID 인증 사용
- openid_identifier
  - OpenID spec에서 OpenID 로그인 필드 지정이 이 이름 사용 권장
  - VeriSign의 OpenID SeatBelt 같은 브라우저 플러그인은 스펙에 따라 페이지가 인식할 수 있는 OpenID 필드에 크리덴셜을 미리 채움
- remember me
  - OpenID 로그인을 사용시 remember me 옵션을 사용하지 않음
  - OpenID 프로바이더 사이트 사이의 리다이렉트시 remember me 체크박스 값이 사라지기 때문

### 스프링 시큐리티에서 OpenID 지원기능 
- security.xml에 http 엘리먼트에 openid 추가
- 흐름
  - 사용자가 OpenID 식별자 제공
    - (j_spring_openid_security_check)
    - o.s.s.openid.OpenIDAuthenticationFilter
    - o.s.s.openid.OpenIdCunsumer
      - 사용자를 리다이렉트
      - o.s.s.openid.OpenId4JavaConsumer
        - org.openid4java.consumer.ConsumerManager
        - OpenID 검색 수행 후 OpenID 프로바이더 검색
- 유효성 검증 흐름
  - 사용작가 OpenID 벤더 사이트 사이에서 로그인 마침
    - /j_spring_openid_security_check 로 리다이렉트
  - o.s.s.openid.OpenIDAuthenticationFilter
    - 응답 유효성 검증
      - o.s.s.openid.OpenIDConsumer
        - o.s.s.openid.OpenID4JavaConsumer 
        - 응답의 정확성 검증
          - org.openid4java.consumer.ConsumerManager
        - 완전히 채워지 OpenIDAuthenticationToken 반환
      - 인증 저장소와 비교 인증
        - 사용자가 ID 인증 저장소에 있는지 검증 후 , 사용자의 GrantedAuthority 획득
    - o.s.s.authenticaion.AuthenticationManager
      - o.s.s.openid.OpenIDAuthenticationProvider
        - Op-Local 식별자 검색
        - o.s.s.core.userdetails.UserDetailsService
          - 크리덴셜 저장소에 식별자가 있는지 확인
          - 부분적으로 채워진 OpenIDAuthenticaionToken 반환
    - 인증성공

#### OpenID를 이용한 사용자 등록 구현
- 등록과 로그인 요청이 동일하고, 이를 구분하기 위해 db에 저장되었는지 여부로 판단
- OpenIDAuthenticationFailureHandler 
  - UsernameNotFoundException 이 있는 경우 /registrationOpenId 로 이동



