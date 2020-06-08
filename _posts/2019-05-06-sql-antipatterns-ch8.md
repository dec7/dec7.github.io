---
layout: post
title: "SQL Anti Patterns Ch8"
description: "SQL Anti patterns"
date: 2019-05-06
tags: [sql,anti,pattern]
comments: true
share: true
---

## 다중 칼럼 속성 
### 목표: 다중 값 속성 저장
- 속성이 한 테이블에 포함되어야 할 것 처럼 보이ㅏ지만, 여러개의 값을 가질 때

### 안티패턴: 여러개의 칼럼 생성
```sql
CREATE TABLE Bugs (
    bug_id SERIAL PRIMARY KEY,
    description    VARCHAR(1000),
    tag1   VARCHAR(20),
    tag2   VARCHAR(20),
    tag3   VARCHAR(20)
);
```

```sql
UPDATE Bugs SET tag2 = 'performance' WHERE bug_id = 3456;

SELECT * FROM Bugs
WHERE tag1 = 'performance'
OR tag2 = 'performance'
OR tag3 = 'performance';
```

```sql
SELECT * FROM Bugs
WHERE (tag1 = 'performance' OR tag2 = 'performance' OR tag3 = 'performance')
AND (tag1 = 'printing' OR tag2 = 'printing' OR tag3 = 'printing')

SELECT * FROM Bugs
WHERE 'performance' IN (tag1, tag2, tag3)
AND 'printing' IN (tag1, tag2, tag3)
```

#### 업데이트 
```sql
UPDATE Bugs SET tag2 = 'performance' WHERE bug_id = 3456;

UPDATE Bugs
SET tag1 = NULLIF(tag1, 'performance'),
    tag2 = NULLIF(tag2, 'performance'),
    tag3 = NULLIF(tag3, 'performance')
WHERE bug_id = 3456;

UPDATE Bugs
SET tag1 = CASE WHEN 'performance' IN (tag2, tag3) THEN tag1 ELSE COALESCE(tag1, 'performance') END,
    tag2 = CASE WHEN 'performance' IN (tag1, tag3) THEN tag2 ELSE COALESCE(tag2, 'performance') END,
    tag3 = CASE WHEN 'performance' IN (tag1, tag2) THEN tag3 ELSE COALESCE(tag3, 'performance') END
WHERE bug_id = 3456;
```

#### 유일성 보장 
- 여러 칼럼에 동일한 값을 나타나지 못하게 막을 수 없음
```sql
INSERT INTO Bugs (description, tag1, tag2, tag3)
VALUES ('printing is slow', 'printing', 'performance', 'performance');
```

#### 값의 수 증가 처리
- 하나의 칼럼에 하나의 값을 유지하기 위해 칼럼이 계속 추가될 수 있음
```sql
ALTER TABLE Bugs ADD COLUMN tag4 VARCHAR(20);
```
- 단점 
    - 이미 데이터를 포함하는 db 구조를 바꾸기 위해 테이블 전체를 잠금 설정하고 다른 클라접근을 차단하는 과정이 필요
    - db 구현체에 따라 새로운 데이블 생성 후 마이그레이션, 데이터가 많을 경우 시간이 오래 걸림
    - 어플리케이션 수정 필요

### 안티패턴 인식 방법 
- 여러개의 값을 할당할 수 있으나, 최대 개수가 제한되어 있는 속성이 있을 때 
    - 의도적으로 개수를 제한할 수도 있지만, 구조적한 한계랑 차이가 있음

### 안티패턴 사용이 합당한 경우 
- 각 칼럼에 들어가는 값은 같은 종류지만 그 의미와 사용처가 논리적으로 다를 때

### 해법: 종속 테이블 생성
- 다중 값 속성을 위한 칼럼을 하나 가지는 종속 테이블을 생성
- 여러개의 값을 여러 개의 칼럼대신 행에 저장

```sql
CREATE TABLE Tags (
    bug_id  BIGINT  UNSIGNED NOT NULL,
    tag     VARCHAR(20),
    PRIMARY KEY (bug_id, tag),
    FOREIGN KEY (bug_id) REFERENCES Bugs (bug_id)
);

INSERT INTO Tags (bug_id, tag)
VALUES (1234, 'crash'), (3456, 'printing'), (3456, 'performance');

```

- 조회 
```sql
SELECT * FROM Bugs JOIN Tags USING (bug_id)
WHERE tag = 'performance';

SELECT * FROM Bugs
    JOIN Tags AS t1 USING (bug_id)
    JOIN Tags AS t2 USING (bug_id)
WHERE t1.tag = 'printing' AND t2.tag = 'performance';

```

