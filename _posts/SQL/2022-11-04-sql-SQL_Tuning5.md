---
title: "SQL Tuning 5"
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

## 소트 튜닝

---

### 소트 연산에 대한 이해

PGA - sort area, Temp table space 를 활용하여 정렬.

디스크 소트가 발생하는 순간 SQL 수행 성능은 나빠질 수 밖에 없음.  
또한, 디스크 I/O가 발생하는 것도 문제지만, **부분범위 처리를 불가능하게 함으로써** OLTP환경에서 애플리케이션 성능을 저하시키는 주요인이 됨.

SORT 오퍼레이션

1. Sort Aggregate
실제로 데이터를 정렬하진 않는다. Sort Area를 사용한다는 의미

>select sum(sal),max(sal),min(sal),avg(sal) from emp;

2. Sort Order By
데이터를 정렬할 때 나타난다.
>select * from emp order by sal desc;

3. Sort Group By
그룹별 집계를 수행할 때 나타난다.(정렬 순서를 보장하지 않음. Order By 명시)
>select sum(sal),max(sal),min(sal),avg(sal) from emp
>group by deptno;

4. Sort Unique
중복을 제거하기 위해 나타난다.(서브쿼리의 유일성이 보장되면 생략)
Union,minus,Intersect,Distinct사용시 발생됨.

5. Sort Join
소트 머지 조인 수행시 나타난다.

6. Window Sort
윈도우 함수 수행시 나타난다.

---

### 소트가 발생하지 않도록 SQL작성

Union대신 Union All

Minus,Distinct 대신 (Not) Exists 활용

Hash방식은 정렬순서를 지키지않음 다른 조인방식으로 변경

---

### 인덱스를 이용한 소트 연산 생략

**3 티어 환경**에서 부분범위처리는 의미 없을까?  
서버리소스를 수 많은 클라이언트가 공유하는 구조  
클라이언트가 특정 DB커넥션을 독점할 수 없다.  
단위 작업을 마치면 커넥션을 풀에 반환 및 결과를 모두 전송하고 커서를 닫아야함.  
따라서 **쿼리 결과 집합을 조금씩 나눠서 전송하는 방식은 사용할 수 없다.**

부분범위 처리 활용은 **결과집합 출력을 바로 시작할 수 있느냐,**
둘째, **앞쪽 일부만 출력하고 멈출 수 있느냐가 핵심.**

하지만,**부분범위 처리 원리는 3티어 환경에서도 여전히 의미가 다.Top N 쿼리**

Top N 쿼리는 전체 결과집합 중 상위 N개 레코드만 선택하는 쿼리.  
Sort Order By오퍼레이션 대신 COUNT(STOPKEY)가 나타난다.
STOPKEY가 없으면 전체범위 처리를 의미.  

이것을 페이징 처리에 활용
```
select * 
from (
  select rownum no, a.*
  from (
    /* SQL */
  ) a
  where rownum <= (:page *10)
)
where no >= (:page -1) * 10 +1
```
부분범위 처리 가능하도록 SQL을 작성한다.   
=> 인덱스 사용 가능하도록 조건절 구사하고, 조인은 NL조인 위주로 처리하고, Order By절이 있어도 소트연산을 생략할 수 있도록 인덱스를 구성해주는 것을 의미.

인덱스를 이용해 First Row stopkey 가능.

Sort Group By는 인덱스를 이용해 생략가능.  
Sort Group By Nosort

---

### Sort Area를 적게 사용하도록 SQL 작성

출력한뒤 정렬하는 것보다 정렬하고 출력하는 것이 Sort Area를 적게 쓸 수 있다.

Top N 소트  
STOPKEY가 발생하면 소트 연산 횟수와 Sort Area사용량을 최소화

페이징 처리시 between으로 작성금지

윈도우 함수 중 rank나 row_number 함수는 max함수보다 소트 부하가 적다. Top N 소트 알고리즘이 작동하기 때문이다.

