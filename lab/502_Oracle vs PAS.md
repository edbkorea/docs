# Day 5. EDB Postgres vs Oracle

## 1. 오라클과의 차이점
#### 1.1. UPDATE key column
```sql
edb=# create table update_pk (a int primary key );
CREATE TABLE
edb=#
edb=# insert into update_pk values(100);
INSERT 0 1
edb=# insert into update_pk values(101);
INSERT 0 1
edb=# insert into update_pk values(102);
INSERT 0 1
edb=#
edb=# update update_pk set a = a+1; 
ERROR:  duplicate key value violates unique constraint "update_pk_pkey"
DETAIL:  Key (a)=(101) already exists.
edb=#
```
* Reason

  PostgreSQL/PPAS 는 기본적으로 해당로우 변경시 그 즉시 중복체크를 수행하므로 에러가 나게 됩니다.

* Solution

	```sql
  CREATE TABLE update_pk
  (
    a integer NOT NULL,
    CONSTRAINT update_pkx PRIMARY KEY (a)
    DEFERRABLE INITIALLY IMMEDIATE
  )
  ```
* 주의사항

  1. ALTER TABLE 명령어로 속성 변경 불가
  2. Immediate checking 대비 성능 저하
    ```
    When a UNIQUE or PRIMARY KEY constraint is not deferrable,
    PostgreSQL checks for uniqueness immediately whenever a row is inserted or modified.
    The SQL standard says that uniqueness should be enforced only at the end of the statement;
    this makes a difference when, for example, a single command updates multiple key values.
    To obtain standard-compliant behavior, declare the constraint as DEFERRABLE but not deferred (i.e., INITIALLY IMMEDIATE).
    Be aware that this can be significantly slower than immediate uniqueness checking.
    ```

#### 1.2. Character Byte
* in Oracle

  ```sql
  SQL> create table column_byte (
    a varchar2 (1),
    b varchar2 (1 byte),
    c varchar2 (1 char)
  );
    2    3    4    5
  Table created.

  SQL> insert into column_byte (a) values ('가');
  insert into column_byte (a) values ('가')
                                      *
  ERROR at line 1:
  ORA-12899: "SYS"."COLUMN_BYTE"."A" 열에 대한 값이 너무 큼(실제: 3, 최대값: 1)

  SQL> insert into column_byte (b) values ('가');
  insert into column_byte (b) values ('가')
                                      *
  ERROR at line 1:
  ORA-12899: "SYS"."COLUMN_BYTE"."B" 열에 대한 값이 너무 큼(실제: 3, 최대값: 1)

  SQL> insert into column_byte (c) values ('가');

  1 row created.

  SQL>
  ```

* in PPAS

  ```sql
  edb=# create table column_byte (a varchar(1));
  CREATE TABLE
  edb=#
  edb=# insert into column_byte (a) values ('가');
  INSERT 0 1
  edb=#
  edb=# select length(a), lengthb(a) from column_byte;
   length | lengthb
  --------+---------
        1 |       3
  (1 row)

  edb=#
  ```

* Reason

  PPAS에서는 byte 가 아닌 글자수를 의미

* 주의사항
  ORACLE에서 PostgreSQL/PPAS 로 데이터를 이관하면 문제될게 전혀없지만, 그 반대의 경우 한글 데이터가 들어가는 컬럼이라면 오라클쪽에서 컬럼 사이즈 조정이 필요

#### 1.3. AUTOCOMMIT & PL/SQL

```sql
edb=# \set AUTOCOMMIT on
edb=#
edb=# create table x (x int primary key);
CREATE TABLE
edb=#
edb=# create table y (y int primary key);
CREATE TABLE
edb=#
edb=# insert into y values(100);
INSERT 0 1
edb=#
edb=# create or replace procedure tx_test()
edb-# is
edb$# begin
edb$# insert into x values(100);
edb$# insert into y values(100);
edb$# exception when others then
edb$# dbms_output.put_line(sqlerrm);
edb$# end;
CREATE PROCEDURE
edb=#
edb=# select tx_test;
duplicate key value violates unique constraint "y_pkey"
 tx_test
---------

(1 row)

edb=#
edb=#
edb=# select count(*) from x;
 count
-------
     0
(1 row)

edb=#
```

