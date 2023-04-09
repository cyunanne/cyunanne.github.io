---
title: "[DB이론] 09. DDL : CREATE, ALTER, DROP"
date: 2023-04-09 15:29:00 +0900
categories: [Database]
tags: [DB이론, Oracle]
---

## DDL
- DDL : Data Definition Language, 데이터 정의 언어
- 객체의 생성(`CREATE`), 수정(`ALTER`), 삭제(`DROP`) 등 데이터의 전체 구조(스키마, schema)를 정의하는 언어로, 주로 DB관리자나 설계자가 사용한다.
- 오라클에서의 객체 : 테이블(TABLE), 뷰(VIEW), 시퀀스(SEQUENCE), 사용자(USER), 인덱스(INDEX), 패키지(PACKAGE), 트리거(TRIGGER), 프로시져(PROCEDURE), 함수(FUNCTION), 동의어(SYNONYM)

<br>

## CREATE : 객체 생성

### TABLE 생성

```sql
CREATE TABLE 테이블명 (
	컬럼명 자료형(크기) 
        [DEFUALT 값]                        -- DEFAULT 값 설정
        [CONSTRAINTS 제약조건명]            -- 제약조건 이름 설정
        [NULL | NOT NULL]                   -- NOT NULL 제약조건 설정 (NULL : 기본값, 생략가능)
        [PRIMARY KEY | UNIQUE ]             -- 기본키, UNIQUE 제약조건 설정 (컬럼레벨)
        [REFERENCES 참조테이블[(참조컬럼)]] -- 외래키 설정 (컬럼레벨)
        [CHEKCH(조건식)],                   -- CHECK 제약조건 설정 (컬럼레벨)

	[PRIMARY KEY(컬럼명, 컬럼명, ...),]                      -- 기본키 제약조건 설정(테이블레벨)
    [UNIQUE(컬럼명, 컬럼명, ...),]                           -- UNIQUE 제약조건 설정 (테이블레벨)
    [FOREIGN KEY(컬럼명) REFERENCES 참조테이블[(참조컬럼)],] -- 외래키 설정 (테이블레벨)
    [CHECK(조건식),]                                         -- CHECK 제약조건 설정 (테이블레벨)
    ...);
```

- 테이블
    - 행(row)과 열(column)으로 구성되는 가장 기본적인 데이터베이스 객체
    - 데이터 배이스 내에서 모든 데이터는 테이블을 통해서 저장된다.

- 자료형
    
    | NUMBER         | 숫자형(정수, 실수)            |   |                                    |
    | CHAR(크기)     | 고정길이 문자형 (최대 2,000B) | → | 선언한 크기만큼의 메모리 공간 유지 |
    | VARCHAR2(크기) | 가변길이 문자형 (최대 4,000B) | → | 불필요한 메모리 공간 반납          |
    | DATE           | 날짜 타입                     |   |                                    |
    | BLOB           | 대용량 이진 데이터 (최대 4GB) |   |                                    |
    | CLOB           | 대용량 문자 데이터 (최대 4GB) |   |                                    |

- 데이터 크기(UTF-8)

    | 데이터 형식          | 크기     |
    | -------------------- | :------: |
    | 영문, 숫자, 특수문자 | 1 바이트 |
    | 한글, 한자           | 3 바이트 |

- DEFUALT
  + 필드의 기본값을 정의하는 키워드
  + DEFAULT는 제약조건보다 먼저 정의해야 한다.
  + `INSERT` 또는 `UPDATE` 수행 시 컬럼의 **값을 입력하지 않거나, `DEFAULT` 키워드를 전달**하면 미리 설정해 둔 기본값이 컬럼에 반영된다.

    ```sql
    -- #1. DEFAULT 키워드 입력
    INSERT INTO 테이블명 VALUES(..., **DEFAULT**);

    -- #2. 해당 필드 미입력
    INSERT INTO 테이블명(컬럼명1, 컬럼명2, ...)
    VALUES('컬럼값1', '컬럼값2', ...);
    ```

- 제약조건 : [08. CONSTRAINTS](/posts/DB이론-08-CONSTRAINTS/) 참조



<br>

### VIEW 생성

```sql
CREATE [OR REPLACE] [FORCE|NOFORCE] VIEW 뷰이름 [컬럼 별칭]
AS 서브쿼리 
[WITH CHECK OPTION] 
[WITH READ ONLY];
```

