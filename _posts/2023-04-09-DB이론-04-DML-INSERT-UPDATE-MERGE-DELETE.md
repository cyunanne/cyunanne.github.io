---
title: "[DB이론] 04. DML : INSERT, UPDATE, MERGE, DELETE"
date: 2023-04-09 00:04:00 +0900
categories: [Database]
tags: [DB이론, Oracle]
---

## DML
- DML(Data Manipulation Language) : 데이터 조작 언어
- 테이블에 값을 삽입하거나(INSERT), 수정하거나(UPDATE), 삭제(DELETE)하는 구문

<br>

## INSERT
테이블에 새로운 행을 추가하는 구문

### **모든** 컬럼에 값 삽입
컬럼명은 생략 가능하나 VALUES에 값을 기입할 때 컬럼 순서를 지켜야 한다.
```sql
INSERT INTO *MY_TABLE*[(*컬럼명, ...*)] VALUES(*값, ...*);
```

### **일부** 컬럼에만 값 삽입
선택되지 않은 컬럼에는 `NULL`이 삽입된다.
⇒ NOT NULL 제약조건이 걸린 컬럼에 값을 삽입하지 않으면 에러 발생
```sql
INSERT INTO 테이블명(컬럼명, ...) VALUES(값, ...);
```
    
### SUBQUERY를 이용한 값 삽입
```sql
INSERT INTO 테이블명[(컬럼명, ...)] 서브쿼리;
```
    
<br>

## UPDATE
- 컬럼값을 수정하는 구문
- 여러 컬럼을 한번에 수정할 시 콤마(,)로 구분

```sql
UPDATE 테이블명 SET 컬럼명 = 바꿀값, ... [WHERE 조건문];
```

> 조건절을 설정하지 않고 UPDATE 구문 실행 시 모든 행의 컬럼 값이 변경되므로 주의한다.
{: .prompt-danger }

### SUBQUERY를 이용한 값 갱신
    
```sql
UPDATE 테이블명 SET 컬럼명 = (서브쿼리);        -- 스칼라 서브쿼리
UPDATE 테이블명 SET (컬럼명, ...) = (서브쿼리); -- 다중행 다중열 서브쿼리
```
    

<br>

## MERGE
- 구조가 같은 두 개의 테이블을 하나로 **병합**하는 구문
- 테이블에서 지정하는 조건의 값이 존재하면 **`UPDATE`**, 없으면 **`INSERT`**가 수행된다.

```sql
MERGE INTO 베이스테이블 USING 병합할테이블 ON(베이스테이블.컬럼명 = 병합할테이블.컬럼명)
WHEN MATCHED THEN
UPDATE SET
베이스테이블.컬럼명1 = 병합할테이블.컬럼명1,
베이스테이블.컬럼명2 = 병합할테이블.컬럼명2,
...
WHEN NOT MATCHED THEN
INSERT VALUES (베이스테이블.컬럼명1, 병합할테이블.컬럼명2, ...);
```

<br>

## DELETE
테이블의 행을 삭제하는 구문

```sql
DELETE FROM 테이블명 WHERE 조건문;
```

> WHERE 조건을 설정하지 않으면 모든 행이 다 삭제되므로 주의해서 사용한다.
{: .prompt-danger }

### cf. TRUNCATE (DDL⭕ DML❌)

- 테이블의 **전체 행**을 삭제하는 **DDL**
- DELETE보다 수행속도가 더 빠르다.

> DDL이므로 수행 시 ROLLBACK으로 복구할 수 없으므로 사용 시 각별한 주의 요함.
{: .prompt-danger }