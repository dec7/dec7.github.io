---
layout: post
title: "SQL Anti Patterns Ch5"
description: "SQL Anti patterns"
date: 2019-02-10
tags: [sql,anti,pattern]
comments: true
share: true
---

## 키가 없는 엔트리

### 목표: 데이터베이스 아키텍쳐 단순화
- 참조정합성
  - 데이터베이스를 적절히 설계하고 운영하는데 있어 중요한 부분
  - FK 제약조건을 선언하면 그 칼럼에 들어가는 부모 테이블의 PK가 존재해야함
- 일부는 FK 무시
  - 업데이트시 제약조건과 충돌할 수 있음
  - DB에서 참조정합성 지원 안함
  - FK에 자동 생성되는 인덱스 때문에 성능에 영향을 받는다고 믿음

### 안티패턴: 제약조건 무시 
- 참조 정합성을 보장하기 위한 코드를 직접 작성해야함

#### 무결점 코드
- 참조 정합성을 유지하면서 추가하기 위한 부가적인 작업이 필요함

```sql
SELECT account_id FROM Accounts WHERE account_id = 1;
INSERT INTO Bugs (reported_by) VALUES (1);
SELECT bug_id FROM Bugs WHERE reported_by = 1;
DELETE FROM Accounts WHERE account_id = 1;
```

#### 벌크 업데이트
- 많은 칼럼을 한번에 갱신할때 불편하기 때문에 FK제약조건을 꺼림

### 안티패턴: 인식방법
- 어떤 값이 변경/추가할때 참조정합성을 확인하기 위한 방법을 확인

### 안티패턴: 사용 합당한 경우

### 해결: 제약조건 선언
#### 여러 테이블 변경 지원
```sql
CREATE TABLE Bugs (
  ...
  reported_by   BIGINT UNSIGNED NOT NULL,
  status        VARCHAR(20) NOT NULL DEFAULT 'NEW',
  FOREIGN KEY (reported_by) REFERENCES Accounts(account_id)
    ON UPDATE CASCADE
    ON DELETE RESTRICT,  -- Bugs 행이 참조하는 한 삭제 불가
  FOREIGN KEY (status) REFERENCES BugStatus(status)
    ON UPDATE CASCADE
    ON DELETE SET DEFAULT -- 상태는 디폴트 값으로 셋팅
);
```

