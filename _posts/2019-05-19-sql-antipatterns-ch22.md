---
layout: post
title: "SQL Anti Patterns Ch22"
description: "SQL Anti patterns"
date: 2019-05-19
tags: [sql,anti,pattern]
comments: true
share: true
---

## 가상키 편집증 
- id를 순차적으로 유지하길 원할때 

### 목표: 데이터 정돈하기 

### 안티패턴: 모든 틈 메우기 
#### 시퀀스에서 벗어난 번호 할당하기 
- 가상키 자동생성 메커니즘을 사용하지 않고, PK값중 사용되지 않은 첫번째 값을 구하여 사용 
    - 불필요한 셀프조인 진행 필요
```sql
SELECT b1.bug_id + 1
FROM Bugs b1
LEFT OUTER JOIN Bugs AS b2 ON (b1.bug_id + 1 = b2.bug_id)
WHERE b2.bug_id IS NULL
ORDER BY b1.bug_id LIMIT 1;
```
- 동시성 문제 발생 
- 비효율적이메 에러 확률 높음 

#### 기존행 번호 다시 매기기 
- 새로운 행을 추가할때, 사용되지 않은 키를 찾고, update문 실행
    - 많을 수록 많은 실행 필요 
    - 값을 전파하기 위해 ON UPDATE CASCADE 옵션 사용했다면 좀 편하겠지만, 아니라면 제약조건 비활성 필요 
- 수정했더라도, 일시적이며 계속 발생할 것 

#### 데이터 불일치 만들기 
- 가상키에 빈값이 있다고 하더라도, 이미 참조하는 칼럼이 있을수 있으므로 그 키를 재할당하면 안됨 

#### 안티패턴 합당 경우
- 없음 

#### 해법: 극복하라
- pk값은 유일하고 null이 아니어서 참조할 수 있으면 됨, 숫자일 필요는 없음 

##### 행에 번호 매기기
- 행번호과 pk는 다른 의미 
- 행 번호는 쿼리 결과의 부분집합만 반환하는 페이지 처리일 경우 그러함 
- windowing function
    - oracle에서는 analytic function 이라고 함 
    - row_numver()

##### guid 사용하기 
- 일반적인 가상키 사용보다 2가지 장점 있음
    - 중복을 걱정하지 않고, 여러 데이터 베이스에서 동시에 가상키 생성 가능
    - 아무도 틈에 대해 불평하지 않음 
    - 단점
        - 값이 너무 길고 복잡 
        - 랜덤하기 때문에 패턴을 추론할 수 없음 
        - 16바이트 필요 



