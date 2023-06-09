# 제 1절 인덱스 기본 원리

## 인덱스 구조

### 인덱스 기본 구조

- **B*Tree Index** :
B-Tree Index 트리 구조로, 여러 노드들이 연결되어 있다. 각 노드는 하나 이상의 키 값을 가지며, 키 값은 정렬되어 있다. 또한 각 키 값은 데이터의 실제 위치를 가리키는 포인터와 연결되어 있다.

**Full Table Scan vs Index Range Scan?**

- **Full Table Scan:** 이는 인덱스가 없거나 인덱스를 사용하지 않는 쿼리에 대해 수행된다. 테이블의 모든 행을 순차적으로 검사하므로, 데이터의 크기에 비례한 시간이 소요된다.
    
    예를 들어, **`Emp`** 테이블에서 **`Sal > 5000`**인 모든 행을 찾으려면, Full Table Scan은 **`Emp`** 테이블의 모든 행을 하나씩 확인하며 **`Sal`**가 5000보다 큰지 검사한다… 이 경우, 테이블의 모든 행을 검사해야 하므로 데이터의 크기에 따라 시간이 오래 걸릴 수 있다...
    
- **Index Range Scan:** 이는 인덱스가 있는 컬럼에 대한 쿼리에 대해 수행된다. B-Tree 인덱스의 특성을 이용해, 인덱스의 키 값을 바탕으로 필요한 행을 빠르게 찾을 수 있다..
    
    예를 들어, **`Sal`** 컬럼에 인덱스가 있고 **`Sal > 5000`**인 모든 행을 찾으려면, 데이터베이스는 **`Sal`** 인덱스의 B-Tree 구조를 탐색한다. B-Tree에서 **`Sal`** 값이 5000보다 큰 첫 번째 키 값을 찾은 다음, 그 키 값과 연결된 포인터를 따라가서 실제 행을 찾는다. 그 후, B-Tree 인덱스의 나머지 키 값을 순차적으로 검사하면서 **`Sal`**가 5000보다 큰 모든 행을 찾아간다. 이 경우, 필요한 행만을 빠르게 찾아갈 수 있으므로, Full Table Scan에 비해 훨씬 빠르게 작업을 수행할 수 있다.
    

**아니, 그러면 인덱스 키 설정을 마구마구 해버리면 개꿀아니야?**

→ 아님.

1. **데이터 쓰기 성능 저하:** 인덱스가 있는 컬럼에 데이터를 추가, 수정, 삭제할 때마다 인덱스도 업데이트되어야 한다. 따라서 쓰기 작업이 많은 시스템에서는 인덱스 관리로 인한 성능 저하가 발생할 수 있다.
2. **저장 공간 증가:** 각 인덱스는 디스크 공간을 사용한다. 모든 컬럼에 인덱스를 적용하면 데이터베이스의 크기가 크게 증가할 수 있다.
3. **오버헤드 증가:** 데이터베이스는 인덱스를 유지하기 위해 추가 작업을 수행해야 한다. 이는 데이터베이스 성능에 추가적인 오버헤드를 만들 수 있다.
4. **쿼리 최적화 복잡성 증가:** 데이터베이스의 옵티마이저는 사용 가능한 인덱스 중에서 최적의 인덱스를 선택해야 한다. 인덱스가 많으면 이 선택 과정이 복잡해지고, 때로는 옵티마이저가 최선의 선택을 하지 못할 수 있다.

### 인덱스 스캔 방식

1. Index Range Scan
2. Index Full Scan
3. Index Unique Scan
4. Index Skip Scan
5. Index Fast Full Scan
6. Index Range Scan Descending

### 인덱스 종류

1. **B*Tree 인덱스**
- Unbalanced Index는 생길 수 없지만, 인덱스 파편화(Fragmentation)에 의해 Index Skew 또는 Sparse 현상이 생길 수 있음.

1. **Rebuild**: 인덱스를 재구성하는 작업. 인덱스 재구성은 인덱스의 모든 엔트리를 새로운 인덱스로 복사하여 새로운 저장 공간에 저장한다. 이는 인덱스의 저장 공간을 최적화하고 인덱스 파편화를 제거하는데 도움이 된다. 다만, 이 작업은 상당히 비용이 많이 들고 시스템에 부하를 줄 수 있다.
    
    ```sql
    ALTER INDEX index_name REBUILD;
    ```
    
