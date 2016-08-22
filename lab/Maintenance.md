# Lab: Routine Maintenance

## Analyze

아래 실습을 통해 통계정보의 생성과 관리의 중요성을 확인해 보자.

* Table 생성
  ```sql
  create extension pgcrypto;

  drop table if exists test;
  create table test
  (
    id integer,
    val1 bytea,
    val2 bytea,
    val3 varchar
  )
    with (autovacuum_enabled = false);

  create index test_idx1 on test(id);
  create index test_idx2 on test(val3);

  insert into test
  select
    generate_series(1, 100000) as id,
    gen_random_bytes(30) as val1,
    gen_random_bytes(30) as val2,
    varchar 'test' as val3;
  ```
* 통계정보 확인
	```
  edb=# explain select * from test where id = 10;
                                  QUERY PLAN
  ---------------------------------------------------------------------------
   Bitmap Heap Scan on test  (cost=11.31..836.05 rows=389 width=100)
     Recheck Cond: (id = 10)
     ->  Bitmap Index Scan on test_idx1  (cost=0.00..11.21 rows=389 width=0)
           Index Cond: (id = 10)
  (4 rows)

  edb=# explain select * from test where val3 = 'test';
                                  QUERY PLAN
  ---------------------------------------------------------------------------
   Bitmap Heap Scan on test  (cost=11.31..836.05 rows=389 width=100)
     Recheck Cond: ((val3)::text = 'test'::text)
     ->  Bitmap Index Scan on test_idx2  (cost=0.00..11.21 rows=389 width=0)
           Index Cond: ((val3)::text = 'test'::text)
  (4 rows)

	edb=# select tablename, attname, avg_width, n_distinct from pg_stats where tablename = 'test';
   tablename | attname | avg_width | n_distinct
  -----------+---------+-----------+------------
  (0 rows)
	```
* 통계정보 갱신
  ```
  edb=# analyze verbose test;
  INFO:  analyzing "enterprisedb.test"
  INFO:  "test": scanned 1235 of 1235 pages, containing 100000 live rows and 0 dead rows; 30000 rows in sample, 100000 estimated total rows
  ANALYZE
  edb=# select tablename, attname, avg_width, n_distinct from pg_stats where tablename = 'test';
   tablename | attname | avg_width | n_distinct
  -----------+---------+-----------+------------
   test      | id      |         4 |         -1
   test      | val1    |        31 |         -1
   test      | val2    |        31 |         -1
   test      | val3    |         5 |          1
  (4 rows)
  edb=# explain select * from test where id = 10;
                                QUERY PLAN                               
  -----------------------------------------------------------------------
   Index Scan using test_idx1 on test  (cost=0.29..8.31 rows=1 width=71)
     Index Cond: (id = 10)
  (2 rows)

  edb=# explain select * from test where val3 = 'test';
                           QUERY PLAN                          
  -------------------------------------------------------------
   Seq Scan on test  (cost=0.00..2485.00 rows=100000 width=71)
     Filter: ((val3)::text = 'test'::text)
  (2 rows)

	```

통계 정보 갱신 전 후의 실행 계획을 분석해 보고 그 차이에 대해 고찰해 본다.

## Vacuum

### vacuum 과 vacuum full

* 테이블 용량 확인
  ```
  edb=# \dt+ test
                           List of relations
      Schema    | Name | Type  |    Owner     |  Size   | Description 
  --------------+------+-------+--------------+---------+-------------
   enterprisedb | test | table | enterprisedb | 9912 kB | 
  (1 row)
  ```

* Update로 dead tuple 생성
  많은 dead tuple이 생성되어 테이블의 크기가 증가한 것을 볼 수 있다.
  ```
  edb=# update test set id = id + 100;
  UPDATE 100000
  edb=# update test set id = id + 100;
  UPDATE 100000
  edb=# \dt+ test
                          List of relations
      Schema    | Name | Type  |    Owner     | Size  | Description 
  --------------+------+-------+--------------+-------+-------------
   enterprisedb | test | table | enterprisedb | 29 MB | 
  (1 row)
  ```