* Reason
	오라클에서는 sql 에서 에러 발생시 에러 발생시킨 문장만 롤백이 되지만 PPAS에서는 문장이 아닌 트랜잭션 블럭 전체가 롤백됨

* Workaround

	```sql
  edb=# create or replace procedure tx_test()
  edb-# is
  edb$# begin
  edb$# set edb_stmt_level_tx = on;
  edb$# insert into x values(100);
  edb$# insert into y values(100);
  edb$# exception when others then
  edb$# dbms_output.put_line(sqlerrm);
  edb$# end;
  CREATE PROCEDURE
  edb=#
  edb=# select tx_test;
  duplicate key value violates unique constraint "y_pkey"
   tx_test
  ---------

  (1 row)

  edb=#
  edb=#
  edb=# select count(*) from x;
   count
  -------
       0
  (1 row)

  edb=#
  ```
* 주의사항!
	`edb_stmt_level_tx`를 사용하면 성능 저하가 발생할 수 있음

#### 1.4. P.K & Unique Constraint

```sql
edb=# create table pk_uc (a int);
CREATE TABLE
edb=# create unique index idx_pk_uc on pk_uc (a);
CREATE INDEX
edb=# alter table pk_uc add constraint pk_pk_uc primary key (a);
ALTER TABLE
edb=#
edb=# select * from pg_indexes where tablename = 'pk_uc';
  schemaname  | tablename | indexname | tablespace |                        indexdef
--------------+-----------+-----------+------------+--------------------------------------------------------
 enterprisedb | pk_uc     | idx_pk_uc |            | CREATE UNIQUE INDEX idx_pk_uc ON pk_uc USING btree (a)
 enterprisedb | pk_uc     | pk_pk_uc  |            | CREATE UNIQUE INDEX pk_pk_uc ON pk_uc USING btree (a)
(2 rows)

edb=#
edb=#
edb=# alter table pk_uc drop constraint pk_pk_uc;
ALTER TABLE
edb=#
edb=# select * from pg_indexes where tablename = 'pk_uc';
  schemaname  | tablename | indexname | tablespace |                        indexdef
--------------+-----------+-----------+------------+--------------------------------------------------------
 enterprisedb | pk_uc     | idx_pk_uc |            | CREATE UNIQUE INDEX idx_pk_uc ON pk_uc USING btree (a)
(1 row)

edb=#
```

* Reason

	PPAS는 ALTER TABLE table_name ADD CONSTRAINT 등의 명령을 이용하여 PK 이나 Unique 제약조건 생성시 인덱스가 자동으로 생성됨
	여기서 오라클과의 차이는 PPAS는 인덱스 구성컬럼이 동일하다 하더라도 인덱스명만 다르면 중복생성이 가능

* 주의사항

	인덱스가 중복 생성 되지 않도록 관리가 필요

#### 1.5. CHAR VS VARCHAR VS VARCHAR2

```sql
edb=# create table test(i int, x varchar(10), y varchar(10), z char(10));
CREATE TABLE
edb=#
edb=#
edb=# insert into test(i,x,y,z) values(1,'abc','abc','abc');
INSERT 0 1
edb=#
edb=# insert into test(i,x,y,z) values(2,'abc       ','abc','abc');  -- abc 문자뒤에 공백 7개
INSERT 0 1
edb=#
edb=# insert into test(i,x,y,z) values(3,'abc    ','abc','abc');   -- abc 문자뒤에 공백 4개
INSERT 0 1
edb=#
edb=# insert into test(i,x,y,z) values(4,'abc    ','abc','abc     '); -- abc 문자뒤에 공백 4개
INSERT 0 1
edb=#
edb=# select i,x,y,z, length(x), length(y), length(z) from test order by i;
 i |     x      |  y  |     z      | length | length | length
---+------------+-----+------------+--------+--------+--------
 1 | abc        | abc | abc        |      3 |      3 |      3
 2 | abc        | abc | abc        |     10 |      3 |      3
 3 | abc        | abc | abc        |      7 |      3 |      3
 4 | abc        | abc | abc        |      7 |      3 |      3
(4 rows)

edb=#
edb=# select i,x,y,z from test where x=y;
 i |  x  |  y  |     z
---+-----+-----+------------
 1 | abc | abc | abc
(1 row)

edb=#
edb=# select i,x,y,z from test where x=z;
 i |     x      |  y  |     z
---+------------+-----+------------
 1 | abc        | abc | abc
 2 | abc        | abc | abc
 3 | abc        | abc | abc
 4 | abc        | abc | abc
(4 rows)

edb=#
```

