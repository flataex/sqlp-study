# 제 1절 SQL 옵티마이징 원리

## 1. 옵티마이저 소개

### 옵티마이저 종류

1. 규칙기반 옵티마이저(Rule-Based Optimizer)
    1. 규칙에 따라 실행계획을 선택
    2. 규칙은 액세스 경로 별 우선순위로서, 인덱스 구조,연산자,조건절 형태가 순위를 결정짓는 주요인
2. 비용기반 옵티마이저(Cost-Based Optimizer)
    1. 비용을 기반으로 최적화를 수행
    2. 비용이란 쿼리를 수행하는데 소요되는 일량 또는 시간

### 최적화 목표

1. 전체 처리속도 최적화
    1. 시스템 리소스를 가장 적게 사용하는 실행계획을 선택
    2. Oracle,SQL Server 등 대부분 DBMS의 기본 옵티마이저 모드는 전체 처리속도 최적화에 맞춰져 있다.

```sql
-- 옵티마이저 모드 변경
alter system set optimizer_mode = all_rows;
alter session set optimizer_mode = all_rows;
select /*+ all_rows */ * from table where;
```

1. 최초 응답속도 최적화

## 2. 옵티마이저 행동에 영향을 미치는 요소

### SQL과 연산자 형태

### 옵티마이징 팩터

### DMBS 제약 설정

### 옵티마이저 힌트

### 통계정보

### 옵티마이저 관련 파라미터

### DMBS 버전과 종류

## 3. 옵티마이저의 한계

### 옵티마이징 팩터의 부족

### 통계정보의 부정확성

### 바인드 변수 사용 시 균등분포 가정

### 비현실적인 가정

### 규칙에 의존하는 CBO

### 하드웨어 성능 특성

## 4. 통계정보를 이용한 비용계산 원리

| 통계 종류 | 정보 |
| --- | --- |
| 테이블 통계 | 행 수, 블록 수, 비어있지 않은 블록 수, 평균 행 길이 |
| 컬럼 통계 | 열의 분포, null이 아닌 값의 수, 최소값, 최대값, 밀도 |
| 인덱스 통계 | 인덱스의 레벨 수, 리프 블록 수, 키의 수, 클러스터링 요소 |
| 시스템 통계 | CPU 성능, I/O 성능, 저장소 |

# 제 2절 SQL 공유 및 재사용

## 1. 소프트 파싱 vs 하드 파싱

- 소프트 파싱 : SQL과 실행계획을 캐시에서 찾음.
- 하드 파싱 : SQL과 실행 계획을 캐시에서 찾지 못해 최적화 과정을 거침.

### SQL 공유 및 재사용의 필요성

- 옵티마이저가 SQL 최적화 과정에 사용하는 정보
    - 테이블,칼럼,인덱스 구조에 관한 기본 정보
    - 오브젝트 통계 : 테이블 통계, 인덱스 통계, 칼럼 통계
    - 시스템 통계 : CPU 속도, Single Block I/O 속도, Multiblock I/O 속도 등
    - 옵티마이저 관련 파라미터

### 실행계획 공유 조건

- 캐시에서 SQL을 찾아야하는데 키값이 `SQL 문장 그 자체`

### 실행계획을 공유하지 못하는 경우

1. 공백 문자 또는 줄바꿈
2. 대소문자 구분
3. 주석
4. 테이블 Owner 명시
5. 옵티마이저 힌트 사용
6. 조건절 비교 값

- 그냥 SQL이 다르면 공유못함…

## 2. 바인드 변수 사용

### 바인드 변수의 중요성

- 바인드 변수를 사용하면, **변수의 값이 달라져도 같은 문장으로 인식**하여 불필요한 hard parse 를 없앰

### 바인드 변수 사용 시 주의사항

- 칼럼의 데이터 분포가 균일하지 않을 때 쿼리 성능이 다르게 나타날 수 있음

### 바인드 변수 부작용을 극복하기 위한 노력