| 옵션                | 설명                                                           |
| :------------------ | :------------------------------------------------------------- |
| `OR REPLACE`        | 동일한 이름의 VIEW가 이미 존재하면 이를 변경, 없으면 새로 생성 |
| `FORCE`             | 서브쿼리에 사용된 테이블이 존재하지 않아도 뷰 생성             |
| `NOFORCE`           | 서브쿼리에 사용된 테이블이 존재할 때만 뷰 생성 **(기본값)**    |
| `컬럼` `별칭`       | 조회되는 VIEW의 컬럼명을 지정                                  |
| `WITH CHECK OPTION` | 컬럼의 값을 수정 불가능하게 함                                 |
| `WITH READ ONLY`    | VIEW에 대해 SELECT만 가능하게 함                               |

> **ORA-42399: 읽기 전용 뷰에서는 DML 작업을 수행할 수 없습니다.**<br>
> `READ ONLY` 옵션이 적용된 뷰에서는 DML 작업 불가
{: .prompt-danger }


<br>

### SEQUENCE 생성

```sql
CREATE SEQUENCE 시퀀스이름
[STRAT WITH 숫자]             -- 처음 발생시킬 시작값 지정, 생략하면 자동 1이 기본
[INCREMENT BY 숫자]           -- 다음 값에 대한 증가치, 생략하면 자동 1이 기본
[MAXVALUE 숫자 | NOMAXVALUE]  -- 발생시킬 최대값 지정 (10의 27제곱 -1)
[MINVALUE 숫자 | NOMINVALUE]  -- 최소값 지정 (-10의 26제곱)
[CYCLE | NOCYCLE]             -- 값 순환 여부 지정
[CACHE 바이트크기 | NOCACHE]; -- 캐쉬메모리 기본값은 20바이트, 최소값은 2바이트
```

SEQUENCE CACHE
: 시퀀스의 캐시 메모리는 할당된 크기만큼 다음 값들을 미리 생성해 저장해 둔다. 시퀀스 호출 시 저장된 값을 가져와 반환하므로 매번 시퀀스를 생성하는 것보다 **DB 속도를 향상**시킬 수 있다. 한번에 대량의 데이터를 DB에 등록할 때 주로 사용한다.

<br>

### INDEX 생성

```sql
CREATE [UNIQUE] INDEX 인덱스명 ON 테이블명(컬럼명|함수명|계산식[, ...]);
```
- `UNIQUE` : 중복값이 없는 컬럼에 UNIQUE INDEX를 생성하기 위한 옵션 ⇒ 속도 빠름
- PK 또는 UNIQUE 제약 조건이 설정된 컬럼에 대해 UNIQUE 인덱스 자동 생성






<br>
<br>
<br>


## ALTER : 객체 스키마 변경

대상 : **컬럼**(추가/수정/삭제/이름변경), **제약조건**(추가/삭제/이름변경), **테이블**(수정/이름변경)

### 컬럼 추가/수정/삭제 : ADD/MODIFY/DROP

#### 컬럼 추가
    
```sql
ALTER TABLE 테이블명 ADD(컬럼명 데이터타입 [DEFAULT 기본값]);
```

#### 컬럼 수정
    
```sql
ALTER TABLE 테이블명 MODIFY 컬럼명 데이터타입;     -- 데이터타입 변경
ALTER TABLE 테이블명 MODIFY 컬럼명 DEFAULT 바꿀값; -- 기본값 변경

-- NOT NULL 제약조건 수정
ALTER TABLE 테이블명 MODIFY 컬럼명 [CONSTRAINT 제약조건명] NOT NULL; -- NOT NULL 제약조건 추가
ALTER TABLE 테이블명 MODIFY 컬럼명 NULL;                             -- NOT NULL 제약조건 삭제
```

> **ORA-01441: 일부 값이 너무 커서 열 길이를 줄일 수 없음**<br>
> 기존에 저장된 값보다 작은 데이터타입으로 수정 불가
{: .prompt-danger }    

> `NOT NULL`은 다른 제약조건과는 달리 `ALTER MODIFY` 구문으로 수정 가능하다.
{: .prompt-tip }

#### 컬럼 삭제 
> cf. 행 삭제 : DLELET (DML)
    
```sql
ALTER TABLE 테이블명 DROP(삭제할컬럼명);
ALTER TABLE 테이블명 DROP COLUMN 삭제할컬럼명;
```

> **ORA-12983: 테이블에 모든 열들을 삭제할 수 없습니다**<br>
> 테이블의 모든 컬럼 삭제 불가. 테이블에는 최소 1개 이상의 컬럼이 존재해야 한다.
{: .prompt-danger } 


### SEQUENCE 수정

