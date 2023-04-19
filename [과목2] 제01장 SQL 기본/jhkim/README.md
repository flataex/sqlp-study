제 1절 관계형 데이터베이스 개요

1. 데이터 베이스?
    - DMBS
    - 관계형 데이터베이스(Relational Database)

1. SQL(Structured Query Language) 

| 분류 | 종류 | 설명 |
| --- | --- | --- |
| 데이터 조작어(DML) | SELECT | 데이터베이스에서 데이터를 조회할 때 사용하는 명령어 |
|  | INSERT | 데이터베이스에 새로운 데이터를 추가할 때 사용하는 명령어 |
|  | UPDATE | 데이터베이스에서 이미 존재하는 데이터를 수정할 때 사용하는 명령어 |
|  | DELETE | 데이터베이스에서 데이터를 삭제할 때 사용하는 명령어 |
| 데이터 정의어(DDL) | CREATE | 데이터베이스에서 새로운 테이블, 인덱스 등을 생성할 때 사용하는 명령어 |
|  | ALTER | 이미 존재하는 테이블의 구조를 변경할 때 사용하는 명령어 |
|  | DROP | 데이터베이스에서 테이블, 인덱스 등을 삭제할 때 사용하는 명령어 |
|  | TRUNCATE | 테이블의 모든 데이터를 삭제할 때 사용하는 명령어 |
| 데이터 제어어(DCL) | GRANT | 데이터베이스 사용자에게 권한을 부여할 때 사용하는 명령어 |
|  | REVOKE | 데이터베이스 사용자의 권한을 회수할 때 사용하는 명령어 |
| 트랜잭션 제어어(TCL) | COMMIT | 데이터베이스 트랜잭션을 완료하고 변경 사항을 저장할 때 사용하는 명령어 |
|  | ROLLBACK | 데이터베이스 트랜잭션을 취소하고 변경 사항을 원래 상태로 되돌릴 때 사용하는 명령어 |
|  | SAVEPOINT | 데이터베이스 트랜잭션에서 중간 저장점을 설정할 때 사용하는 명령어 |
1. STANDARD SQL 개요
2. 테이블
3. ERD
4. 데이터 유형

| 데이터 유형 | 설명 |
| --- | --- |
| CHAR(n) | 고정 길이 문자열, n은 최대 길이 |
| VARCHAR(n) | 가변 길이 문자열, n은 최대 길이 |
| TEXT | 가변 길이 문자열, 매우 큰 길이 |
| INT | 정수, 일반적으로 4바이트 |
| BIGINT | 큰 정수, 일반적으로 8바이트 |
| FLOAT(p) | 단정도 부동 소수점, p는 유효 자릿수 |
| DOUBLE(p) | 배정도 부동 소수점, p는 유효 자릿수 |
| DECIMAL(p,s) | 고정 소수점, p는 전체 자릿수, s는 소수점 이하 자릿수 |
| DATE | 날짜, 'YYYY-MM-DD' 형식 |
| TIME | 시간, 'HH:MM:SS' 형식 |
| DATETIME | 날짜와 시간, 'YYYY-MM-DD HH:MM:SS' 형식 |
| TIMESTAMP | 날짜와 시간, 'YYYY-MM-DD HH:MM:SS' 형식, 보통 시간대까지 포함 |

**char**와 **varchar**는 데이터베이스에서 문자열 데이터를 저장하기 위해 사용되는 두 가지 데이터 타입이다. 각각의 데이터 타입은 저장 방식과 속성이 다르므로, 사용 사례에 따라 적합한 타입을 선택해야 한다.

1. **`char`** (고정 길이 문자열) : **`char`** 타입은 고정 길이의 문자열을 저장한다. 선언할 때 지정한 길이보다 짧은 문자열이 저장되면, 나머지 공간에 공백 문자가 채워진다. 이로 인해 공간 낭비가 발생할 수 있지만, 문자열 길이가 일정한 경우에는 빠른 검색 속도를 제공함.

