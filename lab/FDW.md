# Lab: Foreign Data Wrapper

## `file_fdw`

### 설치

`file_fdw`는 이미 기본으로 포함되어 있기 때문에 `create extension`만으로 간단히 설치 가능하다.

```
edb=# create extension file_fdw;
CREATE EXTENSION
edb=# \dx file_fdw
                         List of installed extensions
   Name   | Version |    Schema    |                Description
----------+---------+--------------+-------------------------------------------
 file_fdw | 1.0     | enterprisedb | foreign-data wrapper for flat file access
(1 row)

edb=# \dx+ file_fdw
     Objects in extension "file_fdw"
           Object Description
-----------------------------------------
 foreign-data wrapper file_fdw
 function file_fdw_handler()
 function file_fdw_validator(text[],oid)
(3 rows)
```

### 테스트 데이터 준비

`file_fdw`는 외부의 파일을 테이블처럼 읽을 수 있게 해 주는 FDW이기 때문에 읽을 데이터가 필요하다.

* 디랙토리 생성

  ```
  [enterprisedb@ppaslab ~]$ mkdir -p ~/work/fdw/file_fdw_data
  ```

* 데이터 파일 생성

  ```
  edb=# copy emp to '/opt/PostgresPlus/9.5AS/work/fdw/file_fdw_data/emp.tsv';
  COPY 14
  edb=# copy emp to '/opt/PostgresPlus/9.5AS/work/fdw/file_fdw_data/emp.dat' with (format binary);
  COPY 14
  ```

  ```
  [enterprisedb@ppaslab ~]$ cat ~/work/fdw/file_fdw_data/emp.tsv 
  7369	SMITH	CLERK	7902	17-DEC-80 00:00:00	800.00	\N	20
  7499	ALLEN	SALESMAN	7698	20-FEB-81 00:00:00	1600.00	300.00	30
  7521	WARD	SALESMAN	7698	22-FEB-81 00:00:00	1250.00	500.00	30
  7566	JONES	MANAGER	7839	02-APR-81 00:00:00	2975.00	\N	20
  7654	MARTIN	SALESMAN	7698	28-SEP-81 00:00:00	1250.00	1400.00	30
  7698	BLAKE	MANAGER	7839	01-MAY-81 00:00:00	2850.00	\N	30
  7782	CLARK	MANAGER	7839	09-JUN-81 00:00:00	2450.00	\N	10
  7788	SCOTT	ANALYST	7566	19-APR-87 00:00:00	3000.00	\N	20
  7839	KING	PRESIDENT	\N	17-NOV-81 00:00:00	5000.00	\N	10
  7844	TURNER	SALESMAN	7698	08-SEP-81 00:00:00	1500.00	0.00	30
  7876	ADAMS	CLERK	7788	23-MAY-87 00:00:00	1100.00	\N	20
  7900	JAMES	CLERK	7698	03-DEC-81 00:00:00	950.00	\N	30
  7902	FORD	ANALYST	7566	03-DEC-81 00:00:00	3000.00	\N	20
  7934	MILLER	CLERK	7782	23-JAN-82 00:00:00	1300.00	\N	10
  [enterprisedb@ppaslab ~]$ file ~/work/fdw/file_fdw_data/emp.dat 
  /opt/PostgresPlus/9.5AS/work/fdw/file_fdw_data/emp.dat: data
  ```

### FDW 생성

Server 생성
```sql
create server filefdw_srv foreign data wrapper file_fdw;
```

#### Plain text data

* Foreign table 생성
  ```sql
  drop foreign table if exists file_fdw_emp;
  create foreign table file_fdw_emp (
    empno numeric(4) not null,
    ename varchar(10),
    job varchar(9),
    mgr numeric(4),
    hiredate timestamp,
    sal numeric(7,2),
    comm numeric(7,2),
    deptno numeric(2)
  ) server filefdw_srv
  options (
    format 'csv',
    filename '/opt/PostgresPlus/9.5AS/work/fdw/file_fdw_data/emp.tsv',
    delimiter E'\t',
    null '\N'
  );
  ```

* 조회
  ```
  edb=# select * from file_fdw_emp;
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

  edb=# select * from emp;
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
  ```

#### Binary format 데이터

