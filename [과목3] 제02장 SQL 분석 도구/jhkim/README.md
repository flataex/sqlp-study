# 제 1절 예상 실행계획

### Explain Plan

```sql
EXPLAIN PLAN FOR
SELECT * FROM emp;
```

### AutoTrace

- AutoTrace를 이용하면 여러가지 실행통계를 확인 할 수 있다.
그런데, `오라클 SQL*Plus` 에서만 실행됨…?

### DBMS_XPLAN

# 제 2절 SQL 트레이스

### SQL 트레이스 수집

```sql
alter session set sql_trace = true;
select * from emp where empno = 7900;
select * from dual;
alter session set sql_trace = false;

select r.value || '/' || lower(t.instance_name) || '_ora_'
 || ltrim(to_char(p.spid)) || '.trc' trace_file
 from v$process p, v$session s, v$parameter r, v$instance t
 where p.addr = s.paddr
 and r.name = 'user_dump_dest'
 and s.sid = (select sid from v$mystat where rownum = 1);
```

- 위 구문으로 trc파일 생성 및 경로확인은 했으나, 현재 사용하고있는 `Oracle Autonomous Database` 에서는 확인할 수 없음…
실행계획때처럼 **DBMS_XPLAN**를  활용하자!!!

```sql
-- 쿼리 실행
select * from emp where empno = 7900;

-- sql ID 확인
SELECT sql_id, sql_text FROM v$sqlarea WHERE sql_text LIKE '%select * from emp%';

-- DBMS_XPLAN
select * from table(dbms_xplan.display_cursor('8am3c6wqp3upw', 0, 'ALLSTATS'));

-- 최근 실행한 SQL 실행계획
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR);
```

- SQL ID는 Oracle 데이터베이스에서 각각의 SQL 문을 구별하는 유일한 식별자이다.
`v$sqlarea` 에서 확인 할 수 있음.

# 제 3절 응답 시간 분석

| 대기 이벤트 유형 | 대기 이벤트 | 설명 |
| --- | --- | --- |
| 라이브러리 캐시 부하 | library cache lock | 다른 세션에서 수정하고 있는 라이브러리 캐시 오브젝트에 액세스하려는 시도 |
|  | library cache pin | 다른 세션에서 이미 고정한 라이브러리 캐시 오브젝트를 고정하려는 시도 |
| 데이터베이스 호출과 네트워크 부하 | SQL*Net message from client | 클라이언트 응용 프로그램에서 서버로 메시지를 보내기를 기다리는 동안 Oracle이 대기할 때 발생 |
|  | SQL*Net message to client | 서버가 클라이언트에 메시지를 보내기를 기다리는 동안 Oracle이 대기할 때 발생 |
| 디스크 I/O 부하 | db file sequential read | Oracle이 한 번에 하나의 블록을 읽을 때 발생 |
|  | db file scattered read | Oracle이 여러 블록을 읽어서 메모리의 여러 위치에 배치할 때 발생 |
| 버퍼 캐시 경합 | buffer busy waits | 세션이 특정 버퍼를 액세스하려고 하지만 해당 버퍼가 다른 세션에 의해 사용 중일 때 발생 |
| 락 관련 대기 이벤트 | enqueue | 세션간의 리소스 경합 때문에 발생 |
|  | latch free | 세션이 래치를 획득하려고 시도하지만 래치가 사용 중일 때 발생 |

### AWR

- `Oracle Autonomous Database` 를 사용할 경우 책에 나온 내용대로 확인이 불가.
오라클 클라우드 로그인해서 해당 인스턴스의 성능허브로 관리해야함.