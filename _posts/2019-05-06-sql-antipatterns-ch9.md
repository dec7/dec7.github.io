---
layout: post
title: "SQL Anti Patterns Ch9"
description: "SQL Anti patterns"
date: 2019-05-06
tags: [sql,anti,pattern]
comments: true
share: true
---

## 메타데이터 트리블 
```sql
CREATE TABLE Customser (
    customer_id NUMBER(9) PRIMARY KEY,
    contact_info    VARCHAR(255),
    business_type   VARCHAR(20),
    revenu          NUMBER(9,2)
)

-- 연도별 매출을 나타내는 칼럼 추가 
ALTER TABLE Customers ADD (revenue2002 NUMBER(9,2));
ALTER TABLE Customers ADD (revenue2003 NUMBER(9,2));
ALTER TABLE Customers ADD (revenue2004 NUMBER(9,2));
```

### 목표: 확장 적응성 지원
- 데이터 양이 늘어나면, 어떤 db든 쿼리 성능이 저하됨
- 쿼리 성능을 향상시키고 지속적으로 크기가 늘어나는 테이블을 지원하도록 db를 구성하는 것

### 안티패턴: 테이블 또는 칼럼 복제 
- 다른 모든 조건이 동일할 경우, 행이 적은 테이블을 조회하는게 빠름
- 어떤 작업시 모든 테이블이 보다 적은 행을 포함하도록 만드는 잘못된 생각을 하게 함
    - 아래 안티패턴으로 이어짐 
        - 많은 행을 가진 큰 테이블을 여러 작은 테이블로 분리
        - 하나의 칼럼을 여러개의 칼럼으로 분리 

#### 테이블이 너무 많아짐 
```sql
CREATE TABLE Bugs_2008 (...);
CREATE TABLE Bugs_2009 (...);
CREATE TABLE Bugs_2010 (...);
```
- 올바른 테이블을 사용하는 건 사용자의 몫이 됨

#### 데이터 정합성 관리
```sql
SELECT * FROM Bugs_2009
WHERE date_reported NOT BETWEEN '2009-01-01' AND '2009-12-31';

CREATE TABLE Bugs_2009 (
    ...
    date_reported   DATE CHECK (EXTRACT(YEAR FROM date_reported) = '2009')
);
-- 다른 년도 테이블 만들때 check 값 조정 필요 
```

#### 데이터 동기화 
- 메타 데이터 정보가 바뀔경우 대응 불가
```sql
UPDATE Bugs_2010
SET date_reported = '2009-12-27'
WHERE bug_id = 1234;

INSERT INTO Bugs_2009 (bug_id, date_reported, ...)
    SELECT bug_id, date_reported, ...
    FROM Bugs_2010
    WHERE bug_id = 1234;

DELETE FROM Bugs_2010 WHERE bug_id = 1234;
```

#### 유일성 보장
- PK는 모든 분할된 테이블에 걸쳐 유일함이 보장되어야 함 
- 시퀀스 객체를 사용하면, 키 값 생성을 위해 모든 분리된 테이블이 하나의 시퀀스를 사용할 수 있음 
- 테이블당 ID 유일성만 보장하는 데이터베이스는 조금 까다로움 -> 별도 테이블 생성

```sql
CREATE TABLE BugsIdGenerator (bug_id SERIAL PRIMARY KEY);

INSERT INFO BugsIdGenerator (bug_id) VALUES (DEFAULT);
ROLLBACK;

INSERT INTO Bugs_2010 (bugs_id, ...)
VALUES (LAST_INSERT_ID(), ...);
```

#### 여러 테이블에 걸쳐 조회하기 
```sql
SELECT b.status, COUNT(*) AS count_per_status FROM (
    SELECT * FROM Bugs_2008
        UNION ALL
    SELECT * FROM Bugs_2009
        UNION ALL
    SELECT * FROM Bugs_2010) AS b
GROUP BY b.status;
)
```
- 메타 테이블 추가시 어플리케이션 변경이 필요 

#### 메타데이터 동기화 
```sql
ALTER TABLE Bugs_2010 ADD COLUMN hours NUMERIC(9,2);
```
- 각 테이블마다 적용이 필요, 한 곳만 했다면 UNION 사용부분에서 에러 발생 

