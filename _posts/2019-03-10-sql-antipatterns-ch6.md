---
layout: post
title: "SQL Anti Patterns Ch6"
description: "SQL Anti patterns"
date: 2019-03-10
tags: [sql,anti,pattern]
comments: true
share: true
---

## 6. 엔티티,속성,값
- 날짜별로 행수를 세기 위해
  - 값이 Bugs.date_reported 와 한 칼럼에 저장된다는 가정이 있음
```sql
SELECT date_reported, COUNT(*)
FROM Bugs
GROUP BY date_reported;
```

### 1. 가변속성 지원
- 일반적인 테이블은 모든 행과, 속성 칼럼으로 이뤄짐
- 각 행은 객체의 인스턴스를 나타냄, 즉 속성 집합이 다르면 객체의 타입이 다르다는 것
  - oop에서는 상속방법으로 객체의 타입도 관계를 가질 수 있음

### 2. 안티 패턴: 범용 속성 테이블 사용
#### EAV (Entity-Attribute-Value)
- open schema, schemaless, name-value pairs
- 덴티티의 속성을 속성 엔티티로 관리하는 방식

#### 테이블
```sql
CREATE TABLE Issues (
  issue_id  SERIAL PRIMARY KEY
);

CREATE TABLE IssueAttributes (
  issue_id  BIGINT UNSIGNED NOT NULL,
  attr_name VARCHAR(100) NOT NULL,
  attr_value  VARCHAR(100),
  PRIAMRY KEY (issue_id, attr_name),
  FOREIGN KEY (issue_id) REFERENCES Issues(issue_id)
);

INSERT INTO Issues (issue_id) VALUES (1234);

INSERT INTO IssueAttributes (issue_id, attr_name, attr_value)
VALUES 
(1234, 'product', '1'),
(1234, 'date_reported', '2009-06-01'),
(1234, 'status', 'Saving does not work'),
...;
```

#### 장점
- 두 테이블 모두 적은 칼럼 유지
- 새로운 속성지원을 위해 칼럼을 추가할 필요 없음
- 불필요한 null을 채울 필요 없음

#### 단점
- 데이터 정합성 지원받을 수 없음
  - 필수 속성, 데이터 타입, 참조 정합성, 속성 이름 강제 불가
- 행을 재구성하기 위해 필드별 조인이 필요

```sql
SELECT 
  i.issue_id, 
  i1.attr_value AS 'date_reported',
  i2.attr_value AS 'status',
  i3.attr_value AS 'priority',
  ...
FROM Isseus AS i
LEFT OUTER JOIN IssueAttributes AS i1
  ON i.issue_id = i1.issue_id AND i1.attr_name = 'date_reported'
LEFT OUTER JOIN IssueAttributes AS i2
  ON i.issue_id = i2.issue_id AND i2.attr_name = 'status'
  ...
WHERE i.issue_id = 1234;
```

### 3. 안티패턴 인식 방법
- 런터임시 메타 데이터 변경없이 확장 가능
  - rdms 는 이런 수준의 유연성을 지원하지 않음
- 하나의 쿼리에서 조인의 수를 걱정할 수준
  - 조인의 수를 걱정하는 것은 설계가 잘못된것

### 4. 안티패턴 사용이 합당한 경우
- EAV 안티패턴 사용을 합리화할 수 없음

### 5. 해법: 서브타입 모델링

#### 1. 단일 테이블 상속
- Martin Fowler: Patterns of Enterprise Application Architecture
- 가장 단순한 설계는 관련 모든 타입을 하나의 테이블에 저장 후, 각 타입에 있는 모든 속성을 별도의 칼럼으로 가지도록 하는 것
- 속성 하나는 행의 서브타입을 나타내는 데 사용해야함
  - issue_type
- 해당속성이 적용되지 않는 경우, null을 넣음

