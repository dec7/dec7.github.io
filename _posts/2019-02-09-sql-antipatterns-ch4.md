---
layout: post
title: "SQL Anti Patterns Ch4"
description: "SQL Anti patterns"
date: 2019-02-10
tags: [sql,anti,pattern]
comments: true
share: true
---

## 아이디가 필요해

```sql
CREATE TABLE ArticleTags (
  id      SERIAL PRIMARY KEY,
  article_id  BIGINT UNSIGNED NOT NULL,
  tag_id      BIGINT UNSIGNED NOT NULL,
  FOREIGN KEY (article_id) REFERENCES Articles(id),
  FOREIGN KEY (tag_id) REFERENCES Tags(id)
);
```
- PK 가 있더라도 중복된 값이 포함될 수 있음

### 목표: PK관례 확립
- 장점
  - PK는 테이블 내의 모든 행이 유일함을 보장함
  - 각행에 접근하는 논리적인 매커니즘
  - 중복행이 저장되는것을 방지
  - FK 로 부터 참조될 수 있음
- 생성
  - 아무런 의미를 가지지 않은 값을 저장할 새로운 칼럼 필요, 그리고 이 컬럼을 PK 로 사용하면  (가상키, 대체키)
    - 다른 속성 칼럼에는 중복값이 들어가는 것을 허용할 수 도 있고, 
    - 특정행에 유일하게 접근할 수 있음
  - 벤터별 유일한 정수값을 생성하는 매커니즘 제공


### 안티패턴: 만능키
- db내 모든 테이블이 아래 특성을 가지는 PK컬럼을 가지도록 하는 문화적 관례를 만듦
  - PK칼럼 이름은 id
  - PK칼럼의 데이터 타입은 32비트 또는 64비트 정수
  - 유일한 값은 자동생성

#### id == PK ?
- id 칼럼이 너무 흔해져서 PK와 동의어가 됨
```sql
CREATE TABLE Bugs (
  id          SERIAL PRIMARY KEY,
  description VARCHAR1(1000),
);
```

#### 중복키 생성
- 테이블의 다른 컬럼이 자연키로 사용될 수 있는 상황이지만, 단지 통념에 따라 id를 PK 로 정의
```sql
CREATE TABLE Bugs (
  id          SERIAL PRIMARY KEY,
  bug_id      VARCHAR(10) UNIQUE,
  description VARCHAR(1000)
);

INSERT INTO Bugs (bug_id, description)
VALUES ('VIS-078', "crashes on save');
```

#### 중복행 허용
- 복합키는 여러 칼럼을 포함하고, 조합이 테이블 안에서 한번만 나타나는 것을 보장
- id를 PK로 쓸 경우, 유일해야하는 두 컬럼에 대해서 제약조건이 적용되지 않음

```sql
CREATE TABLE BugsProducts (
  id        SERIAL PRIMARY KEY,
  bug_id    BIGINT UNSIGNED NOT NULL,
  product_id  BIGINT UNSINGED NOT NULL,
  FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
  FOREIGN KEY (product_id) REFERENCES Products(product_id)
);

INSERT INTO BugsProducts (bug_id, product_id) 
VALUES (1234, 1), (1234, 1), (1234, 1);

```
- 추가적인 제약조건이 필요
```sql
CREATE TABLE BugsProducts (
  id        SERIAL PRIMARY KEY,
  bug_id    BIGINT UNSIGNED NOT NULL,
  product_id  BIGINT UNSINGED NOT NULL,
  UNIQUE KEY (bug_id, product_id),
  FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
  FOREIGN KEY (product_id) REFERENCES Products(product_id)
);
```

#### 모호한 키 의미
- 불명확한 컬럼 이름 의미

#### using 사용
```sql
SELECT * FROM Bugs AS b JOIN BugsProducts AS bp ON (b.bug_id = bp.bug_id);
SELECT * FROM Bugs JOIN BugsProducts USING (bug_id);
SELECT * FROM Bugs AS b JOIN BugsProducts AS bp ON (b.id = bp.bug_id);
```


#### 어려운 복합키

### 안티패턴: 인식방법
- PK가 불필요할것 같다는 인식
- 다대다 연결시 중복 발생시

### 안티패턴: 사용이 합당한 경우
- 가상키는 지나치케 긴 자연키를 대체하기 위해 선택일 뿐, 반드시 가상키를 사용할 필요 없음

### 해법: 상황에 맞추기
- PK는 제약조건일뿐, 데이터 타입이 아님
  - 데이터 타입이 인덱스를 지원하기만 하면, 어떤 칼럼이든 PK를 선언할 수 있음
  - 테이블의 특정칼럼을 PK로 선언하지 않아도, 자동증가하는 정수값을 가지도록 정의할 수 있음

#### 있는 그대로 말하기
- PK는 의미있는 이름을 선택
  - 이름은 PK가 식별하는 인티티의 타입을 나타내야 함
  - Bugs -> bug_id
  - FK에서도 가능하면 같은 이름을 사용해야 함
    - 예외는, 연결의 본질을 더 잘 표현하는 경우

```sql
CREATE TABLE Bugs (
  ...
  reported_by BIGINT UNSIGNED NOT NULL,
  FOREIGN KEY (reported_by) REFERENCES Accounts(account_id)
);
``` 

#### 자연키와 복합키 포용
- 유일함이 보장되고, null을 가지는 경우가 없고, 행을 식별하는 용도로 사용할 수 있는 경우
  - 가상키를 추가할 필요는 없음

