---
title: "[DB이론] 06. TCL : COMMIT, ROLLBACK, SAVEPOINT"
date: 2023-04-09 01:18:00 +0900
categories: [Database]
tags: [DB이론, Oracle]
---

## TRANSACTION
- 데이터베이스의 논리적 연산 단위
- 데이터 변경 사항을 묶어 하나의 트랜잭션에 담아 처리
- 트랜잭션의 대상이 되는 데이터 변경 사항 : DML (`INSERT`, `UPDATE`, `DELETE`)

### 트랜잭션 수행 과정

| INSERT 수행      | → | 트랜잭션에 추가          |   |           | → | DB 반영 ❌ |
| INSERT 수행      | → | 트랜잭션에 추가          | → | COMMIT    | → | DB 반영 ⭕ |
| INSERT * 10 수행 | → | 단일트랜잭션에 10개 추가 | → |  ROLLBACK | → | DB 반영 ❌ |

<br>

## TCL : 트랜잭션 제어 언어
TCL : Transaction Control Language

| COMMIT    | 메모리 버퍼(트랜잭션)에 임시 저장된 데이터 변경 사항(DML)을 DB에 반영            |
| ROLLBACK  | 메모리 버퍼(트랜잭션)를 비우고 마지막 COMMIT 상태로 복귀                         |
| SAVEPOINT | 메모리 버퍼(트랜잭션)에 저장 지점을 정의하여 ROLLBACK 수행 시 특정 지점으로 복귀 |

### SAVEPOINT 사용법
```sql
SAVEPOINT 포인트명1;
...(DML 수행)...
SAVEPOINT 포인트명2;
...(DML 수행)...
ROLLBACK TO 포인트명1; -- 포인트1 지점 이후 데이터 변경사항 삭제
```