#### 참조 정합성 관리 
- Comments 테이블에서 Bugs_xxx 를 참조할때, FK 선언 불가
```sql
CREATE TABLE Comments (
    comment_id  SERIAL PRIMARY KEY,
    bug_id      BIGINT UNSINGED NOT NULL,
    FOREIGN KEY (bug_id) REFERENCES Bugs_xxx (bug_id)
);
```

- 자식 연관시에도 문제 발생 
```sql
SELECT * FROM Accounts a
JOIN (
    SELECT * FROM Bugs_2008
    UNION ALL
    SELECT * FROM Bugs_2009
    UNION ALL
    SELECT * FROM Bugs_2010
) t ON (a.account_id = t.reported_by)
```

### 안티패턴 인식 방법 
- 테이블당 추가 칼럼이 필요해 
- 테이블으 최대 몇개가지 만들수 있는지 확인 할 때
- 새로운 데이터 추가 실패가 테이블이 없어서 그럴 때 
- 여러 테이블을 한꺼번에 검색하는 쿼리가 필요 할 때 
- 테이블 이름을 파라미터로 넘길 필요가 생기 때 

### 안티패턴 사용이 합당할 때 
- 오래된 데이터를 분리해 별도 보관하는 방식일 때 

### 해법: 파티션과 정규화 
- 테이블이 커졌을때 테이블을 직접 분리하는 것보다, 수평분할, 수직분할, 종속테이블을 사용하는 방법이 있음 

#### 수평분할 사용 
- 행을 여러 파티션으로 분리하는 규칙과 함께 논리적 테이블을 하나 정희하면 나머지는 데이터베이스가 알아서해결해 줌
- 물리적으로는 테이블이 분리되어 있지만, SQL에서는 하나의 테이블인 것처럼 사용 가능 
- 각 테이블에서 자신의 행을 별도 스토리지로 분리하는 방식으로 정의 가능 

```sql
CREATE TABLE Bugs (
    bug_id  SERIAL PRIMARY KEY,
    ...
    date_reported   DATE,
) PARTITION BY HASH (YEAR(date_reported))
PARTITIONS 4;
```

- date_reported 칼럼의 연도별로 행을 분리하는 것, 테이블을 직접 분리하는 것과 비교했을 때 행이 잘못된 분리 테이블로 들어갈 위험이 없다는 장점이 있음 
- date_reported 없을 업데이트 해도 상관 없음 

#### 수직분할 사용 
- 수직분할은 칼럼으로 테이블을 나눔, 
- 크기가 매우 크거나 거의 사용되지 않는 칼럼이 있을 때 유리함
    - TEXT, BLOB
    - 만약 분리된 컬럼을 빼고 조회하면 좀더 효율적으로 접근 가능, * 사용시 포함 조회
```sql
CREATE TABLE ProductInstallers (
    product_id  BIGINT UNSIGNED PRIMARY KEY,
    installer_image BLOB,
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```
- mysql MyISAM 스토리지 엔진에서는 행이 고정크기일때 조회성능이 가장 좋음 
- VARCHAR 는 가변길이 데이터타입이므로 VARCHAR 타입이 하나라도 있으면 성능이점을 얻을 수 없음 
- 가변길이 칼럼을 모두 별로 테이블로 저장하면 쿼리시 조금이라도 이점을 얻을 수 있음 

```sql
CREATE TABLE Bugs (
    bug_id  SERIAL PRIMARY KEY,
    summary CHAR(80),
    date_reported   DATE,
    reported_by BIGINT UNSIGNED,
    FOREIGN KEY (reported_by) REFERENCES Accounts(account_id)
);

CREATE TABLE BugDescriptions (
    bug_id  BIGINT UNSIGNED PRIMARY KEY,
    description VARCHAR(1000),
    resolution  VARCHAR(1000),
    FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id)
);
```

#### 해결방법 
```sql
CREATE TABLE ProjectHistory (
    project_id  BIGINT,
    year        SMALLINT,
    bugs_fixed  INT,
    PRIMARY KEY (project_id, year),
    FOREIGN KEY (project_id) REFERENCES Projects(project_id)
);
```
- 결론적으로 데이터가 메타데이터를 낳도록 하지 말자 

