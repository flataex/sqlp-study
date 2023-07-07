# 제 1절 소트 튜닝

## 1. 소트와 성능

### 메모리 소트와 디스크 소트

- 메모리 소트
    - 데이터를 메인 메모리영역에서 로드하여 정렬
    - 처리속도 빠름
    - 메모리 크기에 따라 처리할 수 있는 데이터가 제한
    
- 디스크 소트
    - 디스크 공간을 사용하여 정렬
    - 메모리 크기에 상관없이 큰 데이터도 처리 가능
    - 디스크 I/O 작업이 필요하기 때문에 처리속도 느림
    

[One-Pass Sort / MultiPass Sort](https://m.blog.naver.com/darkturtle/50105294679)

### 예시

1. **메모리 정렬 (Memory Sort):**
학교에 충분한 공간(메모리)이 있어서 한 번에 모든 학생들의 성적을 보고 정렬할 수 있다면, 이것이 메모리 정렬의 사례이다. 모든 데이터(학생들의 성적)를 한 번에 처리하며, 메모리에 있는 데이터를 정렬하는 방식

1. **디스크 정렬 (Disk Sort):**
하지만, 학생 수가 너무 많아서 한 번에 모두 처리할 수 없다면, 클래스별로 성적을 나누고 각각을 정렬해야 한다. 이 경우, 디스크에 데이터를 저장하고 필요할 때마다 일부분을 메모리에 로드해 정렬하는 디스크 정렬 방식을 사용한다.
    - **Onepass Sort:** 각 클래스의 정렬된 성적을 모두 한 번에 보고 전체 성적을 정렬할 수 있다면, 이것이 Onepass Sort이다. 모든 데이터를 한 번에 메모리에 올릴 수 있어 한 번의 패스로 정렬이 가능하다.
    - **Multipass Sort:** 하지만, 여전히 데이터가 너무 많아 한 번에 모두 보지 못한다면, 두 개의 클래스 성적을 가져와 합치고 정렬한 뒤, 그 결과를 다음 클래스의 성적과 합친다(Merge). 이 과정을 반복해야 하는데, 이것이 Multipass Sort. 모든 데이터를 한 번에 메모리에 올릴 수 없어 여러 패스를 거쳐 정렬을 진행한다.

### 소트를 발생시키는 오퍼레이션

1. Sort Aggregate
    1. 집계 함수 (예: COUNT, SUM, AVG 등)를 사용할 때 발생
    2. 실제로 소트처리가 발생하지는 않음
2. Sort Order By
    1. Order By절을 통해 실행
    2. 지정된 열을 기준으로 선택한 행들을 정렬
3. Sort Group By
    1. GROUP BY 절에 사용
    2. 데이터를 그룹화 기준으로 정렬
4. Sort Unique 
    1. DISTINCT 키워드가 포함되어있는 경우 사용
    2. 정렬 후 중복행을 제거
5. Sort Join
    1. SORT MERGE JOIN을 수행할때 실행
    2. 두 테이블을 정렬한 후 병합하여 조인을 수행
6. Window Sort 
    1. 윈도우 함수를 수행할 때 나타남
    2. RANK() 함수 등을 계산하기 위해 데이터 정렬 후 계산

### 소트 튜닝 요약

- 데이터 모델 측면에서의 검토
- 소트가 발생하지 않도록 SQL 작성
- 인덱스를 이용한 소트 연산 대체
- 소트 영역을 적게 사용하도록 SQL 작성
- 소트 영역 크기 조정

## 2. 데이터 모델 측면에서의 검토

- [ ]  우리의 DB 사례를 찾아보자.

## 3. 소트가 발생하지 않도록 SQL 작성

### Union을 Union All으로 대체

- 중복가능성이 없는 테이블일 경우 UNION ALL이 더 효율적
- 중복가능성이 있다면 중복제거를 위한 쿼리가 추가로 필요해, 적은 데이터셋에서는 효율성 보장이안됨.

### Distinct를 Exists로 대체

### 불필요한 Count연산 제거

## 4. 인덱스를 이용한 소트 연산 대체

### Sort Order By 대체

### Sort Group By 대체

### Min, Max 구하기

## 5. 소트 영역을 적게 사용하도록 SQL 작성

### 소트 완료 후 데이터 가공

### TOP N 쿼리 사용

```sql
SELECT *
FROM (
  SELECT *
  FROM EMPLOYEE
  ORDER BY SALARY DESC
)
WHERE ROWNUM <= 10;
```

### TOP N 쿼리를 이용한 효과적인 이력조회

- 인덱스를 사용하면 소트 영역을 더욱 줄일 수 있다. 인덱스가 **`ORDER BY`** 절에 사용되는 열을 이미 정렬하고 있으면, 데이터베이스는 해당 열을 다시 정렬할 필요가 없으므로 소트 영역의 사용을 크게 줄일 수 있다.

## 6. 소트 영역 크기 조정

- 값이 작을수록 디스크 I/O가 발생가능성 높아짐
- 값이 클수록 더 많은 정렬 작업이 메모리에서 수행
- 적절하게 조정해야함. → 어떻게?

> **다음주 숙제**
> 

…**플랫타 DB** **시스템 모니터링 해보기**

# 제 2절 DML 튜닝

## 인덱스 유지 비용

- 인덱스 개수가 많을수록 DML 성능이 나빠짐

**WHY?**

1. INSERT → 테이블의 모든 인덱스를 수정해야함. 
새로운 엔트리를 추가하고, 인덱스를 재정렬
2. UPDATE → 변경된 데이터를 반영하기 위해 인덱스도 수정.
3. DELETE → 해당 행을 가리키는 모든 인덱스 엔트리를 제거해야함
4. Redo, Undo 로그도 생성해야함
    1. Redo : 데이터베이스에서 발생하는 모든 변경에 대한 정보를 저장하는 로그
    2. Undo : 데이터베이스에서 변경이 발생하기 이전의 원래 상태를 저장하는 로그
        1. 원본 데이터블록의 Undo 레코드 생성
        2. 인덱스 엔트리의 Undo 레코드 생성

## Insert 튜닝

### Oracle Insert 튜닝

- **Direct Path Insert : 데이터베이스 버퍼 캐시를 우회하고, 데이터를 직접 디스크에 쓰는 방법**

```sql
-- tbl_order 분리할때…?
INSERT /*+ APPEND */ INTO tbl_order_old 
SELECT * FROM tbl_order
WHERE reg_dtm < '2023-07-01';
```

- **nologging 모드 Insert**

## Update 튜닝

## Truncate & Insert 방식 사용

### 조인을 내포한 Update 튜닝

1. **Oracle 수정 가능 조인 뷰 활용**
2. **Merge into 활용**

# 제 3절 데이터베이스 Call 최소화

## 1. 데이터베이스 Call 종류

### -SQL 커서에 대한 작업 요청에 따른 구분

1. Parse Call 
2. Execute Call
3. Fetch Call

### -Call 발생 위치에 따른 구분

1. User Call
    1. Loop 쿼리 자제할것.
    2. Array Proccessing
        
        ```csharp
        context.entities.AddRange(entitiesToInsert);
        context.RemoveRange(entitiesToDelete);
        context.SaveChanges();
        ```
        
    3. 부분범위처리 원리 활용
    4. 페이지 처리
    5. 사용자 정의 함수, 프로시저, 트리거의 적절한 활용
2. Recursive Call
    1. 내부 call

## 2. 데이터베이스 Call과 성능

### - One SQL 구현의 중요성

### - 데이터베이스 Call과 시스템 확장성

## 3. Array Processing 활용

## 4. Fetch Call 최소화

- EF Core에 설정하는것 없음. `excute rawquery` 에는 가능.

## 5. 페이지 처리 활용

- 조회 위주의 어드민은 페이징 처리를 꼭 해주자!

## 6. 분산 쿼리 활용

- 네트워크 데이터 전송량을 줄이는 것이 핵심.

## 7. 사용자 정의 함수 프로시저의 특징과 성능

- 적절히 섞어서 잘 쓰자.

# 제 4절 파티셔닝

## 1. 파티션 개요

- 파티션 단위로 테이블 또는 인덱스를 나누어 저장

## 2. 파티션 유형

- **Range**
    - Range 파티션은 일반적으로 시간 기반의 데이터를 처리할 때 유용하며, 특히 로그 데이터나 거래 데이터와 같은 시계열 데이터에서 자주 사용된다. 예를 들어, 월별로 데이터를 파티션하면 특정 월에 대한 처리를 빠르게 할 수 있음.(Order,Contract,MatchOrder…)
- **Hash**
    - 데이터가 균등하게 분포되어 있지 않을 때 유용
    - 사용자 ID와 같은 칼럼에 hash 파티션을 적용하면, 모든 파티션에 걸쳐 균등한 데이터 분포를 보장
- **List**
    - 칼럼의 특정 값이나 값의 집합을 기반으로 데이터를 분리할 때 유용
    - 국가별로 고객 데이터를 분리하려는 경우, 각 국가 코드를 사용하여 데이터를 파티션할 수 있다.
- **Composite**
    - 두가지 파티션 유형을 조합
    - 먼저 데이터를 날짜별로 `range` 파티션하고, 그다음 각 날짜 파티션 내에서 사용자 ID에 따라 `hash` 파티션할 수 있다.

> **사용중인 테이블은 파티셔닝이 불가해 아래의 작업이 필요하다.**
> 

```sql
-- 새로운 파티션 테이블 생성
CREATE TABLE TEST_FULL_SCAN_EFFECTIVE_NEW
(
    ID    NUMBER,
    VALUE VARCHAR2(100)
)
    PARTITION BY RANGE (ID)
(
    PARTITION p1 VALUES LESS THAN (30001),
    PARTITION p2 VALUES LESS THAN (60001),
    PARTITION p3 VALUES LESS THAN (90001),
    PARTITION p4 VALUES LESS THAN (MAXVALUE)
);

-- 기존 테이블의 데이터를 새로운 테이블로 이동
INSERT INTO TEST_FULL_SCAN_EFFECTIVE_NEW SELECT * FROM TEST_FULL_SCAN_EFFECTIVE;

-- 기존 테이블 삭제
DROP TABLE TEST_FULL_SCAN_EFFECTIVE;

-- 새로운 테이블의 이름을 기존 테이블의 이름으로 변경
RENAME TEST_FULL_SCAN_EFFECTIVE_NEW TO TEST_FULL_SCAN_EFFECTIVE;
```

## 3. 파티션 프루닝

### 정적 파티션 프루닝

- 파티션 키 칼럼을 조건 조회 할 때 발생

### 동적 파티션 프루닝

- 바인드 변수로 조회할 때 발생

### 주의사항

- 파티션 키 칼럼에 대해 가공이 발생하면 안됨.

## 4. 인덱스 파티셔닝

- `Local Index` **vs** `Global Partitioned Index`
- `Prefix Index` **vs** `Non-prefixed Index`
    
    ```sql
    -- Prefixed
    CREATE INDEX sales_idx ON sales (sale_date, product_id) LOCAL;
    -- Non-prefixed
    CREATE INDEX sales_idx ON sales (product_id, sale_date) LOCAL;
    ```
    

# 제 5절 대용량 배치 프로그램 튜닝

## 병렬 처리 활용

- ****FULL과 PARALLEL 힌트를 사용한 예시****

```sql
SELECT /*+ FULL(e) PARALLEL(e, 4) */
    e.NAME
  , e.SALARY
FROM EMPLOYEE e
WHERE e.SALARY > 50000;
```

- ****INDEX-FFS와 PARALLEL_INDEX 힌트를 사용한 예시****

```sql
SELECT /*+ INDEX_FFS(e) PARALLEL_INDEX(e, salary_idx, 4) */
       e.NAME
     , e.SALARY
FROM EMPLOYEE e
WHERE e.SALARY > 50000;
```

- **병렬처리 힌트 종류**

| 힌트 | 설명 |
| --- | --- |
| PARALLEL(alias, degree) | 테이블 또는 인덱스를 병렬로 처리하되, 병렬 처리 수준을 degree로 설정하도록 지시. alias는 테이블 또는 인덱스의 별칭. |
| NOPARALLEL(alias) | 테이블 또는 인덱스를 병렬로 처리하지 않도록 지시. |
| PARALLEL_INDEX(alias, index, degree) | index라는 이름의 인덱스를 병렬로 처리하되, 병렬 처리 수준을 degree로 설정하도록 지시. |
| NOPARALLEL_INDEX(alias, index) | index라는 이름의 인덱스를 병렬로 처리하지 않도록 지시. |
| PQ_DISTRIBUTE(alias, qc_method, producer_method) | 병렬 질의에서 사용할 데이터 분배 방식을 지정. alias는 테이블의 별칭이며, qc_method는 query coordinator가 사용할 분배 방식, producer_method는 producer가 사용할 분배 방식을 나타낸다. |
| NO_PQ_DISTRIBUTE(alias) | 병렬 질의에서 사용할 데이터 분배 방식을 지정하지 않도록 지시. alias는 테이블의 별칭을 나타낸다. |

- **병렬처리 정리 링크**

[오라클 병렬처리(Parallel Processing) 개념 및 용어 정리, 종합페이지](https://jack-of-all-trades.tistory.com/198)

# 제 6절 고급 SQL 활용

## 1. CASE문 활용

- 분기 조건을 포함하는 쿼리를 One-SQL로 구현

## 2. 데이터 복제 기법 활용

- 카티션 곱을 이용해 데이터를 복제하여 활용

## 3. Union All을 활용한 M:M 관계의 조인

- Union All을 이용해 Full Outer Join에서 나타나는 테이블 중복엑세스를 방지

## 4. 페이징 처리

- Call을 줄이기위한 노력

## 5. 윈도우 함수 활용

## 6. With 구문 활용