* Foreign table 생성

  ```sql
  drop foreign table if exists file_fdw_emp2;
  create foreign table file_fdw_emp2 (
    empno numeric(4) not null,
    ename varchar(10),
    job varchar(9),
    mgr numeric(4),
    hiredate timestamp,
    sal numeric(7,2),
    comm numeric(7,2),
    deptno numeric(2)
  ) server filefdw_srv
  options (
    format 'binary',
    filename '/opt/PostgresPlus/9.5AS/work/fdw/file_fdw_data/emp.dat'
  );
  ```

* 조회

  ```
  edb=# select * from file_fdw_emp2;
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
  ```

## `postgres_fdw`

### 설치

`postgres_fdw`는 이미 기본으로 포함되어 있기 때문에 `create extension`만으로 간단히 설치 가능하다.

```
edb=# create extension postgres_fdw;
CREATE EXTENSION
edb=# \dx postgres_fdw
                                List of installed extensions
     Name     | Version |    Schema    |                    Description
--------------+---------+--------------+----------------------------------------------------
 postgres_fdw | 1.0     | enterprisedb | foreign-data wrapper for remote PostgreSQL servers
(1 row)

edb=# \dx+ postgres_fdw
     Objects in extension "postgres_fdw"
             Object Description
---------------------------------------------
 foreign-data wrapper postgres_fdw
 function postgres_fdw_handler()
 function postgres_fdw_validator(text[],oid)
(3 rows)
```

### FDW 생성

* Server 및 User mapping 생성

  ```sql
  CREATE SERVER localpg_server
          FOREIGN DATA WRAPPER postgres_fdw
          OPTIONS (host '127.0.0.1', port '5444', dbname 'edb');
  ```

  `enterprisedb` 사용자가 `localpg_server` 서버를 이용할때 `enterprisedb` / `ppas`로 인증하도록 mapping한다.
  ```sql
  CREATE USER MAPPING FOR enterprisedb  -- local user
          SERVER localpg_server
          OPTIONS (user 'enterprisedb', password 'ppas'); -- remote user
  ```

* Foreign table 생성

  ```sql
  drop foreign table if exists postgres_fdw_emp;
  create foreign table postgres_fdw_emp (
    empno numeric(4) not null,
    ename varchar(10),
    job varchar(9),
    mgr numeric(4),
    hiredate timestamp,
    sal numeric(7,2),
    comm numeric(7,2),
    deptno numeric(2)
  ) server localpg_server
  options (
    schema_name 'public', table_name 'emp'
  );
  ```

### 테스트

* 조회

  ```
  edb=# select * from postgres_fdw_emp;
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

  edb=# explain select * from postgres_fdw_emp;
                                   QUERY PLAN                                 
  ----------------------------------------------------------------------------
   Foreign Scan on postgres_fdw_emp  (cost=100.00..124.43 rows=481 width=146)
  (1 row)
  ```

* DML

  ```
  edb=# update postgres_fdw_emp set sal = sal * 1.2;
  UPDATE 14
  edb=# select * from emp;
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

* Transaction 처리

  ```
  edb=# begin work;
  BEGIN
  edb=# update postgres_fdw_emp set sal = sal * 1.2;
  UPDATE 14
  edb=# select * from postgres_fdw_emp;
   empno | ename  |    job    | mgr  |      hiredate      |   sal   |  comm   | deptno 
  -------+--------+-----------+------+--------------------+---------+---------+--------
    7369 | SMITH  | CLERK     | 7902 | 17-DEC-80 00:00:00 | 1152.00 |         |     20
    7499 | ALLEN  | SALESMAN  | 7698 | 20-FEB-81 00:00:00 | 2304.00 |  300.00 |     30
    7521 | WARD   | SALESMAN  | 7698 | 22-FEB-81 00:00:00 | 1800.00 |  500.00 |     30
    7566 | JONES  | MANAGER   | 7839 | 02-APR-81 00:00:00 | 4284.00 |         |     20
    7654 | MARTIN | SALESMAN  | 7698 | 28-SEP-81 00:00:00 | 1800.00 | 1400.00 |     30
    7698 | BLAKE  | MANAGER   | 7839 | 01-MAY-81 00:00:00 | 4104.00 |         |     30
    7782 | CLARK  | MANAGER   | 7839 | 09-JUN-81 00:00:00 | 3528.00 |         |     10
    7788 | SCOTT  | ANALYST   | 7566 | 19-APR-87 00:00:00 | 4320.00 |         |     20
    7839 | KING   | PRESIDENT |      | 17-NOV-81 00:00:00 | 7200.00 |         |     10
    7844 | TURNER | SALESMAN  | 7698 | 08-SEP-81 00:00:00 | 2160.00 |    0.00 |     30
    7876 | ADAMS  | CLERK     | 7788 | 23-MAY-87 00:00:00 | 1584.00 |         |     20
    7900 | JAMES  | CLERK     | 7698 | 03-DEC-81 00:00:00 | 1368.00 |         |     30
    7902 | FORD   | ANALYST   | 7566 | 03-DEC-81 00:00:00 | 4320.00 |         |     20
    7934 | MILLER | CLERK     | 7782 | 23-JAN-82 00:00:00 | 1872.00 |         |     10
  (14 rows)

  edb=# select * from emp;
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

  edb=# rollback;
  ROLLBACK
  edb=# select * from postgres_fdw_emp;
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