* 주의사항

	PPAS의 char 타입의 경우 오라클과 같이 크기가 고정이 되지 않고 문자열 뒤의 공백은 제거됨

#### 1.6. NULL & `''`

```sql
edb=# create table null_empty (a varchar);
CREATE TABLE
edb=#
edb=# insert into null_empty values ('aaa');
INSERT 0 1
edb=#
edb=# insert into null_empty values ('not null');
INSERT 0 1
edb=#
edb=# insert into null_empty values (NULL); -- 명시적으로 NULL 값 입력
INSERT 0 1
edb=#
edb=# insert into null_empty values ('');   -- 공백 문자열 입력
INSERT 0 1
edb=#
edb=# select count(*) from null_empty where a is null;
 count
-------
     1
(1 row)

edb=#
edb=# select count(*) from null_empty where a is not null;
 count
-------
     3
(1 row)

edb=#
edb=# select a from null_empty where a is not null;
    a
----------
 aaa
 not null
                -- there are empty string data
(3 rows)

edb=#
```

* 주의사항!

	오라클에서는 ''(empty string) 과 NULL 은 동일하지만 PPAS에서는 '' 은 절대 NULL이 아님

#### 1.7. NULL & Index

```sql
edb=# create table null_idx as select generate_series(1,1000) x;
SELECT 1000
edb=#
edb=# create unique index idx_null_idx on null_idx (x);
CREATE INDEX
edb=#
edb=# insert into null_idx values (null);
INSERT 0 1
edb=# insert into null_idx values (null);
INSERT 0 1
edb=#
edb=# explain analyze select * from null_idx where x=1;
                                                         QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------
 Index Only Scan using idx_null_idx on null_idx  (cost=0.28..8.29 rows=1 width=4) (actual time=0.049..0.050 rows=1 loops=1)
   Index Cond: (x = 1)
   Heap Fetches: 1
 Planning time: 0.092 ms
 Execution time: 0.087 ms
(5 rows)

edb=#
edb=# explain analyze select * from null_idx where x is null;
                                                         QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------
 Index Only Scan using idx_null_idx on null_idx  (cost=0.28..4.29 rows=1 width=4) (actual time=0.021..0.024 rows=2 loops=1)
   Index Cond: (x IS NULL)
   Heap Fetches: 2
 Planning time: 0.070 ms
 Execution time: 0.048 ms
(5 rows)

edb=#
```

* Reason

  카탈로그 테이블 조회
  ```sql
  edb=# select amname, amsearchnulls from pg_am;
   amname | amsearchnulls
  --------+---------------
   btree  | t
   hash   | f
   gist   | t
   gin    | f
   spgist | t
   brin   | t
  (6 rows)

  edb=#
  ```

* pg_am Catalog Table Column Description

  - amname : Name of the access method
  - amsearchnulls :  "Does the access method support IS NULL/NOT NULL searches?"
  - btree 타입의 인덱스에서는 인덱스 스캔을 통해서 null 검색이 가능

#### 1.8. DATE vs DATE

```sql
edb=# select (sysdate - (sysdate - interval '2' hour)) as gap from dual;
   gap
----------
 02:00:00
(1 row)

edb=#
edb=#
edb=# -- data type 확인
edb=# select pg_typeof (sysdate - (sysdate - interval '2' hour)) as "Data Type";
 Data Type
-----------
 interval
(1 row)

edb=#
```

* Reason

	PPAS에서 date와 date 간의 연산의 결과 값은 오라클이 숫자형 결과를 리턴하는것과 달리 INTERVAL 타입을 리턴함

* Workaround

  ```sql
  edb=# select (extract(epoch from sysdate) - extract(epoch from (sysdate - interval '2' hour)))/60/60 as gap;
   gap
  -----
     2
  (1 row)

  edb=#
  edb=# select pg_typeof((extract(epoch from sysdate) - extract(epoch from (sysdate - interval '2' hour)))/60/60::int) as "Data Type";
      Data Type
  ------------------
   double precision
  (1 row)

  edb=#
  ```

