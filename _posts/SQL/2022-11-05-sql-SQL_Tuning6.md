---
title: "SQL Tuning 6"
categories:
  - sql
tags:
  - tuning
toc: true
toc_label: "Index"
toc_icon: ""
toc_sticky: true
---
# SQL 튜닝

---

## DML 튜닝

---

### 기본 DML 튜닝

DML 성능에 미치는 요소
+ 인덱스
+ 무결성 제약
+ 조건절
+ 서브쿼리
+ Redo 로깅
+ Undo 로깅
+ Lock
+ 커밋


Array Processing 활용하여 Call부하를 줄일 수 있다.

인덱스 및 제약 해제를 한 뒤 대량 DML처리.  
이후에 제약과 인덱스 생성.

Merge문 활용
```
merge into <table1> using <table or view> on (condition)
when matched then update
  set <col1> = <col2>
  <추가 condition>
  <delete where condition>
when not matched then insert
  (col1,col2,col3,...) values 
  (col1,col2,col3,...)
  <추가 condition>
```
delete문의 경우 조인에 성공하였으며, 조건에 맞는 row만 삭제 가능하다.

---

### Direct Path I/O 활용

Direct Path I/O이 작동할 때

1. 병렬 쿼리로 Full Scan할 때
2. 병렬 DML을 수행할 때
3. Direct Path Insert를 할 때

Direct Path Insert방식으로 입력하는 법
1. Insert ... Select 문에 append 힌트사용
2. parallel 힌트를 이용해 병렬 모드로 Insert
3. CTAS문 수행

Direct Path Insert 방식이 빠른 이유
1. Freelist참조하지 않고 HWM 바깥 영역에 순차 입력
2. 블록을 버퍼캐시에 탐색하지 않음
3. 버퍼캐시에 적재하지 않고, 데이터파일에 직접 기록
4. Undo 로깅을 하지 않음.
5. Redo 로깅을 안하게 할 수 있음.
(ALTER TABLE T NOLOGGING;)

---

### 파티션을 활용한 DML 튜닝

Unique 인덱스를 파티셔닝 하려면, 파티션 키가 모두 인덱스 구성 컬럼이어야 한다.

서비스 중단 없이 파티션 구조를 빠르게 변경하려면, PK를 포함한 모든 인덱스가 로컬 파티션 인덱스여야 한다.

일반적으로 입력/수정/삭제 하는 데이터 비중이 5%를 넘는다면,
인덱스를 그대로 둔 상태에서 작업하기 보다 인덱스 없이 작업한 후에 재생성하는 게 빠르다.

파티션 Exchange를 이용한 대량 데이터변경  
임시테이블을 만들어 기존 테이블 파티션과 exchange함.

---

### Lock과 트랜잭션 동시성 제어

MVCC모델의 오라클은 select 문에 로우 lock을 사용하지 않는다.  
DML과 select 문은 서로 방해하지 않는다.

만약 select문에 lock을 사용한다면 DML과 select문은 서로 진행을 방해할 수 있다.(SQLserver등)


