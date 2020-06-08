---
layout: post
title: "SQL Anti Patterns Ch18"
description: "SQL Anti patterns"
date: 2019-05-12
tags: [sql,anti,pattern]
comments: true
share: true
---

## 스파게티 쿼리 
- 한방 쿼리 
```sql
SELECT COUNT(bp.product_id) AS how_many_products,
    COUNT(dev.account_id) AS how_many_developers,
    COUNT(b.bug_id) / COUNT(dev.account_id) AS avg_bugs_per_developers,
    COUNT(cust.account_id) AS how_manY_customers 
FROM Bugs b JOIN BugsProducts bp ON (b.bug_id = bp.bug_id)
    JOIN Accounts dev ON (b.assigned_to = dev.account_id)
    JOIN Accounts cust ON (b.reported_by = cust.account_id)
WHERE cust.email NOT LIKE '%@example.com'
GROUP BY bp.product_id;
```

### 목표: SQL쿼리 줄이기 
- 한방쿼리가 만능이라고생각 

### 안티패턴: 복잡한 무제를 한번에 풀기 
- 한방쿼리는 좋은 접근이 아님 

#### 의도하지 않은 제품 
- 한방쿼리를 만들때 흔한 결과는 카테시안 곱 
- 카테시안 곱은 쿼리에 사용된 두 테이블에 이들의 관계를 제한 하는 조건이 없을 때 발생 

### 안티패턴 인식 방법 
- 결과가 너무 많을 때 
- sql이 너무 복잡할 때 

### 해법: 분할해서 정복하기 
- 검약률: 두개의 이론이 동일한 예측을 한다면 단순한 쪽이 좋은 이론 

#### 한번에 하나씩 
- 의도하지 않은 카테시안 곱이 생기는 두 테이블 사이에 논리적 조인 조건을 찾을 수 없으면, 조건이 없는 걸수도.있음 
    - 불필요한 카테시안 곱일 없애기 위해서는 스파케티 쿼리를 단순한 여러개로 나눠야 함 

```sql
-- 1
SELECT p.product_id, COUNT(f.bug_id) AS count_fixed 
FROM BugsProducts p
LEFT OUTER JOIN Bugs f ON (p.bug_id = f.bug_id AND f.status = 'FIXED')
WHERE p.product_id = 1
GROUP BY p.product_id;

-- 2
SELECT p.product_id. COUNT(o.bug_id) AS count_open
FROM BugsProduct p
LEFT OUTER JOIN Bugs o ON (p.bug_id = o.bug_id AND o.status = 'OPEN')
WHERE p.product_id = 1
GROUP BY p.product_id;

```
- 위 쿼리는 불필요한 카테시안 곱을 생성하지 않음 
- 새로운 요구사항이 추가되면 간단히 쿼리를 추가할 수 있음 
- SQL이 복잡한 쿼리보다 단순한 쿼리를 훨씬 잘 최적화 할 수 있음 
- 가독성 좋음 

#### union 연산 
- 여러 쿼리를 하나의 결과로 묶을 수 있음 

```sql
(
    SELECT p.product_id, f.status, COUNT(f.bug_id) AS bug_count
    FROM BugsProducts p
    LEFT OUTER JOIN Bugs f ON (p.bug_id = f.bug_id AND f.status = 'FIXED')
    WHERE p.product_id = 1
    GROUP BY p.product_id, f.status
)
UNION ALL
(
    SELECT p.product_id. o.status, COUNT(o.bug_id) AS bug_count
    FROM BugsProduct p
    LEFT OUTER JOIN Bugs o ON (p.bug_id = o.bug_id AND o.status = 'OPEN')
    WHERE p.product_id = 1
    GROUP BY p.product_id, o.status
)
ORDER BY bug_count;

```
- 서브쿼리의 칼럼이 호환될 때만 union 연산을 사용할 수 있음 
- 결과집합을 만드는 중간 칼럼 개수, 이름, 데이터 타입을 바꿀 수 없음 


#### 상사의 문제 해결 
- sql 을 분리해서 작성 

```sql
-- 작업하는 제품 개수 
SELECT COUNT(*) AS how_many_products
FROM Products;


-- 버그를 수정한 개발자 수 
SELECT COUNT(DISTINCT assigned_to) AS how_many_developers
FROM Bugs
WHERE status = 'FIXED';

-- 개발자당 평균 수정버그 개수 
SELECT AVG(bugs_per_developer) AS average_bugs_per_devloper
FROM (
    SELECT dev.account_id, COUNT(*) AS bugs_per_developer
    FROM Bugs b JOIN Accounts dev
        ON (b.assigned_to = dev.account_id)
    WHERE b.status = 'FIXED'
    GROUP BY dev.accont_id
) t;

-- 수정한 버그 중 고객이 보고한 개수 
SELECT COUNT(*) AS how_many_customer_bugs
FROM Bugs b JOIN Accounts cust ON (b.reported_by = cust.account_id)
WHERE b.status = 'FIXED' AND cust.email NOT LIKE '%@example.com';
```