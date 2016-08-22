# Lab: Extensions

## pgcrypto

### 설치

* Installation

  ```
  edb=# create extension pgcrypto;
  CREATE EXTENSION
  edb=# \dx pgcrypto
                  List of installed extensions
     Name   | Version |    Schema    |       Description       
  ----------+---------+--------------+-------------------------
   pgcrypto | 1.2     | enterprisedb | cryptographic functions
  (1 row)
  ```

* 내용 확인
  ```
  edb=# \dx+ pgcrypto
              Objects in extension "pgcrypto"
                    Object Description                   
  -------------------------------------------------------
   function armor(bytea)
   function armor(bytea,text[],text[])
   function crypt(text,text)
   function dearmor(text)
   function decrypt(bytea,bytea,text)
   function decrypt_iv(bytea,bytea,bytea,text)
   function digest(bytea,text)
   function digest(text,text)
   function encrypt(bytea,bytea,text)
   function encrypt_iv(bytea,bytea,bytea,text)
   function gen_random_bytes(integer)
   function gen_random_uuid()
   function gen_salt(text)
   function gen_salt(text,integer)
   function hmac(bytea,bytea,text)
   function hmac(text,text,text)
   function pgp_armor_headers(text)
   function pgp_key_id(bytea)
   function pgp_pub_decrypt(bytea,bytea)
   function pgp_pub_decrypt_bytea(bytea,bytea)
   function pgp_pub_decrypt_bytea(bytea,bytea,text)
   function pgp_pub_decrypt_bytea(bytea,bytea,text,text)
   function pgp_pub_decrypt(bytea,bytea,text)
   function pgp_pub_decrypt(bytea,bytea,text,text)
   function pgp_pub_encrypt_bytea(bytea,bytea)
   function pgp_pub_encrypt_bytea(bytea,bytea,text)
   function pgp_pub_encrypt(text,bytea)
   function pgp_pub_encrypt(text,bytea,text)
   function pgp_sym_decrypt_bytea(bytea,text)
   function pgp_sym_decrypt_bytea(bytea,text,text)
   function pgp_sym_decrypt(bytea,text)
   function pgp_sym_decrypt(bytea,text,text)
   function pgp_sym_encrypt_bytea(bytea,text)
   function pgp_sym_encrypt_bytea(bytea,text,text)
   function pgp_sym_encrypt(text,text)
   function pgp_sym_encrypt(text,text,text)
  (36 rows)
  ```

### `encrypt` and `decrypt`

* Blowfish, CBC, PKCS 알고리즘으로 문자열을 암/복호화 해 보자.

	```
  edb=# select encrypt('hello world. EDB Postgres!!', 'my secret key!', 'bf-cbc/pad:pkcs');
                                encrypt                               
  --------------------------------------------------------------------
   \xe59cf5fcf31879739619718eba4843f468db5d67d64e0db6f859427660871602
  (1 row)

  edb=# select convert_from(decrypt(E'\\xe59cf5fcf31879739619718eba4843f468db5d67d64e0db6f859427660871602', 'my secret key!', 'bf-cbc/pad:pkcs'), 'utf-8');
          convert_from         
  -----------------------------
   hello world. EDB Postgres!!
  (1 row)
  ```

* AES, CBC, PKCS 알고리즘으로 문자열을 암/복호화 해 보자.

  ```
  edb=# select encrypt('hello world. EDB Postgres!!', 'my secret key!', 'aes-cbc/pad:pkcs');
                                encrypt                               
  --------------------------------------------------------------------
   \x617dc84e0aff85795ec24cd82ecc69577825f708603eeb32498dabb41bb89b25
  (1 row)

  edb=# select convert_from(decrypt(E'\\x617dc84e0aff85795ec24cd82ecc69577825f708603eeb32498dabb41bb89b25', 'my secret key!', 'aes-cbc/pad:pkcs'), 'utf-8');
          convert_from         
  -----------------------------
   hello world. EDB Postgres!!
  (1 row)
  ```

