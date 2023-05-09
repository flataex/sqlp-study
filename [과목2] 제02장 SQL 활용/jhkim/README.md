# 제 1절 서브쿼리

<aside>
💡 **서브쿼리란 하나의 SQL 문안에 포함된 또 다른 SQL 문**

</aside>

### 동작하는 방식에 따른 서브쿼리 분류

| 구분 | 비연관 서브쿼리 | 연관 서브쿼리 |
| --- | --- | --- |
| 정의 | 메인 쿼리의 컬럼과 서브쿼리의 컬럼이 서로 비교되지 않는 서브쿼리 | 메인 쿼리의 컬럼과 서브쿼리의 컬럼이 서로 비교되는 서브쿼리 |
| 실행 횟수 | 서브쿼리가 먼저 실행되고 결과값을 메인 쿼리에 적용 | 메인 쿼리 실행 도중에 서브쿼리가 실행됩니다. |
| 성능 | 대체로 느리고 비효율적입니다. | 대체로 빠르고 효율적입니다. |
| 사용 예시 | WHERE, HAVING 절에서 주로 사용합니다. | FROM, WHERE, HAVING 절에서 주로 사용합니다. |

- 비연관 서브쿼리 예시

```sql
SELECT e1.ID, e1.NAME, e1.DEPARTMENT_ID, e1.SALARY
FROM EMPLOYEE e1
WHERE e1.SALARY > (SELECT AVG(e2.SALARY) FROM EMPLOYEE e2);
```

EMPLOYEE 테이블에서 평균 급여보다 높은 급여를 받는 직원의 ID, 이름, 부서 ID, 급여를 검색합니다. 서브쿼리가 먼저 실행되어 평균 급여를 반환한 다음, 메인 쿼리가 실행됩니다.

- 연관 서브쿼리 예시

```sql
SELECT e1.ID, e1.NAME, e1.DEPARTMENT_ID, e1.SALARY
FROM EMPLOYEE e1
WHERE NOT EXISTS (
    SELECT 1
    FROM EMPLOYEE e2
    WHERE e2.DEPARTMENT_ID = e1.DEPARTMENT_ID -- 메인 쿼리와 상호작용
    AND e2.SALARY > e1.SALARY -- 메인 쿼리와 상호작용
);
```

이 쿼리는 연관 서브쿼리를 사용하여 각 부서에서 더 높은 급여를 받는 동료가 없는 직원을 찾습니다. 서브쿼리는 메인 쿼리의 각 행에 대해 실행되며, 서브쿼리의 결과는 메인 쿼리와 상호 작용하여 최종 결과를 생성합니다. 이 경우, 결과는 각 부서의 최고 급여를 받는 직원의 ID, 이름, 부서 ID, 급여를 포함합니다.

## 반환되는 데이터의 형태에 따른 서브쿼리 분류

| 구분 | 설명 |
| --- | --- |
| Single row | 서브쿼리가 단일 행을 반환합니다. 이 경우 단일 값을 반환하며, 비교 연산자를 사용할 수 있습니다. |
| Multi row | 서브쿼리가 여러 행을 반환합니다. 이 경우 여러 값을 반환하며, IN, ANY, ALL 등의 연산자를 사용할 수 있습니다. |
| Multi column | 서브쿼리가 여러 열을 반환합니다. 이 경우 행과 열 모두를 반환하며, EXISTS, NOT EXISTS 등의 연산자를 사용할 수 있습니다. |

1. **Single row 서브쿼리 예시**

> **이 예시에서는 부서 ID가 가장 작은 부서의 평균 급여를 계산합니다.**
> 

```sql
SELECT d.ID, d.DEPARTMENT_NAME, AVG(e.SALARY) AS average_salary
FROM DEPARTMENT d
JOIN EMPLOYEE e ON d.ID = e.DEPARTMENT_ID
WHERE d.ID = (SELECT MIN(d2.ID) FROM DEPARTMENT d2);
```

1. **Multi row 서브쿼리 예시**

> **이 예시에서는 백엔드팀의 모든 직원중 가장 높은급여의 이상의 급여를 받는 직원을 찾습니다.**
> 

```sql
SELECT e.ID, e.NAME, e.DEPARTMENT_ID, e.SALARY
FROM EMPLOYEE e
WHERE e.SALARY > ALL (
    SELECT e2.SALARY
    FROM EMPLOYEE e2
    JOIN DEPARTMENT d ON e2.DEPARTMENT_ID = d.ID
    WHERE d.DEPARTMENT_NAME = '백엔드'
);
```

- ANY 사용:

> **이 예시에서는 백엔드팀의 적어도 한 명의 직원보다 높은 급여를 받는 직원을 찾습니다.**
> 

```sql
SELECT e.ID, e.NAME, e.DEPARTMENT_ID, e.SALARY
FROM EMPLOYEE e
WHERE e.SALARY > ANY (
    SELECT e2.SALARY
    FROM EMPLOYEE e2
    JOIN DEPARTMENT d ON e2.DEPARTMENT_ID = d.ID
    WHERE d.DEPARTMENT_NAME = '백엔드'
);
```

