# 제 1절 NL Join

## 기본 메커니즘

- 두개 이상의 테이블에서, 하나의 집합을 기준으로 순차적으로 상대방 테이블의 row 를 결합하여 원하는 결과를 추출하는 테이블 연결 방식
- 결합하기 위해 기준이 되는 테이블(선행) : driving 테이블( OUTER 테이블, 즉 바깥쪽 테이블)
- 결합되어지는 테이블(후행) : driven 테이블(INNER 테이블, 즉 안쪽 테이블)
- NL 조인에서는 드라이빙 테이블의 각 row 에 대하여 loop 방식으로 조인이 되는데 드라이빙 테이블의 집합을 어느정도 줄일 수 있는가에 따라 NL 조인의 성능이 결정됨.

## Ordered vs Leading ?

```sql
SELECT /*+ ORDERED USE_NL(D) */ E.NAME AS EMPLOYEE_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEE E
JOIN DEPARTMENT D ON E.DEPARTMENT_ID = D.ID;
```

- 위의 쿼리는 **`EMPLOYEE`** 테이블을 먼저 검색하고, **`DEPARTMENT`** 테이블을 NL 조인으로 처리하도록 지시.

```sql
SELECT /*+ LEADING(E D) USE_NL(D) */ E.NAME AS EMPLOYEE_NAME, D.DEPARTMENT_NAME
FROM EMPLOYEE E
JOIN DEPARTMENT D ON E.DEPARTMENT_ID = D.ID;
```

- 위의 쿼리는 **`LEADING`** 힌트를 사용하여 조인 순서를 **`EMPLOYEE`** -> **`DEPARTMENT`**로 명시적으로 지정하고, **`USE_NL(D)`**를 사용하여 **`DEPARTMENT`** 테이블에 대해 NL 조인을 사용하도록 지시.

### 차이점

**`LEADING`** 힌트는 조인 순서를 더 세밀하게 제어할 수 있다. 즉, 조인할 테이블의 순서를 명시적으로 지정할 수 있다. 반면에 **`ORDERED`** 힌트는 SQL 문의 FROM 절에 나열된 테이블 순서대로 조인 순서를 강제하므로, 테이블의 순서를 변경하면서 쿼리를 조정해야 할 수 있다. 또한, **`ORDERED`** 힌트는 Oracle 옵티마이저가 테이블의 통계를 완전히 무시하고, FROM 절의 테이블 순서를 그대로 따르게 한다.

- https://coding-factory.tistory.com/756

# 제 2절 Sort Merge Join

## 기본 메커니즘

- 각 테이블에 대해 동시에 독립적으로 데이터를 먼저 읽어 들인다.
- 읽혀진 각 테이블의 데이터를 조인을 위한 연결고리에 대하여 정렬을 수행한다.
- 정렬이 모두 끝난 후에 조인 작업이 수행한다.

## 특징

- **ACCESS 하는 속도를 향상 시킨다.**

SORT MERGE JOIN은 가장 먼저 양쪽 테이블을 Access 하는 과정을 거쳐야 합니다. 이 속도를 빠르게 해 준다면 속도 향상에 도움이 되겠죠. 테이블을 Access 할 때 FULL TABLE SCAN이냐 INDEX RANGE SCAN이냐 하는 등 테이블을 Access 하는 방법을 다양한 방법을 통해 최적화시킨다면 SORT MERGE JOIN의 속도도 자연스럽게 최적화할 수 있습니다.

- **정렬 속도의 향상**

SORT MERGE JOIN은 양쪽 테이블에서 조회한 데이터들을 정렬시켜야 합니다. 이때 조인 조건 컬럼이 이미 정렬되어 있다면 정렬을 하는 작업을 단축시켜 검색 속도 향상에 도움이 될 것입니다.

- **양쪽의 정렬까지 완료되는 속도를 맞추어줌**

SORT MERGE JOIN은 양쪽 테이블을 ACCESS하고 조회한 데이터들을 정렬할때 어느 한쪽이라도 정렬 작업이 종료되지 않으면 한쪽이 대기 상태가 되고 다른 한쪽의 정렬이 완전히 끝날 때까지 조인이 시작될 수 없습니다. 그렇기에 두 테이블 ACCESS속도와 정렬 속도를 최대한 비슷하게 맞추어주는 것이 좋습니다. 비교해야 할 두 테이블의 데이터 양이나 정렬 속도를 고려하여 최대한 맞춰주는 것이 효율성 측면에서 좋습니다.

- **SORT_AREA_SIZE 최적화**

SORT MERGE JOIN은 두 테이블 간의 비교가 이루어지기 전에 수행하는 정렬 작업을 위해 별도의 정렬 공간이 필요하며 이 공간은 SORT_AREA_SIZE 크기만큼 메모리를 할당받아 사용하게 되고, 메모리가 부족하다면 Temporary Table Space를 이용하여 정렬을 수행하게 됩니다. 이때 Temporary Table Space를 사용하면 딜레이가 생기므로 SORT_AREA_SIZE를 적당한 크기로 설정해두는 것이 속도 향상에 도움이 됩니다.

# 제 3절 Hash Join

## 기본 메커니즘

- 둘 중 작은 집합(Build Input)을 읽어 Hash Area에 해시 테이블을 생성한다. (해시 함수에서 리턴 받은 버킷 주소로 찾아가 해시 체인에 엔트리를 연결)
- 반대쪽 큰 집합(Probe Input)을 읽어 해시 테이블을 탐색하면서 JOIN 한다.
- 해시 함수에서 리턴 받은 버킷 주소로 찾아가 해시 체인을 스캔하면서 데이터를 찾는다.

