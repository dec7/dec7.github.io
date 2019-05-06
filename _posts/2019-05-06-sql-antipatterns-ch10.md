---
layout: post
title: "SQL Anti Patterns Ch10"
description: "SQL Anti patterns"
date: 2019-05-06
tags: [sql,anti,pattern]
comments: true
share: true
---

## 반올림 오류 
```sql
SELECT b.bug_id, b.hours * a.hourly_rate AS cost_per_bug
FROM Bugs AS b
    JOIN Accounts AS a ON (b.assigned_to = a.account_id);
```
- 소수를 지원하도록 테이블 변경 
```sql
ALTER TABLE Bugs ADD COLUMN hours FLOAT;
ALTER TABLE Accounts ADD COLUMN hourly_rate FLOAT;
```

### 목표: 정수대신 소수 사용 

### 안티패턴: FLOAT 데이터 타입 사용
- float는 IEEE754 표준에 따라 실수를 이진 형식으로 부호화 

### 안티패턴 인식 방법 
- float, real, double precision 타입이 사용되는 곳 

### 안테패턴 사용이 합당한 경우 
- integer, numeric 타입이 지원하는 것보다 큰 범뷔의 실수 값을 사용해야할떄 float가 적절, 
- float 사용이 적절한 예는 과학 계산용 어플리케이션 
- oracle에서 float는 정확한 자릿수를 가지는 수치 타입이고 binary_float 타입이 IEEE754 를 사용 

### 해법: numeric 테이터 타입 사용
- 고정소수점 수에는 numberic 또는 decimal 타입을 사용해야 함
```sql
ALTER TABLE Bugs ADD COLUMN hours NUMERIC(9,2);
ALTER TABLE Accounts ADD COLUMN hourly_rate NUMERIC(9,2);
```
- 칼럼이 9자리 수를 저장 가능 
- 두번째 인수로 스케일 지정 가능
- 총 9자리 중 지정될 스케일만큼 사용 가능 
- 장점 
    - 유리수가 float타입에서와 같이 반올림 되지 않고 저장됨 
