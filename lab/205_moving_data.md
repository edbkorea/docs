# Lab. Moving Data

## COPY

### From/To File

* `COPY` 명령어를 사용하려면 superuser 권한이 있어야 한다. 새로운 사용자를 만들고 superuser 권한을 부여해 보자.

  ```
  edb=# create user copyuser password 'copyuser';
  CREATE ROLE
  edb=# alter user copyuser with superuser;
  ALTER ROLE
  edb=# \du copyuser
                List of roles
   Role name |   Attributes    | Member of
  -----------+-----------------+-----------
   copyuser  | Superuser      +| {}
             | Profile default |
  ```

* pgbench DB의 pgbench_accounts 테이블의 데이터를 파이프(|)를 구분자로 하여 언로드(Unload)해 보자.

  ```
  edb=# \c pgbench copyuser
  Password for user copyuser:
  You are now connected to database "pgbench" as user "copyuser".
  pgbench=# copy pgbench_accounts to '/tmp/pgbench_accounts.csv' with csv header DELIMITER '|;
  COPY 10000000

  $ ls -l /tmp/pgbench_accounts.csv
  -rw-r--r--. 1 enterprisedb enterprisedb 978088897 Feb 12 19:49 /tmp/pgbench_accounts.csv
  $ head -10 /tmp/pgbench_accounts.csv
  aid|bid|abalance|filler
  1|1|0|
  2|1|0|
  3|1|0|
  4|1|0|
  5|1|0|
  6|1|0|
  7|1|0|
  8|1|0|
  9|1|0|
  ```

* 앞서 내려 받은 파일을 다시 로드해 보자.

  ```
  pgbench=# create table test (like pgbench_accounts);
  CREATE TABLE
  pgbench=# copy test from '/tmp/pgbench_accounts.csv' with csv header DELIMITER '|';
  COPY 10000000
  pgbench=# select * from test limit 10;
   aid | bid | abalance |                                        filler
  -----+-----+----------+--------------------------------------------------------------------------------------
     1 |   1 |        0 |
     2 |   1 |        0 |
     3 |   1 |        0 |
     4 |   1 |        0 |
     5 |   1 |        0 |
     6 |   1 |        0 |
     7 |   1 |        0 |
     8 |   1 |        0 |
     9 |   1 |        0 |
    10 |   1 |        0 |
  (10 rows)
  ```

### From STDIN/To STDOUT

* `copy to stdout`
  ```
  edb=# copy emp to stdout;
  7369	SMITH	CLERK	7902	17-DEC-80 00:00:00	960.00	\N	20
  7499	ALLEN	SALESMAN	7698	20-FEB-81 00:00:00	1920.00	300.00	30
  7521	WARD	SALESMAN	7698	22-FEB-81 00:00:00	1500.00	500.00	30
  7566	JONES	MANAGER	7839	02-APR-81 00:00:00	3570.00	\N	20
  7654	MARTIN	SALESMAN	7698	28-SEP-81 00:00:00	1500.00	1400.00	30
  7698	BLAKE	MANAGER	7839	01-MAY-81 00:00:00	3420.00	\N	30
  7782	CLARK	MANAGER	7839	09-JUN-81 00:00:00	2940.00	\N	10
  7788	SCOTT	ANALYST	7566	19-APR-87 00:00:00	3600.00	\N	20
  7839	KING	PRESIDENT	\N	17-NOV-81 00:00:00	6000.00	\N	10
  7844	TURNER	SALESMAN	7698	08-SEP-81 00:00:00	1800.00	0.00	30
  7876	ADAMS	CLERK	7788	23-MAY-87 00:00:00	1320.00	\N	20
  7900	JAMES	CLERK	7698	03-DEC-81 00:00:00	1140.00	\N	30
  7902	FORD	ANALYST	7566	03-DEC-81 00:00:00	3600.00	\N	20
  7934	MILLER	CLERK	7782	23-JAN-82 00:00:00	1560.00	\N	10
  edb=# copy emp to stdout with (format csv);
  7369,SMITH,CLERK,7902,17-DEC-80 00:00:00,960.00,,20
  7499,ALLEN,SALESMAN,7698,20-FEB-81 00:00:00,1920.00,300.00,30
  7521,WARD,SALESMAN,7698,22-FEB-81 00:00:00,1500.00,500.00,30
  7566,JONES,MANAGER,7839,02-APR-81 00:00:00,3570.00,,20
  7654,MARTIN,SALESMAN,7698,28-SEP-81 00:00:00,1500.00,1400.00,30
  7698,BLAKE,MANAGER,7839,01-MAY-81 00:00:00,3420.00,,30
  7782,CLARK,MANAGER,7839,09-JUN-81 00:00:00,2940.00,,10
  7788,SCOTT,ANALYST,7566,19-APR-87 00:00:00,3600.00,,20
  7839,KING,PRESIDENT,,17-NOV-81 00:00:00,6000.00,,10
  7844,TURNER,SALESMAN,7698,08-SEP-81 00:00:00,1800.00,0.00,30
  7876,ADAMS,CLERK,7788,23-MAY-87 00:00:00,1320.00,,20
  7900,JAMES,CLERK,7698,03-DEC-81 00:00:00,1140.00,,30
  7902,FORD,ANALYST,7566,03-DEC-81 00:00:00,3600.00,,20
  7934,MILLER,CLERK,7782,23-JAN-82 00:00:00,1560.00,,10
  ```

