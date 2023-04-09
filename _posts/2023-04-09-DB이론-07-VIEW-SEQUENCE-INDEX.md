---
title: "[DB이론] 07. VIEW, SEQUENCE, INDEX"
date: 2023-04-09 02:45:00 +0900
categories: [Database]
tags: [DB이론, Oracle]
---

> VIEW, SEQUENCE, INDEX 의 생성/수정/삭제 구문은 [09. DDL : CREATE, ALTER, DROP](/posts/DB이론-09-DDL-CREATE-ALTER-DROP/)을 참조한다.
{: .prompt-tip }

## VIEW
- **논리적 가상 테이블**로, RESULT SET을 저장하는 객체
- 테이블과 흡사하지만 실제 데이터가 저장되어 있는 것은 아니다.

### 사용목적
1. 복잡한 SELECT문을 쉽게 재사용
2. 테이블의 실제 모습을 감출 수 있어 보안상 유리

### 사용 시 주의사항
1. 실체가 없는 가상 테이블이므로 ATLER 구문 사용 불가
2. 대부분 조회(SELECT) 용도로 사용
> VIEW를 이용한 DML(INSERT, UPDATE, DELETE)에 제약이 많이 따르기 때문

### VIEW에서 DML 작업 수행
뷰를 이용해서 DML 작업을 수행하면 **원본 테이블이 변경**되며, 이에 따른 오류가 발생할 수 있다.<br>
예를 들어, INSERT 구문 수행 시 VIEW를 통해 확인할 수 없는 컬럼에는 자동으로 NULL 값이 삽입된다. 그 중 NOT NULL 제약조건이 설정된 컬럼이 있다면 오류가 발생하며 INSERT를 완료할 수 없게된다.<br>
👉 **VIEW에서 DML 수행 지양**

> **ORA-01400: NULL을 ("유저명"."테이블명"."컬럼명") 안에 삽입할 수 없습니다**<br>
> NOT NULL 제약조건이 설정된 컬럼에 NULL 삽입 불가<br>
{: .prompt-danger }
> 특별한 이유가 없는 한 데이터 무결성을 위해 대부분의 컬럼에 NOT NULL 제약조건을 추가하게 되므로  VIEW에서는 INSERT가 거의 불가능하다.

<br>

## SEQUENCE
- 일정한 간격의 숫자(번호)를 발생시키는 객체
- 활용 : PRIMARY KEY가 지정된 컬럼에 삽입할 값을 생성

### SEQUENCE 사용

```sql
-- 현재 시퀀스넘버 확인
SELECT 시퀀스명.CURRVAL FROM DUAL;

-- 컬럼에 다음 시퀀스넘버 삽입
UPDATE 테이블명 
SET 컬럼명 = 시퀀스명.NEXTVAL 
WHERE 조건절; 
```

- `시퀀스명.NEXTVAL`
  - 다음 시퀀스 번호(INXREMENT BY 만큼 증가한 값) 반환
  - 최초 호출 시 START WITH에 작성된 값 반환
  - MAXVALUE 도달 시 NAXTVAL 호출할 경우 오류 발생
  
  > **ORA-08004: 시퀀스 시퀀스명.NEXTVAL exceeds MAXVALUE은 사례로 될 수 없습니다**<br>
  > SEQUENCE가 MAXVALUE에 도달하면 NEXTVAL 호출 불가
  {: .prompt-danger }

- `시퀀스명.CURRVAL`
  - 현재 시퀀스 번호(마지막으로 반환된 NEXTVAL 값) 반환
  - 시퀀스가 생성되자마자 (NEXTVAL을 호출하지 않고) 호출할 경우 오류 발생

  > **ORA-08002: 시퀀스 시퀀스명.CURRVAL은 이 세션에서는 정의 되어 있지 않습니다**<br>
  > SEQUENCE 생성 직후 CURRVAL 호출 불가
  {: .prompt-danger }


<br>

## INDEX

- **SELECT의 처리 속도를 향상**시키기 위해 컬럼에 대하여 생성하는 객체
- 인덱스 내부 구조는 B* 트리 형식으로 되어있다.
- `UNIQUE INDEX` : 중복값이 없는 컬럼에 UNIQUE INDEX를 생성하면 **더 빠른 속도**로 검색할 수 있다.
- PK 또는 UNIQUE 제약 조건이 설정된 컬럼에 대해 **UNIQUE 인덱스가 자동 생성**된다.

### INDEX 사용의 장단점

| 장점 | 이진트리 형식으로 구성                                          | → | 자동정렬, 검색속도 증가 |
|      | 조회 시 테이블의 전체 내용이 아닌 인덱스 컬럼만을 이용해서 조회 | → | 시스템 부하 감소 ⬇️      |
| 단점 | 데이터변경(DML) 시 이진트리 구조 변경 필요                      | → | 시스템 부하 증가 ⬆️      |
|      | 인덱스도 하나의 객체이므로 별도 저장 공간이 필요                | → | 추가 저장 공간          |
|      | 인덱스 생성시간 필요                                            | → | 추가 소요 시간          |
  
> SELECT ⟩⟩⟩⟩⟩⟩ INSERT/UPDATE/DELETE 인 테이블에 사용하면 성능적 이점을 볼 수 있다.
{: .prompt-info }

### INDEX 사용
WHERE 절에 인덱싱 된 컬럼을 언급
```sql
SELECT * FROM 테이블명
WHERE 인덱스컬럼 = '0'; -- 항상 TRUE
```