- EXISTS 사용:

> **이 예시에서는 마케팅 부서에서 근무하는 직원을 찾습니다.**
> 

```sql
SELECT e.ID, e.NAME, e.DEPARTMENT_ID, e.SALARY
FROM EMPLOYEE e
WHERE EXISTS (
    SELECT 1
    FROM DEPARTMENT d
    WHERE d.ID = e.DEPARTMENT_ID AND d.DEPARTMENT_NAME = 'Marketing'
);
```

1. **Multi column 서브쿼리 예시**

> **이 예시에서는 각 부서에서 최고 급여를 받는 직원을 찾습니다.**
> 

```sql
SELECT e.ID, e.NAME, e.DEPARTMENT_ID, e.SALARY
FROM EMPLOYEE e
WHERE EXISTS (
    SELECT d.ID, MAX(e2.SALARY)
    FROM DEPARTMENT d
    JOIN EMPLOYEE e2 ON d.ID = e2.DEPARTMENT_ID
    GROUP BY d.ID
    HAVING d.ID = e.DEPARTMENT_ID AND MAX(e2.SALARY) = e.SALARY
);
```

1. **그 이외의 서브쿼리 예시**

- SELECT 절에서 서브쿼리 사용

```sql
SELECT e.ID, e.NAME, e.DEPARTMENT_ID, e.SALARY,
    (SELECT d.DEPARTMENT_NAME FROM DEPARTMENT d WHERE d.ID = e.DEPARTMENT_ID) 
AS department_name
FROM EMPLOYEE e;
```

- FROM절에서 서브쿼리 사용

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY
FROM (
    SELECT *
    FROM EMPLOYEE
    WHERE SALARY > 5000
) e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID;
```

## 뷰

<aside>
💡 뷰는 가상의 테이블로 하나 이상의 기본테이블로부터 데이터를 가져와 생성되며, 복잡한 쿼리를 단순화하거나 데이터 접근을 제한하는 데 사용된다.

</aside>

- 사용예시

```sql
CREATE VIEW employee_department AS
SELECT e.ID, e.NAME, e.SALARY, e.DEPARTMENT_ID, d.DEPARTMENT_NAME
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID;

