---
layout: post
title: "SQL Anti Patterns Ch6"
description: "SQL Anti patterns"
date: 2019-05-01
tags: [sql,anti,pattern]
comments: true
share: true
---

## 엔티티, 속성, 값

- 날짜별로 행 수를 세기 위한 쿼리
```sql
SELECT date_reported, COUNT(*) FROM Bugs
GROUP BY date_reported;
```
- 숨어 있는 가정
  - 값이 Bugs.date_reported 와 같은 한 컬럼에 저장 
  - 값을 서로 비교할 수 있어 Groups by에서 같은 값끼리 모을 수 있음 
- 만약 
  - 날짜라 다은 칼럼에 저장 
  - 날짜라 서로 다양한 형식으로 입력되어 컴퓨터에서 두 날짜를 쉽게 비교할 수 없는 경우 

### 6.1 가변 속성 지원 
- 관계형 모델에서 속성집합이 다른 경우 객체 타입도 다르다는 뜻이므로 다른 테이블에 있어야 함
- 현대적 객체지향 프로그래밍 모델에서는 확장 방식으로 객체의 타입도 관계를 가질 수 있음

### 6.2 안티패턴 
#### 범용 속성 테이블 사용
```sql
CREATE TABLE Issues (
    issue_id    SERIAL PRIMARY KEY
)
INSERT INTO Issues (issue_id) VALUES (1234);

CREATE TABLE IssueAttributes (
    issue_id    BIGINT UNSIGNED NOT NULL,
    attr_name   VARCHAR(100) NOT NULL,
    attr_value  VARCHAR(100),
    PRIMARY KEY (issue_id, attr_name),
    FOREIGN KEY (issue_id) REFERENCES Issues(issue_id)
)
INSERT INTO IssueAttributes (issue_id, attr_name, attr_value)
VALUES
    (1234, 'product', '1'),
    (1234, 'date_reported', '2009-06-01'),
    (1234, 'status', 'NEW'),
    (1234, 'description', 'Saving does not work'),
    (1234, 'reported_by', 'Bill'),
    (1234, 'version_affected', '1.0'),
    (1234, 'severity', 'loss of functionality'),
    (1234, 'priority', 'high),
```

- 적은 칼럼, 추가 속성을 지원하기 위해 칼럼수를 늘릴 필요 없는 것, 불필요한 null 칼럼을 줄일 수 있는 것처럼 보임

#### 활용 
##### 속성 조회
```sql
-- SELECT issue_id, date_reported FROM Issues;
SELECT issue_id, attr_value AS "date_reported"
FROM IssueAttributes
WHERE attr_name = 'date_reported';
```

##### 데이터 정합성
- 필수속성 사용불가

- SQL 데이터 타입 사용 불가 
```sql
-- INSERT INTO Issues (date_reported) VALUES ('banana'); -- error
INSERT INTO IssueAttributes (issue_id, attr_name, attr_value)
VALUES (1234, 'date_reported', 'banana');
```

- 참조 정합성 강제 불가 
```sql
-- normal
CREATE TABLE Issues (
    issue_id    SERIAL PRIMARY KEY,
    status      VARCHAR(20) NOT NULL DEFAULT 'NEW',
    FOREIGN KEY (status) REFERENCES BugStatus(status)
);

-- eav model
CREATE TABLE IssueAttributes (
    issue_id    BIGINT UNSIGNED NOT NULL,
    attr_name   VARCHAR(100) NOT NULL,
    attr_value  VARCHAR(100),
    FOREIGN KEY (attr_value) REFERENCES BugStatus(status)
);
```

- 속성 이름 강제 불가

##### 행 재구성하기 
- 모든 행과 그 행의 속성을 칼럼으로 조회하는게 자연스러움
```sql
SELECt i.issue_id,
    i1.attr_value AS "date_reported",
    i2.attr_value AS "status",
    i3.attr_value AS "priority",
    i4.attr_value AS "description",
FROM Issues AS i
    LEFT ORTER JOIN IssueAttributes AS i1
        ON i.issue_id = i1.issue_id AND i1.attr_name = 'date_reported'
    LEFT ORTER JOIN IssueAttributes AS i2
        ON i.issue_id = i2.issue_id AND i1.attr_name = 'status'
    LEFT ORTER JOIN IssueAttributes AS i3
        ON i.issue_id = i3.issue_id AND i1.attr_name = 'priority'
    LEFT ORTER JOIN IssueAttributes AS i4
        ON i.issue_id = i4.issue_id AND i1.attr_name = 'description'
WHERE i.issue_id = 1234;
```

### 안티패턴 인식 방법
- 메타데이터 변경없이 확장 가능 할때 
- 데이터베이스 한계를 넘는 조인

### 안티패턴 사용이 합당한 경우 
- 어떤 경우든 합리화 어려움
- 앱에서 대부분의 경우 한 두곳에 적용할 수는 있음
- 비관계형 테이터가 필요할 경우 다른 솔루션 사용 권장

### 해법: 서브타입 모델링
#### 단일 테이블 상속
- 가장 단순한 설계 
- 모든 타입을 하나의 테이블에 저장, 각 타입에 있는 모든 속성을 별도의 칼럼으로 가지도록 함
- 속성 하나는 서브타입을 나타내는데 사용 

