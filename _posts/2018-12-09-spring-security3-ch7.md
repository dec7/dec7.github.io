---
layout: post
title: "SpringSecurity3 Ch7"
description: "Spring Security 3"
date: 2018-12-08
tags: [spring,security]
comments: true
share: true
---

## 접근 제어 목록
- 비지니스 티어 및 비지니스 객체에 대한 보안
    - 접근제어 목록 / ACL 기술을 활용해 구현됨
    - 그룹, 비지니스 객체, 논리적인 작업을 조합한 대상에 대해 그룹 퍼미션을 지정할 수 있는 모듈
- 예
    - 사용자명: Amy : 객체-프로필123 : 퍼미션-읽기쓰기
    - 그룹명: ROLE_USER : 객체-프로필123 : 퍼미션-읽기
    - 그룹명: ANONYMOUS : 객체-모든프로필 : 퍼미션-없음

### 스프링 시큐리티에서 접근제어 목록
- OS 처럼, 스프링 시큐리티의 ACL 컴포넌트 이용시 비지니스 객체, 그룹, 주체에 관한 논리적인 트리를 만들 수 있음
- 접근 혀용여부는 요청자와 요청된 대상에 대한 퍼미션이 중첩되는 영역이 사용됨
- 스프링 ACL 시스템의 주된 역할은 보안 식별자 (SID) 가 담당
    - SID는 개별주체, 그룹의 신원을 추상화하는데 사용하는 논리적인 구성체
    - 나머지 영역은 보호된 객체 자체에 대한 정의가 담당
- ACL 구현체는 ACL 규칙을 개별 객체 인스턴스 레벨에서 정의하도록 규정하며, 이 말은 필요시 모든 객체가 개별적인 규칙을 가질 수 있다는 것
- 개별 접근 규칙
    - 접근제어 엔트리 (access control entry, ACE)
    - 다음 요소의 조함
        - 규칙이 적용되는 대상에 대한 SID
        - 규칙이 적용되는 객체 식별자
        - 주어진 SID와 지정된 객체 식별자에 적용될 퍼미션
        - 주어진 SID와 객체 식별자에 대해 지정된 퍼미션 허용 여부
- 사용 객체
    - SID
        - o.s.s.acls.model.Sid
    - 객체 식별자
        - o.s.s.acls.model.ObjectIdentity
    - ACL
        - o.s.s.acls.model.Acl
    - ACE
        - o.s.s.acls.model.AccessControlEntry

### 스프링 시큐리티 ACL지원 기본설정
#### 시나리오 가정
- ROLE_ADMIN 제외한 모든 사용자가 첫번째 카테고리에 대한 읽기 접근 거부

#### 접근 결정 관리자 설정
- acl 체크에 사용되는 접근 결정 관리자는 웹 url권한 부여 규칙에 사용되는 관리자와 달라야 함

#### ACL 지원 빈 설정
- AclEntryVoter
    - db에 있는 ACL 저장소를 참고하여 런타임시 권한부터 및 접근결정을 내리는 AccessDecisionVoter 구현체
    - VOTE_CATEGORY_READ
        - 보터가 보트할 수 있는 보안 속성값의 이름을 나타냄
        - 이 값은 ACL을 사용해 보호할 @Secured 어노테이션과 일치
    - 요청자에 대한 보트가 성공하기 위해 필요한 ACL속성을 선언하는데 사용됨
        - 선언된 VOTE_CATEGORY_READ 접근을 필요로 하는 메소드에 대하 권한 부여를 받기 위해 요청자가 Category 도메인 객체에 READ 접근을 허가 받았음을 증명하는 ACL엔트리를 가지고 있어야 한다고 규정 
    - ACL 보터 선언은 코드 리소스에 선언한 권한부여 조건과 ACL SID 자신에 대한 도메인 객체와 관련한 퍼미션 부여 사이의 추상 레이어
- JdbcAclService
    - 기본으로 제공되는 클래스, 예제 스키마를 바로 사용할 수 있도록 함
    - 재귀 SQL과 후처리를 통해 SID 계층 구조를 이해하고, 이 구조가 AclEntryVoter로 전달되도록 보장
    - LookupStrategy 구현체에 작업을 위임
