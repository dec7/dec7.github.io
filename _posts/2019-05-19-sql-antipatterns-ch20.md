---
layout: post
title: "SQL Anti Patterns Ch20"
description: "SQL Anti patterns"
date: 2019-05-19
tags: [sql,anti,pattern]
comments: true
share: true
---

## 읽을 수 있는 패스워드 
### 목표: 패스워드를 복구하거나 재설정하기 
### 안티패턴: 패스워드를 평문으로 저장하기 
#### 패스워드 저장
```sql
CREATE TABLE Accounts (
    account_id  SERIAL PRIMARY KEY,
    account_name    VARCHAR(20) NOT NULL,
    email       VARCHAR(100) NOT NULL,
    password    VARCHAR(30) NOT NULL,
);

INSERT INTO Accounts (account_id, account_name, email, password)
VALUES (123, 'billkarwin', 'bill@example.com', 'xyzzy');
```

- 평문상태로 저장, 평문상태로 네트워크를 통해 전달하는 것은 안전하지 않음 
    - 공격자가 sql문을 가로채서 읽을 수 있음 
- 문제점
    - 네트워크 패킷을 가로채 sql문을 확인할 수 있음 
    - db서버에서 sql 쿼리 로그를 검색할 수 있음 
    - 서버나 백업 미디어에 저장된 db백업 파일로부터 데이터 읽을 수 있음 

#### 패스워드 인증 
- 사용자가 로그인을 시도할 때, 어플리케이션은 사용자의 입력과 데이터베이스에 저장된 패스워드 문자열을 비교
```sql
SELECT CASE WHEN password = 'opensesame' THEN 1 ELSE 0 END AS password_matches
FROM Accounts
WHERE account_id = 123;
```
- 인증시 패스워드를 평분으로 보내면, 공격자에게 노출될 위험이 증가함 
- account_id, password 를 모두 조건절에 넣는 경우 
    - 두가지 인증실패 원인을 구분할 수 없음
    - 공격시도 여부를 판단하기 위한 조건이 될 수 있음 

#### 이메일로 패스워드 보내기 
- 패스워드를 평문으로 담은 이메일을 보낼 경우, 해커가 이메일을 가로챘을때 노출될 수 있음

### 안티패턴 인식 방법 
- 어플리케이션이 패스워드를 평문으로 저장하거나, 역변환할 수 있은 암호화 기법을 사용하는 경우

### 안티패턴 사용이 합당한 경우 
- 어플리케이션에서 패스워드를 사용해 다른 서드파티 서비스에 접근할 수 있고, 이 경우 패스워드를 읽을 수 있는 형식으로 저장해야함 
    - identification 신원확인, authentication 인증
    - 신원확인은 자신이 누구라고 주장할 때 그 사람이 맞는지 증명하는 것 
    - 패스워드는 인증에 사용되는 흔한 방법 
- 충분히 강력한 보안을 강제할 수 없으면, 인증매커니즘은 있지만, 신뢰할 수 있는 신원확인 메커니즘이 없는 것
    - 신원확인 메커니즘은 꼭 필요한 것은 아님
    - 모든 프로그램이 공격위험이 있는 것도 아니고, 보호할 만큼의 민감한 정보를 가지고 있는 것도 아님
- 단순한 로그인 설계도 문제 없을 수 있으나, 시스템은 진화하기 마련 

### 해법: 패스워드에 소금 친 해시값을 저장 
- 패스워드를 읽을 수 있는 형태로 저장하는게 문제 

#### 해시함수 
- 단방향 해시함수를 사용하여 패스워드를 부호화 
    - 해시함수는 입력 문자열을 해시라 불리는 알아볼수 없는 문자열로 변환 
    - sha256 알고리즘의 경우, 64의 16진수 문자열이 나옴 
    - 역을 구하기가 어려움 

#### sql에서 해시 사용하기 
```sql
CREATE TABLE Accounts (
    account_id  SERIAL PRIMARY KEY,
    account_name    VARCHAR(20),
    email       VARCHAR(100) NOT NULL,
    password_hash   CHAR(64) NOT NULL
);

INSERT INTO Accounts (account_id, account_name, email, password_hash)
VALUES (123, 'billkarwin', 'bill@example.com', SHA2('xyzzy'));

SELECT CASE WHEN password_hash = SHA2('xyzzy') THEN 1 ELSE 0 END
    AS password_matches
FROM Accounts
WHERE account_id = 123;
```

#### 해시에 소금 추가하기 
- 해시값을 저장했더라도, 공격자가 db에 접근한 후 레인보우(dictionary attack) 어택 가능
- 패스워드를 부호화할떄 salt처리 
    - salt는 해시값을 구하기 전에 패스워드에 덧붙이는 무의미한 바이트열 
```sql
CREATE TABLE Accounts (
    account_id  SERIAL PRIMARY KEY,
    account_name    VARCHAR(20),
    email       VARCHAR(100) NOT NULL,
    password_hash   CHAR(64) NOT NULL,
    salt        BINARY(8) NOT NULL
);

INSERT INTO Accounts (account_id, account_name, email, password_hash, salt)
VALUES (123, 'billkarwin', 'bill@example.com', SHA2('xyzzy'||'-OxT!sp9'), '-OxT!sp9');

SELECT (password_hash = SHA2('xyzzy'||'-OxT!sp9')) AS password_matches
FROM Accounts
WHERE account_id = 123;
```

#### sql에서 패스워드 숨기기 
- 패스워드 저장시 해시함수, 사전공격을 막기위한 솔트 사용
- sql문장에서는 여전히 패스워드가 평문으로 노출되고 있음 
- 어플리케이션에서 해시처리하여 저장하면 막을 수 있음 
- 웹 어플리케이션의 경우 사용자 브라우저와 웹 어플리케이션 서버 사이에서 데이터를 가로챌수 있음
    - 사용자가 로그인 폼을 제출할 때 브라우저는 평문 형태의 패스워드를 포함 
    - https 와 같은 보안 프로토콜을 사용하는게 좋음 

#### 패스워드 복구가 아닌 패스워드 재설정 사용 
- 사용자가 비밀번호를 잊어버렸을때
    - db에는 해시값으로 저장되었으므로 원래의 값을로 되돌릴 수 없음 
- 대안
    1. 이메일로 임시 비밀번호 전송, 만료기능 추가, 반드시 변경하도록 강제 
    2. 이메일로 임시 비밀번호 전송, 요청을 db에 기록하고 id로 유일한 토큰을 생성 , 토큰을 발송, 낯선사람이 요청해도 메일은 원소유자에게만 전달 

```sql
CREATE TABLE PasswordResetRequest (
    token       CHAR(32) PRIMARY KEY,
    account_id  BIGINT UNSIGNED NOT NULL,
    expiration  TIMESTAMP NOT NULL,
    FOREIGN KEY (account_id) REFERENCES Accounts(account_id)
);
SET @token = MD5('billkarwin' || CURRENT_TIMESTAMP || RAND());

INSERT INTO PasswordResetRequest (token, account_id, expiration)
VALUES (@token, 123, CURRENT_TIMESTAMP + INTERVAL 1 HOUR);
```