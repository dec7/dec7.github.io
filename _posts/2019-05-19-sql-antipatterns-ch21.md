---
layout: post
title: "SQL Anti Patterns Ch21"
description: "SQL Anti patterns"
date: 2019-05-19
tags: [sql,anti,pattern]
comments: true
share: true
---

## SQL 인젝션 
### 목표: 동적 sql 쿼리 생성하기 
- 쿼리 문자열과 어플리케이션 변수를 엮어 만든 sql 

### 안티패턴: 검증되지 않은 입력을 코드로 실행하기 

```sql
SELECT * FROM Bugs WHERE bug_id = $bug_id;
$bug_id = 1234; DELETE FROM Bugs;

SELECT * FROM Bugs WHERE bug_id = 1234; DELETE FROM Bugs;
```
- 

```sql
UPDATE Accounts SET password_hash = SHA2('$password') WHERE account_id = $user_id;

user_id = 123 OR TRUE
UPDATE Accounts SET password_hash = SHA2('xyzzy') WHERE account_id = 123 OR TRUE;
```

#### 치료를 위한 탐구 
##### 값을 이스케이프하기 
```sql
SELECT * FROM Projects WHERE project_name = 'O''Hare'
SELECT * FROM Projects WHERE project_name = 'O\'Hare'
```

##### 쿼리 파라미터 
```sql
SELECT * FROM Projects WHERE project_name = ?;
```
- 컬럼이름, 수식, sql 키워드 전달 불가 

##### 저장 프로시져 
- 프로시져라도 동적 파라미터가 사용될 수 있음

##### 데이터 접근 프레임웍 
- 어떤 프레임워크도 안전한 sql을 보장할 수 없음 

### 안티패턴 인식방법
- 모든 db 어플리케이션은 sql문을 동적 생성하며, sql문 어느부분이든 문자연 연결, 값을 문자열ㅇ레 삽입하는 문장이 있는 경우 잠재적인 인젝션 공격대상이 될 수 있음 

### 안티패턴 사용이 합당한 경우 
- 없음 

### 해법: 아무도 믿지 마라
#### 입력값 필터링 
- 해당 입력에 대해 유효하지 않은 문자는 모두 제거해야함 
    - 정수 필요시, 정수만 넣어야 함 

#### 파라미터를 통한 값 전달 
- 쿼리 파라미터와 sql 문법을 분리해야함
- 어플리케이션 변수를 sql문의 리터럴 값으로 엮어야 할 경우 쿼리 파라미터 써야함 

#### 동적 값 인용하기 
- 쿼리 파라미터가 최상의 방법이나, 강혹 쿼리 파리미터 사용으로 쿼리 옵티마이져가 어떤 인덱스를 사용할지에 대해 잘못된 결정을 할 때가 있음 
- 가령 Accounts 테이블에 is_active 칼럼이 있고, 99%에 대해 true값이 있다고 가정
    - is_active=false 조건의 경우 쿼리 인덱스를 사용하는게 유리
    - is_active=true 조건의 경우 인덱스를 읽는 것은 낭비 
    - 만약 is_active=? 파라미터를 사용하는 경우, 옵티마이져가 어떤 값이 들어올지 모르므로 잘못된 최적화 계획을 세울 수 있음 
    - 이 경우엔, sql문에 해당 값을 직접 넣는게 나을 수도 있음 

#### 사용자 입력을 코드와 격리
#### 코드리뷰하기 