1. **바인드 변수 피킹 (Bind Variable Peeking)**
이 기능을 통해 최초 쿼리 실행 시점의 바인드 변수 값을 고려하여 실행 계획을 생성합니다. 즉, 처음 쿼리가 실행될 때 바인드 변수의 값을 "피킹"하고, 그 값을 기반으로 통계를 분석하여 최적의 실행 계획을 만듭니다. 이 기능은 일반적으로 성능을 향상시키지만, 바인드 변수의 값이 크게 달라질 경우 문제가 될 수 있음.
2. **적응적 커서 공유 (Adaptive Cursor Sharing)**
다양한 입력 값에 대해 서로 다른 실행 계획을 선택할 수 있게 해주는 기능. 쿼리가 처음 실행될 때 Oracle은 쿼리의 특성을 분석하고 이후에 다른 값이 입력될 경우 새로운 실행 계획을 선택하게 된다. 이는 바인드 변수 피킹의 단점을 극복하기 위한 기능이다.
3. **SQL 플랜 관리 (SQL Plan Management)**
성능 저하를 방지하기 위해 여러 실행 계획을 비교하고 관리하는 기능. 실행 계획이 변경될 때 발생할 수 있는 성능 저하를 방지하기 위해 존재하며, 새로운 실행 계획이 기존 계획보다 성능이 좋은 경우에만 새로운 계획을 사용하도록 관리.
    
    ```sql
    SELECT sql_handle, plan_name, enabled, accepted
    FROM dba_sql_plan_baselines;
    ```
    

## 3. 애플리케이션 커서 캐싱

1. **공유 커서(Shared Cursor):** 오라클의 라이브러리 캐시에 저장되는 커서로, SQL 또는 PL/SQL 코드를 처리하기 위한 데이터베이스 자원이다. 이러한 커서는 같은 SQL 문장을 여러 세션에서 실행하려고 할 때 재사용될 수 있다. 공유 커서를 사용하면 동일한 쿼리를 실행하는 세션 간에 파싱 정보를 공유함으로써 리소스를 절약하고 성능을 향상시킬 수 있다.
    - 공유 커서는 PGA에 저장
2. **세션 커서(Session Cursor):** 이는 특정 데이터베이스 세션 내에서 사용되는 커서로서, 해당 세션에서 실행중인 SQL 문장을 처리한다. 세션 커서는 주로 사용자가 발행한 SQL 문을 실행하는 데 사용되며, 각 세션은 자체 세트의 세션 커서를 가진다. 세션 커서는 공유 커서를 가리키는 포인터 역할을 수행할 수 있다.
    - 세션 커서는 SGA에 저장
3. **애플리케이션 커서(Application Cursor):** 이는 애플리케이션 레벨에서 관리되는 커서로, 애플리케이션 코드 내에서 SQL 문을 처리하는 데 사용된다. 애플리케이션 커서는 일반적으로 SQL 문의 결과를 가져오거나, 데이터를 처리하는 데 사용되는 커서이다. PL/SQL 블록 내에서 선언되고 사용되는 커서는 이 범주에 속한다. 애플리케이션 커서는 명시적 커서와 암시적 커서로 분류될 수 있다.
    - 애플리케이션 Memory에 저장

## 4. Static SQL vs. Dynamic SQL

### Static SQL

```csharp
using (var connection = new SqlConnection("ConnectionString"))
{
    connection.Open();

    using (var command = new SqlCommand("SELECT * FROM Employee WHERE ID = 123", connection))
    {
        using (var reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine(reader["EmployeeName"]);
            }
        }
    }
}
```

### Dynamic SQL

```csharp
string tableName = "Employee";
string columnName = "Id";
int value = 123;

using (var connection = new SqlConnection("ConnectionString"))
{
    connection.Open();

    var commandText = $"SELECT * FROM {tableName} WHERE {columnName} = @value";
    using (var command = new SqlCommand(commandText, connection))
    {
        command.Parameters.AddWithValue("@value", value);

        using (var reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine(reader["EmployeeName"]);
            }
        }
    }
}
```

# 제 3절 쿼리변환

## 1. 쿼리 변환이란?

- 휴리스틱 쿼리변환
- 비용기반 쿼리변환

## 2. 서브쿼리 Unnesting

- 서브쿼리 Unnesting을 사용하면 옵티마이저가 쿼리의 실행 계획을 더 효율적으로 생성할 수 있다.

## 3. 뷰 Merging

- 뷰 Merging으로 성능이 나빠질 수 있는 경우
    - group by 절 포함
    - select-list에 distinct 포함
- 뷰 Merging이 불가능한 경우
    - 집합 연산자
    - connect by 절
    - ROWNUM pseudo 칼럼
    - select-list에 집계 함수 사용
    - 분석 함수

## 4. 조건절 Pushing

- 조건절 Pushdown
- 조건절 Pullup
- 조인 조건

## 5. 조건절 이행

- 추론을 통해 새로운 조건절을 내부적으로 생성해줌

## 6. 불필요한 조인 제거

## 7. OR 조건을 Union으로 변환

## 8. 기타 쿼리 변환

- 집합 연산을 조인으로 변환
- 조인 칼럼에 IS NOT NULL 조건 추가
- 필터 조건 추가
- 조건절 비교 순서 조정