SELECT * FROM employee_department;
```

> **가상 테이블이기 때문에 실제로는 아래와 같은 실행계획을 갖는다!!!**
> 

| 순서 | 연산 | 속성 | 개수 | 크기 | Access 방법 | Time |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | SELECT STATEMENT |  | 13 | 390 | 6 (17) | 00:00:01 |
| 1 | MERGE JOIN |  | 13 | 390 | 6 (17) | 00:00:01 |
| 2 | TABLE ACCESS BY INDEX ROWID | DEPARTMENT | 5 | 50 | 2 (0) | 00:00:01 |
| 3 | INDEX FULL SCAN | SYS_C0022090 | 5 |  | 1 (0) | 00:00:01 |
| 4 | SORT JOIN |  | 13 | 260 | 4 (25) | 00:00:01 |
| 5 | TABLE ACCESS STORAGE FULL | EMPLOYEE | 13 | 260 | 3 (0) | 00:00:01 |

---

# 제 2절 집합 연산자

| 집합 연산자 | 설명 |
| --- | --- |
| UNION | 두 SELECT 문의 결과를 결합하고 중복된 행을 제거합니다. |
| UNION ALL | 두 SELECT 문의 결과를 결합하며 중복된 행도 포함합니다. |
| INTERSECT | 두 SELECT 문의 결과에서 공통된 행만 반환합니다. |
| MINUS | 첫 번째 SELECT 문의 결과에서 두 번째 SELECT 문의 결과를 제거합니다. |
- **`EMPLOYEE`** 및 **`DEPARTMENT`** 테이블을 사용한 집합 연산자의 예
1. **UNION:**

```sql
-- 부서 1의 직원 이름과 부서 2의 직원 이름을 반환합니다.
SELECT NAME FROM EMPLOYEE WHERE DEPARTMENT_ID = 1
UNION
SELECT NAME FROM EMPLOYEE WHERE DEPARTMENT_ID = 6;
```

1. **UNION ALL**

```sql
-- 부서 1의 직원 이름과 부서 2의 직원 이름을 반환하며 중복된 이름도 포함합니다.
SELECT NAME FROM EMPLOYEE WHERE DEPARTMENT_ID = 1
UNION ALL
SELECT NAME FROM EMPLOYEE WHERE DEPARTMENT_ID = 6;
```

<aside>
💡 실제 위의 두 쿼리의 결과를 확인해보면 정렬 순서가 다른것을 확인 할 수 있다. 
**`UNION`**과 **`UNION ALL`** 연산자의 결과에서 정렬 순서의 차이는 집합 연산자의 동작 방식 때문이다.

**`UNION`**은 두 SELECT 문의 결과를 결합할 때 중복된 행을 제거한다. 이를 위해 오라클은 내부적으로 중복 행을 제거하기 전에 정렬 작업을 수행한다. 따라서 **`UNION`**을 사용하면 결과가 암시적으로 정렬된다.

반면, **`UNION ALL`**은 두 SELECT 문의 결과를 결합하고 중복된 행도 포함한다. 중복 행 제거가 필요하지 않으므로 정렬 작업이 수행되지 않는다. 그 결과, **`UNION ALL`**을 사용하면 암시적인 정렬이 발생하지 않는다.

결론적으로, **`UNION`**과 **`UNION ALL`**의 결과에서 정렬 순서의 차이는 중복 행 제거를 위해 필요한 정렬 작업 때문이다.

</aside>

> 아래는 실행 계획입니다.
> 

- **UNION**

| Id | Operation | Name | Rows | Bytes | Cost (%CPU) | Time |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | SELECT STATEMENT |  | 5 | 65 | 8 (25) | 00:00:01 |
| 1 | SORT UNIQUE |  | 5 | 65 | 8 (25) | 00:00:01 |
| 2 | UNION-ALL |  |  |  |  |  |
| * 3 | TABLE ACCESS STORAGE | EMPLOYEE | 2 | 26 | 3 (0) | 00:00:01 |
| * 4 | TABLE ACCESS STORAGE | EMPLOYEE | 3 | 39 | 3 (0) | 00:00:01 |
- **UNION ALL**

| Id | Operation | Name | Rows | Bytes | Cost (%CPU) | Time |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | SELECT STATEMENT |  | 5 | 65 | 6 (0) | 00:00:01 |
| 1 | UNION-ALL |  |  |  |  |  |
| * 2 | TABLE ACCESS STORAGE FULL | EMPLOYEE | 2 | 26 | 3 (0) | 00:00:01 |
| * 3 | TABLE ACCESS STORAGE FULL | EMPLOYEE | 3 | 39 | 3 (0) | 00:00:01 |
1. **INTERSECT**

```sql
-- 부서 1과 부서 2에 모두 속한 직원 이름을 반환합니다
SELECT NAME FROM EMPLOYEE WHERE DEPARTMENT_ID = 1
INTERSECT
SELECT NAME FROM EMPLOYEE WHERE DEPARTMENT_ID = 6;
```

1. **MINUS**

```sql
-- 부서 1의 직원 이름 중 부서 2의 직원 이름에 해당하지 않는 이름을 반환합니다.
SELECT NAME FROM EMPLOYEE WHERE DEPARTMENT_ID = 1
MINUS
SELECT NAME FROM EMPLOYEE WHERE DEPARTMENT_ID = 6;
```

---

# 제 3절 그룹함수

## 함수의 종류

1. **Aggregate Function (집계 함수):**
집계 함수는 여러 행의 데이터를 기반으로 계산되는 함수입니다. 이러한 함수는 데이터베이스에서 데이터를 분석하는 데 유용하며, 특히 여러 행의 값을 집계하거나 요약할 때 사용됩니다.
2. **Group Function (그룹 함수):**
그룹 함수는 GROUP BY 절과 함께 사용되어 특정 그룹별로 집계 값을 계산할 수 있는 함수입니다. 그룹 함수는 대부분 집계 함수와 동일하게 작동하며, GROUP BY 절을 사용하여 그룹별로 결과를 분리할 수 있습니다.
3. **Window Function (윈도우 함수):**
윈도우 함수는 결과 집합의 행을 기반으로 계산되는 함수로, 각 행에 대한 결과를 반환합니다. 윈도우 함수는 OVER 절과 함께 사용되어 행 간의 연산을 수행할 수 있습니다.

## GROUP BY절의 그룹화 기능함수

| 그룹화 기능 | 설명 |
| --- | --- |
| ROLLUP | ROLLUP은 GROUP BY 절과 함께 사용되며, 여러 그룹별로 하위 집계를 계산하고, 상위 수준까지 총계를 계산하는 데 사용됩니다. ROLLUP은 주어진 열의 계층 구조를 따라 다양한 레벨에서 합계를 계산합니다. 결과는 서브 그룹별 합계와 총계를 포함합니다. |
| CUBE | CUBE는 GROUP BY 절과 함께 사용되며, 여러 차원에 대한 집계를 계산하는 데 사용됩니다. CUBE는 주어진 열들 간의 모든 가능한 조합에 대한 집계 값을 생성합니다. 결과는 각 차원별로 합계와 총계를 포함합니다. |
| GROUPING SETS | GROUPING SETS는 GROUP BY 절과 함께 사용되며, 여러 그룹화 기준을 한 번에 계산하는 데 사용됩니다. GROUPING SETS는 여러 개의 GROUP BY 절을 결합하여 다양한 집계 수준을 계산할 수 있습니다. 결과는 지정된 그룹화 기준에 따른 합계를 포함합니다. |

- `ROLLUP` 함수의 예시

```sql
SELECT d.DEPARTMENT_NAME, e.POSITION, COUNT(*) AS EMPLOYEE_COUNT, SUM(e.SALARY) AS TOTAL_SALARY
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY ROLLUP(d.DEPARTMENT_NAME, e.POSITION);
```

- `ROLLUP` + `ORDER BY` 예시

```sql
SELECT d.DEPARTMENT_NAME
, e.POSITION
, COUNT(*) AS EMPLOYEE_COUNT
, SUM(e.SALARY) AS TOTAL_SALARY
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY ROLLUP(d.DEPARTMENT_NAME, e.POSITION)
ORDER BY d.DEPARTMENT_NAME, e.POSITION;
```

- `GROUPING` 함수의 예시

```sql
SELECT d.DEPARTMENT_NAME
, e.POSITION, COUNT(*) AS EMPLOYEE_COUNT
, SUM(e.SALARY) AS TOTAL_SALARY
, GROUPING(d.DEPARTMENT_NAME) AS IS_TOTAL
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY ROLLUP(d.DEPARTMENT_NAME, e.POSITION);
```

- `GROUPING` + `CASE` 사용의 예시

```sql
SELECT d.DEPARTMENT_NAME
, e.POSITION
, COUNT(*) AS EMPLOYEE_COUNT
, SUM(e.SALARY) AS TOTAL_SALARY,
   CASE GROUPING(d.DEPARTMENT_NAME)
       WHEN 1 THEN 'Total'
       ELSE 'Subtotal'
   END AS SALARY_TYPE
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY ROLLUP(d.DEPARTMENT_NAME, e.POSITION);
```

<aside>
💡 **`GROUPING(column)`** 함수는 그룹화된 열이 결과 집합에서 집계된 행을 나타내는 경우 1을 반환하고, 그렇지 않으면 0을 반환합니다. 이는 ROLLUP을 사용할 때 특정 열이 집계 값인지 아닌지를 판별하는 데 도움이 됩니다.

**`WHEN 1`**은 **`GROUPING(d.DEPARTMENT_NAME)`** 함수가 1을 반환하는 경우를 나타냅니다. 이는 DEPARTMENT_NAME 열이 집계된 행(즉, 총계 행)임을 의미합니다. 따라서 **`WHEN 1 THEN 'Total'`** 부분은 DEPARTMENT_NAME이 총계를 나타내는 행인 경우에 "Total"이라는 값을 반환하게 됩니다.

**`ELSE 'Subtotal'`**은 **`GROUPING(d.DEPARTMENT_NAME)`** 함수가 0을 반환하는 경우를 처리합니다. 이는 DEPARTMENT_NAME 열이 일반 행(즉, 소계 행)임을 의미합니다. 따라서 이 경우에 "Subtotal"이라는 값을 반환하게 됩니다.

</aside>

- `DECODE` 사용의 예시 **(ORACLE)**

```sql
SELECT DECODE(GROUPING(d.DEPARTMENT_NAME), 1, 'ALL-DEPT-Total', d.DEPARTMENT_NAME) AS DEPARTMENT_NAME,
       DECODE(GROUPING(e.POSITION), 1, 'ALL-Position-Total', e.POSITION)           AS POSITION_NAME,
       COUNT(*)                                                                    AS EMPLOYEE_COUNT,
       SUM(e.SALARY)                                                               AS TOTAL_SALARY
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY ROLLUP (d.DEPARTMENT_NAME, e.POSITION);
```

- `CUBE` 사용의 예시

```sql
SELECT d.DEPARTMENT_NAME, e.POSITION, COUNT(*) AS EMPLOYEE_COUNT, SUM(e.SALARY) AS TOTAL_SALARY
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY CUBE(d.DEPARTMENT_NAME, e.POSITION);
```

- `CUBE` 를 `UNION ALL`로  변경한다면?

```sql
-- 부서별, 포지션별 직원 수와 급여 합계
SELECT d.DEPARTMENT_NAME
     , e.POSITION
     , COUNT(*)      AS EMPLOYEE_COUNT
     , SUM(e.SALARY) AS TOTAL_SALARY
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY d.DEPARTMENT_NAME, e.POSITION