```sql
CREATE TABLE Issues (
    issue_id    SERIAL PRIMARY KEY,
    reported_by BIGINT UNSINGED NOT NULL,
    product_id  BIGINT UNSIGNED,
    priority    VARCHAR(20),
    version_resolved    VARCHAR(20),
    status      VARCHAR(20),
    issue_type  VARCHAR(10),
    severity    VARCHAR(20),
    version_affected    VARCHAR(20),
    sponsor     VARCHAR(50),
    FOREIGN KEY (reported_by) REFERENCES Accounts(account_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

- 한계 
    - 새로운 타입이 생기면, 데이터베이스는 새로 생긴 객체 타입의 속성을 수용해야함 
    - 어떤 속성이 어느 서브타입에 속하는지 정의하는 메테 데이터가 없음 
- 적용 대상 
    - 서브타입 개수가 적고, 특정 서브타입에만 속하는 속성개수가 적을 때 

#### 구체 테이블 상속
- 서브 타입별로 별도의 테이블을 만드는 것

```sql
CREATE TABLE Bugs (
    issue_id    SERIAL PRIMARY KEY,
    reported_by BIGINT UNSIGNED NOT NULL,
    product_id  BIGINT UNSINGED,
    priority VARCHAR(20),
    version_resolved VARCHAR(20),
    status  VARCHAR(20),
    severity    VARCHAR(20),
    version_affected    VARCHAR(20),
    FOREIGN KEY (reproted_by) REFERENCES Accounts(account_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);

CREATE TABLE FeatureRequests (
    issue_id    SERIAL PRIMARY KEY,
    reported_by BIGINT UNSIGNED NOT NULL,
    product_id  BIGINT UNSIGNED,
    priority    VARCHAR(20),
    version_affected    VARCHAR(20),
    status      VARCHAR(20),
    sponsor     VARCHAR(50),
    FOREIGN KEY (reproted_by) REFERENCES Accounts(account_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
)
```

- 장점 
    - 각 서브타입을 나타내는 부가적 속성 불필요
- 활용 
    - 모든 서브타입을 한꺼번에 조회할 필요가 거의 없는 경우 
```sql
CREATE VIEW Issues AS 
    SELECT b.*, 'bug' AS issue_type
    FROM Bugs AS b
        UNION ALL
    SELECT f.*, 'feature' AS issue_type
    FROM FeatureRequests AS f;
```

#### 클레스 테이블 상속 
- 테이블을 객체지향 클래스인 것처럼 생각해 상속을 흉내내는 것
```sql
CREATE TABLE Issues (
    issue_id    SERIAL PRIMARY KEY,
    reported_by BIGINT UNSIGNED NOT NULL,
    product_id  BIGINT UNSIGNED,
    priority    VARCHAR(20),
    version_affected    VARCHAR(20),
    status      VARCHAR(20),
    FOREIGN KEY (reported_by) REFERENCES Accounts(account_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);

CREATE TABLE Bugs (
    issue_id    BIGINT UNSIGNED PRIMARY KEY,
    severity    VARCHAR(20),
    version_affected    VARCHAR(20),
    FOREIGN KEY (issue_id) REFERENCES Issues(issue_id)
);

CREATE TABLE FeatureRequests (
    issue_id    SERIAL UNSIGNED PRIMARY KEY,
    sponsor     VARCHAR(50),
    FOREIGN KEY (issue_id) REFERENCES Issues(issue_id)
);
```

- 메타데이터에 의해 1:1 관계가 강제됨 
- 베이스테이블에 종속된 테이블의 FK는 PK이기도 함
- 모든 서브타입에 대한 검색을 하는 데 효율적인 방법 제공

- 모두 조회 
```sql
SELECT i.*, b.*, f.*
FROM Issues AS i
    LEFT OUTER JOIN Bugs AS b USING (issue_id),
    LEFT OUTER JOIN FeatrueRequests AS f USING (issue_id);
```
- 위 쿼리는 좋은 view 후보
- 모든 서브타입에 대한 조회가 많고, 공통 칼럼을 참조하는 경우가 많은 경우 적합 

#### 반 구조적 데이터
- 서브타입 수가 많거나, 새로운 속성을 지원하는 경우가 많은 경우 
    - 데이터 속성 이름과 값을 XML, JSON 형식으로 부호화해 TEXT, CLOB 칼럼으로 저장가능

```sql
CREATE TABLE Issues (
    issue_id    SERIAL PRIMARY KEY,
    reported_by BIGINT UNSINGED NOT NULL,
    product_id  BIGINT UNSIGNED,
    priority    VARCHAR(20),
    version_resolved    VARCHAR(20),
    status      VARCHAR(20),
    issue_type  VARCHAR(10),
    attributes  TEXT NOT NULL,
    FOREIGN KEY (reported_by) REFERENCES Accounts(account_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```
- 장점 
    - 확장이 쉬움
- 단점 
    - SQL이 특정 속성에 접근하는 것을 지원하지 못함 
- 대상 
    - 서브타입 개수를 제한할 필요 없고, 새로운 속성을 정의할 수 있는 완전한 유연성이 필요 할 때 

