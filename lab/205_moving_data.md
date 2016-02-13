# Lab. Moving Data

## COPY

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