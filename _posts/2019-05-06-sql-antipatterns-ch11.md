---
layout: post
title: "SQL Anti Patterns Ch11"
description: "SQL Anti patterns"
date: 2019-05-06
tags: [sql,anti,pattern]
comments: true
share: true
---

## 31가지 맛 
- 특정 목록을 데이터타입이나 제약조건을 이용해 칼럼 정의에 지정할 수 있음 
```sql
CREATE TABLE PersonalContacts (
    ...
    salutation VARCHAR(4)
        CHECK (salutation IN ('Mr.','Mrs.','Ms.','Dr.','Rev.'))
);
```
- 만약 타 국가의 호칭도 포함되어야 하는 상황 발생

### 목표: 칼럼을 특정 값으로 제한하기 

### 안티패턴: 컬럼 정의에 값 지정 
- 칼럼 정의는 메타데이터 (테이블 구조정의) 의 일부 
```sql
CREATE TABLE Bugs (
    ...
    status  VARCHAR(20) CHECK (status IN ('NEW','IN PROGRESS','FIXED'))
);
```
- CHECK 제약조건을 정의할 수 있지만, insert, update를 정의할 수 없음 
```sql
-- MySQL은 칼럼을 특정 값의 집합으로 제한하는 ENUM 이라는 비표준 데이터 타입을 지원 
CREATE TABLE Bugs (
    ...
    status ENUM('NEW','IN PROGRESS','FIXED')
);
```
- 값은 문자열로 선언되나, 내부적으로는 열거된 목록에서 문자열이 서수로 저장되도록 구현됨, 
- 공간은 덜 차지하지만, 정렬시 서수 저장 순서로 정렬됨 

#### 모든 상태를 조회하기 
```sql
SELECT column_type
FROM information_scheme.columns
WHERE table_schema = 'bugtracer_schema'
    AND table_name = 'bugs'
    AND column_name = 'status';
``` 
- 위 방식으로 체크 제약조건, 도메인, 사용자 정의 타입 확인시 쿼리는 점점더 복잡해짐

#### 새로운 타입 추가 
- ENUM 이나 체크 제약조건에 값을 추가하거나 삭제하는 문법은 없고, 단지 새로운 값으로 재정의 
```sql
ALTER TABLE Bugs MODIFY COLUMN status
    ENUM ('NEW','IN PROGRESS', 'FIXED','DUPLICATE');
```
- 먼저 (new, in progress, fixed) 사 허용되었다는 것을 알아야 하지만, 현재 허용되는 값 집합을 조회하기 어려움 
- alter table은 비용이 많이 들어가는 작업 
- 정책적으로 메타데이터를 변경하는 것은 (테이블, 칼럼) 적거나 주의를 요하는 일 

#### 이전 값은 없어지지 않음 
- FIXED 상태가 삭제될 수 있음
```sql
ALTER TABLE Bugs MODIFY COLUMN status
    ENUM('NEW','IN PROGRESS','CODE COMPLETE','VERIFIED');
```
#### 포팅이 어려움 
- sql 제품마다 균일하게 지원되는게 아님

### 안티패턴 인식방법 
- db를 종료해야 메뉴를 추가할 수 있고, 시간이 오래걸림 
- status 칼럼은 정의된 목록의 값만 가질 수 있음 
- DB 상태 목록와 어플리케이션에서 관리되는 상태 목록이 달라짐 

### 안티패턴 사용이 합당한 경우 
- 값이 변하지 않는 경우 
- LEFT/RIGHT, ACTIVE/INACTIVE, ON/OFF, INTERNAL/EXTERNAL 같은 상호 배타적인 값
- check 제약조건은 시작시간, 종료시간 검증 등으로 사용할 수 있음 

### 해법: 테이버로 값 지정 
- Bugs.status 칼럼에 들어갈 수 있는 값을 행으로 하는 색인 테이블을 만들고 FK로 제약 조건 
```sql
CREATE TABLE BugStatus (
    status  VARCAHR(20) PRIMARY KEY
);

INSERT INTO BugStatus (status)
VALUES ('NEW'), ('IN PROGRESS'), ('FIXED');

CREATE TABLE Bugs (
    ...
    status  VARCHAR(20),
    FOREIGN KEY (status) REFERENCES BugStatus (status)
        ON UPDATE CASCADE
)
```

#### 갑의 집합 쿼리 
```sql
SELECT status FROM BugStatus ORDER BY status;
```

#### 색인 테이블 값 갱신하기 
```sql
INSERT INTO BugStatus (status) VALUES ('DUPLICATE');

-- ON UPDATE CASCADE 선언시 
UPDATE BugStatus SET status = 'INVALID' WHERE status = 'BOGUS';
```

#### 더이상 사용하지 않는 값 지원 
- Bugs에 있는 행이 참조되는 한 색인 테이블에서 행을 삭제할 수 없음 
- status 칼럼의 FK가 참조정합성을 강제하므로 색인 테이블에 값이 존재해야함
```sql
ALTER TABLE BugStatus ADD COLUMN active
    ENUM('INACTIVE','ACTIVE') NOT NULL DEFAULT 'ACTIVE';

UPDATE BugStatus SET active = 'INACTIVE' WHERE status = 'DUPLICATE';
```