* Using `gen_random_bytes`

  무작위 byte string을 생성하는 함수인 `gen_random_bytes()`를 사용해 보고, 이를 암호화 키로 사용해 보자.

  ```
  edb=# select gen_random_bytes(20);
                gen_random_bytes              
  --------------------------------------------
   \xbe80ee60710dd74d9fda2f4fe74ed244a11fbea2
  (1 row)

  edb=# select encrypt('hello world. EDB Postgres!!', E'\\xbe80ee60710dd74d9fda2f4fe74ed244a11fbea2', 'aes-ecb/pad:pkcs');
                                encrypt                               
  --------------------------------------------------------------------
   \xa2924c68183ed6628877a34ca82fce935a81f1a70eb24fc4f4c119fb791fccab
  (1 row)

  edb=# select encrypt('hello world. EDB Postgres!!', E'\\xbe80ee60710dd74d9fda2f4fe74ed244a11fbea2', 'aes-cbc/pad:pkcs');
                                encrypt                               
  --------------------------------------------------------------------
   \xa2924c68183ed6628877a34ca82fce93ee55d47773a8ee89a71a2fdaa5ae9743
  (1 row)

  edb=# select encrypt('hello world. EDB Postgres!!', E'\\xbe80ee60710dd74d9fda2f4fe74ed244a11fbea2', 'bf-cbc/pad:pkcs');
                                encrypt                               
  --------------------------------------------------------------------
   \xeb10ed95a1c21ed242345c34361e9f81fa77c92b4ee785d547d345f437f544ea
  (1 row)
  ```

### Hash functions

* MD5 hash 알고리즘을 이용하여 문자열 hashing을 해 보자.

  ```
  edb=# select digest('hello world. EDB Postgres!!', 'md5');
                 digest               
  ------------------------------------
   \x24c367e528576ed3bc5ce2ffd0da8ff9
  (1 row)
  ```

* SHA256 hash 알고리즘을 이용하여 문자열 hashing을 해 보자.

  ```
  edb=# select digest('hello world. EDB Postgres!!', 'sha256');
                                 digest                               
  --------------------------------------------------------------------
   \xd3132f5b34b931f403ec6e4c1d5863aebbcb2a8b2279f3ba5682ef92cad568c6
  (1 row)
  ```

* SHA512 hash 알고리즘을 이용하여 문자열 hashing을 해 보자.

  ```
  edb=# select digest('hello world. EDB Postgres!!', 'sha512');
                                                                 digest                                                               
  ------------------------------------------------------------------------------------------------------------------------------------
   \xc6af845609d8e19672afb8a871299887912b3a1302788ec148ad2231df98a2056fdcf610bb62a4dc19ca33fb574f935104e3a529d50589b913bda5226cfae3bf
  (1 row)
  ```

## pg_buffercache

### 설치

* Installation

  ```
  edb=# create extension pg_buffercache;
  CREATE EXTENSION
  edb=# \dx pg_buffercache
                         List of installed extensions
        Name      | Version |    Schema    |           Description           
  ----------------+---------+--------------+---------------------------------
   pg_buffercache | 1.1     | enterprisedb | examine the shared buffer cache
  (1 row)
  ```

* 내용 확인
  ```
  edb=# \dx+ pg_buffercache
  Objects in extension "pg_buffercache"
         Object Description        
  ---------------------------------
   function pg_buffercache_pages()
   view pg_buffercache
  (2 rows)
	```

### Buffer cache의 내용 확인

* 테이블 별 buffer 점유 현황
  아래 쿼리로 각 테이블당 몇개의 buffer를 점유하고 있는지 확인 할 수 있다.

  ```sql
  SELECT c.relname, count(*) AS buffers
    FROM pg_buffercache b INNER JOIN pg_class c
    ON b.relfilenode = pg_relation_filenode(c.oid) AND
       b.reldatabase IN (0, (SELECT oid FROM pg_database
                             WHERE datname = current_database()))
    GROUP BY c.relname
    ORDER BY 2 DESC
    LIMIT 10;
  ```