* EXTRACT(EPOCH FROM ...) function

  - For timestamp with time zone values, the number of seconds since 1970-01-01 00:00:00 UTC (can be negative);
  - For date and timestamp values, the number of seconds since 1970-01-01 00:00:00 local time;
  - For interval values, the total number of seconds in the interval

#### 1.9. DATE, TIME, TIMESTAMP, TIMESTAMP WITH TIME ZONE

```sql
edb=# create table date_time (a date, b time, c timestamp, d timestamp with time zone);
CREATE TABLE
edb=#
edb=# insert into date_time values (now(), now(), now(), now());
INSERT 0 1
edb=#
edb=# table date_time ;
-[ RECORD 1 ]-----------------------
a | 16-FEB-16 12:36:58.706515
b | 12:36:58.706515
c | 16-FEB-16 12:36:58.706515
d | 16-FEB-16 12:36:58.706515 +09:00

edb=#
```
* DATE는 날짜와 시간 모두 저장
* TIME은 시간 정보만 저장
* TIMESTAMP는 날짜와 시간 모두 저장하며 명시적으로 WITH TIME ZONE 옵션을 사용하지 않으면 TIMESTAMP WITHOUT TIME ZONE으로 지정됨

#### 1.10. DATE_PART FUNCTION
```sql
edb=# select
edb-#   now() as now, date_part('year',now()) as year,
edb-#   date_part('month',now()) as month, date_part('day',now()) as day,
edb-#   date_part('hour',now()) as hour, date_part('minute',now()) as minute, date_part('second',now()) as second;
-[ RECORD 1 ]----------------------------
now    | 16-FEB-16 13:25:33.782363 +09:00
year   | 2016
month  | 2
day    | 16
hour   | 13
minute | 25
second | 33.782363

edb=#
```

#### 1.11. sysdate & now, current_timestamp, localtimestamp

```sql
edb=# \! vi b.sql
select sysdate, now, current_timestamp, localtimestamp;
select pg_sleep(1);
select sysdate, now, current_timestamp, localtimestamp;

BEGIN;      -- transaction start

select sysdate, now, current_timestamp, localtimestamp;
select pg_sleep(1);
select sysdate, now, current_timestamp, localtimestamp;
select pg_sleep(1);
select sysdate, now, current_timestamp, localtimestamp;

ROLLBACK;      -- transaction end

select sysdate, now, current_timestamp, localtimestamp;
select pg_sleep(1);
select sysdate, now, current_timestamp, localtimestamp;

edb=#
edb=# \i b.sql
-[ RECORD 1 ]-----+--------------------------------
sysdate           | 16-FEB-16 14:44:09
now               | 16-FEB-16 14:44:09.28651 +09:00
current_timestamp | 16-FEB-16 14:44:09.28651 +09:00
localtimestamp    | 16-FEB-16 14:44:09.286643

-[ RECORD 1 ]
pg_sleep |

-[ RECORD 1 ]-----+---------------------------------
sysdate           | 16-FEB-16 14:44:10
now               | 16-FEB-16 14:44:10.287492 +09:00
current_timestamp | 16-FEB-16 14:44:10.287492 +09:00
localtimestamp    | 16-FEB-16 14:44:10.287646

BEGIN

-[ RECORD 1 ]-----+---------------------------------
sysdate           | 16-FEB-16 14:44:10
now               | 16-FEB-16 14:44:10.287806 +09:00
current_timestamp | 16-FEB-16 14:44:10.287899 +09:00
localtimestamp    | 16-FEB-16 14:44:10.288149

-[ RECORD 1 ]
pg_sleep |

-[ RECORD 1 ]-----+---------------------------------
sysdate           | 16-FEB-16 14:44:11
now               | 16-FEB-16 14:44:10.287806 +09:00
current_timestamp | 16-FEB-16 14:44:11.290238 +09:00
localtimestamp    | 16-FEB-16 14:44:11.290355

-[ RECORD 1 ]
pg_sleep |

-[ RECORD 1 ]-----+---------------------------------
sysdate           | 16-FEB-16 14:44:12
now               | 16-FEB-16 14:44:10.287806 +09:00
current_timestamp | 16-FEB-16 14:44:12.292301 +09:00
localtimestamp    | 16-FEB-16 14:44:12.292451

ROLLBACK

-[ RECORD 1 ]-----+---------------------------------
sysdate           | 16-FEB-16 14:44:12
now               | 16-FEB-16 14:44:12.292813 +09:00
current_timestamp | 16-FEB-16 14:44:12.292813 +09:00
localtimestamp    | 16-FEB-16 14:44:12.292929

-[ RECORD 1 ]
pg_sleep |

-[ RECORD 1 ]-----+---------------------------------
sysdate           | 16-FEB-16 14:44:13
now               | 16-FEB-16 14:44:13.294558 +09:00
current_timestamp | 16-FEB-16 14:44:13.294558 +09:00
localtimestamp    | 16-FEB-16 14:44:13.294667

edb=#
```

  * now() 함수의 경우, 트랜잭션으로 묶이게되면 트랜잭션 동안 트랜잭션이 시작된 시점으로 시간이 고정됨
  * sysdate는 오라클 호환성 기능으로 제공되어 사용이 가능하나 current_timestamp를 사용하는 것을 권장장


