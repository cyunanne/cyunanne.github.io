---
title: "[DB이론] 08. CONSTRAINTS"
date: 2023-04-09 14:53:00 +0900
categories: [Database]
tags: [DB이론, Oracle]
---

## 제약조건(CONSTRAINTS)

- 사용자가 원하는 조건의 데이터만 유지하기 위해서 컬럼에 설정하는 제약
- 제약조건 종류 : `PRIMARY KEY`, `NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY`
- 모든 제약조건 설정 시 `CONSTRAINT` 키워드로 제약조건 이름을 부여할 수 있다. → 디버깅 시 용이
>
제약조건 이름 설정 ❌ → ORA-00001: 무결성 제약조건(KH_CYN.**SYS_C0031039**)에 위배됩니다<br>
제약조건 이름 설정 ⭕ → ORA-00001: 무결성 제약조건(KH_CYN.**USER_ID_UK**)에 위배됩니다

### 제약조건 설정 목적

1. **데이터 무결성 유지** (중복 ❌, NULL ❌)
2. 입력 데이터에 문제가 없는지 자동으로 검사
3. 데이터의 수정/삭제 가능여부 검사


### 제약조건명으로 제약조건 정보 조회

```sql
SELECT UCC.TABLE_NAME, UCC.COLUMN_NAME, UC.CONSTRAINT_TYPE
FROM USER_CONSTRAINTS UC, USER_CONS_COLUMNS UCC
WHERE UCC.CONSTRAINT_NAME = UC.CONSTRAINT_NAME
AND UCC.CONSTRAINT_NAME = '제약조건명';
```

### 테이블명으로 제약조건 조회

```sql
SELECT *
FROM USER_CONSTRAINTS C1
JOIN USER_CONS_COLUMNS C2 USING(CONSTRAINT_NAME)
WHERE C1.TABLE_NAME = '테이블명';
```





<br>

## NOT NULL

- 해당 컬럼에 반드시 값이 기록되어야 하는 경우 사용
- 삽입/수정시 NULL값을 허용하지 않도록 컬럼레벨에서 제한
- **컬럼레벨에서만 작성 가능**

```sql
CREATE TABLE 테이블명(
	컬럼명 컬럼타입 [CONSTRAINT 제약조건명] NOT NULL
);
```

> **ORA-01400: NULL을 ("DB명"."테이블명"."컬럼명") 안에 삽입할 수 없습니다**<br>
> NOT NULL 제약조건이 설정된 컬럼에 NULL 삽입 불가
{: .prompt-danger }





<br>

## UNIQUE (UNIQUE KEY)

- 컬럼에 입력 값에 대해서 중복을 제한하는 제약조건 (단, NULL 값은 중복 삽입 가능)
- 컬럼레벨 / 테이블레벨에서 설정 가능
- UNIQUE 복합키
  - 두 개 이상의 컬럼을 묶어서 하나의 UNIQUE 제약조건 설정
  - 지정된 모든 컬럼의 값이 같아야 중복 데이터로 판정
  - 복합키는 테이블레벨로만 지정 가능

```sql
CREATE TABLE 테이블명(
	컬럼명 컬럼타입 [CONSTRAINT 제약조건명] UNIQUE        -- 컬럼레벨
	...
	[CONSTRAINT 제약조건명] UNIQUE(컬럼명[, 컬럼명, ...]) -- 테이블레벨
);
```

> **ORA-00001: 무결성 제약조건(DB명.제약조건명)에 위배됩니다**<br>
> UNIQUE 제약조건이 설정된 컬럼에 중복 데이터 삽입 불가
{: .prompt-danger }




<br>

## PRIMARY KEY

- 테이블에서 한 행을 특정하는 용도의 컬럼 (즉, 식별자(IDENTIFIER) 역할)
- 의미상 NOT NULL + UNIQUE 와 동등
- 한 테이블당 한 개만 설정 가능
- 컬럼레벨 / 테이블레벨에서 설정 가능
- 단일키 / 복합키 모두 설정 가능
  - 복합키인 경우 모든 컬럼이 동일해야 중복 데이터로 판정
  - 복합키에 포함된 모든 컬럼에 NULL 삽입 불가
  - 복합키는 테이블레벨로만 설정 가능