* dirty buffer 확인

  `pg_buffercache`의 `isdirty`로 dirty buffer의 수를 확인할 수 있다.

  ```
  edb=# select isdirty, count(*) from pg_buffercache group by isdirty;
   isdirty | count
  ---------+-------
           | 54180
   f       |  5892
  (2 rows)
  ```

  지금은 dirty인 buffer가 없다. dirty buffer를 생성해 보자.

  ```
  edb=# update test set id = id + 100;
  UPDATE 100000
  edb=# select isdirty, count(*) from pg_buffercache group by isdirty;
   isdirty | count
  ---------+-------
           | 52050
   t       |  3647
   f       |  4375
  (3 rows)
  ```

  dirty buffer (`isdirty`가 `t`인 버퍼들)가 발생했다. dirty buffer를 어떻게 제거할 수 있을까? 앞서 배운 내용을 바탕으로 dirty buffer를 제거해 보자.

## pg_freespacemap

`pg_freespacemap`을 이용하면 FSM의 내용을 확인 할 수 있다.

### 설치

* Installation

  ```
  edb=# create extension pg_freespacemap;
  CREATE EXTENSION
  edb=# \dx pg_freespacemap              
                          List of installed extensions
        Name       | Version |    Schema    |           Description            
  -----------------+---------+--------------+----------------------------------
   pg_freespacemap | 1.0     | enterprisedb | examine the free space map (FSM)
  (1 row)
  ```

* 내용 확인
  ```
  edb=# \dx+ pg_freespacemap
   Objects in extension "pg_freespacemap"
             Object Description           
  ----------------------------------------
   function pg_freespace(regclass)
   function pg_freespace(regclass,bigint)
  (2 rows)
	```

### FSM 내용 확인

* 테스트용 테이블을 준비한다.
	```sql
  drop table if exists test;
  create table test
  (
    id integer,
    val1 bytea,
    val2 bytea,
    val3 varchar
  )
    with (autovacuum_enabled = false);

  insert into test
  select
    generate_series(1, 100) as id,
    gen_random_bytes(30) as val1,
    gen_random_bytes(30) as val2,
    varchar 'test' as val3;

	update test set id = id + 100;
	update test set id = id + 100;
	update test set id = id + 100;
	```

* test 테이블의 FSM 정보를 확인해 보자.

  ```
  edb=# select * from pg_freespace('test');
   blkno | avail
  -------+-------
       0 |    64
       1 |    64
       2 |    64
       3 |    64
       4 |     0
  (5 rows)
  ```

* vacuum으로 dead tuple을 제거하고 FSM을 update 한다.

  ```
  edb=# vacuum verbose test;
  INFO:  vacuuming "enterprisedb.test"
  INFO:  "test": removed 100 row versions in 4 pages
  INFO:  "test": found 100 removable, 100 nonremovable row versions in 5 out of 5 pages
  DETAIL:  0 dead row versions cannot be removed yet.
  There were 18 unused item pointers.
  Skipped 0 pages due to buffer pins.
  0 pages are entirely empty.
  CPU 0.00s/0.00u sec elapsed 0.00 sec.
  INFO:  vacuuming "pg_toast.pg_toast_16884"
  INFO:  index "pg_toast_16884_index" now contains 0 row versions in 1 pages
  DETAIL:  0 index row versions were removed.
  0 index pages have been deleted, 0 are currently reusable.
  CPU 0.00s/0.00u sec elapsed 0.00 sec.
  INFO:  "pg_toast_16884": found 0 removable, 0 nonremovable row versions in 0 out of 0 pages
  DETAIL:  0 dead row versions cannot be removed yet.
  There were 0 unused item pointers.
  Skipped 0 pages due to buffer pins.
  0 pages are entirely empty.
  CPU 0.00s/0.00u sec elapsed 0.00 sec.
  VACUUM
  ```

* 다시 FSM 정보를 확인한다.

  ```
  edb=# select * from pg_freespace('test');
   blkno | avail
  -------+-------
       0 |  7840
       1 |  5952
       2 |  4128
       3 |  3808
       4 |  7840
  (5 rows)
  ```

## pageinspect

`pageinspect`를 이용하여 low level의 db page 정보를 확인해 보자.

### 설치

* Installation

  ```
  edb=# create extension pageinspect;
  CREATE EXTENSION
  edb=# \dx pageinspect
                                   List of installed extensions
      Name     | Version |    Schema    |                      Description                      
  -------------+---------+--------------+-------------------------------------------------------
   pageinspect | 1.3     | enterprisedb | inspect the contents of database pages at a low level
  (1 row)
  ```

