---
layout: post
title: "SQL Anti Patterns Ch17"
description: "SQL Anti patterns"
date: 2019-05-12
tags: [sql,anti,pattern]
comments: true
share: true
---

## 가난한 자의 검색 엔진 
- 문서 집합이 커지면 검색기능이 필요해짐
- 문서를 카테고리로 구분하더라도 규모가 큼
- 검색어를 입력하면 단어가 포함된 문서를 표현하는게 가장 직관적인 인터페이스 

### 목표: 목표: 전체 텍스트 검색 
- SQL (관계형 이론)의 기본원리중 하나는 칼럼에 들어 있는 값이 원자적이어야 한다는 것 (전체값과 비교해야한 다는 것)
- SQL에서 부분문자열 비교는 비효율적이거나 부정확 

### 안티패턴: 패턴매칭 사용 
- SQL은 문자열 비교를 위해 패턴매칭 기능을 제공 
    - LIKE
    - LIKE 연산자는 0개 이상의 문자와 매치되는 와일드카드를 지원 
```sql
SELECT * FROM Bugs WHERE description LIKE '%crash%';

SELECT * FROM Bugs WHERE description REGEXP 'crash';

SELECT * FROM Bugs WHERE description REGEXP '[[:<:]]one[[:>:]]';
```
- 단점
    - 패턴매칭의 단점은 성능이 나쁘다는 것 
        - 인덱스 확용 불가, 풀스캔 
    - 결과가 부정확 할 수 있음 
        - 단어 경계를 구분할 수 없음, 단어 경계를 위한 특별 정규식을 포함하더라도 성능/확장적응성에 문제를 일으킴

### 안티패턴 인식 방법 
- like를 사용할때, 두 와일드 카드 사이에 변수를 넣을 수 있을지 
- 특정 문자열을 포함하거나, 포함하고 싶지 않을 때 
- 쿼리 성능이 느릴 때 

### 안티패턴 사용이 합당한 경우 
- 사용된 쿼리는 모두 문제가 없고, 사용도 간편
- 성능도 중요하지만, 최적화하기엔 불필요할 수도 있음 (비지니스 요구사항 상 )

### 해법: 작업에 맞는 올바른 도구 사용하기 
- sql대신 특화된 검색엔진을 쓰는게 제일 좋음 
- 대안은 검색 결과를 저장해 반복되는 비율 줄이기 

#### 벤더 확장 기능 
- 제품간 전체 텍스트 검색 요구에 대응하기 위해 각자의 해법을 찾음, 하지만 표준이 아니고, 제품간 호환성도 없음 

##### mysql에서 전체 텍스트 검색 
- MySQL 에서는 MyISAM 스토리지 엔진에서만 간단한 전체 텍스트 인덱스 타입을 제공 
    - CHAR, VARCHAR, TEXT 타입의 칼럼에 전체 엑스트 인덱스를 정의할 수 있음 
    - 인덱스가 걸린 텍스트에서 키워드를 검색할 때 MATCH 함수를 사용하고, 인덱스 칼럼을 지정해야함

```sql
ALTER TABLE Bugs ADD FULLTEXT INDEX bugfts (summary, description);

SELECT * FROM Bugs WHERE MATCH(summary, description) AGAINST ('crash');

-- up to mysql4.1
SELECT * FROM Bugs WHERE MATCH(summary, description) AGAINST ('+crash -save' IN BOOLEAN MODE);
```

##### oracle 에서 텍스트 인덱싱 
- Oracle8 부터 텍스트 인덱싱 기능을 제공, ConText 라는 데이터 카드리지의 일부였고, 지금은 DB SW에 통합됨 
- Oracle의 텍스트 인덱싱 기술은 복잡/풍부 

###### CONTEXT
- 하나의 텍스트 칼럼에 대해서 이 타입의 인덱스를 생성 
- 이 인덱스를 사용하는 검색에서는 CONTAINS() 연산자 사용 
- 이 인덱스는 데이터가 변경되어도 일관적인 상태를 유지하지 않으므로 스케쥴을 걸어 주기적으로 재구성해야줘야함
```sql
CREATE INDEX BugsText ON Bugs(summary) INDEXTYPE IS CTSSYS.CONTEXT;
SELECT * FROM Bgus WHERE CONTAINS(summary, 'crash') > 0;
```

###### CTXCAT
- 이 인덱스는 짧은 텍스트 샘플과 동일 테이블의 다른 칼럼을 함께 사용하는데 특화됨 
- 이 인덱스는 데이터가 변경되어도 일관적인 상태를 유지 

```sql
CTX_DDL.CREATE_INDEX_SET ('BugsCatalogSet');
CTX_DDL.ADD_INDEX('BugsCatalogSet', 'status');
CTX_DDL.ADD_INDEX('BugsCatalogSet', 'priority');

CREATE INDEX BugsCatalog ON Bugs(summary)
    INDEXTYPE IS CTSSYS.CTXCAT PARAMETERS ('BugsCatalogSet');

SELECT * FROM Bugs
WHERE CATSEARCH(summary, '(crash save)', 'status = "NEW"') > 0;
```


###### CTXXPATH
- existsNode() 연산자로 XML 문서를 검색하는 데 특화됨 
```sql
CREATE INDEX BugTextXml ON Bugs(testoutput) INDEXTYPE IS CTSSYS.CTXXPATH;

SELECT * FROM Bugs
WHERE testoutput.existsNode('/testsuite/test[@status="fail"]') > 0;
```

###### CTXRULE
- 문서를 분석해 분류에 규칙을 설계할 수 있음 

#### 서드파티 검색엔진 
- DB 제품에 관계없이 동일한 방식으로 텍스트를 검색해야할 경우, SQL DB와 독립적으로 실행되는 검색엔진을 사용해야함 
    - Sphinex, Apache Lucene 