```sql
CREATE TABLE Issues (
  issue_id    SERIAL PRIMARY KEY,
  reported_by BIGINT UNSIGNED NOT NULL,
  product_id  BIGINT UNSIGNED,
  priority    VARCHAR(20),
  version_resolved  VARCHAR(20),
  status      VARCHAR(20),
  issue_type  VARCHAR(10),
  severity    VARCHAR(20),
  version_affected  VARCHAR(20),
  sponsor     VARCHAR(50),
  FOREIGN KEY (reported_by) REFERENCES Accounts(account_id),
  FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

- 한계
  - 추가되는 타입에 따라 속성 추가 필요
  - 어떤 속성이 서브타입에 해당하는지 정의된 메타데이터가 없음
  
#### 2. 구체 테이블 상속
- 서브 타입별로 별도의 테이블을 만드는 것
- 장점
  - 특정 서브타입에 지정되지 않은 속성은 저장할 수 없음
  - 서브타입을 나타내는 부가적 속성이 불필요
- 단점
  - 어떤 속성이 공통 속성인지 알 수 없음 
  - 공통 속성이 추가되면 모든 서브타입 테이블 수정 필요
  - 관련 객체가 서브타입 테이블에 저장되었다는 알려주는 메타데이터도 없음
    - (개발자는 두 테이블간 어떤 논리적 관계가 있는지 알 수 없음)
  - 서브타입에 관계없이 모든 객체를 보는 것이 복잡해짐
    - 서브타입 테이블에서 공통속성만 선택 후, union으로 묶은 뷰를 정의해야함
    
```sql
CREATE VIEW Issues AS 
  SELECT b.*, 'bug' AS issue_type
  FROM Bugs AS b
    UNION ALL
  SELECT f.*, 'feature' AS issues_type
  FROM FeatureRequests AS f;
```

#### 3. 클래스 테이블 상속
- oop에서 상속 흉내
  - 공통속성을 포함하는 베이스 타입 테이블 만들고, 각 서브타입에 대한 테이블 만듦

```sql
CREATE TABLE Issues (
  issue_id    SERIAL PRIMARY KEY,
  reported_by BIGINT UNSIGNED NOT NULL,
  product_id  BIGINT UNSIGNED,
  priority    VARCHAR(20),
  version_resolved  VARCHAR(20),
  status      VARCHAR(20),
  FOREIGN KEY (reported_by) REFERENCES Accounts(account_id),
  FOREIGN KEY (product_id) REFERENCES Products(product_id)
);

CREATE TABLE Bugs (
  issue_id      BIGINT UNSIGNED PRIMARY KEY,
  severity    VARCHAR(20),
  version_affected  VARCHAR(20),
  FOREIGN KEY (issue_id) REFERENCES Issues(issue_id)
);

CREATE TABLE FeatureRequests (
  issue_id      BIGINT UNSIGNED PRIMARY KEY,
  sponsor     VARCHAR(50),
  FOREIGN KEY (issue_id) REFERENCES Issues(issue_id)
)

```

- 메타데이터에 의해 일대일 관계가 강제됨
- 검색에서 베이스 타입에 있는 속성만 참조하는 한 , 모든 서브 타입에 대한 검색하는데 효율적 방법 지원 
- 베이스 테이블의 행이 어떤 서브 타입을 나타내는지 알 필요가 없음
  - 서브타입개수가 적은 경우, 각 서브타입과 조인하는 쿼리를 작성

```sql
SELECT i.*, b.*, f.*
  FROM Isseus AS i
    LEFT OUTER JOIN Bugs AS b USING (issue_id)
    LEFT OUTER JOIN FeatureRequests AS f USING (issue_id);
```

#### 4. 반구조적 데이터
- 서브타입 수가 너무 많거나 새로운 타입을 지원해야 하는 경우
  - 데이터의 속성이름과 값을 XML, JSON 형식으로 TEXT, CLOB 칼럼으로 저장
  - Martin Fowler: Serialzed LOB pattern

```sql
CREATE TABLE Issues (
  issue_id    SERIAL PRIMARY KEY,
  reported_by BIGINT UNSIGNED NOT NULL,
  product_id  BIGINT UNSIGNED,
  priority    VARCHAR(20),
  version_resolved  VARCHAR(20),
  status      VARCHAR(20),
  issue_type  VARCHAR(10),
  attributes  TEXT NOT NULL,
  FOREIGN KEY (reported_by) REFERENCES Accounts(account_id),
  FOREIGN KEY (product_id) REFERENCES Products(product_id)
)
```

- 장점
  - 확장이 쉬움
- 단점
  - SQL이 특정 속성에 접근하는것을 거의 지원하지 않고, 해석을 위한 어플리케이션 코드작성
- 서브타입 개수를 제한할 수 없고, 새로운 속성을 정의할 수 있는 유연성이 필요한 경우 사용됨