* 내용 확인
  ```
  edb=# \dx+ pageinspect
      Objects in extension "pageinspect"
              Object Description            
  ------------------------------------------
   function brin_metapage_info(bytea)
   function brin_page_items(bytea,regclass)
   function brin_page_type(bytea)
   function brin_revmap_data(bytea)
   function bt_metap(text)
   function bt_page_items(text,integer)
   function bt_page_stats(text,integer)
   function fsm_page_contents(bytea)
   function get_raw_page(text,integer)
   function get_raw_page(text,text,integer)
   function gin_leafpage_items(bytea)
   function gin_metapage_info(bytea)
   function gin_page_opaque_info(bytea)
   function heap_page_items(bytea)
   function page_header(bytea)
	```

### Heap page

* Page header
  PostgreSQL의 모든 파일은 동일한 형태의 8K page로 포맷팅 되어 있다. 각 page는 공통된 header 정보를 가지고 있는데 아래와 같이 확인이 가능하다.

  ```
  edb=# select * from page_header(get_raw_page('test', 0));    
      lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid
  ------------+----------+-------+-------+-------+---------+----------+---------+-----------
   0/5665FF78 |        0 |     5 |   348 |  8192 |    8192 |     8192 |       4 |         0
  (1 row)

  edb=# select * from page_header(get_raw_page('test', 1));    
      lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid
  ------------+----------+-------+-------+-------+---------+----------+---------+-----------
   0/56662148 |        0 |     5 |   492 |  6464 |    8192 |     8192 |       4 |         0
  (1 row)

  edb=# select * from page_header(get_raw_page('emp', 0));
      lsn    | checksum | flags | lower | upper | special | pagesize | version | prune_xid
  -----------+----------+-------+-------+-------+---------+----------+---------+-----------
   0/2403D90 |        0 |     0 |    80 |  7160 |    8192 |     8192 |       4 |         0
  (1 row)

  edb=# select * from page_header(get_raw_page('emp', 1));
  ERROR:  block number 1 is out of range for relation "emp"
  ```

* Tuple list
  테이블의 page에는 튜플들이 들어있다. 아래와 같이 정보를 가져올 수 있다.

  ```
  edb=# select * from heap_page_items(get_raw_page('emp', 0));
   lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff |  t_bits  | t_oid
  ----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+----------+-------
    1 |   8120 |        1 |     66 |   1819 |   1819 |       40 | (0,1)  |           8 |       2451 |     24 | 11111101 |      
    2 |   8040 |        1 |     79 |   1819 |   1819 |       42 | (0,2)  |           8 |       2450 |     24 |          |      
    3 |   7968 |        1 |     71 |   1819 |   1819 |       44 | (0,3)  |           8 |       2450 |     24 |          |      
    4 |   7896 |        1 |     66 |   1819 |   1819 |       46 | (0,4)  |           8 |       2451 |     24 | 11111101 |      
    5 |   7816 |        1 |     79 |   1819 |   1819 |       48 | (0,5)  |           8 |       2450 |     24 |          |      
    6 |   7744 |        1 |     66 |   1819 |   1819 |       50 | (0,6)  |           8 |       2451 |     24 | 11111101 |      
    7 |   7672 |        1 |     66 |   1819 |   1819 |       52 | (0,7)  |           8 |       2451 |     24 | 11111101 |      
    8 |   7600 |        1 |     66 |   1819 |   1819 |       54 | (0,8)  |           8 |       2451 |     24 | 11111101 |      
    9 |   7528 |        1 |     66 |   1819 |   1819 |       56 | (0,9)  |           8 |       2451 |     24 | 11101101 |      
   10 |   7448 |        1 |     77 |   1819 |   1819 |       58 | (0,10) |           8 |       2450 |     24 |          |      
   11 |   7376 |        1 |     66 |   1819 |   1819 |       60 | (0,11) |           8 |       2451 |     24 | 11111101 |      
   12 |   7304 |        1 |     66 |   1819 |   1819 |       62 | (0,12) |           8 |       2451 |     24 | 11111101 |      
   13 |   7232 |        1 |     66 |   1819 |   1819 |       64 | (0,13) |           8 |       2451 |     24 | 11111101 |      
   14 |   7160 |        1 |     66 |   1819 |   1819 |       66 | (0,14) |           8 |       2451 |     24 | 11111101 |      
  (14 rows)
  ```

