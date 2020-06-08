---
layout: post
title: "SQL Anti Patterns Ch7"
description: "SQL Anti patterns"
date: 2019-05-05
tags: [sql,anti,pattern]
comments: true
share: true
---

## 다형성 연관 
- 버그에 코멘트를 달때 1:N 관계 

```sql
CREATE TABLE Comments (
    comment_id  SERIAL PRIMARY KEY,
    bug_id      BIGINT UNSIGNED NOT NULL,
    author_id   BIGINT UNSINGED NOT NULL,
    comment_date    DATETIME NOT NULL,
    comment     TEXT NOT NULL,
    FOREIGN KEY (author_id) REFERENCES Accounts(account_id),
    FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id)
);
```
- 여러개의 부모 테이블을 참조하는 FK를 만들 수 없음 

### 여러 부모 참조
### 안티패턴: 이중 목적 FK 사용 
- 다양성 연관 == 난잡한 연관 

#### 다양성 연관 정의 
- FK 칼럼 issue_id 옆에 문자열 타입의 별도 칼럼 추가 후 참조하는 부모테이블이름을 넣음
```sql
CREATE TABLE Comments (
    comment_id  SERIAL PRIMARY KEY,
    issue_type  VARCHAR(20) -- Bugs, FeatureRequests
    issue_id    BIGINT UNSIGNED NOT NULL,
    author_id    BIGINT UNSINGED NOT NULL,
    comment_date    DATETIME NOT NULL,
    comment     TEXT NOT NULL,
    FOREIGN KEY (author_id) REFERENCES Accounts(account_id),
);
```
- 차이점 
    - issue_id에 대한 FK 선언이 빠짐 
    - FK로는 하나의 테이블만 참조 가능
    - Comments.issue_id 값이 부모 테이블의 참조정합성 확인 불가
    - Comments.issue_type 문자열이 db에 있는 테이블인지 확인 불가

#### 다양성 연관에서 조회
- Comments.issue_id 는 Bugs, FeatureRequests 양쪽 테이블의 PK로 나타날 수 있음
```sql
SELECT * FROM Bugs AS b JOIN Comments AS c
    ON (b.issue_id = c.issue_id AND c.issue_type = 'Bugs')
WHERE b.issue_id = 1234;


SELECT * FROM Comments AS c
    LEFT OUTER JOIN Bugs AS b
        ON (b.issue_id = c.issue_id AND c.issue_type = 'Bugs')
    LEFT OUTER JOIN FeatureRequests AS f
        ON (f.issue_id = c.issue_id AND c.issue_type = 'FeatureRequests');
```
- 단일 테이블일 경우는 잘 동작하지만, FeatureRequests 와 연관시 문제 발생

### 안티패턴 인식 방법 
- 어떤 테이블에도 태그를 달 수 있음 (무제한적 유연성)
- FK 선언 불가

### 안티패턴 사용이 합당한 경우 
- 다형성 연관 안티패턴 사용을 피하고 FK 제약 조건을 사용해 참조정합성을 지켜야 함
- 객체관계 프레임웍을 사용하는 경우 안타패턴 사용이 불가피 할 수도 있으나, 참조정합성을 유지하기 위한 로직을 캡슐화하므로 사용할 수도 있음

### 해법: 관계 단순화
- 다형성 연관을 피하면서 데이터 모델을 지원하기 위해 db를 다시 설계하는게 나음
- 데이터 관계를 그대로 수용하면서 정합성 강제하기 위해 메타데이터를 더 잘 활용하는 방법으로 개선 

#### 역참조
- 다형성 연관에서는 관계의 방향이 거꾸로

#### 교차 테이블 생성
- 자식 테이블 Comments에 있는 FK를 여러 부모 테이블이 참조할 수 없으므로, Comments 테이블을 참조하는 여러개 FK를 사용하도록 함
- 각 부모 테이블에 대해 별도의 교차 테이블 생성
- 교차 테이블에는 각 부모 테이블에 대한 FK와 Comments에 대한 FK도 포함

```sql
CREATE TABLE BugsComents (
    issue_id    BIGINT UNSINGED NOT NULL,
    comment_id  BIGINT UNSINGED NOT NULL,
    PRIMARY KEY (issue_id, comment_id),
    FOREIGN KEY (issue_id) REFERENCES Bugs (issue_id),
    FOREIGN KEY (comment_id) REFERENCES Comments (comment_id)
);

CREATE TABLE FeaturesComments (
    issue_id    BIGINT UNSINGED NOT NULL,
    comment_id  BIGINT UNSIGNED NOT NULL,
    PRIMARY KEY (issue_id, comment_id),
    FOREIGN KEY (issue_id) REFERENCES FeatureRequests(issue_id),
    FOREIGN KEY (comment_id) REFERENCES Comments (comment_id)
);
```