* `copy from stdin`
  ```
  edb=# create table emp2 (like emp);
  CREATE TABLE
  edb=# copy emp2 from STDIN with (format csv);
  Enter data to be copied followed by a newline.
  End with a backslash and a period on a line by itself.
  >>
  ```

  이제 입력할 데이터를 직접 입력하고 마지막 줄에 `\.`를 입력하면 된다. 아래 데이터를 복사해서 붙여 넣어 보자.

  ```
  7369,SMITH,CLERK,7902,17-DEC-80 00:00:00,960.00,,20
  7499,ALLEN,SALESMAN,7698,20-FEB-81 00:00:00,1920.00,300.00,30
  7521,WARD,SALESMAN,7698,22-FEB-81 00:00:00,1500.00,500.00,30
  7566,JONES,MANAGER,7839,02-APR-81 00:00:00,3570.00,,20
  7654,MARTIN,SALESMAN,7698,28-SEP-81 00:00:00,1500.00,1400.00,30
  7698,BLAKE,MANAGER,7839,01-MAY-81 00:00:00,3420.00,,30
  7782,CLARK,MANAGER,7839,09-JUN-81 00:00:00,2940.00,,10
  7788,SCOTT,ANALYST,7566,19-APR-87 00:00:00,3600.00,,20
  7839,KING,PRESIDENT,,17-NOV-81 00:00:00,6000.00,,10
  7844,TURNER,SALESMAN,7698,08-SEP-81 00:00:00,1800.00,0.00,30
  7876,ADAMS,CLERK,7788,23-MAY-87 00:00:00,1320.00,,20
  7900,JAMES,CLERK,7698,03-DEC-81 00:00:00,1140.00,,30
  7902,FORD,ANALYST,7566,03-DEC-81 00:00:00,3600.00,,20
  7934,MILLER,CLERK,7782,23-JAN-82 00:00:00,1560.00,,10
  \.
  ```

  아래와 같이 데이터가 입력된 것을 볼 수 있다.

  ```
  >> 7369,SMITH,CLERK,7902,17-DEC-80 00:00:00,960.00,,20
  >> 7499,ALLEN,SALESMAN,7698,20-FEB-81 00:00:00,1920.00,300.00,30
  >> 7521,WARD,SALESMAN,7698,22-FEB-81 00:00:00,1500.00,500.00,30
  >> 7566,JONES,MANAGER,7839,02-APR-81 00:00:00,3570.00,,20
  >> 7654,MARTIN,SALESMAN,7698,28-SEP-81 00:00:00,1500.00,1400.00,30
  >> 7698,BLAKE,MANAGER,7839,01-MAY-81 00:00:00,3420.00,,30
  >> 7782,CLARK,MANAGER,7839,09-JUN-81 00:00:00,2940.00,,10
  >> 7788,SCOTT,ANALYST,7566,19-APR-87 00:00:00,3600.00,,20
  >> 7839,KING,PRESIDENT,,17-NOV-81 00:00:00,6000.00,,10
  >> 7844,TURNER,SALESMAN,7698,08-SEP-81 00:00:00,1800.00,0.00,30
  >> 7876,ADAMS,CLERK,7788,23-MAY-87 00:00:00,1320.00,,20
  >> 7900,JAMES,CLERK,7698,03-DEC-81 00:00:00,1140.00,,30
  >> 7902,FORD,ANALYST,7566,03-DEC-81 00:00:00,3600.00,,20
  >> 7934,MILLER,CLERK,7782,23-JAN-82 00:00:00,1560.00,,10
  >> \.
  COPY 14
  edb=# select * from emp2;
   empno | ename  |    job    | mgr  |      hiredate      |   sal   |  comm   | deptno 
  -------+--------+-----------+------+--------------------+---------+---------+--------
    7369 | SMITH  | CLERK     | 7902 | 17-DEC-80 00:00:00 |  960.00 |         |     20
    7499 | ALLEN  | SALESMAN  | 7698 | 20-FEB-81 00:00:00 | 1920.00 |  300.00 |     30
    7521 | WARD   | SALESMAN  | 7698 | 22-FEB-81 00:00:00 | 1500.00 |  500.00 |     30
    7566 | JONES  | MANAGER   | 7839 | 02-APR-81 00:00:00 | 3570.00 |         |     20
    7654 | MARTIN | SALESMAN  | 7698 | 28-SEP-81 00:00:00 | 1500.00 | 1400.00 |     30
    7698 | BLAKE  | MANAGER   | 7839 | 01-MAY-81 00:00:00 | 3420.00 |         |     30
    7782 | CLARK  | MANAGER   | 7839 | 09-JUN-81 00:00:00 | 2940.00 |         |     10
    7788 | SCOTT  | ANALYST   | 7566 | 19-APR-87 00:00:00 | 3600.00 |         |     20
    7839 | KING   | PRESIDENT |      | 17-NOV-81 00:00:00 | 6000.00 |         |     10
    7844 | TURNER | SALESMAN  | 7698 | 08-SEP-81 00:00:00 | 1800.00 |    0.00 |     30
    7876 | ADAMS  | CLERK     | 7788 | 23-MAY-87 00:00:00 | 1320.00 |         |     20
    7900 | JAMES  | CLERK     | 7698 | 03-DEC-81 00:00:00 | 1140.00 |         |     30
    7902 | FORD   | ANALYST   | 7566 | 03-DEC-81 00:00:00 | 3600.00 |         |     20
    7934 | MILLER | CLERK     | 7782 | 23-JAN-82 00:00:00 | 1560.00 |         |     10
  (14 rows)
  ```