* vacuum 수행
  vacuum을 verbose 모드로 실행하고 그 출력 결과들을 분석 해 본다.
  ```
  edb=# vacuum verbose test;
  INFO:  vacuuming "enterprisedb.test"
  INFO:  scanned index "test_idx1" to remove 200000 row versions
  DETAIL:  CPU 0.00s/0.03u sec elapsed 0.03 sec.
  INFO:  scanned index "test_idx2" to remove 200000 row versions
  DETAIL:  CPU 0.00s/0.03u sec elapsed 0.05 sec.
  INFO:  "test": removed 200000 row versions in 2470 pages
  DETAIL:  CPU 0.00s/0.00u sec elapsed 0.01 sec.
  INFO:  index "test_idx1" now contains 100000 row versions in 1099 pages
  DETAIL:  200000 index row versions were removed.
  1 index pages have been deleted, 0 are currently reusable.
  CPU 0.00s/0.00u sec elapsed 0.00 sec.
  INFO:  index "test_idx2" now contains 100000 row versions in 1034 pages
  DETAIL:  200000 index row versions were removed.
  682 index pages have been deleted, 0 are currently reusable.
  CPU 0.00s/0.00u sec elapsed 0.00 sec.
  INFO:  "test": found 100000 removable, 100000 nonremovable row versions in 3704 out of 3704 pages
  DETAIL:  0 dead row versions cannot be removed yet.
  There were 0 unused item pointers.
  Skipped 0 pages due to buffer pins.
  0 pages are entirely empty.
  CPU 0.00s/0.08u sec elapsed 0.13 sec.
  INFO:  vacuuming "pg_toast.pg_toast_16740"
  INFO:  index "pg_toast_16740_index" now contains 0 row versions in 1 pages
  DETAIL:  0 index row versions were removed.
  0 index pages have been deleted, 0 are currently reusable.
  CPU 0.00s/0.00u sec elapsed 0.00 sec.
  INFO:  "pg_toast_16740": found 0 removable, 0 nonremovable row versions in 0 out of 0 pages
  DETAIL:  0 dead row versions cannot be removed yet.
  There were 0 unused item pointers.
  Skipped 0 pages due to buffer pins.
  0 pages are entirely empty.
  CPU 0.00s/0.00u sec elapsed 0.00 sec.
  VACUUM
  ```

* vacuum은 공간을 회수하지 않음을 볼 수 있다.
  ```
  edb=# \dt+ test
                          List of relations
      Schema    | Name | Type  |    Owner     | Size  | Description 
  --------------+------+-------+--------------+-------+-------------
   enterprisedb | test | table | enterprisedb | 29 MB | 
  (1 row)
  ```

* 다시한번 dead tuple들을 생성하여 본다.
  이때 vacuum으로 확보된 공간을 다시 사용하고 있음을 볼 수 있다.
  ```
  edb=# update test set id = id + 100;
  UPDATE 100000
  edb=# update test set id = id + 100;
  UPDATE 100000
  edb=# \dt+ test                     
                          List of relations
      Schema    | Name | Type  |    Owner     | Size  | Description 
  --------------+------+-------+--------------+-------+-------------
   enterprisedb | test | table | enterprisedb | 29 MB | 
  (1 row)
  ```

* vacuum full을 실행하여 공간이 회수되는지 확인한다.
  ```
  edb=# vacuum full test;
  VACUUM
  edb=# \dt+ test        
                           List of relations
      Schema    | Name | Type  |    Owner     |  Size   | Description 
  --------------+------+-------+--------------+---------+-------------
   enterprisedb | test | table | enterprisedb | 9888 kB | 
  (1 row)
  ```


### vacuum full의 특성
vacuum full은 테이블에 lock을 걸고, 새로운 파일을 만들어 live tuple들만 옮겨 담고, 기존 파일을 삭제한다. 즉 파일이 변경된다. 이를 확인해 보자.

```
edb=# select pg_relation_filepath('test'); 
 pg_relation_filepath 
----------------------
 base/14793/16750
(1 row)

edb=# vacuum full test;
VACUUM
edb=# select pg_relation_filepath('test');
 pg_relation_filepath 
----------------------
 base/14793/16758
(1 row)
```


## Index rebuilding

### `REINDEX`