UNION ALL

-- 부서별 직원 수와 급여 합계
SELECT d.DEPARTMENT_NAME
     , NULL AS POSITION
     , COUNT(*) AS EMPLOYEE_COUNT
     , SUM(e.SALARY) AS TOTAL_SALARY
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY d.DEPARTMENT_NAME

UNION ALL

-- 포지션별 직원 수와 급여 합계
SELECT NULL AS DEPARTMENT_NAME
     , e.POSITION, COUNT(*) AS EMPLOYEE_COUNT
     , SUM(e.SALARY) AS TOTAL_SALARY
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY e.POSITION

UNION ALL

-- 전체 직원 수와 급여 합계
SELECT NULL AS DEPARTMENT_NAME
     , NULL AS POSITION
     , COUNT(*) AS EMPLOYEE_COUNT
     , SUM(e.SALARY) AS TOTAL_SALARY
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID;
```

- `GROUPING SETS` 사용 예시

```sql
SELECT d.DEPARTMENT_NAME
     , e.POSITION
     , COUNT(*)      AS EMPLOYEE_COUNT
     , SUM(e.SALARY) AS TOTAL_SALARY
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY GROUPING SETS ((d.DEPARTMENT_NAME, e.POSITION), (d.DEPARTMENT_NAME), (e.POSITION), ());
```

- 그냥 이것저것 사용해서 만든 SQL

```sql
SELECT d.DEPARTMENT_NAME
     , e.POSITION
     , COUNT(*)      AS EMPLOYEE_COUNT
     , SUM(e.SALARY) AS TOTAL_SALARY
     , CASE
           WHEN GROUPING(d.DEPARTMENT_NAME) = 1 AND GROUPING(e.POSITION) = 1 THEN 'Total'
           WHEN GROUPING(d.DEPARTMENT_NAME) = 1 THEN 'Subtotal by Position'
           WHEN GROUPING(e.POSITION) = 1 THEN 'Subtotal by Department'
           ELSE 'Detailed'
    END              AS GROUPING_TYPE
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
GROUP BY GROUPING SETS ((d.DEPARTMENT_NAME, e.POSITION), (d.DEPARTMENT_NAME), (e.POSITION), ());
```

# 제 4절 윈도우 함수

| 종류 | 함수명 | 설명 |
| --- | --- | --- |
| 그룹내 순위 관련 함수 | ROW_NUMBER() | 해당 윈도우에서 고유한 순위를 부여합니다. |
|  | RANK() | 동일한 값이 있을 경우 동일한 순위를 부여하고 건너뛰기가 발생합니다. |
|  | DENSE_RANK() | 동일한 값이 있을 경우 동일한 순위를 부여하고 건너뛰기 없이 순위가 증가합니다. |
|  | NTILE(n) | 결과 집합을 n 개의 동일한 크기의 그룹으로 나눕니다. |
| 그룹내 집계 관련 함수 | SUM() | 윈도우 내의 합계를 계산합니다. |
|  | AVG() | 윈도우 내의 평균을 계산합니다. |
|  | MIN() | 윈도우 내의 최솟값을 반환합니다. |
|  | MAX() | 윈도우 내의 최댓값을 반환합니다. |
|  | COUNT() | 윈도우 내의 행 개수를 계산합니다. |
| 그룹내 행 순서 관련 함수 | FIRST_VALUE() | 윈도우 내에서 첫 번째 행의 값을 반환합니다. |
|  | LAST_VALUE() | 윈도우 내에서 마지막 행의 값을 반환합니다. |
|  | LEAD() | 현재 행에서 지정된 수만큼 앞의 행의 값을 반환합니다. |
|  | LAG() | 현재 행에서 지정된 수만큼 뒤의 행의 값을 반환합니다. |
| 그룹내 비율 관련 함수 | PERCENT_RANK() | 행의 백분위 순위를 계산합니다. |
|  | CUME_DIST() | 특정 행의 분포 누적 백분위를 반환합니다. |

## WINDOW FUNCTION SYNTAX

```sql
<window_function> (<arguments>) OVER (
    [PARTITION BY <partition_expression_list>]
    [ORDER BY <order_expression_list>]
    [ROWS | RANGE <window_frame>]
)
```

| 구성 요소 | 설명 |
| --- | --- |
| window_function | 사용할 윈도우 함수 (예: ROW_NUMBER, RANK, DENSE_RANK, NTILE, LEAD, LAG, SUM, AVG 등) |
| arguments | 윈도우 함수에 전달할 인수 (필요한 경우) |
| PARTITION BY | 결과를 나눌 그룹을 정의하는 표현식 목록 |
| partition_expression_list | 각 파티션을 나누기 위해 사용되는 열이나 표현식 목록 |
| ORDER BY | 각 파티션 내에서 행의 순서를 정의하는 표현식 목록 |
| order_expression_list | 각 파티션 내에서 행 순서를 결정하는데 사용되는 열이나 표현식 목록 |
| ROWS | RANGE |
| window_frame | 현재 행에 대한 윈도우 프레임 범위를 지정하는 표현식 (예: UNBOUNDED PRECEDING, n PRECEDING, CURRENT ROW, n FOLLOWING, UNBOUNDED FOLLOWING) |

### 🧐윈도우 프레임 형식?

| 윈도우 프레임 형식 | 설명 |
| --- | --- |
| ROWS BETWEEN | 현재 행으로부터 지정된 행 수만큼 범위를 지정합니다. |
| RANGE BETWEEN | 현재 행으로부터 지정된 값의 범위만큼 범위를 지정합니다. |

| 옵션 | 설명 |
| --- | --- |
| UNBOUNDED PRECEDING | 파티션의 첫 번째 행부터 현재 행까지 범위를 지정합니다. |
| UNBOUNDED FOLLOWING | 현재 행부터 파티션의 마지막 행까지 범위를 지정합니다. |
| CURRENT ROW | 현재 행을 기준으로 범위를 지정합니다. |
| n PRECEDING | 현재 행으로부터 n행 이전까지 범위를 지정합니다. |
| n FOLLOWING | 현재 행으로부터 n행 이후까지 범위를 지정합니다. |

## 그룹 내 순위 함수

### RANK

```sql
SELECT e.ID
     , e.NAME
     , d.DEPARTMENT_NAME
     , e.SALARY
     , RANK() OVER (PARTITION BY d.DEPARTMENT_NAME ORDER BY e.SALARY DESC) AS SALARY_RANK
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.DEPARTMENT_NAME, e.SALARY DESC;
```

### DENSE_RANK

```sql
SELECT e.ID
     , e.NAME
     , d.DEPARTMENT_NAME
     , e.SALARY
     , DENSE_RANK() OVER (ORDER BY e.SALARY DESC) AS SALARY_RANK
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.DEPARTMENT_NAME, e.SALARY DESC;
```

### ROW_NUMBER

```sql
SELECT e.ID
     , e.NAME
     , d.DEPARTMENT_NAME
     , e.SALARY
     , ROW_NUMBER() OVER (PARTITION BY d.DEPARTMENT_NAME ORDER BY e.SALARY DESC) AS SALARY_RANK
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.DEPARTMENT_NAME, e.SALARY DESC;
```

## 일반집계함수

### SUM

```sql
SELECT e.NAME,
       d.DEPARTMENT_NAME,
       e.SALARY,
       SUM(e.SALARY) OVER (PARTITION BY d.DEPARTMENT_NAME) AS DEPARTMENT_TOTAL_SALARY
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.DEPARTMENT_NAME;
```

### MAX

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY,
       MAX(e.SALARY) OVER (PARTITION BY d.ID) AS MAX_SALARY
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### MIN

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY,
       AVG(e.SALARY) OVER (PARTITION BY d.ID) AS AVG_SALARY
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### AVG

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY,
       AVG(e.SALARY) OVER (PARTITION BY d.ID) AS AVG_SALARY
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### COUNT

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY,
       COUNT(*) OVER (PARTITION BY d.ID) AS EMPLOYEE_COUNT
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

## 그룹 내 행 순서 함수

### FIRST_VALUE

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY,
       FIRST_VALUE(e.NAME) OVER (PARTITION BY d.ID ORDER BY e.SALARY DESC ROWS UNBOUNDED PRECEDING) AS HIGHEST_PAID_EMPLOYEE
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### LAST_VALUE

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY,
       LAST_VALUE(e.NAME) OVER (PARTITION BY d.ID ORDER BY e.SALARY ASC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS LOWEST_PAID_EMPLOYEE
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### LAG

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY,
       LAG(e.SALARY) OVER (PARTITION BY d.ID ORDER BY e.ID) AS PREVIOUS_EMPLOYEE_SALARY
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### LEAD

```sql
SELECT e.ID, e.NAME, d.DEPARTMENT_NAME, e.SALARY,
       LEAD(e.SALARY) OVER (PARTITION BY d.ID ORDER BY e.ID) AS NEXT_EMPLOYEE_SALARY