## `oracle_fdw`

### 설치

`oracle_fdw`는 소스를 컴파일해서 설치해야 하는데 Oracle client를 필요로 한다. 우리는 instant client를 사용하여 설치해 보자.

#### Oracle client 설치

* Instant client 설치
  ```
  [enterprisedb@ppaslab ~]$ cd /opt/PostgresPlus/
  [enterprisedb@ppaslab PostgresPlus]$ unzip /opt/pkgs/instantclient-basic-linux.x64-11.2.0.4.0.zip 
  [enterprisedb@ppaslab PostgresPlus]$ unzip /opt/pkgs/instantclient-sdk-linux.x64-11.2.0.4.0.zip 
  [enterprisedb@ppaslab PostgresPlus]$ cd instantclient_11_2/
  [enterprisedb@ppaslab instantclient_11_2]$ ln -s libclntsh.so.11.1 libclntsh.so
  [enterprisedb@ppaslab instantclient_11_2]$ ln -s libocci.so.11.1 libocci.so
  ```

* 환경변수 설정
  `pgplus_env.sh`에 아래 내용을 추가해 준다.
  ```bash
  export ORACLE_HOME=/opt/PostgresPlus/instantclient_11_2
  export TNS_ADMIN=$ORACLE_HOME
  export LD_LIBRARY_PATH=$ORACLE_HOME:$LD_LIBRARY_PATH
  ```

  ```
  [enterprisedb@ppaslab ~]$ . pgplus_env.sh
  ```

* `data/postgresql.conf`에 아래 내용 추가
  ```ini
  oracle_home ='/opt/PostgresPlus/instantclient_11_2'
  ```

* DB 재기동
  환경변수 적용을 위해 DB를 재기동 해 준다.

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl restart
  waiting for server to shut down.... done
  server stopped
  server starting
  ```

#### `oracle_fdw` 설치

* 컴파일 및 설치
  ```
  [enterprisedb@ppaslab ~]$ mkdir -p ~/work/fdw
  [enterprisedb@ppaslab ~]$ cd ~/work/fdw
  [enterprisedb@ppaslab fdw]$ tar -zxvf /opt/pkgs/oracle_fdw-ORACLE_FDW_1_3_0.tar.gz 
  [enterprisedb@ppaslab fdw]$ cd oracle_fdw-ORACLE_FDW_1_3_0/
  [enterprisedb@ppaslab oracle_fdw-ORACLE_FDW_1_3_0]$ make
  [enterprisedb@ppaslab oracle_fdw-ORACLE_FDW_1_3_0]$ make install
  ```

* Extension 설치
  ```
  edb=# create extension oracle_fdw;
  CREATE EXTENSION
  edb=# \dx oracle_fdw 
                           List of installed extensions
      Name    | Version |    Schema    |              Description               
  ------------+---------+--------------+----------------------------------------
   oracle_fdw | 1.1     | enterprisedb | foreign data wrapper for Oracle access
  (1 row)

  edb=# \dx+ oracle_fdw
       Objects in extension "oracle_fdw"
              Object Description             
  -------------------------------------------
   foreign-data wrapper oracle_fdw
   function oracle_close_connections()
   function oracle_diag(name)
   function oracle_fdw_handler()
   function oracle_fdw_validator(text[],oid)
  (5 rows)
  ```

#### FDW 생성

* Server 및 User mapping 생성

  ```
