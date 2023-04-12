---
title: "[DB이론] 01. DQL : SELECT"
date: 2023-04-08 21:49:00 +0900
categories: [Database]
tags: [DB이론, Oracle]
---

## SELECT 구문

```sql
/* 구문해석순서 */
/*5*/ SELECT *|컬럼명|함수|계산식|리터럴 [[AS] 별칭]
/*1*/ FROM 테이블명 [JOIN 테이블명 ON(컬럼명)|USING(조건식)]
/*2*/ WHERE 조건식 [AND|OR ...]
/*3*/ GROUP BY column|function
/*4*/ HAVING 그룹함수를_이용한_조건식
/*6*/ ORDER BY 컬럼명|별칭|컬럼순서 [ASC|DESC] [NULLS FIRST|LAST];
```

**DQL** : Data Query Language

**컬럼 값** : 행과 열이 교차되는 테이블의 한 칸(한 셀)에 작성된 값(DATA)

**RESULT SET** : SELECT문의 실행 결과로 조회되는 행의 집합(행의 수는 0개 이상)

**리터럴(Literal)** : 고정된 값, '…'(홑따옴표)로 표현
> cf. "…"(쌍따옴표) : 계정명, 비밀번호, 컬럼명, 테이블명 등 값이 아닌 문자열 표현 (대소문자 구분, 공백 포함 가능)

### 집합연산(Set Operation)

2개 이상의 SELECT 결과(RESULT SET)을 이용해 하나의 결과를 조회하는 연산
> 집합 연산에 사용되는 RESULT SET의 타입, 순서, 개수가 동일해야 한다.
{: .prompt-warning }

| UNION      | 합집합                  |
| INTERSECT  | 교집합                  |
| UNION ALL  | 합집합 + 교집합 (중복O) |
| MINUS      | 차집합                  |

```sql
SELECT * FROM 테이블명
UNION|INTERSECT|UNION ALL|MINUS -- 집합연산자
SELECT * FROM 테이블명;
```





<br>

## SELECT 절

### 날짜 조회

```sql
SELECT SYSDATE FROM DUAL;                        -- 오늘 날짜 조회

SELECT SYSDATE-1, SYSDATE, SYSDATE+1 FROM DUAL;  -- 어제, 오늘, 내일 조회

SELECT SYSDATE, SYSDATE-(1/24) FROM DUAL;        -- 1시간 후
SELECT SYSDATE, SYSDATE-(1/24/60) FROM DUAL;     -- 1분 후
SELECT SYSDATE, SYSDATE-(1/24/60*30) FROM DUAL;  -- 30분 후

SELECT (SYSDATE - BIRTHDAY) / 365 FROM 테이블명; -- DATE 끼리 연산
```

- `SYSDATE` : 시스템 상의 현재 날짜(시간)
- `DUAL`(**DU**mmy t**A**b**L**e) : 실제 테이블이 아닌 임시 테이블
- DATE 타입으로 +, - 연산 가능(**일Day 단위**)
- 대소비교 연산 : 과거 < 미래

### 별칭(ALIAS)

| 작성법           | 문자 | 띄어쓰기 | 특수문자 |
| ---------------- | :--: | :------: | :------: |
| 컬럼명 AS 별칭   | O    | X        | X        |
| 컬럼명 AS "별칭" | O    | O        | O        |
| 컬럼명 별칭      | O    | X        | X        |
| 컬럼명 "별칭"    | O    | O        | O        |

### DISTINCT

조회 시 지정된 컬럼에 중복 제거

```sql
SELECT DISTINCT 컬럼명 FROM 테이블명;
```

### 연결 연산자( || )

여러 값을 하나의 컬럼 값으로 연결



<br>

## WHERE 절

조건식을 활용하여 조건을 충족하는 행만 조회할 때 사용
> cf. 조건식 : 결과값이 `TRUE` 또는 `FALSE`인 식
    
**조건식에서 사용하는 연산자 종류**

| 비교 연산자 | >, >=, <, <=, =(같다), !=, <>(같지 않다) |
| 논리 연산자 | AND, OR, NOT                             |
    
### BTWEEN A AND B

컬럼 값이 A**이상** B**이하**인 경우 `TRUE`

| BETWEEN A AND B         | 경곗값(A, B) 포함 ⭕ |
| **NOT** BETWEEN A AND B | 경곗값(A, B) 포함 ❌ |

```sql
SELECT * FROM 테이블명 WHERE 컬럼명 [NOT] BETWEEN 값1 AND 값2;
```