### B-Tree index

* B-Tree index meta page
  B-Tree index의 첫번째 page는 특수한 정보가 들어있는 meta page이다. 아래와 같이 정보를 조회할 수 있다.

  ```
  edb=# SELECT * FROM bt_metap('emp_pk');             
   magic  | version | root | level | fastroot | fastlevel
  --------+---------+------+-------+----------+-----------
   340322 |       2 |    1 |     0 |        1 |         0
  (1 row)
  ```

* B-Tree index의 각 page별 요약 정보는 아래와 같이 조회 할 수 있다.
  ```
  edb=# SELECT * FROM bt_page_stats('emp_pk', 0);
  ERROR:  block 0 is a meta page
  edb=# SELECT * FROM bt_page_stats('emp_pk', 1);
   blkno | type | live_items | dead_items | avg_item_size | page_size | free_size | btpo_prev | btpo_next | btpo | btpo_flags
  -------+------+------------+------------+---------------+-----------+-----------+-----------+-----------+------+------------
       1 | l    |         14 |          0 |            16 |      8192 |      7868 |         0 |         0 |    0 |          3
  (1 row)

  edb=# SELECT * FROM bt_page_stats('emp_pk', 2);
  ERROR:  block number out of range
  ```

## pg_prewarm

`pg_prewarm`을 이용하면 테이블을 cache에 미리 올려 놓을 수 있다.

### 설치

* Installation

  ```
  edb=# create extension pg_prewarm;
  CREATE EXTENSION
  edb=# \dx pg_prewarm
                  List of installed extensions
      Name    | Version |    Schema    |      Description      
  ------------+---------+--------------+-----------------------
   pg_prewarm | 1.0     | enterprisedb | prewarm relation data
  (1 row)
  ```

* 내용 확인
  ```
  edb=# \dx+ pg_prewarm
             Objects in extension "pg_prewarm"
                    Object Description                   
  -------------------------------------------------------
   function pg_prewarm(regclass,text,text,bigint,bigint)
  (1 row)
	```

### 테스트

* 테스트용 테이블 생성

	```sql
  drop table if exists test;
  create table test
  (
    id integer,
    val1 bytea,
    val2 bytea,
    val3 varchar
  )
    with (autovacuum_enabled = false);

  insert into test
  select
    generate_series(1, 10000) as id,
    gen_random_bytes(30) as val1,
    gen_random_bytes(30) as val2,
    varchar 'test' as val3;
	```