예시: 주민등록번호 (항상 13자리

```sql
CREATE TABLE people (
  id INT PRIMARY KEY,
  social_security_number CHAR(13)
);
```

1. **varchar** (가변 길이 문자열) : **`varchar`** 타입은 가변 길이의 문자열을 저장한다. 저장되는 문자열의 길이에 따라 공간을 할당하므로, 공간 사용이 효율적이다. 하지만, 문자열 길이가 다양하면 검색 속도가 느려질 수 있음!!!

예시: 이메일 주소 (길이가 다양함)

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  email VARCHAR(255)
);
```

요약하면, **char**는 고정 길이의 문자열을 저장할 때 사용하며, **varchar**는 가변 길이의 문자열을 저장할 때 사용한다. 문자열의 길이가 일정하면 **char**를 사용하여 검색 속도를 높이고, 문자열의 길이가 다양하면 **varchar**를 사용하여 공간 사용을 최적화할 수 있다.

제 2절 SELECT 문

1. SELECT
2. 산술 연산자와 합성 연산자

제 3절 함수

1. 내장 함수
 <br/><br/>   
    
2. 문자형 함수

| 함수 종류 | 설명 | 예시 |
| --- | --- | --- |
| LENGTH | 문자열의 길이를 반환합니다. | SELECT LENGTH('Hello, world!') FROM dual; 결과: 13 |
| CONCAT | 두 개의 문자열을 연결합니다. | SELECT CONCAT('Hello', ', world!') FROM dual; 결과: 'Hello, world!' |
|  |  | (더블 파이프) |
| SUBSTR | 문자열의 일부분을 추출합니다. | SELECT SUBSTR('Hello, world!', 1, 5) FROM dual; 결과: 'Hello' |
| UPPER | 문자열의 모든 문자를 대문자로 변환합니다. | SELECT UPPER('Hello, world!') FROM dual; 결과: 'HELLO, WORLD!' |
| LOWER | 문자열의 모든 문자를 소문자로 변환합니다. | SELECT LOWER('Hello, world!') FROM dual; 결과: 'hello, world!' |
| TRIM | 문자열의 앞뒤 공백을 제거합니다. | SELECT TRIM(' Hello, world! ') FROM dual; 결과: 'Hello, world!' |
| REPLACE | 문자열에서 지정한 문자를 다른 문자로 대체합니다. | SELECT REPLACE('Hello, world!', 'world', 'Oracle') FROM dual; 결과: 'Hello, Oracle!' |
| INSTR | 문자열에서 지정한 문자가 처음 나타나는 위치를 반환합니다. | SELECT INSTR('Hello, world!', 'world') FROM dual; 결과: 8 |
| REVERSE | 문자열의 문자 순서를 뒤집습니다. | SELECT REVERSE('Hello, world!') FROM dual; 결과: '!dlrow ,olleH' |

**From daul;이 도대체 뭐야?**

오라클에서는 "FROM dual"을 사용하여 문자열 함수를 테스트할 수 있다. 이는 가상 테이블로, 오라클 데이터베이스에서 제공하는 기능이다.

3. 숫자형 함수

| 함수 종류 | 설명 | 예시 |
| --- | --- | --- |
| ABS | 절대값을 반환합니다. | SELECT ABS(-42) FROM dual; 결과: 42 |
| CEIL | 주어진 숫자보다 크거나 같은 가장 작은 정수를 반환합니다. | SELECT CEIL(42.3) FROM dual; 결과: 43 |
| FLOOR | 주어진 숫자보다 작거나 같은 가장 큰 정수를 반환합니다. | SELECT FLOOR(42.9) FROM dual; 결과: 42 |
| ROUND | 숫자를 반올림하여 가장 가까운 정수 또는 지정된 자릿수까지 반환합니다. | SELECT ROUND(42.5) FROM dual; 결과: 43 |
| TRUNC | 숫자의 소수점 이하를 잘라내고 정수 또는 지정된 자릿수까지 반환합니다. | SELECT TRUNC(42.9) FROM dual; 결과: 42 |
| MOD | 첫 번째 인수를 두 번째 인수로 나눈 나머지를 반환합니다. | SELECT MOD(10, 3) FROM dual; 결과: 1 |
| POWER | 숫자의 거듭제곱 값을 반환합니다. | SELECT POWER(2, 3) FROM dual; 결과: 8 |
| SQRT | 숫자의 제곱근 값을 반환합니다. | SELECT SQRT(16) FROM dual; 결과: 4 |
| SIN | 주어진 각도(라디안 단위)의 사인 값을 반환합니다. | SELECT SIN(1) FROM dual; 결과: 0.841470... |
| COS | 주어진 각도(라디안 단위)의 코사인 값을 반환합니다. | SELECT COS(1) FROM dual; 결과: 0.540302... |
| TAN | 주어진 각도(라디안 단위)의 탄젠트 값을 반환합니다. | SELECT TAN(1) FROM dual; 결과: 1.557407... |
| ASIN | 주어진 숫자의 역사인(아크사인) 값을 라디안 단위로 반환합니다. | SELECT ASIN(0.5) FROM dual; 결과: 0.523598... |
| ACOS | 주어진 숫자의 역코사인(아크코사인) 값을 라디안 단위로 반환합니다. | SELECT ACOS(0.5) FROM dual; 결과: 1.047197... |
| ATAN | 주어진 숫자의 역탄젠트(아크탄젠트) 값을 라디안 단위로 반환합니다. | SELECT ATAN(1) FROM dual; 결과: 0.785398... |
| ATAN2 | 두 숫자에 대한 역탄젠트(아크탄젠트) 값을 라디안 단위로 반환합니다. | SELECT ATAN2(1, 1) FROM dual; 결과: 0.785398... |
| EXP | 주어진 인수를 사용하여 자연상수 e의 거듭제곱 값을 반환합니다. | SELECT EXP(1) FROM dual; 결과: 2.718281... |
| --- | --- | --- |
| LOG | 주어진 숫자의 자연로그 값을 반환합니다. | SELECT LOG(2.718281) FROM dual; 결과: 1 |
| LN | 주어진 숫자의 자연로그 값을 반환합니다. LOG와 동일합니다. | SELECT LN(2.718281) FROM dual; 결과: 1 |
| LOG10 | 주어진 숫자의 상용로그(밑이 10인 로그) 값을 반환합니다. | SELECT LOG10(100) FROM dual; 결과: 2 |
| GREATEST | 주어진 인수 중 가장 큰 값을 반환합니다. | SELECT GREATEST(10, 20, 30) FROM dual; 결과: 30 |
| LEAST | 주어진 인수 중 가장 작은 값을 반환합니다. | SELECT LEAST(10, 20, 30) FROM dual; 결과: 10 |
| SIGN | 숫자의 부호를 반환합니다. (음수: -1, 양수: 1, 0: 0) | SELECT SIGN(-42) FROM dual; 결과: -1 |
4. 날짜형 함수

| 함수 종류 | 설명 | 예시 |
| --- | --- | --- |
| SYSDATE | 현재 시스템 날짜와 시간을 반환합니다. | SELECT SYSDATE FROM dual; 결과: 17-APR-23 10:24:32 |
| CURRENT_DATE | 세션 시간대의 현재 날짜와 시간을 반환합니다. | SELECT CURRENT_DATE FROM dual; 결과: 17-APR-23 10:24:32 |
| EXTRACT | 날짜에서 지정된 필드(예: YEAR, MONTH, DAY 등)의 값을 추출합니다. | SELECT EXTRACT(YEAR FROM SYSDATE) FROM dual; 결과: 2023 |
| TO_DATE | 문자열을 날짜 형식으로 변환합니다. | SELECT TO_DATE('2023-04-17', 'YYYY-MM-DD') FROM dual; 결과: 17-APR-23 |
| TO_CHAR | 날짜를 문자열 형식으로 변환합니다. | SELECT TO_CHAR(SYSDATE, 'YYYY-MM-DD') FROM dual; 결과: '2023-04-17' |
| ADD_MONTHS | 날짜에 지정된 개월 수를 더하거나 뺍니다. | SELECT ADD_MONTHS(SYSDATE, 3) FROM dual; 결과: 17-JUL-23 |
| LAST_DAY | 지정된 날짜의 달에서 마지막 날짜를 반환합니다. | SELECT LAST_DAY(SYSDATE) FROM dual; 결과: 30-APR-23 |
| NEXT_DAY | 지정된 날짜의 다음 요일을 반환합니다. | SELECT NEXT_DAY(SYSDATE, 'FRIDAY') FROM dual; 결과: 21-APR-23 |
| MONTHS_BETWEEN | 두 날짜 사이의 개월 수를 반환합니다. | SELECT MONTHS_BETWEEN(TO_DATE('2023-04-17', 'YYYY-MM-DD'), TO_DATE('2023-01-01', 'YYYY-MM-DD')) FROM dual; 결과: 3.5161... |
| ROUND | 날짜를 지정한 단위(예: YEAR, MONTH, DAY 등)로 반올림합니다. | SELECT ROUND(SYSDATE, 'MONTH') FROM dual; 결과: 01-MAY-23 |
| TRUNC | 날짜를 지정한 단위(예: YEAR, MONTH, DAY 등)로 절삭합니다. | `SELECT TRUNC(SYSDATE, 'MONTH') FROM dual; 결과: 01-APR-23` |
5. 변환형 함수

| 함수 | 설명 | 예시 |
| --- | --- | --- |
| TO_NUMBER | 문자열을 숫자로 변환합니다. | TO_NUMBER('1000') |
| TO_CHAR | 숫자를 문자열로 변환합니다. | TO_CHAR(1000) |
| TO_DATE | 문자열을 날짜로 변환합니다. | TO_DATE('2021-09-01', 'YYYY-MM-DD') |
| TO_CHAR | 날짜를 문자열로 변환합니다. | TO_CHAR(SYSDATE, 'YYYY-MM-DD') |
| TO_NCHAR | 숫자, 날짜, 문자열을 NVARCHAR2 데이터 유형으로 변환합니다. | TO_NCHAR(1000) |

**?명시적 변환과 암시적 변환?**

<aside>
💡

명시적 데이터 유형 변환과 암시적 데이터 유형 변환은 프로그래밍 및 데이터베이스에서 데이터 유형 간의 변환을 수행하는 두 가지 방법이다.

- 명시적 데이터 유형 변환:
명시적 변환은 개발자가 직접 코드에서 데이터 유형을 변환하도록 지정하는 방식이다. 이 방식은 변환하려는 유형을 명확히 지정하기 때문에, 의도치 않은 결과를 방지할 수 있다. 명시적 변환은 변환 함수를 사용하거나, 캐스팅 연산자를 사용하여 수행된다.

    - **`TO_NUMBER('100')`**: 문자열 '100'을 숫자 100으로 변환.
    - **`TO_DATE('2021-09-01', 'YYYY-MM-DD')`**: 문자열 '2021-09-01'을 날짜로 변환.
    - **`CAST('100' AS NUMBER)`**: 문자열 '100'을 숫자 100으로 변환.
- 암시적 데이터 유형 변환:
암시적 변환은 프로그래밍 언어 또는 데이터베이스 관리 시스템(DBMS)이 자동으로 데이터 유형을 변환하는 방식. 이 방식은 코드가 간결해지지만, 의도치 않은 결과가 발생할 수 있으므로 주의가 필요하다. 암시적 변환은 데이터 유형이 호환되는 경우에만 발생하며, 호환되지 않는 경우 오류가 발생한다.

    - **`SELECT salary + '100' FROM employee;`**: 여기서 '100'이 숫자 100으로 암시적 변환이 발생. 결과적으로 salary 컬럼의 각 값에 100이 더해진다.
    - **`SELECT hire_date + 7 FROM employee;`**: 여기서 7이 날짜 간격으로 암시적 변환되어 hire_date 컬럼의 각 날짜에 7일이 더해진다.

명시적 변환과 암시적 변환의 주요 차이점은 개발자가 데이터 유형 변환을 직접 지정하는지 여부이다.

</aside>

6. Null 관련 함수
    
    Oracle의 NVL 함수는 SQL에서 사용되는 함수로, NULL 값을 다른 값으로 대체하는 데 사용된다.
    
    ```sql
    NVL(expression1, expression2)
    ```
    
    이 함수는 두 개의 인수를 사용
    
    - expression1: NULL 값을 확인할 표현식.
    - expression2: expression1이 NULL인 경우 대체할 값.
    
    NVL 함수는 expression1이 NULL이 아닌 경우 해당 값을 반환하고, NULL인 경우 expression2를 반환한다.
    
    >예시1
    
    ```sql
    SELECT employee_id, NVL(salary, 3000) AS adjusted_salary
    FROM employees;
    ```
    
    >예시2. NVL 함수를 사용하여 두 개의 날짜 간의 차이를 계산하고, 둘 중 하나가 NULL인 경우 기본값으로 0을 반환할 수 있다.
    
    ```sql
    SELECT employee_id, NVL((end_date - start_date), 0) AS work_duration
    FROM work_history;
    ```
    
    이처럼 NVL 함수는 데이터베이스에서 NULL 값을 다루는 데 유용하며, 특정 열의 값이 없거나 누락된 경우 기본값을 제공하거나 결과를 조정하는 데 사용할 수 있다.

    - MYSQL : IFNULL
    - MSSQL : ISNULL
    - ORACLE : NVL
    

제 4절 WHERE 조건절

제 5절 GROUP BY, HAVING 절

제 6절 ORDER BY 절

제 7절 JOIN

제 8절 표준조인

1. **USING 절** 

USING 절은 두 테이블에서 동일한 이름의 열을 기준으로 조인을 수행할 때 사용된다. USING을 사용하면 두 테이블 간의 공통 열을 한 번만 명시하면 되어 편리함. 
```sql
SELECT *
FROM table1
JOIN table2
USING (common_column);
```

2. **Cross Join**
크로스 조인(cross join)은 두 테이블의 모든 행에 대한 조합을 생성한다.
**그럼 도대체 얘는 어느상황에 쓰는가???**

- 크로스 조인은 경우의 수를 구할 때 유용할 수 있다.
    - 상품과 프로모션의 모든 가능한 조합을 찾고자 할 때
    - 국가와 도시의 모든 가능한 조합을 찾고자 할 때
    - 색상과 사이즈의 모든 가능한 조합을 찾고자 할 때