## 2. PostgreSQL 고유 기능
#### 2.1. Domain Data Type

```sql
/**
 * Biz Requirement : TEST1 과 TEST2 테이블의 컬럼은 아래 CHECK 제약 조건에 의해 VARCHAR 형이지만 2자리 숫자 타입의 값만 받아들여야함!
 */
edb=# CREATE TABLE TEST1 (COL1 varchar(2)  CHECK (COL1 ~'[[:digit:]]{2}'));  -- 정규식 (regular expression)
CREATE TABLE
edb=#
edb=# CREATE TABLE TEST2 (COL1 varchar(2)  CHECK (COL1 ~'[[:digit:]]{2}'));
CREATE TABLE
edb=#
edb=# INSERT INTO TEST1 VALUES('01');
INSERT 0 1
edb=#
edb=# INSERT INTO TEST2 VALUES('12');
INSERT 0 1
edb=#
edb=# INSERT INTO TEST1 VALUES('9'); -- check 조건에 의해 에러발생
ERROR:  new row for relation "test1" violates check constraint "test1_col1_check"
DETAIL:  Failing row contains (9).
edb=#
edb=# SELECT * FROM TEST1;
col1
------
01
(1 row)

edb=# SELECT * FROM TEST2;
col1
------
12
(1 row)

edb=#
```

위 CHECK 조건을 만족하는 컬럼값이 많은 테이블에서 정의되어야 한다면, 매번 CHECK 조건을 적어주기 매우 번거로움. 이때 DOMAIN data type을 활용

```sql
edb=# CREATE DOMAIN DIGITV AS VARCHAR(2) CHECK( VALUE ~'[[:digit:]]{2}');
CREATE DOMAIN
edb=#
edb=# DROP TABLE TEST1;
DROP TABLE
edb=# DROP TABLE TEST2;
DROP TABLE
edb=#
edb=# CREATE TABLE TEST1(COL1 DIGITV);
CREATE TABLE
edb=# CREATE TABLE TEST2(COL1 DIGITV);
CREATE TABLE
edb=#
edb=# INSERT INTO TEST1 VALUES('02');
INSERT 0 1
edb=# INSERT INTO TEST2 VALUES('13');
INSERT 0 1
edb=#
edb=# \d test1
    Table "public.test1"
 Column |  Type  | Modifiers
--------+--------+-----------
 col1   | digitv |
edb=#
edb=# \d test2
    Table "public.test2"
 Column |  Type  | Modifiers
--------+--------+-----------
 col1   | digitv |
edb=#
```

#### 2.2. Bind Variable 있는 SQL 실행 계획 보기

```sql
edb=# explain select * from emp where empno=$1;
ERROR:  there is no parameter $1
LINE 1: explain select * from emp where empno=$1;
                                              ^
edb=#
edb=# prepare stmt(int) as select * from emp where empno=$1;
PREPARE
edb=#
edb=# explain execute stmt(7902);
                     QUERY PLAN
----------------------------------------------------
 Seq Scan on emp  (cost=0.00..1.18 rows=1 width=45)
   Filter: (empno = '7902'::numeric)
(2 rows)

edb=# deallocate stmt ;
DEALLOCATE
edb=#
```
* 주의사항

	바인드 변수를 사용하는 SQL의 실행 계획 조회를 위해서는 PREPARE 단계를 거쳐야 하며, 바인드 변수값 또한 알아야함

