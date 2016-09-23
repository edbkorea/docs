## pg_stat_statements
`pg_stat_statements` Postgres 성능 관련 extension으로, 데이터베이스에서 실행된 SQL 구문의 실행 통계 정보를 트랙킹함

### 1. Installation
pg_stat_statements extension은 이미 라이브러리가 lib 경로에 저장되어 있어 별도의 컴파일 등의 작업이 불필요.
따라서 database 레벨에서 해당 extension을 생성하고 postgresql.conf의 파라미터 수정으로 설치가 가능

1. Configuration Parameters
- pg_stat_statements.max (integer) : 트래킹할 SQL 구문 갯수 지정. 기본값은 5000이며 변경 시 서버 재시작 필요
- pg_stat_statements.track (enum) : 트래킹 레벨 지정. top은 클라이언트가 실행한 구문만, all은 네스티드 구문까지 트래킹, none은 트래킹 비활성. 기본값은 top
- pg_stat_statements.track_utility (boolean) : INSERT, UPDATE, DELETE, SELECT를 제외한 구문의 트래킹 여부 지정. 기본값은 on
- pg_stat_statements.save (boolean) : 서버 종료 시 트래킹 정보를 계속 저장할지 여부 지정. 기본값은 on

2. `shared_preload_libraries` in `postgresql.conf`
    이 extension은 별도의 shared memory를 사용하기 때문에 DB 기동시에 미리 로딩이 되어야함
`postgresql.conf`의 `shared_preload_libraries`에 아래와 같이 `pg_stat_statements`를 추가

    ```
shared_preload_libraries = '$libdir/dbms_pipe,$libdir/edb_gen,$libdir/pg_stat_statements'
pg_stat_statements.max = 10000
pg_stat_statements.track = all
    ```

3. DB 재기동
    ```
    [enterprisedb@ppaslab ~]$ pg_ctl restart
    waiting for server to shut down.... done
    server stopped
    server starting
    ```

4. pg_stat_statements extension 설치
    ```sql
    edb=# create extension pg_stat_statements;
    CREATE EXTENSION
    edb=# \dx pg_stat_statements
                                          List of installed extensions
            Name        | Version |    Schema    |                        Description                        
    --------------------+---------+--------------+-----------------------------------------------------------
     pg_stat_statements | 1.3     | enterprisedb | track execution statistics of all SQL statements executed
    (1 row)
    ```

4. 설치 확인
    ```sql
    edb=# \dx+ pg_stat_statements
    Objects in extension "pg_stat_statements"
            Object Description          
    --------------------------------------
    function pg_stat_statements(boolean)
    function pg_stat_statements_reset()
    view pg_stat_statements
    (3 rows)
    ```

### 3. 테스트
1. 통계정보 reset
  ```
  edb=# select pg_stat_statements_reset();
   pg_stat_statements_reset
  --------------------------

  (1 row)
  ```

2. 쿼리 실행

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

3. 정보 확인
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


#### Appendix. pg_stat_statements View
|	Name	|	Type	|	Description	|
|-|-|-|
|	userid	|	oid	|	OID of user who executed the statement	|
|	dbid	|	oid	|	OID of database in which the statement was executed	|
|	queryid	|	bigint	|	Internal hash code, computed from the statement's parse tree	|
|	query	|	text	|	Text of a representative statement	|
|	calls	|	bigint	|	Number of times executed	|
|	total_time	|	double precision	|	Total time spent in the statement, in milliseconds	|
|	min_time	|	double precision	|	Minimum time spent in the statement, in milliseconds	|
|	max_time	|	double precision	|	Maximum time spent in the statement, in milliseconds	|
|	mean_time	|	double precision	|	Mean time spent in the statement, in milliseconds	|
|	stddev_time	|	double precision	|	Population standard deviation of time spent in the statement, in milliseconds	|
|	rows	|	bigint	|	Total number of rows retrieved or affected by the statement	|
|	shared_blks_hit	|	bigint	|	Total number of shared block cache hits by the statement	|
|	shared_blks_read	|	bigint	|	Total number of shared blocks read by the statement	|
|	shared_blks_dirtied	|	bigint	|	Total number of shared blocks dirtied by the statement	|
|	shared_blks_written	|	bigint	|	Total number of shared blocks written by the statement	|
|	local_blks_hit	|	bigint	|	Total number of local block cache hits by the statement	|
|	local_blks_read	|	bigint	|	Total number of local blocks read by the statement	|
|	local_blks_dirtied	|	bigint	|	Total number of local blocks dirtied by the statement	|
|	local_blks_written	|	bigint	|	Total number of local blocks written by the statement	|
|	temp_blks_read	|	bigint	|	Total number of temp blocks read by the statement	|
|	temp_blks_written	|	bigint	|	Total number of temp blocks written by the statement	|
|	blk_read_time	|	double precision	|	Total time the statement spent reading blocks, in milliseconds (if track_io_timing is enabled, otherwise zero)	|
|	blk_write_time	|	double precision	|	Total time the statement spent writing blocks, in milliseconds (if track_io_timing is enabled, otherwise zero)	|