FROM EMPLOYEE e
JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

## 그룹 내 비율 함수

### RATIO_TO_REPORT

```sql
-- 각 부서별로 직원들의 급여 비율을 계산.
-- RATIO_TO_REPORT 함수는 각 직원의 급여를 해당 부서의 총 급여로 나눈 값을 반환.
SELECT e.ID,
       e.NAME,
       d.DEPARTMENT_NAME,
       e.SALARY,
       RATIO_TO_REPORT(e.SALARY) OVER (PARTITION BY d.ID) AS SALARY_RATIO
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### PERCENT_RANK

```sql
-- 각 부서별로 직원들의 급여 순위를 백분율로 계산. 
-- PERCENT_RANK 함수는 각 직원의 급여 순위를 해당 부서의 직원 수로 나눈 값을 반환.
SELECT e.ID,
       e.NAME,
       d.DEPARTMENT_NAME,
       e.SALARY,
       PERCENT_RANK() OVER (PARTITION BY d.ID ORDER BY e.SALARY DESC) AS SALARY_PERCENT_RANK
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### CUME_DIST

```sql
-- 각 부서별로 직원들의 급여 분포를 계산.
-- CUME_DIST 함수는 각 직원의 급여가 해당 부서의 급여 분포에서 차지하는 위치를 반환.

SELECT e.ID,
       e.NAME,
       d.DEPARTMENT_NAME,
       e.SALARY,
       CUME_DIST() OVER (PARTITION BY d.ID ORDER BY e.SALARY DESC) AS SALARY_CUME_DIST
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

### NTILE

```sql
SELECT e.ID,
       e.NAME,
       d.DEPARTMENT_NAME,
       e.SALARY,
       NTILE(2) OVER (ORDER BY e.SALARY DESC) AS SALARY_NTILE
