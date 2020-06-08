---
layout: post
title: "SQL Anti Patterns Ch14"
description: "SQL Anti patterns"
date: 2019-05-12
tags: [sql,anti,pattern]
comments: true
share: true
---

## 모르는 것에 대한 두려움 
```sql
SELECT first_name || ' ' || last_name as full_name FROM Accounts;
```
- 중간이름의 첫 글자를 테이블에 저장하도록 하려면 
```sql
ALTER TABLE Accounts ADD COLUMN middle_initial CHAR(2);

UPDATE Accounts SET middle_initial = 'J.' WHERE account_id = 123;
UPDATE Accounts SET middle_initial = 'C.' WHERE account_id = 321;

SELECT first_name || ' ' || middle_name || ' ' || last_name AS full_name FROM Accounts;
```

### 목표: 누락된 값 구분 하기 
- SQL은 특수한 값인 NULL을 지원 
- NULL을 생산적으로 쓸수 있는 다양한 방법 
    - 행을 생성할 수 없을때나, 저장할 값이 없을 때 
    - 평가한 값이 유효하지 않을 때 
    - 외부 조인에서 매치되지 않은 행의 칼럼을 채우기 위해 

### 안티패턴: NULL을 일반 값처럼 사용
#### 수식에서 null사용 
- null 칼럼에서 수식계산을 적용하면 null이 반환됨 
- 표준 SQL에서 어떤 문자열도 null과 연결하면 null이 됨 
- NULL은 false와 다름, NULL이 들어간 boolean 수식은 AND, OR, NOT을 사용해도 null 

#### NULL을 가질 수 있는 칼럼 검색 
```sql
SELECT * FROM Bugs WHERE NOT (assigned_to = 123);
```
- NULL인 행은 반환하지 않음 

```sql
SELECT * FROM Bugs WHERE assigned_to = NULL;
SELECT * FROM Bugs WHERE assigned_to <> NULL;
```
- where 절 조건은 수식이 true인 경우만 만족되고, NULL과의 비표는 절대 true가 되지 않음

#### 쿼리 파라미터로 null 사용
```sql
SELECT * FROM Bugs WHERE assigned_to = ?;
```
- NULL을 파라미터로 사용할 수 없음


#### 문제 회피하기 
- null 은 누락된 값을 나타내기 위한 것, 사용하지 않는다고 하더라도 다른 대체 값이 필요 

```sql
CREATE TABLE Bugs (
    bug_id      SERIAL PRIMARY KEY,
    ...
    assigned_to BIGINT UNSIGNED NOT NULL,
    hours       NUMERIC(9,2) NOT NULL,
    FOREIGN KEY(assigned_to) REFERENCES Accounts (account_id)
);

INSERT INTO Bugs (assigned_to, hours) VALUES (-1, -1);

SELECT AVG(hours) AS average_hours_per_bug FROM Bugs
WHERE hours <> -1;
```
- NULL 이 아닌 값으로 추가할 경우 다른 수식함수에 포함되며, 제외하기 위한 추가 코드가 필요
- FK로 선언된 경우 -1 에 대한 참조값이 있어야함 

### 안티패턴 인식 방법 
- 아무것도 설정되어 있지 않은 값을 찾는 방법 
- 일부 컬럼이 표시 되지 않는 경우 

### 안티패턴 사용이 합당한 경우 
- null을 사용하는 것은 문제 없고, null을 일반적인 값처럼 사용하는 것이 안티패턴 
- null을 일반적인 값처럼 취급하는 경우는 외부데이터를 가져오거나 내보낼때 

### 해법: 유일한 값으로 NULL을 사용 
- NULL값과 관련된 대부분의 문제는 SQL의 세가지 값의 동작을 제대로 이해하지 못한 배경 

#### 스칼라 수식에서 NULL
- NULL = 0, NULL
- NULL = 1234, NULL
- NULL <> 1234, NULL
- NULL + 1234, NULL
- NULL || 'string', NULL
- NULL = NULL, NULL
- NULL <> NULL, NULL

#### 불리언 수식에서 NULL
- NULL AND TRUE, NULL,
- NULL AND FALSE, NULL,
- NULL OR FALSE, NULL,
- NULL OR TRUE, NULL,
- NOT (NULL), NULL

#### NULL 검색하기 
```sql
SELECT * FROM Bugs WHERE assigned_to IS NULL;
SELECT * FROM Bugs WHERE assigned_to IS NOT NULL;
```

- SQL99, is distinct from
    - oracle 미지원 
    - mysql <=>
```sql
SELECT * FROM Bugs WHERE assigned_to IS NULL OR assigned_to <> 1;
SELECT * FROM Bugs WHERE assigned_to IS DISTINCT FROM 1;
```

#### 칼럼을 NOT NULL로 선언 
- NULL 값이 어플리메이션 정책을 위반하거나 의미가 없는 경우엔 NOT NULL 제약조건을 선언하는게 권장 
    - 어플리케이션 코드보다 데이터베이스가 제약조건을 균일하게 강제하도록 하는게 더 좋은 방법 

#### 동적 디폴트 
```sql
SELECT first_name || COALESCE(' ' || middle_initial || ' ', ' ') || last_name as full_name
FROM Accounts;
```
- COALESCE, NVL, ISNULL




