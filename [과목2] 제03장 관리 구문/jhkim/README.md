### 제 1절 DML

1. INSERT
- 단일 행

```sql
INSERT INTO 테이블명 (칼럼1,칼럼2,...) VALUES (값1,값2,...);
```

- 서브쿼리를 이용한 다중 행

```sql
INSERT INTO 테이블명 (칼럼1,칼럼2,...)
서브쿼리;
```

2. UPDATE

```sql
UPDATE 테이블명
	SET 수정할 컬럼명1 = 수정할 값,
			수정할 칼럼명2 = [서브쿼리]
[WHERE 수정 대상 식별 조건식]
```

3. DELETE

```sql
DELETE [FROM] 테이블명
[WHERE 삭제 대상 식별 조건식];
```

> 알아두면 좋을 내용
> 

<aside>
💡  **DELETE** 문과 **TRUNCATE** 문은 둘 다 SQL에서 데이터를 제거하는 명령어이다. 하지만 이 두 명령어 사이에는 몇 가지 중요한 차이점이 있다.
</aside>

1. 범위
    - DELETE: **`DELETE`** 문은 테이블에서 특정 레코드 또는 조건에 맞는 레코드를 제거하는 데 사용됩니다. WHERE 절을 사용하여 특정 조건에 맞는 레코드만 삭제할 수 있습니다.
    - TRUNCATE: **`TRUNCATE`** 문은 테이블의 모든 레코드를 제거하는 데 사용됩니다. 조건을 지정할 수 없으며, 테이블의 모든 데이터를 일괄 삭제합니다.
2. 속도
    - DELETE: **`DELETE`** 문은 행 단위로 작업하므로, 삭제할 레코드가 많을 경우 작업 속도가 느려질 수 있습니다.
    - TRUNCATE: **`TRUNCATE`** 문은 테이블의 데이터를 일괄 삭제하므로, 일반적으로 **`DELETE`** 문보다 빠릅니다.
3. 트랜잭션 및 로그
    - DELETE: **`DELETE`** 문은 트랜잭션 로그에 기록되므로, 삭제 작업 후 롤백할 수 있습니다. 이로 인해 트랜잭션 로그 공간이 더 빨리 차고, 추가 오버헤드가 발생할 수 있습니다.
    - TRUNCATE: **`TRUNCATE`** 문은 트랜잭션 로그에 최소한의 정보만 기록되므로, 롤백 옵션이 제한적입니다. 그러나 이로 인해 더 적은 로그 공간을 사용하고, 일반적으로 더 빠른 작업을 가능케 합니다.
4. 트리거
    - DELETE: 테이블에서 **`DELETE`** 작업이 수행되면, 관련 트리거가 실행됩니다.
    - TRUNCATE: **`TRUNCATE`** 작업은 테이블 트리거를 실행하지 않습니다.
5. IDENTITY 속성 재설정
    - DELETE: **`DELETE`** 문을 사용하여 테이블의 모든 데이터를 삭제해도, IDENTITY 속성이 있는 열의 값은 재설정되지 않습니다.
    - TRUNCATE: **`TRUNCATE`** 문을 사용하면, IDENTITY 속성이 있는 열의 값이 초기 값으로 재설정됩니다.
    

**결론적으로, `DELETE` 문은 조건부 삭제와 트랜잭션 롤백이 필요한 경우 사용하며, `TRUNCATE` 문은 테이블의 모든 데이터를 빠르게 삭제하고 IDENTITY 값을 초기화할 때 사용합니다.**

1. MERGE

```sql
MERGE
 INTO 타겟 테이블명
USING 소스 테이블명
   ON (조인 조건식)	
 WHEN MATCHED WHEN
		UPDATE
				SET 수정할 칼럼명1 = 수정될 새로운 값1,
						수정할 칼럼명2 = 수정될 새로운 값2,...
		WHEN NOT MATCHED THEN
			INSERT (칼럼1,칼럼2,...)
			VALUES (값1,값2,...)
;
```

### 제 2절 TCL

1. 트랜잭션 개요

<aside>
💡 트랜잭션의 특성 (ACID)

</aside>

| 속성 | 설명 |
| --- | --- |
| Atomicity | 원자성은 트랜잭션의 모든 연산이 완전히 수행되거나 전혀 수행되지 않아야 함을 의미합니다. 트랜잭션 내의 모든 작업이 성공적으로 완료되면 커밋(commit)되며, 그렇지 않으면 롤백(rollback)되어 이전 상태로 되돌립니다. |
| Consistency | 일관성은 트랜잭션의 시작과 종료 시점에 데이터베이스가 일관된 상태를 유지해야 함을 의미합니다. 트랜잭션 수행 전후에 모든 데이터베이스 제약 조건, 트리거, 비즈니스 규칙 등이 만족되어야 합니다. |
| Isolation | 고립성은 동시에 실행되는 트랜잭션들이 서로 영향을 주지 않도록 함을 의미합니다. 이를 통해 각 트랜잭션은 독립적으로 실행되며, 동시성 제어 기법을 사용하여 데이터의 일관성을 보장할 수 있습니다. |
| Durability | 영속성은 트랜잭션이 성공적으로 커밋된 후에는 해당 결과가 데이터베이스에 영구적으로 저장되어야 함을 의미합니다. 시스템 오류, 정전, 장애 등이 발생하더라도 커밋된 트랜잭션 결과는 보존되어야 합니다. |

