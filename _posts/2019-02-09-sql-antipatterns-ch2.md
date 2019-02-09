---
layout: post
title: "SQL Anti Patterns Ch2"
description: "SQL Anti patterns"
date: 2019-02-09
tags: [sql,anti,pattern]
comments: true
share: true
---

## 무단횡단
### 목표
- Product에 여러 관리자가 등록 될 수 있음
- 현재는 1명의 계정만 등록할 수 있음

```sql
CREATE TABLE Products (
    product_id    SERIAL PRIMARY KEY,
    product_name  VARCHAR(1000),
    account_id    BIGINT UNSIGNED
    FOREIGH KEY (account_id) REFERENCES Accounts(account_id)
);

INSERT INTO Products (product_id, product_name, account_id)
VALUES (DEFAULT, 'Visual TurboBuilder', 12);
```

### 안티패턴
```sql
CREATE TABLE Products (
  product_id    SERIAL PRIMARY KEY,
  product_name  VARCHAR(1000),
  account_id    VARCHAR(100)
)
INSERT INTO Products (product_id, product_name, account_id)
VALUES (DEFAULT, 'Visual TurboBuilder', '12,34');
```

- 데이터 타입만 변경하여 여러개의 계정을 쉼표로 구분하여 포함
- 문제: 성능, 데이터 정합성

#### 특정 게정에 대한 제품 조회

```sql
SELECT * FROM Products WHERE account_id REGEXP '[[:<:]]12[[:>:]]';
```

- 패턴매칭을 사용할 경우
  - 잘못된 결과가 반환될 수 있음
  - 인덱스 활용 불가
  - 벤더중립적이지 않음: 패턴매칭 문법이 벤더에 따라 다를 수 있음

#### 주어진 제품에 대한 계정 정보 조회
```sql
SELECT * FROM Products AS p JOIN Account AS a 
ON p.account_id REGEXP '[[:<:]]' || a.account_id || '[[:>:]]'
WHERE p.product_id = 123;
```

- 문제점
  - 인덱스 사용 불가
  - 카테시안 곱을 생성한 뒤에 모든 행의 조합에 대해 정규식 평가

#### 집계쿼리 만들기 

```sql
SELECT 
  product_id, 
  LENGTH(account_id) - LENGTH(REPLACE(account_id, ',' '')) + 1 AS contact_per_product
FROM Products;
```
- 문제점
  - count, sum, avg와 같은 집계쿼리 사용 불가
  - 부정확하며 디버깅이 어렵고, 부정확함

#### 특정 제품에 대한 계정 갱신

```sql
UPDATE Products
SET account_id = account || ',' || 56
WHERE product_id = 123;
```

- 문제점
  - 새로운 ID 추가는 가능하나, 정렬된 상태를 유지 불가
  - 삭제시, 2번의 sql를 쿼리해야함 (조회, 갱신)

#### 제품 아이디 유효성 검증

```sql
INSERT INTO Products (product_id, product_name, account_id)
VALUES (DEFAULT, 'Visual TurboBuilder', '12,34,banana');
```

#### 구분자 문자 선택
- 문자열 구분자를 사용하는 경우, 목록에 구분자가 포함되지 않는다고 보장할 수 없음

#### 목록 길이 제한
- 목록에 저장할 수 있는 한계가 존재함


### 안티패턴 인식 방법
- 프로젝트 진행시 아래의 상황이 발생할 수 있음
  - 목록이 지원하는 최대 항목 수는 ?
    - account_id 컬럼 길이 선정시, 목록이 지원할 수 있는 최대 항목수에 대한 의문이 있을 수 있음
  - sql에서 단어의 경계는 어떻게 알 수 있는지 ?
    - 문자열의 일부를 찾기 위해 정규식을 사용할때
  - 목록에 절대 나오지 않을 문자에 대한 의문
    - 어떤 구분자를 사용하더라도 목록의 값에 사용될 수 있음

### 안티패턴 사용이 합당한 경우
- 반정규화를 적용해 성능을 향상시킬 수 있음
  - 목록을 쉼표로 구분된 문자열로 저장하는 것도 반정규화의 일종
  - 어플리케이션에서 쉼표로 구분된 형식의 데이터가 필요할 때
- 반정규화를 사용하기로 결정할때는 보수적이어야 함
  - 정규화는 어플리케이션 코드를 좀 더 유연하게 만들어 주고, 정합성을 유지시켜줌

### 해법
- 교차테이블 생성
  - 조인 테이블, 다대다 테이블, 매핑 테이블 등으로 불림
  - 두 테이블 사이의 다대다 관계를 구현

```sql
CREATE TABLE Contacts (
  product_id    BIGINT UNSINGED NOT NULL,
  account_id    BIGINT UNSIGNED NOT NULL,
  PRIMARY KEY (product_id, account_id),
  FOREIGN KEY (product_id) REFERENCES Products(product_id),
  FOREIGN KEY (account_id) REFERENCES Accounts(account_id)
);

INSERT INTO Contacts (product_id, account_id)
VALUES (123, 13), (123, 34), (345,23), (567,12), (567,34)
```

#### 계정으로 제품 조회, 제품으로 계정 조회

- 결과
  - 조인을 포함한 쿼리가 성능이 나쁠수도 있지만, 안티패턴 방법보다 인덱스를 훨씬 잘 활용함
  
##### 계정으로 제품 조회
```sql
SELECT p.* FROM Products AS p JOIN Contacts AS c
ON (p.product_id = c.product_id)
WHERE c.account_id = 34;
```

##### 제품으로 계정 조회
```sql
SELECT p.* FROM Products AS p JOIN Contacts AS c
ON (p.product_id = c.product_id)
WHERE c.product_id = 123;
```


#### 집계 쿼리 만들기

##### 제품당 계정 수
```sql
SELECT product_id, COUNT(*) AS accounts_per_product
FROM Contacts
GROUP BY product_id
```

##### 계정당 제품 수
``` sql
SELECT account_id, COUNT(*) AS products_per_account
FROM Contacts
GROUP BY account_id
```

##### 가장 많은 담당자를 할당받은 제품
``` sql
SELECT c.product_id, c.contacts_per_product
FROM (
  SELECT product_id, COUNT(*) AS accounts_per_product
  FROM Contacts
  GROUP BY product_id
) AS c
ORDER BY c.contacts_per_product DESC LIMIT 1
```

#### 특정 제품에 대한 계정 갱신
```sql
INSERT INTO Contacts(product_id, account_id) VALUES (456,34);
DELETE FROM Contacts WHERE product_id = 456 AND account_id = 34;
```

- 교차테이블에 대한 조작만으로 갱신 가능

#### 제품 아이디 유효성 검증
- FK를 사용하여  account_id, product_id 값이 적절한지 확인 할 수 있음
- sql 데이터 타입을 사용하여 유효한 데이터 형식이 맞는지 검증 가능

#### 구분자 문자 선택
- 별도 행 이므로 사용 가능

#### 목록 길이 제한
- 각 항목이 별도 행으로 구분되므로 한 테이블에 물리적으로 저장할 수 있는 행 수에 대해서 제한을 받음

### 교차 테이블의 다른 장점
- Contacts.account_id 에 걸린 인덱스를 활용하면 성능 향상
- 컬럼에 FK 선언시, DB가 내부적으로 그 컬럼에 대한 인덱스 생성
- 추가적인 정보 추가 가능