```sql
CREATE TABLE 테이블명(
	컬럼명 데이터타입 [CONSTRAINT 제약조건명] PRIMARY KEY     -- 컬럼레벨
	...
	[CONSTRAINT 제약조건명] PRIMARY KEY(컬럼명[, 컬럼명,...]) -- 테이블레벨
);
```

> **ORA-00001: 무결성 제약조건(DB명.제약조건명)에 위배됩니다**<br>
> **ORA-01400: NULL을 ("DB명"."테이블명"."컬럼명") 안에 삽입할 수 없습니다**<br>
> PRIMARY KEY 제약조건이 설정된 컬럼에 중복 데이터 또는 NULL 삽입 불가
{: .prompt-danger }




<br>

## FOREIGN KEY (외부키=외래키)

![foreign-key-relation](/assets/img/2023/foreign-key-relation.png){: width="300"}
_화살표 머리쪽이 부모 테이블(USER_GRADE)_

- FOREIGN KEY 제약조건에 의해 테이블간의 관계(RELATIONSHIP)가 형성됨
- 참조(REFERENCES)된 다른 테이블의 컬럼이 제공하는 값(+ NULL)만 사용할 수 있음
- 참조될 수 있는 컬럼 : PRIMARY KEY 컬럼**(기본값, 생략가능)**, UNIQUE 컬럼

```sql
CREATE TABLE 테이블명(
    컬럼명 데이터타입 [CONSTRAINT 제약조건명] REFERENCES 참조TB[(참조컬럼)]          -- 컬럼레벨
    ...
    [CONSTRAINT 제약조건명] FOREIGN KEY(컬럼) REFERENCES 참조TB[(참조컬럼)] [삭제룰] -- TB레벨
);
```

> **ORA-02291: 무결성 제약조건(DB명.제약조건명)이 위배되었습니다- 부모 키가 없습니다**<br>
> 외래키에 없는 값 삽입 불가(NULL 제외)
{: .prompt-danger }

### 삭제 옵션
부모 테이블의 데이터 삭제 시 연관된 자식 테이블의 데이터 처리방식 설정

| ON DELETE RESTRICTED | 자식 테이블에서 사용되고 있는 부모키 삭제 불가 (기본값) |
| ON DELETE SET NULL   | 부모키 삭제 시 자식키를 NULL로 변경                     |
| ON DELETE CASCADE    | 부모키 삭제시 자식키가 포함된 행도 함께 삭제            |

> **ORA-02292: 무결성 제약조건(DB명.제약조건명)이 위배되었습니다- 자식 레코드가 발견되었습니다**<br>
> 부모 테이블, 부모 레코드(참조되어지고있는 테이블, 컬럼) 삭제 불가<br>
> 👉 `ON DELETE SET NULL` 또는 `ON DELETE CASCADE` 삭제옵션 설정
{: .prompt-danger }





<br>

## CHECK

- 컬럼에 기록되는 값에 조건을 설정하기 위한 키워드
- CHECK 제약조건은 조건식을 범위로도 설정 가능
- 컬럼레벨 / 테이블레벨에서 설정 가능
- **주의** : **비교값은 리터럴만 사용**할 수 있고, 변하는 값이나 함수 사용 불가

```sql
CREATE TABLE 테이블명(
	컬럼명 데이터타입 [CONSTRAINT 제약조건명] CHECK(조건식) -- 컬럼레벨
	...
	[CONSTRAINT 제약조건명] CHECK(조건식)                   -- 테이블레벨
);
```

> **ORA-02290: 체크 제약조건(KH_CYN.GENDER_CHECK)이 위배되었습니다**<br>
> CHECK 제약조건에 위배되는 값 입력 불가
{: .prompt-danger }