- BasicLookupStrategy
    - ObjectIdentity 목록을 데이터베이스로부터 가져온 실제 적용 가능한 ACE목록으로 해석 -> 보호를 목적
    - ObjectIdentity 선언이 재귀적일 수 있다는 것이 문제가 될수 있음
    - AclCache
        - 불필요한 쿼리를 피함
    - AuditLogger
        - ACL, ACE 룩업 기록시 사용
    - AclAuthorizationStrategy
        - db에서 ACL을 로드할땐 일 안함
        - 변화 유형을 기준으로 ACL또는 ACE에 대한 런타임시 변화가 허용될지 여부를 판단
        - GrantedAuthorityImpl
            - 수정가능 ACL에 대해 런타임시 특정 작업을 허용하도록 세개의 지정자를 제공
- [link](https://github.com/dec7/study/commit/aba2007db4753a8c8063fb776e21d9a587751ff2)

### 고급 ACL 주제
- ACE퍼미션, ACL에 대한 특정 타입의 변화가 허용되는지 여부를 결정시, ACL환경울 지원해주는 GrantedAuthority 의 사용과 관련된 내용

#### 퍼미션 동작 원리
- AclEntryVoter
    - 메서드 자체에 대해 선언된 퍼미션(@Secured)을 실제 ACL과 연결 
- 흐름
    - o.s.s.access.AccessDecisionManager
        - 도메일 객체 접근 요청에 대한 보트를 요청
    - o.s.s.acls.AclEntryVoter
        - 런타임시에 받은 정보를 ACL 데이터 객체로 변환
            - o.s.s.acls.model.ObjectIdentityRetrievalStrategy
                - 체크되는 도메인 객체를 바탕으로 ObjectIdentity 가져옴
            - o.s.s.acls.model.SidRetrievalStrategy
                - 관련 Authentication 바탕으로 적용 가능한 Sid 가져옴
        - 런터임 요청 정보를 바탕으로 ACL 검색
            - o.s.s.acls.model.AclService
            - o.s.s.acls.model.JdbcAclService
                - 재귀 sql 위임
                    - o.s.s.acls.jdbc.LokupStrategy
                - 적용 가능한 AccessControlEntry 로 채워진 Acl 객체 반환
                    - 런타임 Sid목록과 Acl비교해 권한부여 판단
- ObjectIdentity
    - Type과 Identifier 라는 두개 속성 있고
    - 런타임시 체크되는 객체로부터 가져오고 ACE엔트리를 선언하는데 사용됨
- AclEntryObejct
    - 모든 퍼미션 조합을 갖추도록 설정하거나
    - ACE가 여러 값들을 저장할 의도록 만든 퍼미션을 무시하고 ACE마다 단일 퍼미션만 저장하는 방식 중 택해야함
- [link](https://github.com/dec7/study/commit/52cd4996a9f6ac40df3c60b9dfbe500d9df5f761)


#### 스프링 시큐리티 JSP 태그 라이브러리 활용한 ACL 활성화
- ACL 시스템에도 적용 가능
  - accesscontrollist 태그 활용
  
#### ACL을 지원하는 SpEL
- 메소드 보안에서 SpEL을 사용하려면 빈설정을 명시적으로 사용해 모든 메소드 보안 설정 필요

#### 수정가능 ACL과 권한부여
- o.s.s.acls.model.MutableAcl 개념으로 수정지원
- 런타임시에 ACL 필드를 수정할 수 있도록 지원
- JDBC 데이터 저장소에 저장하는 방법 지원
  - o.s.s.acls.jdbc.JdbcMutableAclService
- [link](https://github.com/dec7/study/commit/6c74ee16d7b3b2c36bcc86516cc7f6edca656bf9)

#### Ehcache ACL 캐싱
- Ehache ACL 캐싱 설정

##### 스프링 ACL이 EhCache를 사용하는 방식
- 스프링 ACL은 아래 객체를 모두 ACL 개킹 전략의 일환으로 키, 값으로 저장
  - ObjectIdentity
  - Sid
  - AccessControlEntry
  - 객체 인스턴스의 Serializable 주키

### 일반적인 ACL 적용에 대한 고려
- [link](https://github.com/dec7/study/commit/361d56aa4ac0bf83ff5971bd43ed46c3daf0b21b)