FROM EMPLOYEE e
         JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID
ORDER BY d.ID, e.ID;
```

## TOP N 쿼리

### ROWNUM

```sql
SELECT *
FROM (
    SELECT E.ID, E.NAME, E.DEPARTMENT_ID, D.DEPARTMENT_NAME, E.SALARY, E.POSITION, ROWNUM as row_num
    FROM EMPLOYEE E
    JOIN DEPARTMENT D ON E.DEPARTMENT_ID = D.ID
    ORDER BY E.SALARY DESC
)
WHERE row_num <= 5;
```

### TOP (only SQL Server)

```sql
SELECT TOP(2) WITH TIES NAME
FROM EMPLOYEE
ORDER BY SALARY DESC;
```

### ROW LIMITTING

```sql
SELECT E.ID
     , E.NAME
     , E.DEPARTMENT_ID
     , D.DEPARTMENT_NAME
     , E.SALARY
     , E.POSITION
FROM EMPLOYEE E
         JOIN DEPARTMENT D ON E.DEPARTMENT_ID = D.ID
WHERE d.ID = 1 -- 필요한 경우 조건 추가
ORDER BY E.SALARY DESC
    FETCH FIRST 5 ROWS ONLY;
```

- 아래는 각 DBMS 벤더 별 `ROW LIMITTING` 구문이다.

| DBMS | 구문 |
| --- | --- |
| Oracle 12c 이상 | FETCH FIRST N ROWS ONLY (OFFSET M ROWS) |
| PostgreSQL 8.4 이상 | LIMIT N (OFFSET M) |
| MySQL 8.0 이상 | LIMIT N (OFFSET M) |
| SQL Server 2012 이상 | OFFSET M ROWS FETCH NEXT N ROWS ONLY |

# 제 6절 계층형 질의와 셀프 조인

## 셀프조인

```sql
SELECT
    m1.id AS menu_id,
    m1.name AS menu_name,
    m1.url AS menu_url,
    m2.id AS parent_menu_id,
    m2.name AS parent_menu_name,
    m2.url AS parent_menu_url
