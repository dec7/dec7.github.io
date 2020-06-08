---
layout: post
title: "SQL Anti Patterns Ch7"
description: "SQL Anti patterns"
date: 2019-03-10
tags: [sql,anti,pattern]
comments: true
share: true
---

## 7. 다형성 연관
- comment 테이블
```sql
CREATE TABLE Comments (
  comment_id    SERIAL PRIMARY KEY,
  bug_id        BIGINT UNSIGNED NOT NULL,
  author        BIGINT UNSIGNED NOT NULL,
  comment_date  DATETIME  NOT NULL,
  comment       TEXT NOT NULL,
  FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
  FOREIGN KEY (author) REFERENCES Accounts(account_id)
);
```
- 버그나, 기능요청 중 어느 이슈타입이든 하나의 테이블에 저장하고 싶더라도 여러개의 부모테이블을 참조하는 FK를 만들 수 없음 
- 쿼리가 실행하면서 테이블이 바뀔수 없음 (쿼리를 제출하는 시점에 모든 테이블의 이름이 정해져 있어야 함)

### 1. 여러 부모 참조

### 2. 안티패턴: 이중 목적의 FK 사용
- 다형성 연관
  - 난잡한 연관이라가 불리기도 함

```sql
CREATE TABLE Comments (
  comment_id    SERIAL PRIMARY KEY,
  issue_type    VARCHAR(20),
  issue_id      BIGINT UNSIGNED NOT NULL,
  author        BIGINT UNSIGNED NOT NULL,
  comment_date  DATETIME  NOT NULL,
  comment       TEXT NOT NULL,
  FOREIGN KEY (author) REFERENCES Accounts(account_id)
);
```
- 차이
  - issue_id에 대한 FK 선언 제외됨
  - EAV 안티패턴과 비슷한 특성을 가짐

- 다형성연관에서 조회
  - issue_id는 Bugs, FeatureRequests PK 칼럼에 나타날 수 있으니, issue_type이 중요한 속성

```sql
-- 
SELECT *
  FROM Bugs AS b JOIN Comments AS c
    ON (b.issue_id = c.issue_id AND c.issue_type = 'Bugs')
WHERE b.issue_id = 1234;

-- 두곳 모두 조회시
SELECT * 
  FROM Comments AS c
    LEFT OUTER JOIN Bugs AS b
      ON (b.issue_id = c.issue_id AND c.issue_type = 'Bugs')
    LEFT OUTER JOIN FeatureRequests AS f
      ON (f.issue_id = c.issue_id AND c.issue_type = 'FeatureRequests');
```

- FK제약조건이 아니므로, 서로 관계가 없는 경우에도 사용 가능

### 3. 안티패턴 인식 방법
- db내의 어떤 리소스에도 태그를 달 수 있음
  - EAV와 같은 무제한적 유연성 존재
- FK선언 불가 
- entity_type의 용도가 불명확한 경우

### 4. 안티패턴 사용이 합당한 경우
- 합당한 경우는 없음, FK 제약조건은 사용해 정합성 보장해야함 
- hibernate 와 같은 객체-관계 프로그래밍 프레임웍을 사용하는 경우 안티패턴 사용이 불가피할 수 있음
  - 프레임웍 로직을 캡슐화하여 연관성 연관으로 생기는 위협을 위험을 완화해줄 수 있음

### 5. 해법: 관계 단순화
- 다형성 연관의 단점을 피하면서 필요한 데이터 모델을 지원하기 위해서, 데이터베이스를 다시 설계하는게 나음

#### 1. 역참조
##### 1. 교차 테이블 생성
- Comments 테이블을 참조하는 여러개의 FK를 사용하도록 함
- Comments.issue_type 칼럼 불필요
- 메타데이터로 데이터 정합성 유지 가능

```sql
CREATE TABLE BugsComments (
  issue_id    BIGINT UNSINGED NOT NULL,
  comment_id  BIGINT UNSINGED NOT NULL,
  PRIMARY KEY (issue_id, comment_id),
  FOREIGN KEY (issue_id) REFERENCES Bugs(issue_id),
  FOREIGN KEY (comment_id) REFERENCES Comments(comment_id)
);

CREATE TABLE FeaturesComments (
  issue_id    BIGINT UNSINGED NOT NULL,
  comment_id  BIGINT UNSIGNED NOT NULL,
  PRIMARY KEY (issue_id, comment_id),
  FOREIGN KEY (issue_id) REFERENCES FeatureRequests(issue_id),
  FOREIGN KEY (comment_id) REFERENCES Comments(comment_id)
);
```

##### 2. 신호등 설치
- 잠재적 약점
  - 허용하고 싶지 않은 연관이 생길 수 있음

#### 2. 공통 슈퍼테이블 생성