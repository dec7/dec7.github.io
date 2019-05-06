---
layout: post
title: "SQL Anti Patterns Ch12"
description: "SQL Anti patterns"
date: 2019-05-06
tags: [sql,anti,pattern]
comments: true
share: true
---

## 유령파일 
- 이미지 저장 

### 목표: 이미지 또는 벌크 미디어 저장 
- 이미지를 저장하고, 이를 사용자 계정이나 버그와 같은 데이터베이스 연티티와 연관을 갖게 하는 것 

### 안티패턴: 파일을 사용해야한다고 가정 
- 개념적으로 이미지는 테이블 속성 
```sql
CREATE TABLE Accounts (
    account_id  SERIAL PRIMARY KEY,
    account_name    VARCHAR(20),
    portrait_image  BLOB
);

CREATE TABLE Screenshots (
    bug_id  BIGINT UNSIGNED NOT NULL,
    image_id    SERIAL NOT NULL,
    screenshot_image    BLOB,
    caption     VARCHAR(100),
    PRIMARY KEY (bug_id, image_id),
    FOREIGN KEY (bug_id) REFERENCES Bugs (bug_id)
);
```
- 이미지 경로만 저장 
```sql
CREATE TABLE Screenshots (
    bug_id      BIGINT UNSINGED NOT NULL,
    image_id    BIGINT UNSIGNED NOT NULL,
    screenshot_path VARCHAR(100),
    caption     VARCHAR(100),
    PRIMARY KEY (bug_id, image_id),
    FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id)
);
```

#### delete 문제 
- 이미지가 외부 경로에 있는 경우, 엔티티를 삭제하더라도 이미지가 그대로 유지됨 

#### 트랜젝션 문제 
- 데이터를 업데이트, 삭제 할때 commit 으로 트랜잭션을 끝내기 전까지 변경사항이 다른 클라이언트에게 보이지 않음 
- 하지만, 외부 저장시 트랜젝션 활용 불가 

#### 롤백 문제 
- 외부에 젖아시 파일 삭제 후 롤백하더라도 돌아오지 않음 

#### 백업 문제 

#### sql접근 권한 문제 
- 외부 파일에 대한 접근권한 처리 불가 

#### sql 데이터 타입 문제 
- 이미지를 경로로 지정ㅇ할 경우, 유요한 경로인지, 실제 파일이 있는지 여부를 검증할 수 없음. 
- 오직 어플리케이션 코드에 의존해야함 


### 안티패턴 인식 방법 
- 데이터 백업과 복원 절차, 백업을 검증할 수 있는 방법 
- 불필요한 이미지가 계속 쌓이는지 아니면 삭제되는지, 
- 어떤 사용자가 이미지를 볼 수 있는 권한이 있는지 
- 이미지에 대한 변경을 취소할 수 있는지. 혹은 이전 상태로 변경이 필요한지 

### 안티패턴 사용이 합당한 경우 
- db를 가볍게 유지해야할 경우 

### 해법: 필요한 경우 blob 데이터 타입을 사용 
```sql
CREATE TABLE Screenshots (
    bug_id  BIGINT UNSIGNED NOT NULL,
    image_id    BIGINT UNSIGNED NOT NULL,
    screenshot_image    BLOB,
    caption     VARCHAR(100),
    PRIMARY KEY (bug_id, image_id),
    FOREIGN KEY (bug_id) REFERENCES Bugs(bug_id)
);
```

```sql
UPDATE Screenshots SET screenshot_image = LOAD_FILE('images/screenshot1234-1.jpg')
WHERE bug_id = 1234 AND image_id = 1;

SELECT screenshot_iamge
INTO DUMPFILE 'image/screenshot1234-1.jpg'
FROM Screenshots
WHERE bug_id = 1234 AND image_id = 1;
```





