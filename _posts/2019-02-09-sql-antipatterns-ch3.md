---
layout: post
title: "SQL Anti Patterns Ch3"
description: "SQL Anti patterns"
date: 2019-02-09
tags: [sql,anti,pattern]
comments: true
share: true
---

## 순진한 트리
### 목적 및 문제점
#### 목적
- 댓글에 대한 대댓글이 달 수 있는 설계 필요
  - 간단하게, 댓글에 대댓글에 대한 참조를 가지도록 개선

```sql
CREATE TABLE Comments (
  comment_id    SERIAL PRIMARY KEY,
  parent_id   BIGINT UNSIGNED,
  comment     TEXT NOT NULL,
  FOREIGN KEY (parent_id) REFERENCES Comments(comment_id)
);
```

#### 문제점
- 무한히 깊이로 반복되는 댓글을 한 sql쿼리로 불러오기 어려워짐
- 간단하게 모든 Coment를 불러온뒤 메모리에서 트리구조를 형성할 수도 있지만 데이터 규모를 생각하면 비현실적임


### 목표: 계층구조 저장 및 조회
- 데이터가 재귀적 관계를 가지는 일은 흔한 일
  - 조직도, 글타래

### 안티패턴: 항상 부모에 의존하기

#### 초보적 방법: parent_id 추가
- 테이블 안의 다른 글을 참조
  - 관계를 강제하기 위해 FK를 걸 수도 있음

```sql
CREATE TABLE Comments (
  commnet_id    SERIAL PRIMARY KEY,
  parent_id     BIGINT UNSINGED,
  bug_id        BIGINT UNSIGNED NOT NULL,
  author        BIGINT UNSINGED NOT NULL,
  comment_date  DATETIME NOT NULL,
  comment       TEXT NOT NULL,
  FOREIGN KEY (parent_id) REFERENCES Comments(comment_id),
  FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
  FOREIGN KEY (author) REFERENCES Accounts(account_id)
);
```

- 인접 목록 (Adjacency List) 설계

#### 인접목록에서 트리 조회하기

##### 2단계 구조
```sql
SELECT c1.*, c2.*
FROM Comments c1 LEFT OUTER JOIN Comments c2
  ON c2.parent_id = c1.comment_id
```

##### 4단계 구조
```sql
SELECT c1.*, c2.*, c3.*, c4.*
FROM Comments c1
  LEFT OUTER JOIN Comments c2
    ON c2.parent_id = c1.comment_id
  LEFT OUTER JOIN Comments c3
    ON c3.parent_id = c2.comment_id
  LEFT OUTER JOIN Comments c4
    ON c4.parent_id = c3.comment_id
```
- 단계가 추가될 수록, 컬럼으로 확장됨

#### 인접 목록에서 트리 유지하기
- 새로운 노드를 추가하거나, 노드를 서브트리로 이동하는 작업은 비교적 간다 

##### 추가
```sql
INSERT INTO Comments (bug_id, parent_id, authro, comment)
VALUES (1234, 7, 'Kukla', 'Thanks!');
```

##### 이동
```sql
UPDATE Comments SET parent_id = 3 WHERE comment_id = 6;
```

##### 삭제
- 트리에서 노드를 삭제하는건 좀 복잡
  - 서브트리 전체를 삭제하기 위해서
    - FK제약조건을 만족하기 위해 여러번 쿼리를 날려 모든 자손을 찾은 다음
    - 가장 아래 단계부터 차례로 삭제하면서 올라가야 함
    
```sql
SELECT comment_id FROM Comments WHERE parent_id = 4; -- 5,6 반환
SELECT comment_id FROM Comments WHERE parent_id = 5; -- 결과 없음
SELECT comment_id FROM Comments WHERE parent_id = 6; -- 7 반환
SELECT comment_id FROM Comments WHERE parent_id = 7; -- 결과 없음

DELETE FROM Comments WHERE comment_id IN ( 7 );
DELETE FROM Comments WHERE comment_id IN ( 5, 6 );
DELETE FROM Comments WHERE comment_id = 4;
```

- 노드의 자손을 이동시키지 않고 항상 삭제한다면 아래 옵션을 사용할 수도 있음
  - FK에 ON DELETE CASCADE

```sql
SELECT parent_id FROM Comments WHERE comment_id = 6; -- 4 반환
UPDATE Comments SET parent_id = 4 WHERE parent_id = 6;

DELETE FROM Comments WHERE comment_ID = 6;
```

### 안티패턴: 인식방법
- 트리에서 얼마나 싶은 단계를 지원해야하는지 ?
- 트리 데이터 구조를 관리하는 코드를 수정하기 어려운지 ?
- 트리에서 고아노드가 주기적으로 발생하는지 ?

### 안티패턴 사용이 허용되는 경우
- 인접목록
  - 장점
    - 어떤 노드의 부모나 자식을 바로 얻을 수 있는 것
    - 새로운 노드를 추가하기 쉬움