* DB restart
  DB를 restart하여 버퍼를 모두 비워 보자

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl restart
  waiting for server to shut down.... done
  server stopped
  server starting
  ```

* Buffer의 내용 확인

  ```
  edb=# SELECT c.relname, count(*) AS buffers
          FROM pg_buffercache b INNER JOIN pg_class c
          ON b.relfilenode = pg_relation_filenode(c.oid) AND
             b.reldatabase IN (0, (SELECT oid FROM pg_database
                                   WHERE datname = current_database()))
          GROUP BY c.relname
          ORDER BY 2 DESC
          LIMIT 10;
               relname             | buffers
  ---------------------------------+---------
   pg_attribute                    |      26
   pg_class                        |      20
   pg_attribute_relid_attnum_index |       8
   pg_proc                         |       8
   pg_proc_oid_index               |       6
   pg_class_oid_index              |       4
   pg_amproc                       |       4
   pg_type                         |       4
   pg_amop                         |       4
   pg_class_relname_nsp_index      |       4
  (10 rows)
  ```

* prewarm

  ```
  edb=# select pg_prewarm('test');
   pg_prewarm
  ------------
          124
  (1 row)
  edb=# SELECT c.relname, count(*) AS buffers
          FROM pg_buffercache b INNER JOIN pg_class c
          ON b.relfilenode = pg_relation_filenode(c.oid) AND
             b.reldatabase IN (0, (SELECT oid FROM pg_database
                                   WHERE datname = current_database()))
          GROUP BY c.relname
          ORDER BY 2 DESC
          LIMIT 10;
               relname             | buffers
  ---------------------------------+---------
   test                            |     124
   pg_attribute                    |      27
   pg_class                        |      20
   pg_proc                         |       9
   pg_attribute_relid_attnum_index |       8
   pg_proc_oid_index               |       6
   pg_class_relname_nsp_index      |       5
   pg_class_oid_index              |       4
   pg_type                         |       4
   pg_amop                         |       4
  (10 rows)
	```

## pg_stat_statements

### 설치

* Installation

	1. `shared_preload_libraries` in `postgresql.conf`
		이 extension은 별도의 shared memory를 사용하기 때문에 DB 기동시에 미리 로딩이 되어야 한다.
    `postgresql.conf`의 `shared_preload_libraries`에 아래와 같이 `pg_stat_statements`를 추가해 준다.

		```
    shared_preload_libraries = '$libdir/dbms_pipe,$libdir/edb_gen,$libdir/pg_stat_statements'
		```

	2. DB 재기동

    ```
    [enterprisedb@ppaslab ~]$ pg_ctl restart
    waiting for server to shut down.... done
    server stopped
    server starting
    ```
	3. 설치

    ```
    edb=# create extension pg_stat_statements;
    CREATE EXTENSION
    edb=# \dx pg_stat_statements
                                          List of installed extensions
            Name        | Version |    Schema    |                        Description                        
    --------------------+---------+--------------+-----------------------------------------------------------
     pg_stat_statements | 1.3     | enterprisedb | track execution statistics of all SQL statements executed
    (1 row)
    ```

* 내용 확인
  ```
  edb=# \dx+ pg_stat_statements
  Objects in extension "pg_stat_statements"
            Object Description          
  --------------------------------------
   function pg_stat_statements(boolean)
   function pg_stat_statements_reset()
   view pg_stat_statements
  (3 rows)
	```

### 테스트

* 통계정보 reset
  ```
  edb=# select pg_stat_statements_reset();
   pg_stat_statements_reset
  --------------------------

  (1 row)
  ```

* 쿼리 실행

  ```
  edb=# select count(*) from test;
   count
  -------
   10000
  (1 row)

  -- 4회 반복 실행 --

  edb=# select sum(id) from test;
     sum    
  ----------
   50005000
  (1 row)

  -- 5회 반복 실행 --
  ```

* 정보 확인
  ```
  edb=# \x
  Expanded display is on.
  edb=# SELECT query, calls, total_time, rows, 100.0 * shared_blks_hit /
                       nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
                   FROM pg_stat_statements ORDER BY total_time DESC LIMIT 3;
  -[ RECORD 1 ]-----------------------------------
  query       | select sum(id) from test;
  calls       | 5
  total_time  | 5.473
  rows        | 5
  hit_percent | 100.0000000000000000
  -[ RECORD 2 ]-----------------------------------
  query       | select count(*) from test;
  calls       | 4
  total_time  | 3.741
  rows        | 4
  hit_percent | 100.0000000000000000
  -[ RECORD 3 ]-----------------------------------
  query       | select pg_stat_statements_reset();
  calls       | 1
  total_time  | 0.1
  rows        | 1
  hit_percent |
  ```

## pg_repack

https://github.com/reorg/pg_repack

pg_repack은 PostgreSQL의 extension으로 bloat된 table과 index를 정리할 수 있게 해 준다. 추가적으로 테이블의 물리적 구조를 clustered index들의 순서에 맞게 정렬해 준다. `CLUSTER`나 `VACUUM FULL`과 다른 점은 이 모든 작업이 온라인 중에 수행되며, exclusie lock을 잡지 않는다는 것이다.

아래 중 한가지 방식을 선택할 수 있다.

* Online CLUSTER (cluster index 순으로 정렬)
* 지정한 컬럼 순으로 정렬
* Online VACUUM FULL
* Rebuild or relocate only the indexes of a table

#### 주의 사항:

* Superuser만 사용할 수 있다.
* 대상 테이블은 반드시 PRIMARY KEY 또는 최소한 한개의 NOT NULL UNIQUE 컬럼을 가지고 있어야 한다.

#### Installation

pg_repack은 UNIX나 Linux에서 make로 컴파일 할 수 있다. PGXS를 사용하기 때문에 컴파일 전에 `pg_config`가 포함된 경로를 PATH에 추가해 주어야 한다.

*추가로 현제 버전의 PPAS에서 pg_repack을 컴파일 할 경우 library들을 찾지 못하는 문제가 있다. 아래와 같이 symbolic link를 만들어 주면 해결 된다.*

```
cd ~/lib/
ln -s libmemcached.so.11 libmemcached.so
ln -s libssl.so.1.0.0 libssl.so
ln -s libcrypto.so.1.0.0 libcrypto.so
ln -s libgssapi_krb5.so.2 libgssapi_krb5.so
ln -s libz.so.1 libz.so
ln -s libedit.so.0 libedit.so
```

이제 컴파일 해 보자.

```
mkdir ~/work
cd ~/work/
unzip /opt/pkgs/pg_repack-master.zip
cd pg_repack-master
make
make install
```

설치가 되었다면 pg_repack extension을 설치해 주어야 한다.

```
CREATE EXTENSION pg_repack;
```

#### Examples

* 테스트용 테이블 준비
	```sql
  drop table if exists test;
  create table test
  (
    id integer constraint test_pk primary key,
    val1 bytea,
    val2 bytea,
		val3 varchar
  )
    with (autovacuum_enabled = false);

  insert into test
  select
    generate_series(1, 100000) as id,
    gen_random_bytes(30) as val1,
    gen_random_bytes(30) as val2,
    varchar 'test' as val3;
	```

* `test` 테이블의 `test_pk`을 clustered index로 지정

  ```
  edb=# alter table test cluster on test_pk;
  ALTER TABLE
  ```

* val2 컬럼의 순서로 online clustering
  난수인 val2 컬럼의 순서로 clustering 하여 index scan의 효율을 저하시켜 보자.

  ```
  $ pg_repack -t enterprisedb.test -o val2 edb
  INFO: repacking table "enterprisedb.test"
  ```

  ```
  edb=# explain (analyze, buffers) select * from test where id >= 100 and id < 300;
                                                       QUERY PLAN                                                     
  --------------------------------------------------------------------------------------------------------------------
   Bitmap Heap Scan on test  (cost=6.30..527.64 rows=196 width=71) (actual time=0.048..0.325 rows=200 loops=1)
     Recheck Cond: ((id >= 100) AND (id < 300))
     Heap Blocks: exact=189
     Buffers: shared hit=191
     ->  Bitmap Index Scan on test_pk  (cost=0.00..6.25 rows=196 width=0) (actual time=0.025..0.025 rows=200 loops=1)
           Index Cond: ((id >= 100) AND (id < 300))
           Buffers: shared hit=2
   Planning time: 0.213 ms
   Execution time: 0.381 ms
  (9 rows)
  ```

* 다시 id 컬럼 순으로 clustering 하여 차이를 확인해 보자.

  ```
  [enterprisedb@ppaslab ~]$ pg_repack -t enterprisedb.test edb
  INFO: repacking table "enterprisedb.test"
  ```

  ```
  edb=# explain (analyze, buffers) select * from test where id >= 100 and id < 300;
                                                       QUERY PLAN                                                     
  --------------------------------------------------------------------------------------------------------------------
   Index Scan using test_pk on test  (cost=0.29..14.35 rows=203 width=71) (actual time=0.009..0.037 rows=200 loops=1)
     Index Cond: ((id >= 100) AND (id < 300))
     Buffers: shared hit=5
   Planning time: 0.229 ms
   Execution time: 0.129 ms
  (5 rows)
  ```

* `edb` DB에 대해 모든 clustered table 들은 online CLUSTER를 non-clustered table 들은 online FULL VACUUM을 실행하는 명령

  ```
  $ pg_repack edb
  ```

* `edb` DB의 테이블 중 `test`와 `emp` 두개의 테이블만 online FULL VACUUM을 하는 명령, Clustering 안함

  ```
  $ pg_repack --no-order --table dept --table emp edb
  ```