#### 2.3. UPDATE Returning

```sql
edb=# create table emp2 as select * from emp;
edb=#
edb=# select * from emp2;
 empno | ename  |    job    | mgr  |      hiredate      |   sal   |  comm   | deptno
-------+--------+-----------+------+--------------------+---------+---------+--------
  7369 | SMITH  | CLERK     | 7902 | 17-DEC-80 00:00:00 |  800.00 |         |     20
  7499 | ALLEN  | SALESMAN  | 7698 | 20-FEB-81 00:00:00 | 1600.00 |  300.00 |     30
  7521 | WARD   | SALESMAN  | 7698 | 22-FEB-81 00:00:00 | 1250.00 |  500.00 |     30
  7566 | JONES  | MANAGER   | 7839 | 02-APR-81 00:00:00 | 2975.00 |         |     20
  7654 | MARTIN | SALESMAN  | 7698 | 28-SEP-81 00:00:00 | 1250.00 | 1400.00 |     30
  7698 | BLAKE  | MANAGER   | 7839 | 01-MAY-81 00:00:00 | 2850.00 |         |     30
  7782 | CLARK  | MANAGER   | 7839 | 09-JUN-81 00:00:00 | 2450.00 |         |     10
  7788 | SCOTT  | ANALYST   | 7566 | 19-APR-87 00:00:00 | 3000.00 |         |     20
  7839 | KING   | PRESIDENT |      | 17-NOV-81 00:00:00 | 5000.00 |         |     10
  7844 | TURNER | SALESMAN  | 7698 | 08-SEP-81 00:00:00 | 1500.00 |    0.00 |     30
  7876 | ADAMS  | CLERK     | 7788 | 23-MAY-87 00:00:00 | 1100.00 |         |     20
  7900 | JAMES  | CLERK     | 7698 | 03-DEC-81 00:00:00 |  950.00 |         |     30
  7902 | FORD   | ANALYST   | 7566 | 03-DEC-81 00:00:00 | 3000.00 |         |     20
  7934 | MILLER | CLERK     | 7782 | 23-JAN-82 00:00:00 | 1300.00 |         |     10
(14 rows)

edb=#
edb=# update emp2 set sal=sal+100 where sal<1000 returning *;
 empno | ename |  job  | mgr  |      hiredate      |   sal   | comm | deptno
-------+-------+-------+------+--------------------+---------+------+--------
  7369 | SMITH | CLERK | 7902 | 17-DEC-80 00:00:00 |  900.00 |      |     20
  7900 | JAMES | CLERK | 7698 | 03-DEC-81 00:00:00 | 1050.00 |      |     30
(2 rows)

UPDATE 2
edb=#
```

오라클 경우에도 UPDATE .. RETURNING 문을 지원하나 PL/SQL 등에서만 사용해야함

#### 2.4. Type Casting

* ANSI 표준인 CAST 함수를 사용하지 않고, "::" 를 이용하여 타입을 변경할수 있음

  ```sql
  edb=# select '2013-03-05', cast('2013-03-15' as date), '2013-03-15'::date;
    ?column?  |        date        |        date
  ------------+--------------------+--------------------
   2013-03-05 | 15-MAR-13 00:00:00 | 15-MAR-13 00:00:00
  (1 row)

  edb=#
  ```

* 오라클과 같이 to_date 함수 등을 사용하지 않아도 날짜 형태의 스트링을 자동으로 날짜형 데이터 타입으로 캐스팅함

  ```sql
  edb=# select * from emp where hiredate < '1981-08-31 00:00:00';
   empno | ename |   job    | mgr  |      hiredate      |   sal   |  comm  | deptno
  -------+-------+----------+------+--------------------+---------+--------+--------
    7499 | ALLEN | SALESMAN | 7698 | 20-FEB-81 00:00:00 | 1600.00 | 300.00 |     30
    7521 | WARD  | SALESMAN | 7698 | 22-FEB-81 00:00:00 | 1250.00 | 500.00 |     30
    7566 | JONES | MANAGER  | 7839 | 02-APR-81 00:00:00 | 2975.00 |        |     20
    7698 | BLAKE | MANAGER  | 7839 | 01-MAY-81 00:00:00 | 2850.00 |        |     30
    7782 | CLARK | MANAGER  | 7839 | 09-JUN-81 00:00:00 | 2450.00 |        |     10
    7369 | SMITH | CLERK    | 7902 | 17-DEC-80 00:00:00 |  900.00 |        |     20
  (6 rows)

  edb=# explain select * from emp where hiredate < '1981-08-31 00:00:00';
                                  QUERY PLAN
  --------------------------------------------------------------------------
   Seq Scan on emp  (cost=0.00..1.18 rows=6 width=45)
     Filter: (hiredate < '31-AUG-81 00:00:00'::timestamp without time zone)
  (2 rows)

  edb=#
  ```