1. COMMIT

```sql
INSERT blah blah;

COMMIT;

UPDATE blah blah;

COMMIT;
```

1. ROLLBACK

```sql
UPDATE TBL_ORDER SET ORD_PRC = 99999999

ROLLBACK;
```

1. SAVEPOINT

```sql
SAVEPOINT save1;

ROLLBACK TO save1;
```

### 제 3절 DDL

1. CREATE TABLE

```sql
CREATE TABLE ORDER
(
		칼럼명1 데이터유형 [기본값] [NOT NULL]
		...
);
```

- 테이블 생성 시 규칙

| 항목 | Oracle | SQL Server | MySQL |
| --- | --- | --- | --- |
| 최대 길이 | 30자 | 128자 | 64자 |
| 시작 문자 | 영문자 | 영문자 | 영문자 |
| 사용 가능한 문자 | A-Z, a-z, 0-9, _ | A-Z, a-z, 0-9, _ | A-Z, a-z, 0-9, _ |
| 공백 및 특수 문자 | 사용 불가 | 사용 불가 | 사용 불가 |
| 예약어 사용 금지 | 사용 불가 | 사용 불가 | 사용 불가 |
| 대소문자 구분 | 대소문자 무시 | 대소문자 무시 | 기본적으로 구분함 |
| 명명 규칙 | 명확하게 기능 설명 | 명확하게 기능 설명 | 명확하게 기능 설명 |

- 제약조건

| 제약조건 종류 | 설명 |
| --- | --- |
| PRIMARY KEY | 테이블의 각 행(row)을 고유하게 식별할 수 있는 속성(column)을 지정합니다. 중복되거나 NULL 값이 허용되지 않습니다. |
| UNIQUE | 특정 속성(column)에 중복된 값이 없도록 제약합니다. NULL 값은 허용됩니다. |
| FOREIGN KEY | 두 테이블 간의 참조 무결성을 보장하기 위해 사용되는 제약조건으로, 한 테이블의 속성(column)이 다른 테이블의 PRIMARY KEY를 참조합니다. |
| CHECK | 특정 속성(column)의 값에 대한 조건을 지정하여, 해당 조건을 만족하는 값만 입력되게 합니다. |
| NOT NULL | 특정 속성(column)에 NULL 값을 허용하지 않도록 제약합니다. |

1. ALTER TABLE

- 열 추가 (Add column)

```sql
ALTER TABLE employees
ADD email VARCHAR(255);
```

- 열 수정 (Modify column)

```sql
ALTER TABLE employees
MODIFY email VARCHAR(300) NOT NULL;
```

- 열 이름 변경 (Rename column)

```sql
ALTER TABLE employees
CHANGE email email_address VARCHAR(255);
```

- 열 삭제 (Drop column)

```sql
ALTER TABLE employees
DROP COLUMN email_address;
```

- 제약조건 추가 (Add constraint)

```sql
ALTER TABLE employees
ADD CONSTRAINT pk_employee_id PRIMARY KEY (employee_id);
```

- 제약조건 삭제 (Drop constraint)

```sql
ALTER TABLE employees
DROP CONSTRAINT pk_employee_id;
```

- 테이블 이름 변경 (Rename table)

```sql
ALTER TABLE employees
RENAME TO staff;
```

1. RENAME TABLE
- **`ALTER TABLE`** 사용:
    
    ```sql
    ALTER TABLE old_table_name
    RENAME TO new_table_name;
    ```
    
- **`RENAME`** 사용:
    
    ```sql
    RENAME old_table_name TO new_table_name;
    ```
    
1. DROP TABLE

```sql
DROP TABLE 테이블명;
```

1. TRUNCATE TABLE

```sql
TRUNCATE TABLE 테이블명;
```

### 제 4절 DCL

| 명령어 | 설명 | SQL 예시 |
| --- | --- | --- |
| GRANT | 사용자에게 데이터베이스 객체에 대한 권한을 부여합니다. | GRANT SELECT ON employees TO user1; |
| REVOKE | 사용자로부터 데이터베이스 객체에 대한 권한을 박탈합니다. | REVOKE SELECT ON employees FROM user1; |
1. GRANT :
- 사용자 user1에게 employees 테이블에 대한 SELECT 권한 부여:
    
    ```sql
    GRANT SELECT ON employees TO user1;
    ```
    
- 사용자 user1에게 employees 테이블에 대한 SELECT, INSERT, UPDATE, DELETE 권한 부여:
    
    ```sql
    GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO user1;
    ```
    
- 사용자 user1에게 모든 테이블에 대한 SELECT 권한 부여:
    
    ```sql
    GRANT SELECT ON * TO user1;
    ```
    
2. REVOKE :
- 사용자 user1로부터 employees 테이블에 대한 SELECT 권한 박탈:
    
    ```sql
    REVOKE SELECT ON employees FROM user1;
    ```
    
- 사용자 user1로부터 employees 테이블에 대한 SELECT, INSERT, UPDATE, DELETE 권한 박탈:
    
    ```sql
    REVOKE SELECT, INSERT, UPDATE, DELETE ON employees FROM user1;
    ```
    
- 사용자 user1로부터 모든 테이블에 대한 SELECT 권한 박탈:
    
    ```sql
    REVOKE SELECT ON * FROM user1;
    ```