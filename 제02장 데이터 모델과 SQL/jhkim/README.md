### 제 1절 정규화

1. 제 1 정규형
- 모든 속성은 반드시 하나의 값을 가져야 한다.

플랫타의 문제점 → tbl_fds_list내의 속성중 fds_states 에서 콤마(,)를 이용한 다중 값이 들어가고있음.

“고령,다회“

→ 개발의 복잡성이 증가함

1. 제 2 정규형
- 엔터티의 일반속성은 주식별자 전체에 종속적이어야 한다

→ tbl_kyc_info.job_type_1의 경우 

근로소득자,자영업자 등의 값이 들어간다.

job_type을 관리하는 엔터티를 생성하고 분리해서 관리해야함.

1. 제 3 정규형
- 엔터티의 일반속성 간에는 서로 종속적이지 않는다.

→ 제 2 정규형과 제 3 정규형이 헷갈림.

1. 반 정규화
- 반 정규화는 데이터베이스의 성능을 향상시키기 위해 정규화된 데이터베이스 구조를 조금 더 단순화하는 과정이다. 반 정규화를 사용하면 쿼리 성능이 향상될 수 있지만, 데이터 무결성이나 저장 공간 측면에서 문제가 발생할 수 있다. 이러한 장단점을 고려하여 반 정규화를 적용해야 한다...
### 제 2절 관계와 조인의 이해


1. 조인

**employees** 

| id | name | department_id |
| --- | --- | --- |
| 1 | 김종훈 | 10 |
| 2 | 권도영 | 20 |
| 3 | 박지은 | 10 |

**departments** 

| id | name |
| --- | --- |
| 10 | BO |
| 20 | AML |


다음과 같은 SQL 쿼리를 사용할 수 있다.

```sql
SELECT *
FROM employees e
JOIN departments d ON e.**department_id**= d.**id**;
```

2. 계층형 데이터 모델

**occupation** 

| occupation_id | parent_id | occupation_name | description |
| --- | --- | --- | --- |
| 1 | NULL | IT Profession | IT 전문직 |
| 2 | 1 | Software Engineer | 소프트웨어 엔지니어 |
| 3 | 1 | Data Analyst | 데이터 분석가 |
| 4 | NULL | Healthcare | 의료 |
| 5 | 4 | Doctor | 의사 |
| 6 | 4 | Nurse | 간호사 |

이 테이블에서 **`parent_id`**가 NULL인 레코드는 최상위 계층에 위치하고, 나머지 레코드들은 상위 직업에 대한 하위 직업으로 구성되어 있다.

**모든 계층을 조회하는 SQL 쿼리**

```sql
SELECT LEVEL, LPAD(' ', (LEVEL - 1) * 2) || occupation_name as occupation_name, description
FROM occupations
START WITH parent_id IS NULL
CONNECT BY PRIOR occupation_id = parent_id
ORDER SIBLINGS BY occupation_name;
```

**쿼리 결과**

| LEVEL | occupation_name | description |
| --- | --- | --- |
| 1 | Healthcare | 의료 |
| 2 | Doctor | 의사 |
| 2 | Nurse | 간호사 |
| 1 | IT Profession | IT 전문직 |
| 2 | Data Analyst | 데이터 분석가 |
| 2 | Software Engineer | 소프트웨어 엔지니어 |

**`START WITH`** 절을 사용하여 최상위 계층부터 시작하고, **`CONNECT BY`** 절을 사용하여 하위 계층을 연결하며 계층 구조를 만든다. **`ORDER SIBLINGS BY`** 절을 사용하여 형제 노드를 **`occupation_name`** 기준으로 정렬한다. 
<br/><br/>

3. 모델이 표현하는 트랜잭션의 이해

~~플랫타의 체결 트랜잭션이……~~

4. Null 속성의 이해

- Null 값의 연산은 언제나 Null이다!
- 집계함수는 Null 값을 제외하고 처리한다.

5. 본질식별자 vs 인조식별자