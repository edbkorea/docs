# Lab: Tablespace

## 정보 확인

테이블 스페이스의 목록은 psql 명령어 `\db` 또는 `pg_catalog.pg_tablespace` 뷰를 통해 조회 할 수 있다.
```
edb=# \db+
                                    List of tablespaces
    Name    |    Owner     | Location | Access privileges | Options |  Size  | Description 
------------+--------------+----------+-------------------+---------+--------+-------------
 pg_default | enterprisedb |          |                   |         | 54 MB  | 
 pg_global  | enterprisedb |          |                   |         | 608 kB | 
(2 rows)

edb=# select oid, * from pg_tablespace;
 oid  |  spcname   | spcowner | spcacl | spcoptions 
------+------------+----------+--------+------------
 1663 | pg_default |       10 |        | 
 1664 | pg_global  |       10 |        | 
(2 rows)

```

## 생성

* 디랙토리 생성
    미리 디랙토리를 생성해 둬야 한다. 별도의 디스크를 사용할 예정이라면 미리 마운트 포인트를 생성하고 마운트 한다.

    ```
    [enterprisedb@ppaslab ~]$ mkdir -p /opt/PostgresPlus/tbs/tbs01
    ```

* 테이블스페이스 생성

    ```
    edb=# \h create tablespace
    Command:     CREATE TABLESPACE
    Description: define a new tablespace
    Syntax:
    CREATE TABLESPACE tablespace_name
        [ OWNER { new_owner | CURRENT_USER | SESSION_USER } ]
        LOCATION 'directory'
        [ WITH ( tablespace_option = value [, ... ] ) ]

    edb=# create tablespace tbs01 location '/opt/PostgresPlus/tbs/tbs01';
    CREATE TABLESPACE
    ```

* 결과 확인
    정상적으로 생성 된 것을 확인할 수 있다.
    ```
    edb=# \db+
                                                  List of tablespaces
        Name    |    Owner     |          Location           | Access privileges | Options |  Size   | Description 
    ------------+--------------+-----------------------------+-------------------+---------+---------+-------------
     pg_default | enterprisedb |                             |                   |         | 54 MB   | 
     pg_global  | enterprisedb |                             |                   |         | 608 kB  | 
     tbs01      | enterprisedb | /opt/PostgresPlus/tbs/tbs01 |                   |         | 0 bytes | 
    (3 rows)

    edb=# select oid, * from pg_tablespace;
      oid  |  spcname   | spcowner | spcacl | spcoptions 
    -------+------------+----------+--------+------------
      1663 | pg_default |       10 |        | 
      1664 | pg_global  |       10 |        | 
     17500 | tbs01      |       10 |        | 
    (3 rows)
    ```

    여기서 oid가 중요한데 `data/pg_tblspc`에 oid를 파일 이름으로 하는 symbolic link가 생성되어 있는 것을 볼 수 있다.

    ```
    [enterprisedb@ppaslab ~]$ cd data/pg_tblspc/
    [enterprisedb@ppaslab pg_tblspc]$ ls -l
    total 0
    lrwxrwxrwx. 1 enterprisedb enterprisedb 27 Feb 13 22:35 17500 -> /opt/PostgresPlus/tbs/tbs01
    ```

## 사용

* 테이블 생성시 지정
  ```
  create extension if not exists pgcrypto;
  drop table if exists test;
  create table test
    tablespace tbs01
  as
  select
    generate_series(1, 10000) as id,
    gen_random_bytes(30) as val1,
    gen_random_bytes(30) as val2,
    varchar 'test' as val3;
  ```

  ```
  edb=# \d test
         Table "enterprisedb.test"
   Column |       Type        | Modifiers 
  --------+-------------------+-----------
   id     | integer           | 
   val1   | bytea             | 
   val2   | bytea             | 
   val3   | character varying | 
  Tablespace: "tbs01"
  ```

* 기존 테이블 이동

  ```
  create extension if not exists pgcrypto;
  drop table if exists test;
  create table test
  as
  select
    generate_series(1, 10000) as id,
    gen_random_bytes(30) as val1,
    gen_random_bytes(30) as val2,
    varchar 'test' as val3;
  ```

  아래와 같이 `alter table` 명령을 이용해 다른 테이블스페이로 이동할 수 있다.

  ```
  edb=# \d test
         Table "enterprisedb.test"
   Column |       Type        | Modifiers 
  --------+-------------------+-----------
   id     | integer           | 
   val1   | bytea             | 
   val2   | bytea             | 
   val3   | character varying | 

  edb=# alter table test set tablespace tbs01;
  ALTER TABLE
  edb=# \d test
         Table "enterprisedb.test"
   Column |       Type        | Modifiers 
  --------+-------------------+-----------
   id     | integer           | 
   val1   | bytea             | 
   val2   | bytea             | 
   val3   | character varying | 
  Tablespace: "tbs01"
  ```

## 테이블 스페이스 이동

아래와 같이 이미 만들어 둔 테이블 스페이스를 다른 경로로 옮길 수 있다.

1. DB shutdown

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl stop
  waiting for server to shut down.... done
  server stopped
  ```

2. 데이터 이동

  ```
  [enterprisedb@ppaslab ~]$ mv /opt/PostgresPlus/tbs/tbs01 /opt/PostgresPlus/tbs01
  ```

3. Symbolic link 수정

  ```
  [enterprisedb@ppaslab ~]$ cd data/pg_tblspc/
  [enterprisedb@ppaslab pg_tblspc]$ ls -l
  total 0
  lrwxrwxrwx. 1 enterprisedb enterprisedb 27 Feb 13 22:35 17500 -> /opt/PostgresPlus/tbs/tbs01
  [enterprisedb@ppaslab pg_tblspc]$ ln -sf /opt/PostgresPlus/tbs01 17500
  [enterprisedb@ppaslab pg_tblspc]$ ls -l
  total 0
  lrwxrwxrwx. 1 enterprisedb enterprisedb 23 Feb 13 22:51 17500 -> /opt/PostgresPlus/tbs01
  ```

4. DB 기동

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl start
  server starting
  ```

5. 데이터 확인

  ```
  [enterprisedb@ppaslab ~]$ psql
  psql.bin (9.5.0.5)
  Type "help" for help.

  edb=# \d test
         Table "enterprisedb.test"
   Column |       Type        | Modifiers 
  --------+-------------------+-----------
   id     | integer           | 
   val1   | bytea             | 
   val2   | bytea             | 
   val3   | character varying | 
  Tablespace: "tbs01"

  edb=# select count(*) from test;
   count 
  -------
   10000
  (1 row)
  ```