- SQL 확장기능을 지원하는 벤더
  - SQL-99 표준에서 WITH 키워드에 CTE (Common Table Expression) 를 사용한 재귀적 문법 정의
  - MS SQL Server 2005, Oracle 11g, IBM DB2, PostgreSQL 8.4

```sql
WITH CommentTree
  (comment_id, bug_id, parent_id, authro, comment, depth)
AS (
  SELECT *, 0 AS depth FROM Comments
  WHERE parent_id IS NULL
  UNION ALL
    SELECT c.*, ct.depth+1 AS depth AS CommentTree ct
    JOIN Comments c ON (ct.comment_id = c.parent_id)
)
SELECT * FROM CommentTree WHERE bug_id = 1234;
```

- oracle 9i, 10g는 WITH절은 지원하나 CTE를 통한 문법을 지원 안함
  - START WITH, CONNECT BY PRIOR 사용 가능

```sql
SELECT * FROM Comments
START WITH comment_id = 9876
CONNECT BY PRIOR parent_id = comment_id;
``` 

### 해법: 대안 트리모델 사용
- 경로열거, 중첩집합, 클로저테이블

#### 경로열거 (Path Enumeration)
- 인접목록의 약점 중 하나는
  - 주어진 노드의 조상을 얻는데 비용이 많이 듦
- 경로열거 방법 해결
  - 일련의 조상을 각 노드의 속성으로 저장해 해결함
  - parent_id 컬럼대신, varchar타입의 path컬럼을 정의

```sql
CREATE TABLE Comments (
  comment_id    SERIAL PRIMARY KEY,
  path          VARCHAR(1000),
  bug_id        BIGINT UNSIGNED NOT NULL,
  author        BIGINT UNSIGNED NOT NULL,
  comment_date  DATETIME NOT NULL,
  comment       TEXT NOT NULL,
  FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
  FOREIGN KEY (author) REFERENCES Accounts(account_id)
);
```

##### 조상 및 후손 조회
```sql
SELECT *
FROM Comments AS c
WHERE '1/4/6/7/' LIKE c.path || '%';
```

```sql
SELECT * 
FROM Comments AS c
WHERE c.path LIKE '1/4/' || '%';
```

##### 통계: 글쓴이가 쓴 답글 수
```sql
SELECT COUNT(*)
FROM Comments AS c
WHERE c.path LIKE '1/4/' || '%'
GROUP BY c.author;
```

##### 새로운 노드 추가
```sql
INSERT INTO Comments (author, comment) 
VALUES ('Ollie', 'Good job!');

UPDATE Comments
  SET path = (SELECT path FROM Comments WHERE comment_id = 7)
WHERE comment_id = LAST_INSERT_ID();
```

##### 단점
- DB는 경로가 올바르게 형성되도록 하거나, 경로값이 실제 노드에 대응되도록 강제할 수 없음
  - 경로 문자열을 유지하는 작업은 어플리케이션에 종속되며, 많은 작업 필요
  - 경로 길이에 대한 제한이 발생 (VARCHAR가 아무리 커도 한계가 있음)

#### 중첩집합 (Nested Set)
- 자신의 부모를 저장하지 않고, 자손의 집합에 대한 정보 저장

```sql
CREATE TABLE Comments (
  comment_id    SERIAL PRIMARY KEY,
  nsleft        INTEGER NOT NULL,
  nsright       INTEGER NOT NULL,
  bug_id        BIGINT UNSIGNED NOT NULL,
  author        BIGINT UNSIGNED NOT NULL,
  comment_date  DATETIME NOT NULL,
  comment       TEXT NOT NULL,
  FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
  FOREIGN KEY (author) REFERENCES Accounts(account_id)
);
```

- 특징
  - nsleft와 nsright 수는 아래 조건을 지켜야함
    - nsleft수는 모든 자식 노드의 nsleft 수보다 작어야 하고
    - nsrigh는 모든 자식 노드의 nsright 수보다 커야 한다
    - comment_id 값과는 무관
  - 트리를 깊이우선 탐색하며 값을 하나씩 증가시켜가면서 할당

##### 자손 조회
```sql
SELECT c2.*
FROM Comments AS c1
  JOIN Comments AS c2
    ON c2.nsleft BETWEEN c1.nsleft AND c1.nsright
WHERE c1.comment_id = 4;
```

##### 조상 조회
```sql
SELECT c2.*
FROM Comments AS c1
  JOIN Comments AS c2
    ON c1.nsleft BETWEEN c2.nsleft AND c2.nsright
WHERE c1.comment_id = 6;
```

- 장점
  - 자식을 가진 노드를 삭제했을때, 자손이 자동으로 삭제된 노드의 부모의 자손이 됨
  - 노드를 삭제해 값들 사이 간격이 생기더라고 트리구조에 문제 없음