FROM menu m1
         LEFT JOIN menu m2 ON m1.parent_id = m2.id
ORDER BY m1.id;
```

## 계층형 질의

| 기능 | 설명 | 예시 |
| --- | --- | --- |
| START WITH | 계층 구조에서 시작할 루트 노드를 지정합니다. | START WITH parent_id IS NULL |
| CONNECT BY | 상위 노드와 하위 노드 간의 관계를 정의합니다. PRIOR 키워드는 이전 레벨의 열을 참조합니다. | CONNECT BY PRIOR id=parent_id |
| NOCYCLE | 계층 구조에 순환 참조가 있는 경우, 에러를 방지하고 결과를 반환합니다. | CONNECT BY NOCYCLE ... |
| LEVEL | 현재 레벨의 깊이를 나타내는 가상 열입니다. | SELECT LEVEL, ... |
| SYS_CONNECT_BY_PATH | 계층 구조에서 루트 노드부터 현재 노드까지의 경로를 표시하는 데 사용됩니다. | SYS_CONNECT_BY_PATH(name, '/') |
| ORDER SIBLINGS BY | 같은 레벨의 노드를 정렬하는 데 사용됩니다. | ORDER SIBLINGS BY name |

- EXAMPLE

```sql
SELECT
        LPAD(' ', LEVEL * 2 - 2) || name AS menu_name,
        url,
        LEVEL,
        SYS_CONNECT_BY_PATH(name, ' > ') AS menu_path
FROM menu
START WITH parent_id IS NULL
CONNECT BY NOCYCLE PRIOR id = parent_id
ORDER SIBLINGS BY name;
```

# 제 7절 PIVOT 절과 UNPIVOT 절

| 연산자 | 설명 | 구문 |
| --- | --- | --- |
| PIVOT | 행 데이터를 열 데이터로 변환합니다. 집계 함수와 함께 사용되어 여러 행을 기준에 따라 하나의 행으로 요약합니다. | SELECT ... FROM (SELECT ...) PIVOT (AGGREGATE_FUNCTION(...) FOR column_name IN (subquery)) |
| UNPIVOT | 열 데이터를 행 데이터로 변환합니다. 여러 열의 데이터를 하나의 열로 합칩니다. | SELECT ... FROM (SELECT ...) UNPIVOT (column_name FOR new_column_name IN (subquery)) |

### PIVOT

```sql
SELECT *
FROM (SELECT e.ID, e.DEPARTMENT_ID, d.DEPARTMENT_NAME, e.POSITION
      FROM EMPLOYEE e
               JOIN DEPARTMENT d ON e.DEPARTMENT_ID = d.ID)
    PIVOT (COUNT(ID) FOR POSITION IN ('사원', '주임', '대리','과장','차장','부장'));