* bloating 된 index 준비
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

  create index test_idx1 on test(id);
  create index test_idx2 on test(val3);

  insert into test
  select
      generate_series(1, 100000) as id,
      gen_random_bytes(30) as val1,
      gen_random_bytes(30) as val2,
      varchar 'test' as val3;

  update test set id = id + 100;
  update test set id = id + 100;
  update test set id = id + 100;
  ```

* Size 확인
  ```
  edb=# \di+ test_idx1                 
                                 List of relations
      Schema    |   Name    | Type  |    Owner     | Table | Size  | Description 
  --------------+-----------+-------+--------------+-------+-------+-------------
   enterprisedb | test_idx1 | index | enterprisedb | test  | 11 MB | 
  (1 row)

  edb=# \di+ test_idx2
                                 List of relations
      Schema    |   Name    | Type  |    Owner     | Table | Size  | Description 
  --------------+-----------+-------+--------------+-------+-------+-------------
   enterprisedb | test_idx2 | index | enterprisedb | test  | 10 MB | 
  (1 row)
  ```

* `reindex` 명령으로 index를 rebuild 하고 size를 다시 확인 한다.
  ```
  edb=# \h reindex        
  Command:     REINDEX
  Description: rebuild indexes
  Syntax:
  REINDEX [ ( { VERBOSE } [, ...] ) ] { INDEX | TABLE | SCHEMA | DATABASE | SYSTEM } name

  edb=# reindex table test;
  REINDEX
  edb=# \di+ test_idx1     
                                  List of relations
      Schema    |   Name    | Type  |    Owner     | Table |  Size   | Description 
  --------------+-----------+-------+--------------+-------+---------+-------------
   enterprisedb | test_idx1 | index | enterprisedb | test  | 2208 kB | 
  (1 row)

  edb=# \di+ test_idx2     
                                  List of relations
      Schema    |   Name    | Type  |    Owner     | Table |  Size   | Description 
  --------------+-----------+-------+--------------+-------+---------+-------------
   enterprisedb | test_idx2 | index | enterprisedb | test  | 2208 kB | 
  (1 row)
  ```

### `CREATE INDEX CONCURRENTLY`

* bloating 된 index 준비
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

  create index test_idx1 on test(id);
  create index test_idx2 on test(val3);

  insert into test
  select
    generate_series(1, 100000) as id,
    gen_random_bytes(30) as val1,
    gen_random_bytes(30) as val2,
    varchar 'test' as val3;

  update test set id = id + 100;
  update test set id = id + 100;
  update test set id = id + 100;
  ```

* Size 확인
  ```
  edb=# \di+ test_idx1                 
                                 List of relations
      Schema    |   Name    | Type  |    Owner     | Table | Size  | Description 
  --------------+-----------+-------+--------------+-------+-------+-------------
   enterprisedb | test_idx1 | index | enterprisedb | test  | 11 MB | 
  (1 row)

  edb=# \di+ test_idx2
                                 List of relations
      Schema    |   Name    | Type  |    Owner     | Table | Size  | Description 
  --------------+-----------+-------+--------------+-------+-------+-------------
   enterprisedb | test_idx2 | index | enterprisedb | test  | 10 MB | 
  (1 row)
  ```

* index 재생성
  ```
  edb=# create index concurrently test_idx1_t on test(id);
  CREATE INDEX
  edb=# create index concurrently test_idx2_t on test(val3);
  CREATE INDEX
  edb=# drop index test_idx1;
  DROP INDEX
  edb=# drop index test_idx2;
  DROP INDEX
  edb=# alter index test_idx1_t rename to test_idx1;
  ALTER INDEX
  edb=# alter index test_idx2_t rename to test_idx2;
  ALTER INDEX
  ```

* Size 확인
  ```
  edb=# \di+ test_idx1  
                                  List of relations
      Schema    |   Name    | Type  |    Owner     | Table |  Size   | Description 
  --------------+-----------+-------+--------------+-------+---------+-------------
   enterprisedb | test_idx1 | index | enterprisedb | test  | 2208 kB | 
  (1 row)

  edb=# \di+ test_idx2  
                                  List of relations
      Schema    |   Name    | Type  |    Owner     | Table |  Size   | Description 
  --------------+-----------+-------+--------------+-------+---------+-------------
   enterprisedb | test_idx2 | index | enterprisedb | test  | 2208 kB | 
  (1 row)
  ```