2. **Coalesce**: 인덱스의 조각 모음 작업이다. 이는 인덱스 내의 연속적인 빈 공간을 병합하여 공간을 재사용할 수 있게 한다. 조각 모음은 인덱스의 공간 효율성을 향상시키지만, 인덱스 내의 모든 공간을 재사용 가능하게 만들지는 않는다.
    
    ```sql
    ALTER INDEX index_name COALESCE;
    ```
    
3. **Shrink Space**: 이 연산은 테이블이나 인덱스의 공간을 축소하여 더 이상 사용되지 않는 공간을 데이터베이스에 반환한다. 이는 테이블이나 인덱스의 크기를 줄이고, 디스크 공간을 절약하는 데 도움이 된다.
    
    ```sql
    ALTER TABLE table_name SHRINK SPACE;
    ```
    

1. 비트맵 인덱스
2. 함수기반 인덱스
3. 리버스 키 인덱스
4. 클러스터 인덱스

### 인덱스 튜닝 기초

1. 범위 스캔이 불가능하거나 인덱스 사용이 불가능한 경우
2. 인덱스 컬럼의 가공
3. 묵시적 형변환

# 제 2절 테이블 엑세스 최소화

## 인덱스 ROWID에 의한 테이블 랜덤 액세스

### 인덱스 ROWID에 의한 테이블 엑세스 구조

- ROWID에 의한 테이블 랜덤 엑세스는 디스크 I/O가 크게 발생하기 때문에 생각보다 효율적이지 않음.

### 클러스터링 팩터

- 클러스터링 팩터(Clustering Factor)는 인덱스의 효율성을 평가하는 중요한 메트릭 중 하나로, 특히 B-tree 인덱스에 적용된다. 클러스터링 팩터는 인덱스가 참조하는 테이블의 행이 얼마나 잘 '클러스터링'되어 있는지, 즉 인덱스의 순서대로 얼마나 잘 정렬되어 있는지를 나타낸다.
- 정렬 순서는 일반적으로 인덱스 키의 데이터 유형에 따라 달라진다. 문자열의 경우 일반적으로 알파벳순 (ABC)으로 정렬한다, 숫자의 경우 일반적으로 오름차순 (123)으로 정렬한다.
→ 하지만 내림차순 인덱스 생성등으로 최적화도 가능!

→ 공통코드 테이블을 인덱스 컬럼의 정렬순으로 테이블 재정의를 해도 좋지않을까?

### 인덱스 손익분기점

- 테이블 전체조회를 할때에는 인덱스 스캔의 효율이 떨어짐

### 손익분기점 극복하기

- Oracle IOT
    - Oracle에서 제공하는 인덱스 조직 테이블(Index-Organized Table, IOT)은 특별한 종류의 테이블로, 데이터와 인덱스를 함께 저장하는 방식을 취한다.
- 테이블 파티셔닝
    
    → 우리의 tbl_order도 파티셔닝으로 처리 할 수 있을까?
    
    ```sql
    CREATE TABLE sales (
                           sale_id NUMBER,
                           sale_date DATE,
                           amount NUMBER
    )
        PARTITION BY RANGE (sale_date)
    (
        PARTITION sales_q1 VALUES LESS THAN (DATE '2023-04-01'),
        PARTITION sales_q2 VALUES LESS THAN (DATE '2023-07-01'),
        PARTITION sales_q3 VALUES LESS THAN (DATE '2023-10-01'),
        PARTITION sales_q4 VALUES LESS THAN (DATE '2024-01-01')
    );
    
    SELECT * FROM sales PARTITION (sales_q1);
    ```
    
- 클러스터 테이블

## 테이블 엑세스 최소화 튜닝

### 인덱스 칼럼 추가

### Covered Index

### Include Index

### IOT, 클러스터형 인덱스, 클러스터 테이블 활용

### 수동으로 클러스터링 팩터 높이기

- 테이블 재정의

### 배치 I/O


# 제 3절 인덱스 스캔 효율화

### 인덱스 선행 칼럼이 범위조건일 때의 비효율

- 범위조건은 뒤로 빼자.

### 범위조건을 In-List로 전환

### 범위조건을 2개 이상 사용할 때의 비효율