```

### UNPIVOT

```sql
SELECT ID, NAME, DEPARTMENT_ID, ATTRIBUTE, VALUE
FROM (SELECT ID, NAME, DEPARTMENT_ID, TO_CHAR(SALARY) AS SALARY, POSITION
      FROM EMPLOYEE)
    UNPIVOT (VALUE FOR ATTRIBUTE IN (SALARY AS 'Salary', POSITION AS 'Position'));
```

### CASE 표현식

```sql
SELECT ID, NAME, DEPARTMENT_ID, ATTRIBUTE, VALUE
FROM (SELECT ID,
             NAME,
             DEPARTMENT_ID,
             'Salary'        AS ATTRIBUTE,
             TO_CHAR(SALARY) AS VALUE
      FROM EMPLOYEE
      UNION ALL
      SELECT ID,
             NAME,
             DEPARTMENT_ID,
             'Position' AS ATTRIBUTE,
             POSITION   AS VALUE
      FROM EMPLOYEE)
ORDER BY ID, ATTRIBUTE;
```

## 제 8 절 정규 표현식

| 함수명 | 설명 |
| --- | --- |
| REGEXP_LIKE | 주어진 문자열이 정규 표현식과 일치하는지 여부를 TRUE 또는 FALSE로 반환합니다. |
| REGEXP_REPLACE | 주어진 문자열에서 정규 표현식과 일치하는 부분을 다른 문자열로 대체한 후 결과 문자열을 반환합니다. |
| REGEXP_SUBSTR | 주어진 문자열에서 정규 표현식과 일치하는 첫 번째 부분을 반환합니다. |
| REGEXP_INSTR | 주어진 문자열에서 정규 표현식과 일치하는 첫 번째 부분의 위치를 반환합니다. |
| REGEXP_COUNT | 주어진 문자열에서 정규 표현식과 일치하는 부분의 개수를 반환합니다. |

| 연산자 | 설명 | 예제 |
| --- | --- | --- |
| * | 0회 이상 반복 | a* |
| + | 1회 이상 반복 | a+ |
| ? | 0회 또는 1회 반복 | a? |
| {m,n} | m회에서 n회까지 반복 | a{2,4} |
| . | 임의의 문자 하나와 일치 | a.b |
| [ ] | 문자 집합 내의 문자 중 하나와 일치 | [a-c]d |
| ^ | 문자열의 시작 부분과 일치 | ^abc |
| $ | 문자열의 끝 부분과 일치 | xyz$ |
| ` | ` | 두 패턴 중 하나와 일치 (OR 연산자) |
| ( ) | 패턴을 그룹화하여 캡처 | (ab)(cd) |
| \d | 숫자와 일치 | \d\d |
| \w | 단어 문자와 일치 (영문자, 숫자, 밑줄 문자) | \w+ |
| \s | 공백 문자와 일치 (스페이스, 탭, 줄바꿈 등) | a\sb |
| \S | 공백 문자가 아닌 문자와 일치 | a\Sb |
| \D | 숫자가 아닌 문자와 일치 | \D+ |
| \W | 단어 문자가 아닌 문자와 일치 | \W+ |
| [[:digit:]] | 숫자와 일치하는 POSIX 문자 클래스 | [[:digit:]]{3} |
| [[:alnum:]] | 영문자 또는 숫자와 일치하는 POSIX 문자 클래스 | [[:alnum:]]+ |
| [[:alpha:]] | 영문자와 일치하는 POSIX 문자 클래스 | [[:alpha:]]+ |
| [[:space:]] | 공백 문자와 일치하는 POSIX 문자 클래스 | a[[:space:]]b |
| *? | 0회 이상 반복 (비탐욕적) | a*? |
| +? | 1회 이상 반복 (비탐욕적) | a+? |
| ?? | 0회 또는 1회 반복 (비탐욕적) | a?? |
| {m,n}? | m회에서 n회까지 반복 (비탐욕적) | a{2,4}? |
| (?i) | 대소문자를 구분하지 않는 일치 | (?i)abc |
| (?<=) | 긍정형 전방 탐색 (lookbehind) | (?<=a)b |
| (?<!) | 부정형 전방 탐색 (lookbehind) | (?<!a)b |
| (?=) | 긍정형 후방 탐색 (lookahead) | a(?=b) |
| (?!) | 부정형 후방 탐색 (lookahead) | a(?!b) |
| (?>) | 독립적 패턴 (atomic group) | (?>ab)c |
| \b | 단어 경계 | \bword\b |
| \B | 단어 경계가 아닌 위치 | \Bword\B |
| \1, \2 등 | 캡처된 그룹 참조 | (a)(b)\1\2 |
| \A | 문자열의 시작 부분과 일치 (다중 라인 모드에서 사용) | \Aabc |
| \z | 문자열의 끝 부분과 일치 (다중 라인 모드에서 사용) | xyz\z |