#### 2.5. Distinct On

* a 컬럼에 대해 distinct 한 값 로우만을 가져오는 동시에, b 컬럼까지 한번에 select 가능

  ```sql
  edb=# create table dist_on (a varchar, b int);
  CREATE TABLE
  edb=#
  edb=# insert into dist_on values ('lion', 1);
  INSERT 0 1
  edb=# insert into dist_on values ('lion', 2);
  INSERT 0 1
  edb=# insert into dist_on values ('tiger', 2);
  INSERT 0 1
  edb=# insert into dist_on values ('tiger', 1);
  INSERT 0 1
  edb=# insert into dist_on values ('rabbit', 1);
  INSERT 0 1
  edb=# insert into dist_on values ('rabbit', 2);
  INSERT 0 1
  edb=# insert into dist_on values ('rabbit', 1);
  INSERT 0 1
  edb=#
  edb=# table dist_on ;
     a    | b
  --------+---
   lion   | 1
   lion   | 2
   tiger  | 2
   tiger  | 1
   rabbit | 1
   rabbit | 2
   rabbit | 1
  (7 rows)

  edb=#
  edb=# select distinct a, b from dist_on order by 1,2;
     a    | b
  --------+---
   lion   | 1
   lion   | 2
   rabbit | 1
   rabbit | 2
   tiger  | 1
   tiger  | 2
  (6 rows)

  edb=#
  edb=# select distinct on (a) a, b from dist_on order by 1,2;
     a    | b
  --------+---
   lion   | 1
   rabbit | 1
   tiger  | 1
  (3 rows)

  edb=#
  ```

#### 2.6. Regular Expression

```sql
edb=# create table reg_exp (a varchar);
CREATE TABLE
edb=#
edb=# insert into reg_exp values ('가나다');
INSERT 0 1
edb=# insert into reg_exp values (100);
INSERT 0 1
edb=# insert into reg_exp values (2);
INSERT 0 1
edb=# insert into reg_exp values ('a한글123');
INSERT 0 1
edb=# insert into reg_exp values ('999test');
INSERT 0 1
edb=# insert into reg_exp values ('a89adf33');
INSERT 0 1
edb=#
edb=# table reg_exp ;
    a
----------
 가나다
 100
 2
 a한글123
 999test
 a89adf33
(6 rows)

edb=#
edb=# -- 로우중에서 숫자가 하나라도 들어가 있는 로우만 추출
edb=# select * from reg_exp where a ~ '[0-9]+';
    a
----------
 100
 2
 a한글123
 999test
 a89adf33
(5 rows)

edb=#
edb=# -- 숫자가 하나도 들어가 있지 않은 로우만 추출
edb=# select * from reg_exp where a !~ '[0-9]+';
   a
--------
 가나다
(1 row)

edb=#
edb=# -- 데이터중 숫자부분만 추출
edb=# select a, regexp_matches (a, '[0-9]+') from reg_exp;
    a     | regexp_matches
----------+----------------
 100      | {100}
 2        | {2}
 a한글123  | {123}
 999test  | {999}
 a89adf33 | {89}
(5 rows)

edb=#
edb=# -- regexp_matches는 배열로 값을 리턴. 배열중 밸류값만 추출
edb=# select a, (regexp_matches (a, '[0-9]+'))[1] from reg_exp;
    a     | regexp_matches
----------+----------------
 100      | 100
 2        | 2
 a한글123  | 123
 999test  | 999
 a89adf33 | 89
(5 rows)

edb=#
edb=# -- 데이터중 모든 숫자부분 추출
edb=# select a, regexp_matches (a, '[0-9]+', 'g') from reg_exp;
    a     | regexp_matches
----------+----------------
 100      | {100}
 2        | {2}
 a한글123  | {123}
 999test  | {999}
 a89adf33 | {89}
 a89adf33 | {33}
(6 rows)

edb=#
edb=# -- 배열에서 밸류만 추출
edb=# select a, (regexp_matches (a, '[0-9]+', 'g'))[1] from reg_exp;
    a     | regexp_matches
----------+----------------
 100      | 100
 2        | 2
 a한글123  | 123
 999test  | 999
 a89adf33 | 89
 a89adf33 | 33
(6 rows)

edb=#
edb=# -- 숫자만으로 이루어진 로우만 필터링
edb=# select a from reg_exp where a ~'^[0-9]+$';
  a
-----
 100
 2
(2 rows)

edb=#
```