##### 깊이 조회
```sql
-- depth: 3
SELECT c1.comment_id, COUNT(c2.comment_id) AS depth
FROM Comments AS c1
  JOIN Comments AS c2
    ON c1.nsleft BETWEEN c2.nsleft AND c2.nsright
WHERE c1.comment_id = 7
GROUP BY c1.comment_id;

-- 삭제
DELETE FROM Comments WHERE comment_id = 6;

-- depth: 2
...
```

##### 부모 노드 조회
```sql
SELECT parent.*
FROM Comments AS c
  JOIN Comments AS parent
    ON c.nsleft BETWEEN parent.nsleft AND parent.nsright
  LEFT OUTER JOIN Comments AS in_between
    ON c.nsleft BETWEEN in_between.nsleft AND in_between.nsright
    AND in_between.nsleft BETWEEN parent.nsleft AND parent.nsright
WHERE c.comment_id = 6
  AND in_between.comment_id IS NULL;
```

##### 노드 추가, 이동
- 추가
  - 새로운 노드를 추가한 경우, 새 노드의 왼쪽 값보다 큰 모든 노드의 왼쪽/오른쪽 값을 재계산 필요
  - 새로 추가한 노드의 오르쪽 형체들, 조상들, 조상의 오른쪽 형제도 포함 됨

```sql
-- NS값 8,9 공간 확보
UPDATE Comments
  SET nsleft = CASE WHEN nsleft >= 8 THEN nsleft + 2 ELSE nsleft END,
      nsright = nsright + 2
WHERE nsright >= 7;

-- #5의 자식 생성, NS값 8,9
INSERT INTO Comments (nsleft, nsright, author, coment)
VALUES (8,9,'Fran','Me Too!');
```

- 각 노드를 조가하기 보다, 서브트리를 쉽고 빠르게 조회하는게 중요할때 적절함
  - 노드 추가/이동은 왼쪽/오른쪽 계산이 포함되기 때문에 복잡함

#### 클로저 테이블 (Closure Table)
- 계층구조를 저장하는 단순하고 우아한 방법
- 부모-자식 관계에 대한 경로와 트리의 모든 경로를 저장
- TreePaths 테이블 생성
  - 각 컬럼은 Comments에 대한 FK

```sql
CREATE TABLE Comments (
  comment_id    SERIAL PRIMARY KEY,
  bug_id        BIGINT UNSIGNED NOT NULL,
  author        BIGINT UNSIGNED NOT NULL,
  comment_date  DATETIME NOT NULL,
  comment       TEXT NOT NULL,
  FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id),
  FOREIGN KEY (author) REFERENCES Accounts(account_id)
);

CREATE TABLE TreePathes (
  ancestor    BIGINT UNSIGNED NOT NULL,
  descendant  BIGINT UNSIGNED NOT NULL,
  PRIMARY KEY (ancestor, descendant),
  FOREIGN KEY (ancestor) REFERENCES Comments(comment_id),
  FOREIGN KEY (descendant) REFERENCES Comments(comment_id)
);
```

##### 자손 조회
```sql
SELECT c.*
FROM Comments AS c
  JOIN TreePaths AS t 
    ON c.comment_id = t.descendant
WHERE t.ancestor = 4;
```

##### 조상 조회
```sql
SELECT c.*
FROM Comments AS c
  JOIN TreePaths AS t ON c.comment_id = t.ancestor
WHERE t.descendant = 6;
```

##### 노드 추가 
```sql
INSERT INTO TreePaths (ancestor, descendant) 
  SELECT t.ancestor, 8
    FROM TreePaths AS t
    WHERE t.descendant = 5
  UNION ALL
    SELECT 8,8
```

##### 노드 삭제
```sql
DELETE FROM TreePaths WHERE descendant = 7;
```

##### 답글과 서브트리 삭제
```sql
DELETE FROM TreePaths
WHERE descendant IN (
  SELECT descendant
  FROM TreePaths
  WHERE ancestor = 4
);
```


##### 서브트리 이동
```sql
-- 삭제
DELETE FROM TreePaths
WHERE descendant IN (
  SELECT descendant
  FROM TreePaths
  WHERE ancestor = 6
)
AND ancestor IN (
  SELECT ancestor
  FROM TreePaths
  WHERE descendant = 6
  AND ancestor != descendant
)

-- 갱신
INSERT INTO TreePath (ancestor, descendant)
  SELECT supertree.ancestor, subtree.descendant
  FROM TreePaths AS supertree
    CROSS JOIN TreePaths AS subtree
  WHERE supertree.descendant = 3
    AND subtree.ancestor = 6;
```

##### depth 정보
- 조회를 쉽게 하도록 path_length를 추가할 수 있음
```sql
SELECT * FROM TreePaths
WHERE ancestor = 4 AND path_length = 1;
```
