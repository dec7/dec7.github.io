---
layout: post
title: "SQL Anti Patterns Ch19"
description: "SQL Anti patterns"
date: 2019-05-12
tags: [sql,anti,pattern]
comments: true
share: true
---

## 암묵적 칼람 
- 어플리케이션이 찾는 칼럼과, sql에서 반환된 칼럼 이름이 다른 경우 null 이 될 수 있음 

### 목표: 타이핑 줄이기 
- 와일드 카드를 사용하면 쿼리가 간결해짐 
```sql
SELECT bug_id, date_reported, summary, description, resoultion, reported_by, assigned_to, verified_by, status, priority, hours 
FROM Bugs;

SELECt * FROM Bugs;
```

### 안티패턴: 지름길만 좋아하면 길을 잃는다 .
- 와일드 카드를 쓰는 습관은 몇가지 위험이 있음 

#### 리팩토링 방해 
- 추가 칼럼이 필요할 때 
```sql
ALTER TABLE Bugs ADD COLUMN date_due DATE;
```

- insert 문 사용할때 에러발생 
    - 테이블의 모든 칼럼에 대해 값은 테이블에 정의된 순서대로 들어가야 함 
    - 특히, insert 문 발생시 동일 타입에 대해 칼럼이 삭제될 경우 에러를 볼 수 없음 

#### 숨겨진 비용 
- 성능과 확장적응성에 문제를 줄수 있음 
- 모든 칼럼을 조회할 경우 server <-> db간 통신이 증가할 수 있음 

### 안티패턴 인식 방법 
- 어플리케이션이 예전 칼럼 이름음 참조해 동작하지 않을 때 
- 네트워크 병목이 있을 때 

### 안티패턴 사용이 합당한 경우 
- 테스트나, 현재 데이터를 확인하는 용도 
- 조인 쿼리에서 각 테이블에서 개별적으로 와일드카드를 사용할 수 있음 
```sql
SELECT b.*, a.first_name, a.email
FROM Bugs b, JOIN Accounts a
ON (b.reported_by = a.account_id);
```

### 해법: 명시적으로 칼럼 이름 지정 
- 와일드카드나, 암묵적 칼럼목록에 의지하기 보다 항상 필요한 칼럼을 나열해야함 

#### 오류검증 
- 컬럼 이름을 지정할 경우 장점 
    - 테이블의 컬럼 위치가 바뀌어도 쿼리 결과는 바뀌지 않음 
    - 테이블의 칼럼이 추가되어도 쿼리 결과에는 나타나지 않음 
    - 테이블에서 칼럼이 삭제되면, 퀄리가 에러를 발생시킴. 
        - 이것은 코드를 고쳐야할 위치를 직접 알려주기 때문에 좋은 코드 
        - insert도 동일 




