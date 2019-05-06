---
layout: post
title: "SQL Anti Patterns Ch13"
description: "SQL Anti patterns"
date: 2019-05-06
tags: [sql,anti,pattern]
comments: true
share: true
---

## 인덱스 샷건 
### 목표: 성능 최적화
- db성능을 향상 시키는 가장 좋은 방법은 인덱스를 잘 활용하는 것 

### 안티패턴: 무계획하게 인덱스 사용하기 
- 인덱스 불충분하거나 없는 경우 
- 너무 많은 인덱스 설정 
- 도움되지 않는 인덱스로 설정 

#### 없는 인덱스 
- INSERT, UPDATE, DELETE시 인덱스를 갱신하기 떄문에 오버헤드가 있으나, 훨씬 높은 성능을 보장 

#### 너무 많은 인덱스 
- 사용되지 않는 인덱스를 생성하면 아무런 이득이 없음 
```sql
CREATE TABLE Bugs (
    bug_id      SERIAL PRIMARY KEY,
    date_reported   DATE NOT NULL,
    summary     VARCHAR(80) NOT NULL,
    status      VARCHAR(10) NOT NULL,
    hours       NUMERIC(9,2),
    INDEX (bug_id),
    INDEX (summary),
    INDEX (hours),
    INDEX (bug_id, date_reported, status)
);
```

- bud_id
    - PK는 자동인덱스 생성되며 중복, 불필요한 오버헤드만 늘어남 
- summary
    - 긴 문자열에 대한 인덱스는 작은 문자열보다 큼, 
    - summary 칼럼 전체로 검색하거나 정렬하는 일은 거의 없을 것
- hours
    - 특정 값으로 검색할 일 없을 것
- bug_id, date_reported, status
    - 복합인덱스는 좋은 이유가 있으나, 중복, 사용되지 않는 인덱스를 생성하는 경우도 있음 
    - 복합인덱스는 칼럼 순서가 중요
    - 검색조건, 조인조건, 정렬순서에 맞춰 왼쪽에서 오른쪽으로 나열 

#### 인덱스가 도움되지 않을 때 
```sql
CREATE INDEX TelephoneBook ON Accounts (last_name, first_name)
```
- SELECT * FROM Accounts ORDER BY first_name, last_name
    - 정렬조건이 반대임 
- SELECT * FROM Bugs WHERE MONTH(date_reported) = 4
    - date_reported에 대한 인덱스가 있더라도 함수를 쓰므로 지원받지 못함 
- SELECT * FROM Bugs WHERE last_name = 'Charles' OR first_name = 'Charles'
    - 위 쿼리는 아래와 동일 
        - SELECT * FROM Bugs WHERE last_name = '?' UNION SELECT * FROM Bugs WHERE first_namt = '?'
        - 이름 찾는데는 도움 못받음 
- SELECT * FROM Bugs WHERE description LINK '%crash%';
    - 패턴 문자열은 어디든 나올 수 있으므로 도움 못받음 

### 안티패턴 인식 방법 
- 쿼리 성능이 필요한 경우 

### 안티패턴 사용이 합당한 경우 
- 최적의 추측이 필요 ?...

### 해법: 인덱스를 mentor 하라 
- 인덱스 샷건은 적절한 이유 없이 인덱스를 생성하거나 삭제하는 것에 대한 안티 패턴 
- MENTOR
    - Measure
    - Explain
    - Nominate
    - Test
    - Optimize
    - Rebuild

#### Measure
- 정보 없이 제대로된 결정을 내릴 수 없음 
- slow query 확인 
- 어플리케이션에서 어떤 쿼리가 가장 많은 시간을 사용하지는 확인 
- 쿼리 성능을 측정할 때는 쿼리 결과 캐싱 기능의 비활성화 필요 

#### Explain
- 모든 db는 옵티마이저를 통해 쿼리가 사용할 수 있는 인덱스를 선택 
- 데이터베이스의 이런 분석 결과를 리포트로 볼수 있음 --> QEP (Query Execution Plan)
    - MySql: Explain
    - Oracle: Explain plan

```sql
EXPLAIN SELECT Bugs.*
FROM Bugs
JOIN (BugsProducts JOIN Products USING (product_id)) USING (bug_id)
WHERE summary LINK '%crash%'
    AND product_name = 'Open RoundFile'
ORDER BY date_reported DESC;
```
- QEP 해석방법은 Optimizing Queries with EXPLAIN 을 읽어야 함 

#### Nominate
- 쿼리에 대한 옵티마이저의 실행계획이 있으므로, 쿼리에서 인덱스를 사용하지 않고 테이블을 접근하는 부분을 살펴봐야 함 
    - Oracle Automate SQL Tuning Advisor
    - MySql Enterprise Query Analyzer
- 인덱스 커버링 
    - 인덱스를 정의할 떄 인덱스에 필요하지 않더라도 추가적인 속성을 포함시킬 수 있음 
    - 만약 쿼리가 인덱스 데이터 구조에 포함된 칼럼만 참조한다면, db는 인덱스만 읽어서 쿼리 결과를 생성할 수 있음 

#### Test
- 중요한 단계 
- 인덱스 생성 후, 쿼리를 다시 프로파일링 함 -> 변경이 제대로 되었다는 것을 확인 

#### Optimize
- 인덱스는 빈번하게 사용되는 데이터 구조로 캐시 메모리에 보관할 좋은 후보 
    - mysql: LOAD INDEX INTO CACHE 

#### Rebuild
- 인덱스는 균형이 잡혀있을 때 가장 효율이 좋음 
- 시간이 지나면서 데이터가 변경되고 인덱스도 점점 균형을 잃는다 
- 주기적으로 인덱스를 정비해줄만한 가치가 있음 
    - Oracle, ALTER INDEX...REBUILD
    - MySql, ANALYZE TABLE or OPTIMIZE TABLE