# 7장 Lock과 트랜잭션 동시성 제어

## 1절 Lock

#### 공유 Lock

> 데이터 조회 시 사용

- 다른 공유 Lock과는 호환되지만 배타적 Lock과는 호환(= 한 리소스에 2개 이상의 Lock 동시 설정) x
- 읽고 있는 리소스를 다른 사용자가 동시에 읽을 순 있지만 변경 x
- 변경중인 리소스 동시에 조회 x

#### 배타적 Lock

> 데이터 변경 시 사용

- Lock 해제될 때까지 다른 트랜잭션 해당 리소스 접근(= 읽기, 수정) x

#### Blocking

> Lock 경합이 발생해 특정 세션이 작업을 진행하지 못하고 멈춘 상태

- 공유 Lock &harr; 배타 Lock 블로킹 가능
- 먼저 Lock을 설정한 트랜잭션 커밋할 때까지 후행 트랜잭션 대기

#### Lock에 의한 성능 저하 최소화 방안

- 트랜잭션 원자성 훼손하지 않는 선에서 트랜잭션 가능한 짧게 구성
- 같은 데이터를 수정하는 트랜잭션이 동시에 수행되지 않도록 구성
- Blocking으로 사용자가 무한정 기다리지 않도록 Lock timeout 설정
- 트랜잭션 격리성 불필요하게 상향 조정 x

#### Row Lock

- insert, update, delete, select for update
- 트랜잭션 커밋 전까지 다른 트랜잭션 변경 불가

#### Table Lock

> Lock을 획득한 트랜잭션이 테이블에서 어떤 작업 수행중인지 알리는 flag<br>
> 후행 트랜잭션은 어떤 테이블 Lock인지 확인 후 진입 여부 결정

- Row Share ex) select for update
- Row Exclusive ex) insert, update, delete
- Share
- Share Row Exclusive
- Exclusive

## 3절 동시성 제어

### 1. 비관적 동시성 제어 vs. 낙관적 동시성 제어

#### 비관적 동시성 제어

> 같은 데이터를 동시에 수정할 것이라고 가정<br>
> 데이터 읽는 시점에 lock 걸고 트랜잭션 완료할 때까지 유지

```sql
select *
from 고객
where 고객번호 = :customer_number for update;

--- 새로운 적립 포인트 계산
update 고객
set 적립 포인트 = :적립포인트
where 고객번호 = :customer_number;
```

```sql
for update nowait # 대기 없이 Exception
for update wait 3 # 3초 대기 후 Exception
```

- `wait`, `nowait` 걸어서 무한정 기다리지 않게함

#### 낙관적 동시성 제어

> 같은 데이터를 동시에 수정하지 않을 것이라 가정<br>
> 데이터 조회 시 Lock 설정 x<br>
> 수정 시점에 다른 사용자에 의해 값이 변경됐는지 검사 필요

```sql
select *
from 고객
where 고객번호 = :customer_number;

--- 새로운 적립 포인트 계산
update 고객
set 적립 포인트 = :적립포인트
where 고객번호 = :customer_number;

if sql%rowcount = 0 then
    alert('다른 사용자에 의해 변경');
end if;
```

```sql
select *
from 고객
where 고객번호 = :customer_number;

--- 새로운 적립 포인트 계산
update 고객
set 적립 포인트 = :적립포인트,
    변경일시 = sysdate
where 고객번호 = :customer_number and
      변경일시 = :mod_dt; # 최종 변경 일시가 앞서 읽은 값과 동일한지 비교
```

#### 다중버전 동시성 제어

> 공유 Lock을 사용하더라도 데이터 일관성이 훼손될 수 있는 문제 해결하기 위해 다중버전 동시성 제어(= MVCC) 도입

MVCC 매커니즘
- 데이터 변경 마다 변경 사항 undo 영역 저장
- 데이터 읽다가 쿼리 시작 시점 이후에 변경된 값을 발견하면 undo 영역에 저장된 정보를 통해 쿼리 시작 시점의 일관성 있는 버전 생성하여 조회