### NULL 처리 연산

| IS NULL         | 컬럼 값이 `NULL`인 경우 `TRUE`          |
| IS **NOT** NULL | 컬럼 값이 `NULL`이 **아닌** 경우 `TRUE` |

```sql
SELECT * FROM 테이블명 WHERE 컬럼명 IS [NOT] NULL;
```

### LIKE & Wildcard

비교하려는 값이 특정한 **패턴**을 만족 시키면(`TRUE`) 조회하는 연산자

```sql
SELECT * FROM 테이블명 WHERE 컬럼명 LIKE 패턴;
```

**와일드카드 문자(%, _)를 활용한 패턴**

| '%A'   | A로 끝나는 문자열            |
| 'A%'   | A로 시작하는 문자열          |
| '%A%'  | A를 포함하는 문자열          |
| 'A_'   | A 뒤에 한 글자만 있는 문자열 |
| '___A' | A 앞에 세 글자가 있는 문자열 |

### ESCAPE

LIKE 패턴 내부에서 와일드카드 문자(%, _)를 일반 문자열로 표현할 때 사용
    
```sql
-- 특정 컬럼에서 ‘_’문자열을 포함하는 행 조회
SELECT * FROM 테이블명 WHERE 컬럼명 LIKE '%_%'; -- 원하는 결과 조회 실패
```

> 와일드카드 문자(_)와 패턴에 사용된 일반 문자가 같은 문자여서 구분 불가<br>
> 👉 ESCAPE 옵션을 이용하여 일반 문자를 구분
{: .prompt-info }
    
```sql
SELECT * FROM 테이블명 
WHERE 컬럼명 LIKE '%$_%' ESCAPE '$'; -- '$' 뒤의 한 글자(_)를 일반 문자로 만듦
```

### 날짜(시간) 비교

```sql
-- TO_DATE() 함수 이용
SELECT * FROM 테이블명
WHERE 날짜컬럼 BETWEEN TO_DATE('1990/01/01') AND TO_DATE(20001231);

-- Oracle DB의 자동 형변환 기능 이용
SELECT * FROM 테이블명 
WHERE 날짜컬럼 BETWEEN 19900101 AND '2000/01/01';
```




<br>

## GROUP BY 절

같은 값을 가진 행끼리 하나의 그룹으로 묶어서 표현

```sql
SELECT * FROM 테이블명
GROUP BY 필드명1, 필드명2;
--       (대그룹) (소그룹)
```

### 집계 함수

GROUP BY절에 작성하여 그룹 별로 산출한 결과를 집계하는 함수

| ROLLUP | 그룹별로 중간 집계와 전체 합계를 처리하는 함수                       |
| CUBE   | 그룹으로 지정된 모든 그룹에 대한 중간 집계와 총 합계를 처리하는 함수 |

```sql
SELECT * FROM 테이블명
GROUP BY ROLLUP|CUBE(필드명, ...);
```



<br>

## HAVING 절

그룹함수로 조회할 그룹에 대한 조건을 설정할 때 사용

```sql
SELECT * FROM 테이블명
GROUP BY 필드명1, 필드명2
HAVING 그룹핑 된 데이터에 대한 조건식;
```
> 예시
```sql
-- 평균이 70점 이상인 학급과 학급 평균 조회
SELECT CLASS, AVG(GRADE) FROM SCHOOL
GROUP BY CLASS
HAVING AVG(GRADE) >= 70;
```



<br>

## ORDER BY 절

SELECT 문의 조회 결과(RESULT SET)를 정렬할때 사용하는 구문으로, **SELECT 문의 제일 마지막에 해석**된다.

```sql
SELECT * FROM 테이블명
ORDER BY 컬럼명|별칭|순서 [ASC|DESC] [NULLS FIRST|LAST];
```
> ORDER BY 절에서만 컬럼명 대신 `별칭`이나 `순서`를 사용할 수 있다.
{: .prompt-warning }
> SELECT문 해석순서에 의해 **WHERE 절**에서는 `별칭|순서` 사용 불가

| NULLS FIRST | 내림차순 기본값 | NULL 데이터 먼저 출력
| NULLS LAST  | 오름차순 기본값 | NULL 데이터를 마지막에 출력

> 기본값 : `ASC`(오름차순) & `NULLS LAST`(널은 마지막에)    

### 정렬 중첩

큰 분류를 먼저 정렬하고 내부 분류를 다음에 정렬

```sql
SELECT * FROM 테이블명 
ORDER BY 대분류, 소분류;
```