#### 2.7. Paging

```sql
edb=# select ename, job, sal from emp order by sal;
 ename  |    job    |   sal
--------+-----------+---------
 SMITH  | CLERK     |  900.00
 JAMES  | CLERK     | 1050.00
 ADAMS  | CLERK     | 1100.00
 MARTIN | SALESMAN  | 1250.00
 WARD   | SALESMAN  | 1250.00
 MILLER | CLERK     | 1300.00
 TURNER | SALESMAN  | 1500.00
 ALLEN  | SALESMAN  | 1600.00
 CLARK  | MANAGER   | 2450.00
 BLAKE  | MANAGER   | 2850.00
 JONES  | MANAGER   | 2975.00
 SCOTT  | ANALYST   | 3000.00
 FORD   | ANALYST   | 3000.00
 KING   | PRESIDENT | 5000.00
(14 rows)

edb=#
edb=# select ename, job, sal from emp order by sal limit 5;
 ename  |   job    |   sal
--------+----------+---------
 SMITH  | CLERK    |  900.00
 JAMES  | CLERK    | 1050.00
 ADAMS  | CLERK    | 1100.00
 WARD   | SALESMAN | 1250.00
 MARTIN | SALESMAN | 1250.00
(5 rows)

edb=#
edb=# select ename, job, sal from emp order by sal offset 0 limit 5;
 ename  |   job    |   sal
--------+----------+---------
 SMITH  | CLERK    |  900.00
 JAMES  | CLERK    | 1050.00
 ADAMS  | CLERK    | 1100.00
 WARD   | SALESMAN | 1250.00
 MARTIN | SALESMAN | 1250.00
(5 rows)

edb=#
edb=# select ename, job, sal from emp order by sal offset 5 limit 5;
 ename  |   job    |   sal
--------+----------+---------
 MILLER | CLERK    | 1300.00
 TURNER | SALESMAN | 1500.00
 ALLEN  | SALESMAN | 1600.00
 CLARK  | MANAGER  | 2450.00
 BLAKE  | MANAGER  | 2850.00
(5 rows)

edb=# select ename, job, sal from emp order by sal offset 10 limit 5;
 ename |    job    |   sal
-------+-----------+---------
 JONES | MANAGER   | 2975.00
 SCOTT | ANALYST   | 3000.00
 FORD  | ANALYST   | 3000.00
 KING  | PRESIDENT | 5000.00
(4 rows)

edb=#
```

* PPAS의 페이징 처리는 오라클처럼 ROWNUM을 이용하여 복잡하게 처리할 필요없이 limit과 offset으로 간단히 처리 가능


#### 2.8. 대소문자 무시한 검색

```sql
edb=# create table up_low (name varchar);
CREATE TABLE
edb=# insert into up_low values ('Abcde');
INSERT 0 1
edb=# insert into up_low values ('aBcde');
INSERT 0 1
edb=# insert into up_low values ('abCde');
INSERT 0 1
edb=# insert into up_low values ('abcDe');
INSERT 0 1
edb=# insert into up_low values ('abcdE');
INSERT 0 1
edb=#
edb=# select name from up_low where name like '%bcd%';
 name
-------
 Abcde
 abcdE
(2 rows)

edb=# select name from up_low where name ilike '%bcd%';
 name
-------
 Abcde
 aBcde
 abCde
 abcDe
 abcdE
(5 rows)

edb=#```

* ILIKE 문을 이용하여 대소문자 구분없이 검색이 가능합니다.
