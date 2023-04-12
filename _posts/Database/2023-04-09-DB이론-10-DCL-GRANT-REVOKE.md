---
title: "[DB이론] 10. DCL : GRANT, REVOKE"
date: 2023-04-09 16:09:00 +0900
categories: [Database]
tags: [DB이론, Oracle]
---

## DCL

- DCL : Data Control Language
- 사용자(USER)의 DB 또는 DB객체에 대한 접근 권한을 부여(GRANT)하고 회수(REVOKE)하는 언어

<br>

## 계정 구분

| **관리자 계정** | DB의 생성과 관리를 담당하는 계정으로, 모든 권한과 책임을 가짐<br>ex. SYS(최고관리자), SYSTEM(SYS에서 권한 몇 개 제외) |
| **사용자 계정** | 데이터베이스의 질의, 갱신, 보고서 작성 등의 작업을 수행할 수 있는 계정<br>원칙상 업무에 필요한 최소한의 권한만 가짐 |

<br>

## 권한의 종류

### 시스템 권한
- DB접속, 객체 생성
- 시스템 권한은 관리자 계정 또는 관리자 권한이 있는 계정이 부여할 수 있다.

| 권한 종류 | 설명 |
| --- | --- |
| CRETAE SESSION | 데이터베이스 접속 권한 |
| CREATE TABLE | 테이블 생성 권한 |
| CREATE VIEW | 뷰 생성 권한 |
| CREATE SEQUENCE | 시퀀스 생성 권한 |
| CREATE PROCEDURE | 함수(프로시져) 생성 권한 |
| CREATE USER | 사용자(계정) 생성 권한 |
| DROP USER | 사용자(계정) 삭제 권한 |
| DROP ANY TABLE | 임의 테이블 삭제 권한 |

### 객체 권한
특정 객체를 조작할 수 있는 권한

| 권한 종류 | 설정 객체 |
| --- | --- |
| SELECT | TABLE, VIEW, SEQUENCE |
| INSERT | TABLE, VIEW |
| UPDATE | TABLE, VIEW |
| DELETE | TABLE, VIEW |
| ALTER | TABE, SEQUENCE |
| REFERENCES | TABLE |
| INDEX | TABLE |
| EXECUTE | PROCEDURE |

<br>

## 시스템 권한 부여/회수

스템 권한은 관리자 계정 또는 관리자 권한이 있는 계정이 부여할 수 있다.

1. SQL 구문 작성 형식을 일반 형식(12c버전 이전 형식)으로 변경
    
    ```sql
    ALTER SESSION SET "_ORACLE_SCRIPT" = TRUE;
    ```
    
2. 사용자 계정 생성
    
    ```sql
    CREATE USER 계정명 IDENTIFIED BY 비밀번호; -- 대소문자 구분 안함
    CREATE USER 계정명 IDENTIFIED BY "비밀번호"; -- 대소문자 구분됨
    ```
    
    > **ORA-01045: 사용자 "계정명"는 CREATE SESSION 권한을 가지고있지 않음; 로그온이 거절되었습니다**<br>
    > 계정 생성 후 바로 접속 불가<br>
    > 👉 관리자 계정에서 계정에 접속 권한(CREATE SESSION) 부여 필요
    {: .prompt-danger }
    
3. 사용자 계정에 권한 부여
    
    ```sql
    GRANT 권한[, 권한,...] TO 계정명;
    ```
    > **ORA-01031: 권한이 불충분합니다**<br>
    > CREATE SESSION 권한으로는 DB 접속만 가능<br>
    > 👉 관리자 계정에서 관련 권한 부여 필요
    {: .prompt-danger }

4. 특정 사용자에게 부여된 시스템 권한 확인
    
    ```sql
    SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE = '사용자명';
    ```
    
5. 객체 생성 공간 할당
    
    ```sql
    ALTER USER 사용자명
    DEFAULT TABLESPACE 테이블스페이스명  -- default tablespace 지정
    QUOTA UNLIMITED ON 테이블스페이스명; -- tablesapce 용량 할당(무제한, 기본값20M)
    ```

6. 시스템 권한 회수
    
    ```sql
    REVOKE 권한명 FROM 사용자명;
    ```

<br>    

## 객체 권한 부여 / 회수

1. 객체 권한 부여
    
    ```sql
    GRANT 권한[, 권한,...] ON 객체명 TO 계정명;
    ```
    
    > **ORA-00942: 테이블 또는 뷰가 존재하지 않습니다**<br>
    > 자신이 생성하지 않은 객체에는 기본적으로 접근할 수 없다.<br>
    > (권한이 없어서 다른 계정의 테이블이 보이지 않음)<br>
    > 👉 접근하려는 객체의 주인이(또는 관리자가) 권한 부여
    {: .prompt-danger }

2. 객체 권한 회수
    
    ```sql
    REVOKE 권한[, 권한,...] ON 객체명 FROM 계정명; 
    ```
    
<br>

## ROLE

### 사용 목적

1. 여러 권한을 하나로 묶어두는 용도
    > ex. RESOURCE : DB사용을 위한 기본 객체 생성 권한을 묶어둔 것<br>
    > RESOURCE 보유 권한(8개) : CREATE SEQUENCE, CREATE PROCEDURE, CREATE CLUSTER, CREATE INDEXTYPE, CREATE OPERATOR, CREATE TYPE, CREATE TRIGGER, CREATE TABLE
    
2. 어려운 권한명(SESSION 등)을 쉽게 사용할 수 있도록 변경
    > ex. CONNECT : CREATE SESSION(DB 접속) 권한 하나만 가진 ROLE
    
### 특정 ROLE이 가지고 있는 권한 목록 확인

```sql
SELECT * FROM ROLE_SYS_PRIVS WHERE ROLE = '역할이름';
```

### 특정 사용자에게 부여된 ROLE 확인

```sql
SELECT * FROM DBA_ROLE_PRIVS WHERE GRANTEE = '사용자명';
```

### 일반적으로 사용자 계정을 생성할 때 부여하는 권한

```sql
GRANT CONNECT, RESOURCE, CREATE VIEW TO 계정명;
```