DROP SERVER IF EXISTS localora_server cascade;
  CREATE SERVER localora_server FOREIGN DATA WRAPPER oracle_fdw
    OPTIONS (dbserver '//127.0.0.1:1521/XE');
  CREATE USER MAPPING FOR enterprisedb SERVER localora_server
    OPTIONS (user 'soe', password 'soe');
  ```

#### Foreign table import

앞서와 같이 테이블들을 하나씩 만들어줄 수도 있지만 9.5에서 생긴 auto import 기능을 이용할 수도 있다.

* 대상 스키마 생성
  한번에 여러 테이블을 가져오게 되면 기존 테이블과 섞이게 될 위험성이 있다. 별도의 schema를 만들어서 여기에 가져온 foreign table들을 모아 두는것을 권장 한다.
  ```
  edb=# create schema oracle_fdw_test;
  CREATE SCHEMA
  edb=# import foreign schema "SOE" from server localora_server into oracle_fdw_test;
  IMPORT FOREIGN SCHEMA
  ```

* 데이터 확인
  `oracle_fdw_test` 스키마에 테이블들이 생성되어 있는 것을 확인할 수 있다.
  ```
  edb=# \d oracle_fdw_test.<TAB>
  oracle_fdw_test.addresses             oracle_fdw_test.orderentry_metadata   oracle_fdw_test.product_prices
  oracle_fdw_test.card_details          oracle_fdw_test.order_items           oracle_fdw_test.products
  oracle_fdw_test.customers             oracle_fdw_test.orders                oracle_fdw_test.warehouses
  oracle_fdw_test.inventories           oracle_fdw_test.product_descriptions  
  oracle_fdw_test.logon                 oracle_fdw_test.product_information   
  edb=# select * from oracle_fdw_test.products limit 10;
   product_id | language_id |    product_name     | category_id |                                                         product_description                                                          | weight_class | warranty_period | supplier_id |  product_status   | list_price | min_price |           catalog_url           
  ------------+-------------+---------------------+-------------+--------------------------------------------------------------------------------------------------------------------------------------+--------------+-----------------+-------------+-------------------+------------+-----------+---------------------------------
          250 | kT          | f 6Lii8OfKeBVuL     |           2 | i1lI3q5ZyZa5gxGJDOn979G7G 3835I84qrHlxNocxmRfkys o3nm2UVG0 fW7y o7 pD5Iz0e1gHP8ks330YNfdW                                            |            5 | 37 years 1 mon  |      953928 | orderable         |    1687.00 |   1968.00 | DEV48zPnKlAPMBjRozb7DwI
          251 | Q           | V Y uiEu6IEqIeRE2   |          16 | q6xkUMgXqWKv1LqgoSoB0PUPxAcRSKd9z0 SlZAk5ZopV8D5h5XDpPB5O CmMq79yid1bFJPLaGkMsRpJeQ0IKsN1tGh2D TrZ24uhqtOcUaNax9xJ                   |            5 | 58 years 1 mon  |      874091 | under development |    4189.00 |   1186.00 | cLt7VZjf35qfV17w0YGaNKvjqnXiyP
          252 | lj          | SXF o VuegFO        |          52 | sY1gVu 2MNJNy2XrtVXahgzA wQedWRwsgv2oHogpmwrfgHX622HKr638NzkguDn7Q79k0OduIlmutiRi9CqVWKzHch                                          |            6 | 42 years 1 mon  |      789468 | orderable         |    4386.00 |    627.00 | rjidean0ODhQw0a
  ...
  edb=# explain select * from oracle_fdw_test.customers where customer_id = 3191;;
                                                                                                                                                                                        QUERY PLAN
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   Foreign Scan on customers  (cost=10000.00..20000.00 rows=1000 width=782)
     Oracle query: SELECT /*bf61c7cef09feecedee5f92bf285665c*/ "CUSTOMER_ID", "CUST_FIRST_NAME", "CUST_LAST_NAME", "NLS_LANGUAGE", "NLS_TERRITORY", "CREDIT_LIMIT", "CUST_EMAIL", "ACCOUNT_MGR_ID", "CUSTOMER_SINCE", "CUSTOMER_CLASS", "SUGGESTIONS", "DOB", "MAILSHOT", "PARTNER_MAILSHOT", "PREFERRED_ADDRESS", "PREFERRED_CARD" FROM "SOE"."CUSTOMERS" WHERE ("CUSTOMER_ID" = 3191)
  (2 rows)
  ```