##### 신호등 설치 `
- 잠재적 약점
    - 허용하고 싶지 않은 연관이 생길 수 있음 
- 교차테이블은 보통 다대다 관계를 모델링하는데 사용되므로 특정댓글이 여러개의 버그나 기능요청과 연관될 수 있음
- 각 댓글은 하나의 버그/기능요청과 관계되어야 하므로 부분적으로 강제할 수 있음 (comment_id, unique 제약조건)

```sql
CREATE TABLE BugsComments (
    issue_id    BIGINT UNSIGNED NOT NULL,
    comment_id  BIGINT UNSIGNED NOT NULL,
    UNIQUE KEY (comment_id),
    PRIMARY KEY (issue_id, comment_id),
    FOREIGN KEY (issue_id) REFERENCES Bugs(issue_id),
    FOREIGN KEY (comment_id) REFERENCES Comments(comment_id)
);
```

- 특정 댓글이 양쪽 교차 테이블이 모두 한번씩 참조되는 것을 방지 못하고, 어플리케이션 코드 책임


##### 양쪽 다 보기 
- 특정 버그 또는 기능 요청에 대한 댓글을 교차 테이블을 이용해 간단히 조회 가능
```sql
SELECT * FROM BugsComments AS b
    JOIN Comments AS c USING (comment_id)
WHERE b.issue_id = 1234;

SELECT * FROM Comments AS c
    LEFT OUTER JOIN (BugsComments JOIN Bugs AS b USING (issue_id)) USING (comment_id)
    LEFT OUTER JOIN (FeaturesComments JOIN FeatureRequests AS f USING (issue_Id)) USING (comment_id)
WHERE c.comment_id = 9876;

```

##### 차선 통합 
- 여러 부모 테이블에 대해 조회한 결과를 하나의 테이블에서 조회한 것처럼 보이게 할 때
- 첫번째 방법 union
```sql
SELECT b.issue_id, b.description, b.reporter, b.priority, b.status, b.severity, b.version_affected, NULL AS sponsor
    FROM Comments AS c
    JOIN (BugsComments JOIN Bugs AS b USING (issue_id)) USING (comment_id)
    WHERE c.comment_id = 9876
UNION
SELECT f.issue_id, f.description, f.reporter, f.priority, f.status, NULL AS severity, NULL AS version_affected, f.sponsor
    FROM Comments AS c
    JOIN (FeaturesComments JOIN FeatureRequests AS f UNSING (issue_id)) USING (comment_id)
    WHERE c.comment_id = 9876;
```

- 두번째 방법 coalesce
    coalesce: 처음으로 NULL이 아닌 값을 가진 인자를 반환

```sql
SELECT c.*,
    COALESCE(b.issue_id, f.issue_id) AS issue_id,
    COALESCE(b.description, f.description) AS description,
    COALESCE(b.reporter, f.reporter) AS reporter,
    COALESCE(b.priority, f.priority) AS priority,
    COALESCE(b.status, f.status) AS status,
    b.severity,
    b.version_affected,
    f.sponsor
FROM Comments AS c
    LEFT OUTER JOIN (BugsComments JOIN Bugs AS b USING (issue_id)) USING (comment_id),
    LEFT OUTER JOIN (FeaturesComments JOIN FeatureRequest AS f USING (issue_id)) USING (comment_id)
WHERE c.comment_id = 9876;
```

#### 공통 슈퍼테이블 생성 
- 모든 부모 테이블이 상속할 베이스 테이블을 생성해 문제 해결 가능
```sql
CREATE TABLE Issues (
    issue_id    SERIAL  PRIMARY KEY
);

CREATE TABLE Bugs (
    issue_id    BIGINT UNSIGNED PRIMARY KEY,
    FOREIGN KEY (issue_id) REFERENCES Issues(issue_id),
    ...
);

CREATE TABLE FeatureRequests (
    issue_id    BIGINT UNSIGNED PRIMARY KEY,
    FOREIGN KEY (issue_id) REFERENCES Issues(issue_id),
    ...
);

CREATE TABLE Comments (
    comment_id  SERIAL PRIMARY KEY,
    issue_id    BIGINT UNSIGNED NOT NULL,
    author      BIGINT UNSINGED NOT NULL,
    comment_date    DATETIME,
    comment     TEXT,
    FOREIGN KEY (issue_id) REFERENCES Issues(issue_id),
    FOREIGN KEY (author) REFERENCES Accounts(account_id)
);
```

