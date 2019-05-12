---
layout: post
title: "SQL Anti Patterns Ch16"
description: "SQL Anti patterns"
date: 2019-05-12
tags: [sql,anti,pattern]
comments: true
share: true
---

## 임의의 선택 
- 쿼리가 큰 규모에서 제대로 동작하지 않는 경우 

### 목표: 샘플 행 가져오기 
- 샘플 데이터만 반환하는 효율적인 SQL 쿼리 작성 

### 안티패턴: 데이터를 임의로 정렬 
```sql
SELECT * FROM Bugs ORDER BY RAND() LIMIT 1;

-- normal
SELECT * FROM Bugs ORDER BY date_reported;
```
- 직관적이나 일반적인 정렬과 비교했을때 rand() 처럼 비결정적 수식으로 정렬시 인덱스 활용 불가 

### 해법: 데이블 전체 정렬 피하기 
- 임의로 정렬하는 방법은 데이터가 적을때는 쓸만하나, 데이터가 많으면 사용하기 어려움 
- 임의로 정렬하는 것은 테이블 스캔과 비용이 많이 드는 수동정렬을 수반하면, 이런 비효율적인 쿼리를 쓰면 안됨

#### 1과 MAX 사이에서 임의의 키값 고르기
- 1과 PK 의 최댓값 사이에서 임의의값을 선택하는 것 
```sql
SELECT b1.*
FROM Bugs AS b1
JOIN (
    SELECT CEIL(RAND() * (SELECT MAX(bug_id) FROM Bugs)) AS rand_id
) AS b2
ON (b1.bug_id = b2.rand_id);
```
- 가정: PK 값이 1부터 시작해 연속적이고, 그 사이에 빈값이 없다 

#### 다음으로 큰 기 값 고르기 
- 이 경우은 1과 최댓값 사이에 빈틈이 있어도 쓸 수 있음 
```sql
SELECT b1.*
FROM Bugs AS b1
JOIN (
    SELECT CEIL(RAND() * (SELECT MAX(bug_id) FROM Bugs)) AS bug_id
) AS b2
WHERE b1.bug_id >= b2.bug_id
ORDER BY b1.bug_id
LIMIT 1;
```

#### 모든 키값의 목록을 구한뒤ㅡ 임의로 하나 고르기 
- 결과 집합의 PK값 하나를 고르는 어플리케이션 코드를 사용할 수 있음 
- db로부터 모든 bug_id 값을 불러올때 목록 크기가 엄청 클 수 있음 
- 쿼리를 두번해야함 

#### 오프셋을 이용해 임의로 고르기 
- 데이터 집합에서 행의 개수를 세고 0과 행 개수 사이의 임의의수를 고른 다음, 데이터 집합을 쿼리할 때 이 수를 오프셋으로 사용하는 것 
- 어플리케이션 종속적 

#### 벤터 종속적 
```sql
-- MS SQL
SELECT * FROM Bugs TABLESAMPLE (1 ROWS);

-- Oracle
-- sample은 테이블에서 1%의 행을 가져오고 임의의 순서로 정렬 후 한 행만 가져옴
SELECT * FROM (
    SELECT * FROM Bugs SAMPLE(1)
    ORDER BY dbms_random.value
)
WHERE ROWNUM = 1;
```