### From/To Program (PIPE)

`to program` 구문을 이용하면 외부 프로그램을 실행해서 결과를 전달할 수 있다.

```
edb=# copy emp to program 'gzip > /tmp/emp.tsv.gz';
COPY 14
edb=# \! gunzip -c /tmp/emp.tsv.gz
7369	SMITH	CLERK	7902	17-DEC-80 00:00:00	960.00	\N	20
7499	ALLEN	SALESMAN	7698	20-FEB-81 00:00:00	1920.00	300.00	30
7521	WARD	SALESMAN	7698	22-FEB-81 00:00:00	1500.00	500.00	30
7566	JONES	MANAGER	7839	02-APR-81 00:00:00	3570.00	\N	20
7654	MARTIN	SALESMAN	7698	28-SEP-81 00:00:00	1500.00	1400.00	30
7698	BLAKE	MANAGER	7839	01-MAY-81 00:00:00	3420.00	\N	30
7782	CLARK	MANAGER	7839	09-JUN-81 00:00:00	2940.00	\N	10
7788	SCOTT	ANALYST	7566	19-APR-87 00:00:00	3600.00	\N	20
7839	KING	PRESIDENT	\N	17-NOV-81 00:00:00	6000.00	\N	10
7844	TURNER	SALESMAN	7698	08-SEP-81 00:00:00	1800.00	0.00	30
7876	ADAMS	CLERK	7788	23-MAY-87 00:00:00	1320.00	\N	20
7900	JAMES	CLERK	7698	03-DEC-81 00:00:00	1140.00	\N	30
7902	FORD	ANALYST	7566	03-DEC-81 00:00:00	3600.00	\N	20
7934	MILLER	CLERK	7782	23-JAN-82 00:00:00	1560.00	\N	10
```



## COPY FREEZE

앞의 작업을 `COPY` 대신 `COPY FREEZE`로 반복 해 보자.

* 현재 xlog 위치를 확인한다.

  ```
  pgbench=# select * from pg_xlogfile_name_offset(pg_current_xlog_location());
          file_name         | file_offset 
  --------------------------+-------------
   0000000100000000000000DB |     8061288
  (1 row)
  ```

* `COPY FREEZE`로 파일을 로딩한다.

  ```
  pgbench=# BEGIN;
  BEGIN
  pgbench=# truncate table test;
  TRUNCATE TABLE
  pgbench=# copy test from '/tmp/pgbench_accounts.csv' with csv header DELIMITER '|' FREEZE;
  COPY 10000000
  pgbench=# COMMIT;
  COMMIT
  ```

* xlog의 위치를 다시 확인한다.
	매우 적은 양의 xlog만 발생한 것을 볼 수 있다. xlog 발생 량을 `COPY` 명령어와 비교해 보자.

  ```
  pgbench=# select * from pg_xlogfile_name_offset(pg_current_xlog_location());
          file_name         | file_offset 
  --------------------------+-------------
   0000000100000000000000DB |     8081224
  (1 row)
  ```

* 참고자료
 * http://eulerto.blogspot.kr/2011/11/understanding-wal-nomenclature.html

## EDB*Loader

* 앞서 사용한 파일을 EDB*Loader를 이용해서 로딩해 보자.

  ```
  $ cat /tmp/pgbench_accounts.ctl 
  LOAD DATA
    INFILE '/tmp/pgbench_accounts.csv' 
    BADFILE '/tmp/pgbench_accounts.bad' 
  APPEND
  INTO TABLE test
  FIELDS TERMINATED BY '|'
    (aid, bid, abalance, "filler")

  $ edbldr -d pgbench userid=copyuser control=/tmp/pgbench_accounts.ctl 
  EDB*Loader: Copyright (c) 2007-2015, EnterpriseDB Corporation.

  Enter the password : 
  Successfully processed (10000000) records
  ```
  
  ```
  pgbench=# select * from test limit 10;
   aid | bid | abalance |                                        filler
  -----+-----+----------+--------------------------------------------------------------------------------------
     1 |   1 |        0 |
     2 |   1 |        0 |
     3 |   1 |        0 |
     4 |   1 |        0 |
     5 |   1 |        0 |
     6 |   1 |        0 |
     7 |   1 |        0 |
     8 |   1 |        0 |
     9 |   1 |        0 |
    10 |   1 |        0 |
  (10 rows)
  ```