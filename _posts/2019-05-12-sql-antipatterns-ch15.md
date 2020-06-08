---
layout: post
title: "SQL Anti Patterns Ch15"
description: "SQL Anti patterns"
date: 2019-05-12
tags: [sql,anti,pattern]
comments: true
share: true
---

## 애매한 그룹 

### 목표: 그룹당 최댓값을 가진 행 찾기 
- 예. 버그 데이터베이스에서 각 제품별로 가장 최근에 보고된 버그를 얻는 쿼리 
```sql
SELECT product_id, MAX(date_reported) AS latest
FROM Bugs JOIN BugsProducts USING (bug_id)
GROUP BY product_id;
```
- 하지만 위의 결과를 신뢰할 수 없음 
    - 목표는 그룹의 최대값뿐 아니라 해당값을 찾은 행의 다른 속성도 포함하도록 쿼리를 작성하는 것

### 안티패턴: 그룹되지 않은 칼럼 참조 
- 근본원인은 개발자가 SQL에서 그룹핑 쿼리의 동작 방식을 제대로 이해하지 못한 것 

#### 단일 값 규칙 
- 각 그룹행은 GROUP BY 절 뒤에 쓴 칼럼 (또는 칼럼 목록) 의 값이 같은 행
- 예, 다음 쿼리에서는 각 product_id 값마다 그룹이 생김 
```sql
SELECT product_id, MAX(date_reported) AS latest
FROM Bugs JOIN BugsProducts USING (bug_id)
GROUP BY product_id;
```

- 단일 값 규칙 
    - 쿼리에서 SELECT 목록에 있는 모든 칼럼은 그룹당 하나의 값을 가져야 함 
    - GROUP BY 절 뒤에 쓴 칼럼이 얼마나 많은 행 그룹에 대응되는지는 상관없이 , 각 그룹당 정확히 하나의 값만 나온다는 것이 보장 
```sql
SELECT product_id, MAX(date_reported) AS latest, bug_id
FROM Bugs JOIN BugsProducts USING (bug_id)
GROUP BY product_id;
```
- bug_id와 같은 여분의 칼럼은 데이터베이스가 단일함을 보장할 수 없기 떄문에, DB는 단일 값을 위반했다고 가정 

### 안티패턴 인식 방법 
- 단일 값 규칙을 위반할 경우 에러 발생 

### 해법: 칼럼을 모호하게 사용하지 않기 
#### 함수 종속인 칼럼만 쿼리하기 
- 가장 간단한 방법으로 모호한 쿼리를 제거하는 것 
```sql
SELECT product_id, MAX(date_reported) AS latest
FROM Bugs JOIN BugsProducts USING (bug_id)
GROUP BY product_id;
```
- bug_id 제외 

#### 상호 연관된 서브쿼리 사용 
- correlated subquery 는 바깥쪽 쿼리에 대한 참조를 가지고 있어 바깥쪽 퀄리의 각 행에 대한 다른 결과를 생성할 수 있음 
```sql
SELECT bp1.product_id, b1.date_reported AS latest, b1.bug_id
FROM Bugs b1 JOIN BugsProducts bp1 USING (bug_id)
WHERE NOT EXISTS (
    SELECT * FROM Bugs b2 JOIN BugsProducts bp2 USING (bug_id)
    WHERE bp1.product_id = bp2.product_id
    AND b1.date_reported < b2.date_reported
);
```
- 간단한 벙법이나, 상호연관 서브쿼리는 바깥쪽 쿼리의 각 행에 대해 한번씩 실행되기 때문에 성능상 최적의 방법은 아님 

#### 유도테이블 사용 
- 서브쿼리를 유도테이블 (derived table) 각 제품에 대한 product_id와 버그보고일자의 최대값만 포함하는 임시결과를 만들 수 있음, 그 대음 결과를 테이블과 조인해 쿼리 결과가 각 제품당 가장 최근 버그만 포함하게 함 
```sql
SELECT m.product_id, m.latest, b1.bug_id
FROM Bugs b1 JOIN BugsProducts bp1 USING(bug_id)
    JOIN (
        SELECT bp2.product_id, MAX(b2.date_reported) AS latest
        FROM Bugs b2 JOIN BugsProducts bp2 USING (bug_id)
        GROUP BY bp2.product_id
    ) m
    ON (bp1.product_id = m.product_id AND b1.date_reported = m.latest);
```
```sql
SELECT m.product_id, m.latest, MAX(b1.bug_id) AS latest_bug_id
FROM Bugs b1 JOIN
    (
        SELECT product_id, MAX(date_reported) AS latest
        FROM Bugs b2 JOIN BugsProducts USING (bug_id)
        GROUP BY product_id
    ) m
    ON (b1.date_ported = m.latest)
GROUP BY m.product_id = m.latest;
```
- 유도테이블 방법은 상호연관된 서브쿼리를 사용하는 방법보다 확장적응성이 좋은 대안 

#### JOIN 사용하기 
- 외부조인 
    - 대응되는 행이 없을 수도 있는 행의 집합에 대하 대응을 시도하는 조인 
```sql
SELECT bp1.product_id, b1.date_reported AS latest, b1.bug_id
FROM Bugs b1 JOIN BugsProducts bp1 ON (b1.bug_id = bp1.bug_id)
LEFT OUTER JOIN 
    (Bugs AS b2 JOIN BugsProducts AS bp2 ON (b2.bug_id = bp2.bug_id))
    ON (
        bp1.product_id = bp2.product_id 
        AND (
            b1.date_reported < b2.date_reported 
            OR b1.date_reported = b2.date_reported 
            AND b1.bug_id < b2.bug_id
        )
    )
WHERE b2.bug_id IS NULL;
```

#### 다른 칼럼에 집계 함수 사용하기 
- 다른 칼럼에 집계 함수를 적용하고 단일 값 규칙을 따르게 할 수 도 있음 
```sql
SELECT product_id, MAX(date_repotred) AS latest,
    MAX(bug_id) AS latest_bug_id
FROM Bugs JOIN BugsProducts USING (bug_id)
GROUP BY product_id;
```
- bug_id가 시간순으로 생성되는 경우만 사용될 수 있음 

#### 각 그룹에 대하 모든 값을 연결하기 
- MySQL, SQLite 는 그룹에 속한 모든 값을 하나의 값으로 연결하는 GROUP_CONCAT() 함수 지원 
    - 위함수는 디폴트로 쉬표로 구분된 문자열을 생성 

```sql
SELECT product_id, MAX(date_reported) AS latest, GROUP_CONCAT(bug_id) AS bug_id_list
FROM Bugs JOIN BugsProducts USING (bug_id)
GROUP BY product_id;
```
- 각 그룹안에 있는 모든 bug_id값을 포함 