## 특징

- **HASH TABLE을 만드는 과정을 효율화 한다.**

HASH JOIN은 해시 테이블을 생성하는 비용이 수반되므로 이 과정을 효율화하는 것이 성능 개선에 있어 가장 중요합니다. 그렇기에 HASH TABLE로 만들 Build Input이 Hash Area에 담길 정도로 충분히 작아야 하며 Build Input 해시 키 칼럼에 중복 값이 거의 없어야 효율적인 동작을 기대할 수 있습니다.

- **CPU의 성능을 향상한다.**

HASH BUCKET이 조인 집합에 구성되어 해시 함수 결과를 저장해야 하는데 기본적으로 HASH_AREA_SIZE에 지정된 크기만큼의 메모리가 할당되어 사용됩니다. 이러한 처리에는 많은 메모리와 CPU 자원을 소모하게 됩니다. 그렇기에 CPU의 자원이 넉넉하다면 다른 조인에 비해 보다 좋은 효율을 내지만 부족한 상황에서는 다른 조인 방법보다 느려질 수도 있습니다. 그러므로 CPU의 성능을 향상한다면 HASH JOIN의 성능을 향상할 수 있습니다.

- **충분한 PGA 메모리 확보**

Hash Area는 PGA 메모리에 할당되는데 Build Input이 HASH_AREA_SIZE를 초과하게 되면 가장 큰 순서대로 Hash Bucket이 Temporary Table Space로 내려가서 구성됩니다. 디스크로 내려간 Hash Bucket에 변경이 일어날 때마다 디스크 I/O가 발생하게 되어 성능이 현저하게 저하됩니다.

<aside>
💡 **HASH_AREA_SIZE**

- HASH JOIN에 사용되는 최대 메모리 SIZE를 지정하는 설정값.
</aside>

# 제 4절 스칼라 서브 쿼리

```sql
SELECT E.NAME AS EMPLOYEE_NAME, 
    (SELECT D.DEPARTMENT_NAME 
     FROM DEPARTMENT D 
     WHERE E.DEPARTMENT_ID = D.ID) AS DEPARTMENT_NAME
FROM EMPLOYEE E;

SELECT /*+ USE_NL(D E) */ 
  E.NAME AS EMPLOYEE_NAME
, D.DEPARTMENT_NAME
FROM EMPLOYEE E
JOIN DEPARTMENT D ON E.DEPARTMENT_ID = D.ID;
```

- 위의 두 쿼리를 같은 값을 반환.

## 스칼라 서브쿼리 Unnesting

```sql
SELECT /*+ UNNEST */ E.NAME, 
       (SELECT D.DEPARTMENT_NAME FROM DEPARTMENT D WHERE D.ID = E.DEPARTMENT_ID) AS DEPARTMENT_NAME
FROM EMPLOYEE E;
```

# 제 5절 고급 조인 기법

## 인라인 뷰 활용

```sql
SELECT E.NAME, IV.MAX_SALARY
FROM EMPLOYEE E
JOIN (
  SELECT DEPARTMENT_ID, MAX(SALARY) AS MAX_SALARY
  FROM EMPLOYEE
  GROUP BY DEPARTMENT_ID
) IV ON E.DEPARTMENT_ID = IV.DEPARTMENT_ID;
```

- 인라인 뷰는 SQL 문 내에서 쿼리의 일부로 사용되는 서브쿼리
- 임시 테이블처럼 작동하여 다른 테이블과 함께 조인

## 배타적 관계의 조인(**Exclusive Relationships Join**)

```sql
SELECT E.NAME, D1.DEPARTMENT_NAME
FROM EMPLOYEE E
JOIN DEPARTMENT1 D1 ON E.DEPARTMENT_ID = D1.ID
WHERE E.DEPARTMENT_TYPE = 'Type1'
UNION ALL
SELECT E.NAME, D2.DEPARTMENT_NAME
FROM EMPLOYEE E
JOIN DEPARTMENT2 D2 ON E.DEPARTMENT_ID = D2.ID
WHERE E.DEPARTMENT_TYPE = 'Type2';
```

## 부등호 조인

```sql
SELECT E1.NAME, E2.NAME
FROM EMPLOYEE E1
JOIN EMPLOYEE E2 ON E1.SALARY < E2.SALARY;
```

- 일반적으로 조인은 두 테이블 간의 등호(=) 조건을 사용하여 수행되지만, 부등호(<, >, <=, >=)를 사용하여 조인을 수행하는 것이 가능함.

## Between 조인

```sql
SELECT E.NAME, S.GRADE
FROM EMPLOYEE E
JOIN SALARY_GRADE S ON E.SALARY 
BETWEEN S.LOWEST_SALARY AND S.HIGHEST_SALARY;
```

## ROWID 활용

```sql
SELECT E1.NAME, E2.NAME 
FROM EMPLOYEE E1, EMPLOYEE E2
WHERE E1.ROWID = E2.ROWID AND E1.DEPARTMENT_ID = 10;
```

- Oracle에서만 지원함
- 같은 테이블에 대해 여러 번 조인을 수행해야 하는 경우에 **`ROWID`**를 사용하면 쿼리의 성능을 크게 향상시킬 수 있음
- **`ROWID`**는 물리적 주소를 나타내므로, 데이터의 물리적 위치가 변경되면 **`ROWID`**도 변경될 수 있다. 따라서 **`ROWID`**는 주로 일시적인 작업에 사용된다.