- `START WITH` 외 `CREATE` 구문과 동일하다.
- 현재 시퀀스 값을 특정 값으로 변경하고 싶으면 시퀀스 삭제 후 `START WITH`를 사용해서 다시 생성해야 한다.

```sql
ALTER SEQUENCE 시퀀스이름
[INCREMENT BY 숫자]            -- 다음 값에 대한 증가치, 생략하면 자동 1이 기본
[MAXVALUE 숫자 | NOMAXVALUE]   -- 발생시킬 최대값 지정 (10의 27제곱 -1)
[MINVALUE 숫자 | NOMINVALUE]   -- 최소값 지정 (-10의 26제곱)
[CYCLE | NOCYCLE]              -- 값 순환 여부 지정
[CACHE 바이트크기 | NOCACHE];  -- 캐쉬메모리 기본값은 20바이트, 최소값은 2바이트
```




### 제약조건 추가/삭제 : ADD/DROP

제약조건은 **수정 불가능** 👉 삭제하고 다시 추가

#### 기존 컬럼에 제약조건 추가
```sql
-- PRIMARY KEY, UNIQUE
ALTER TABLE 테이블명 ADD [CONSTRAINT 제약조건명] PRIMARY KEY|UNIQUE(컬럼명);

-- FOREIGN KEY
ALTER TABLE 테이블명 ADD [CONSTRAINT 제약조건명]
FOREIGN KEY(외래키가 될 컬럼명) REFERENCES 참조할 테이블명[(참조할 컬럼명)];

-- CHECK
ALTER TABLE 테이블명 ADD [CONSTRAINT 제약조건명]
CHECK(조건식);

-- NOT NULL
ALTER TABLE 테이블명 MODIFY 컬럼명 [CONSTRAINT 제약조건명] NOT NULL;
```
    
#### 제약조건 삭제

`NOT NULL` 제약조건은 1)일반적인 제약조건 삭제 방법과 2)`MODIFY`를 이용한 방법 모두 사용 가능

```sql
ALTER TABLE 테이블명 DROP CONSTRAINT 제약조건명;
ALTER TABLE 테이블명 MODIFY 컬럼명 [CONSTRAINT 제약조건명] NULL; -- NOT NULL 삭제#2
```

<br>

### 객체 이름 변경 : RENAME

```sql
ALTER TABLE 테이블명 RENAME TO 변경할 테이블명;                       -- 테이블명 변경
ALTER TABLE 테이블명 RENAME COLUMN 컬럼명 TO 변경할 테이블명;         -- 컬럼명 변경
ALTER TABLE 테이블명 RENAME CONSTRAINT 제약조건명 TO 변경할 테이블명; -- 제약조건명 변경
```









<br>

## DROP : 객체 삭제

```sql
DROP TABLE 테이블명 [CASCADE CONSTRAINS]; -- 테이블 삭제
DROP SEQUENCE 시퀀스이름;                 -- 시퀀스 삭제
DROP INDEX 인덱스이름;                    -- 인덱스 삭제
DROP VIEW 뷰이름;                         -- 뷰 삭제
DROP USER 유저명 [CASCADE];               -- 유저 삭제
```

- DROP **TABLE** 테이블명 `CASCADE CONSTRAINTS` : 테이블을 삭제할 때 모든 제약조건을 함께 삭제 
  (즉, 테이블이 참조되고있는 경우 참조관계를 삭제한다. 이 때 자식키는 삭제되지 않는다.)
- DROP **USER** 유저명 `CASCADE` : 해당 유저(스키마)가 보유하는 모든 객체를 함께 삭제

> **ORA-02449: 외래 키에 의해 참조되는 고유/기본 키가 테이블에 있습니다**<br>
> 외래키에 의해 참조되는 테이블 삭제 불가<br>
> 👉 해결 방법 1 : 자식 테이블 → 부모 테이블 순서로 삭제<br>
> 👉 해결 방법 2 : **`CASCADE CONSTRAINTS`** 옵션 사용
{: .prompt-danger }

### TRUNCATE

- 테이블의 **전체 행** 삭제 (DML ❌, 테이블 스키마 유지)
- DELETE(DML)보다 수행속도가 더 빠르다.
  
```sql
TRUNCATE TABLE 테이블명;
```

<br>

## COMMNET ON : 주석 달기

```sql
COMMENT ON COLUMN 테이블명.컬럼명 IS '주석내용';
```

### 주석 내용 확인
```sql
SELECT * FROM USER_COL_COMMENTS WHERE TABLE_NAME = '테이블명';
```