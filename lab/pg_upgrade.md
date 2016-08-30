# EDB PAS Version Upgrade Guide

### Agenda

* [Objective](#objective)
* [`pg_upgrade` Overview](#pg_upgrade-overview)
* [Upgrade Procedure](#upgrade-procedure)
* [Data Copy Method](#data-copy-method)
* [Hard Link Method](#hard-link-method)
* [Appendix](#appendix)

## Objective

EDB PAS Upgrade Guide를 통해 EDB PAS의 메이저 버전 업그레이드 방법에 대해 알아보고 현재 운영중인 환경에 적합한 방법을 통한 버전 업그레이트를 할 수 있도록 함

## `pg_upgrade` Overview

EDB PAS의 버전 업그레이드는 `pg_upgrade` 유틸리티를 통해 이루어짐

> `pg_dump`, `pg_restore` 를 이용해서도 업그레이드를 할 수 있지만, 기존 디비의 2배 이상의 추가 용량이 필요하며 성능적으로도 좋지 않아 이 문서에서는 다루지 않음

pg_upgrade를 이용한 업그레이드 방법으로는 2가지가 지원되며,
첫번째는 이전 버전의 데이터를 읽어서 새로운 버전의 클러스터로 옮기는 `data copy` 방법
두번째는 이전 버전의 데이터의 물리적 정보를 복제하는 방식의 `hard link` 방법

1. Data Copy

  * **장점**
    * 실패해도 얼마든지 다시 시도 가능
  * **단점**
    * 기존 데이터 크기 만큼의 추가 용량 필요
    * 데이터 크기에 비례해 마이그레이션 시간 소요

2. Hard Link

  * **장점**
    * 추가 디스크 용량 불필요
    * 빠른 마이그레이션
  * **단점**
    * 실패하면 기존 데이터도 사용 불가 (백업 반드시 필요)

* pg_upgrade 사용 방법

  ```
  옵션:
    -b, --old-bindir=BINDIR       old cluster executable directory
    -B, --new-bindir=BINDIR       new cluster executable directory
    -c, --check                   check clusters only, don't change any data
    -d, --old-datadir=DATADIR     old cluster data directory
    -D, --new-datadir=DATADIR     new cluster data directory
    -j, --jobs                    number of simultaneous processes or threads to use
    -k, --link                    link instead of copying files to new cluster
    -o, --old-options=OPTIONS     old cluster options to pass to the server
    -O, --new-options=OPTIONS     new cluster options to pass to the server
    -p, --old-port=PORT           old cluster port number (default 50432)
    -P, --new-port=PORT           new cluster port number (default 50432)
    -r, --retain                  retain SQL and log files after success
    -U, --username=NAME           cluster superuser (default "enterprisedb")
    -v, --verbose                 enable verbose internal logging
    -V, --version                 display version information, then exit
    -?, --help                    show this help, then exit
  ```
  
  ```
  사용 예시:
    pg_upgrade -d oldCluster/data -D newCluster/data -b oldCluster/bin -B newCluster/bin -p oldClusterPort -N newClusterPort
  or
    $ export PGDATAOLD=oldCluster/data
    $ export PGDATANEW=newCluster/data
    $ export PGBINOLD=oldCluster/bin
    $ export PGBINNEW=newCluster/bin
    $ export PGPORTOLD=5444
    $ export PGPORTNEW=5445
    $ pg_upgrade
  ```


## Upgrade Procedure

pg_upgrade를 통한 업그레이드는 다음의 과정을 따름

1. New 버전 PAS 설치

  업그레이드할 새 버전의 PAS 소프트웨어 설치

2. initdb

  새버전의 `initdb`로 빈 클러스터 생성

3. 클러스터 인증

  업그레이드 과정 동안 Old/New 클러스터에 접속을 해야하기 때문에 인증없이 접속할 수 있도록 수정
  방법으로 pg_hba.conf 파일을 수정하거나 .pgpass 파일을 수정하는 방법이 있음

  * `pg_hba.conf` 수정

    로컬 접속 부분을 **trust** 로 변경

    ```
    # "local" is for Unix domain socket connections only
    local   all             all                                     trust
    ```

  * `~/.pgpass` 설정

    .pgpass 파일에 슈퍼유저 계정 정보 입력을 통해 인증 절차 해결

    ```
    -bash-4.1$ cat .pgpass
    localhost:5444:edb:enterprisedb:password
     ```

4. Old/New 클러스터 종료

  Old/New 클러스터를 모두 종료시킴

  ```
  pg_ctl -D /opt/PostgresPlus/9.4AS/data stop -mf
  pg_ctl -D /opt/PostgresPlus/9.5AS/data stop -mf
  ```

5.  pg_upgrade 실행

  pg_upgrade 실행을 위해 필수 항목과 옵션 항목이 있음

  * 필수 항목

    - -d : Old 클러스터 데이터 경로
    - -D : New 클러스터 데이터 경로
    - -b : Old 클러스터 바이너리 (bin 디렉토리) 경로
    - -B : New 클러스터 바이너리 (bin 디렉토리) 경로
    - -p : Old 클러스터 포트 번호
    - -P : New 클러스터 포트 번호

  * `-c` or `--check` option

    업그레이드 실행 이전에 현재 상태를 파악하여 문제점 여부 (디렉토리 권한, 클러스터 프로세스 종료, 테이블스페이스 디렉토리 등)를 사전 검증해주는 옵션

  * `-j` or `'--jobs` option

    업그레이드 시 지정한 숫자만큼의 쓰레드가 병렬로 작업을 하기 때문에 업그레이드 시간 단축에 이점이 있음

  * `-k` or `--link` option

    링크 옵션을 사용하면 데이터를 옮기는 것이 아니라 기존 데이터가 가지고 있는 동일한 inode를 바라보게하여 기존의 파일을 그대로 사용하도록 하는 옵션

    데이터 복제의 시간이 없기 때문에 용량이 클수록 시간 단축에 효과가 있음

    * **주의사항!**

      > 링크방식으로 업그레이드가 진행되면 Old 클러스터와 New 클러스터가 동일한 inode를 바라게되는데, New 클러스터가 실행이 되면 Old 클러스터는 사용할 수 없게됨.
      > 또한 링크방식으로 업그레이드가 진행이 되면 Old 클러스터의 `global` 디렉토리의 `pg_control` 파일이 `pg_control.old`로 변경이 되어 클러스터를 실행할 수 없음

  * 사용 예시

    ```
    pg_upgrade -d oldCluster/data -D newCluster/data -b oldCluster/bin -B newCluster/bin -p oldClusterPort -N newClusterPort
    ```

    ```
    $ export PGDATAOLD=oldCluster/data
    $ export PGDATANEW=newCluster/data
    $ export PGBINOLD=oldCluster/bin
    $ export PGBINNEW=newCluster/bin
    $ export PGPORTOLD=5444
    $ export PGPORTNEW=5445
    $ pg_upgrade
    ```

6. 인증 파일 원복

  업그레이드 완료 후 New 클러스터의 pg_hba.conf 또는 .pgpass 파일을 기존의 상태로 돌려놓기

7. New 클러스터 시작

  업그레이드 이후 후속 작업을 위해 New 클러스터를 시작

8. 통계 생성

  pg_upgrade로 Old 클러스터의 옵티마이저 통계 정보가 갱신되지 않기 때문에 pg_upgrade가 업그레이드를 마치면서 가이드해준 클러스터 통계 정보 갱신 스크립트를 실행하여 통계 생성

  ```
  Optimizer statistics are not transferred by pg_upgrade so,
  once you start the new server, consider running:
      ./analyze_new_cluster.sh
  ```

9. 이전 클러스터 삭제 (Optional)

  업그레이드가 성공적으로 완료되었다면 Old 클러스터의 데이터 및 바이너리가 불필요하다 판단할 수 있음
  이 경우 pg_upgrade가 가이드해준 스크립트를 이용하여 이전 버전 디렉토리들을 삭제할 수 있음

  ```
  Running this script will delete the old cluster's data files:
      ./delete_old_cluster.sh
  ```

## Data Copy Method

Data copy 방법을 이용하여 PAS 9.4.5.12 버전의 클러스터를 PAS 9.5.4.9 로 업그레이드하는 데모
[Upgrade Procedure](#upgrade-procedure) 챕터에서 설명한 업그레이드 과정에 의한 업그레이드 진행

* **Old Cluster**
  * bin : /opt/PostgresPlus/9.4AS/bin
  * data : /opt/PostgresPlus/9.4AS/data
  * port : 5432
  * version : 9.4.5.12
  * tablespace : /pg_tblspc
* **New Cluster**
  * bin : /opt/PostgresPlus/9.5AS/bin
  * data : /opt/PostgresPlus/9.5AS/data
  * port : 5444
  * version : 9.5.4.9

> 본 데모에서는 9.5 설치 과정은 생략함

1. 사전 단계

  PAS 9.4.5.12 버전에 테이블스페이스 (tbs) 와 해당 테이블스페이스를 사용하는 테이블을 생성

  ```sql
  -bash-4.1$
  -bash-4.1$ psql
  Timing is on.
  psql.bin (9.4.5.12)
  Type "help" for help.

  edb=# \dt
              List of relations
   Schema |  Name   | Type  |    Owner
  --------+---------+-------+--------------
   public | dept    | table | enterprisedb
   public | emp     | table | enterprisedb
   public | jobhist | table | enterprisedb
  (3 rows)

  edb=#
  edb=# create tablespace tbs location '/pg_tblspc';
  CREATE TABLESPACE
  edb=#
  edb=# create table emp2 tablespace tbs
  edb-# as select * from emp;
  SELECT 14
  edb=#
  edb=# \dt
                 List of relations
      Schema    |  Name   | Type  |    Owner
  --------------+---------+-------+--------------
   enterprisedb | emp2    | table | enterprisedb
   public       | dept    | table | enterprisedb
   public       | emp     | table | enterprisedb
   public       | jobhist | table | enterprisedb
  (4 rows)

  edb=# \d emp2
               Table "enterprisedb.emp2"
    Column  |            Type             | Modifiers
  ----------+-----------------------------+-----------
   empno    | numeric(4,0)                |
   ename    | character varying(10)       |
   job      | character varying(9)        |
   mgr      | numeric(4,0)                |
   hiredate | timestamp without time zone |
   sal      | numeric(7,2)                |
   comm     | numeric(7,2)                |
   deptno   | numeric(2,0)                |
  Tablespace: "tbs"

  edb=#
  edb=# show port;
   port
  ------
   5432
  (1 row)

  edb=#
  ```

2. `initdb`

  New 버전에 initdb로 유저 오브젝트가 전혀 만들어지지 않은 clean 상태의 클러스터 생성

  * **주의사항!**
    * 클러스터 생성 시 기존 환경에 맞는 characterset과 locale 설정이 필요
    * initdb로 클러스터를 생성하면 포트번호가 `5444` 이고 `pg_hba.conf`의 모든 인증 방식은 `trust`로 설정되어 있음
    * Post upgrade 단계에서 이 부분의 적절한 재설정이 필요

  ```
  -bash-4.1$ cd /opt/PostgresPlus/9.5AS
  -bash-4.1$
  -bash-4.1$ . pgplus_env.sh
  -bash-4.1$
  -bash-4.1$ initdb -D /opt/PostgresPlus/9.5AS/data
  The files belonging to this database system will be owned by user "enterprisedb".
  This user must also own the server process.
  
  The database cluster will be initialized with locale "en_US.UTF-8".
  The default database encoding has accordingly been set to "UTF8".
  The default text search configuration will be set to "english".
  
  Data page checksums are disabled.
  
  creating directory /opt/PostgresPlus/9.5AS/data ... ok
  creating subdirectories ... ok
  selecting default max_connections ... 100
  selecting default shared_buffers ... 128MB
  selecting dynamic shared memory implementation ... posix
  creating configuration files ... ok
  creating template1 database in /opt/PostgresPlus/9.5AS/data/base/1 ... ok
  initializing pg_authid ... ok
  initializing dependencies ... ok
  loading EDB-SPL server-side language ... ok
  creating system views ... ok
  loading system objects' descriptions ... ok
  creating collations ... ok
  creating conversions ... ok
  creating dictionaries ... ok
  setting privileges on built-in objects ... ok
  creating information schema ... ok
  loading PL/pgSQL server-side language ... ok
  creating edb sys ... ok
  creating Redwood-style casts ... ok
  loading edb contrib modules ...
  dbms_alert_public.sql ok
  dbms_alert.plb ok
  dbms_job_public.sql ok
  dbms_job.plb ok
  dbms_lob_public.sql ok
  dbms_lob.plb ok
  dbms_output_public.sql ok
  dbms_output.plb ok
  dbms_pipe_public.sql ok
  dbms_pipe.plb ok
  dbms_sql_public.sql ok
  dbms_sql.plb ok
  dbms_utility_public.sql ok
  dbms_utility.plb ok
  dbms_profiler_public.sql ok
  dbms_profiler.plb ok
  dbms_random_public.sql ok
  dbms_random.plb ok
  dbms_lock_public.sql ok
  dbms_lock.plb ok
  dbms_scheduler_public.sql ok
  dbms_scheduler.plb ok
  dbms_crypto_public.sql ok
  dbms_crypto.plb ok
  dbms_mview_public.sql ok
  dbms_mview.plb ok
  dbms_session_public.sql ok
  dbms_session.plb ok
  edb_bulkload.sql ok
  edb_gen.sql ok
  object_source.sql ok
  utl_encode_public.sql ok
  utl_encode.plb ok
  utl_http_public.sql ok
  utl_http.plb ok
  utl_file.plb ok
  utl_tcp_public.sql ok
  utl_tcp.plb ok
  utl_smtp_public.sql ok
  utl_smtp.plb ok
  utl_mail_public.sql ok
  utl_mail.plb ok
  
  utl_url_public.sql ok
  utl_url.plb ok
  utl_raw_public.sql ok
  utl_raw.plb ok
  commoncriteria.sql ok
  waitstates.sql ok
  installing extension edb_dblink_libpq ... ok
  installing extension edb_dblink_oci ... ok
  installing extension pldbgapi ... ok
  snap_tables.sql ok
  snap_functions.sql ok
  dblink_ora.sql ok
  edb_infinitecache.sql ok
  sys_stats.sql ok
  vacuuming database template1 ... ok
  copying template1 to template0 ... ok
  copying template1 to postgres ... ok
  syncing data to disk ... ok
  
  WARNING: enabling "trust" authentication for local connections
  You can change this by editing pg_hba.conf or using the option -A, or
  --auth-local and --auth-host, the next time you run initdb.
  
  Success. You can now start the database server using:
  
      pg_ctl -D /opt/PostgresPlus/9.5AS/data -l logfile start
  
  -bash-4.1$
  ```

3. 클러스터 인증 설정

  Old/New 클러스터의 `local` 접속에 대한 인증없이 접속 가능하도록 설정

  > New 클러스터의 경우 initdb로 만들면 `pg_hba.conf`의 local 인증 방식이 기본적으로 `trust`로 되어 있기 때문에 별도의 설정이 필요 없음
  >
  > 본 데모에서는 Old 클러스터의 pg_hba.conf의 local 설정을 trust로 변경해줌

  ```
  -bash-4.1$ vi /opt/PostgresPlus/9.4AS/data/pg_hba.conf
  -bash-4.1$
  # "local" is for Unix domain socket connections only
  local   all             all                                     trust
  ```

4. Old/New 클러스터 종료

  업그레이드를 위해 실행중인 클러스터 프로세스를 종료함

  ```
  -bash-4.1$ ps -efH | grep postgres
  501      22381 17461  0 19:18 pts/0    00:00:00             grep postgres
  501      22145     1  0 19:11 pts/0    00:00:00   /opt/PostgresPlus/9.4AS/bin/edb-postgres -D /opt/PostgresPlus/9.4AS/data
  501      22146 22145  0 19:11 ?        00:00:00     postgres: logger process
  501      22148 22145  0 19:11 ?        00:00:00     postgres: checkpointer process
  501      22149 22145  0 19:11 ?        00:00:00     postgres: writer process
  501      22150 22145  0 19:11 ?        00:00:00     postgres: wal writer process
  501      22151 22145  0 19:11 ?        00:00:00     postgres: autovacuum launcher process
  501      22152 22145  0 19:11 ?        00:00:00     postgres: stats collector process
  -bash-4.1$
  -bash-4.1$
  -bash-4.1$ pg_ctl -D /opt/PostgresPlus/9.4AS/data stop -mf
  waiting for server to shut down.... done
  server stopped
  -bash-4.1$
  ```

5. pg_upgrade

  Data copy 방식을 통한 pg_upgrade를 실행
  환경 변수에 클러스터 실행파일 경로, 데이터 경로, 포트번호를 지정하고 실행하거나 또는 pg_upgrade에 모든 옵션을 붙여 사용 가능

  * 환경 변수 세팅

    ```
    -bash-4.1$ export PGDATAOLD=/opt/PostgresPlus/9.4AS/data
    -bash-4.1$ export PGDATANEW=/opt/PostgresPlus/9.5AS/data
    -bash-4.1$ export PGBINOLD=/opt/PostgresPlus/9.4AS/bin
    -bash-4.1$ export PGBINNEW=/opt/PostgresPlus/9.5AS/bin
    -bash-4.1$ export PGPORTOLD=5432
    -bash-4.1$ export PGPORTNEW=5444
    ```

  * 업그레이드에 앞서 `--check` 옵션으로 업그레이트 준비 상태 점검

    ```
    -bash-4.1$ pg_upgrade --check
    Performing Consistency Checks
    -----------------------------
    Checking cluster versions                                   ok
    Checking database user is the install user                  ok
    Checking database connection settings                       ok
    Checking for prepared transactions                          ok
    Checking for reg* system OID user data types                ok
    Checking for contrib/isn with bigint-passing mismatch       ok
    Checking for presence of required libraries                 ok
    Checking database user is the install user                  ok
    Checking for prepared transactions                          ok

    *Clusters are compatible*
    -bash-4.1$
    ```

  * 실제 업그레이드 실행

    ```
    -bash-4.1$
    -bash-4.1$ pg_upgrade
    Performing Consistency Checks
    -----------------------------
    Checking cluster versions                                   ok
    Checking database user is the install user                  ok
    Checking database connection settings                       ok
    Checking for prepared transactions                          ok
    Checking for reg* system OID user data types                ok
    Checking for contrib/isn with bigint-passing mismatch       ok
    Creating dump of global objects                             ok
    Creating dump of database schemas
                                                                ok
    Checking for presence of required libraries                 ok
    Checking database user is the install user                  ok
    Checking for prepared transactions                          ok

    If pg_upgrade fails after this point, you must re-initdb the
    new cluster before continuing.

    Performing Upgrade
    ------------------
    Analyzing all rows in the new cluster                       ok
    Freezing all rows on the new cluster                        ok
    Deleting files from new pg_clog                             ok
    Copying old pg_clog to new server                           ok
    Setting next transaction ID and epoch for new cluster       ok
    Deleting files from new pg_multixact/offsets                ok
    Copying old pg_multixact/offsets to new server              ok
    Deleting files from new pg_multixact/members                ok
    Copying old pg_multixact/members to new server              ok
    Setting next multixact ID and offset for new cluster        ok
    Resetting WAL archives                                      ok
    Setting frozenxid and minmxid counters in new cluster       ok
    Restoring global objects in the new cluster                 ok
    Restoring database schemas in the new cluster
                                                                ok
    Copying user relation files
                                                                ok
    Setting next OID for new cluster                            ok
    Sync data directory to disk                                 ok
    Creating script to analyze new cluster                      ok
    Creating script to delete old cluster                       ok

    Upgrade Complete
    ----------------
    Optimizer statistics are not transferred by pg_upgrade so,
    once you start the new server, consider running:
        ./analyze_new_cluster.sh

    Running this script will delete the old cluster's data files:
        ./delete_old_cluster.sh
    -bash-4.1$
    ```

6. 인증 설정

  New 클러스터의 인증 설정을 기존의 설정대로 원복시킴

7. New 클러스터 시작 & 업그레이드 확인

  업그레이드가 정상적으로 이루어졌는지 확인하고 post upgrade 단계를 실행하기 위해 New 클러스터를 시작

  * 클러스터 시작

    ```
    -bash-4.1$ ps -efH | grep postgres
    501      22601 17461  0 19:21 pts/0    00:00:00             grep postgres
    -bash-4.1$
    -bash-4.1$
    -bash-4.1$ pg_ctl -D /opt/PostgresPlus/9.5AS/data start -w
    waiting for server to start....2016-08-19 19:21:21 KST LOG:

    ** EnterpriseDB Dynamic Tuning Agent ********************************************
    *       System Utilization: 66 %                                                *
    *         Database Version: 9.5.4.9                                             *
    *            Database Size: 0.1    GB                                           *
    *                      RAM: 1.9    GB                                           *
    *            Shared Memory: 1877   MB                                           *
    *       Max DB Connections: 112                                                 *
    *               Autovacuum: on                                                  *
    *       Autovacuum Naptime: 60   Seconds                                        *
    *********************************************************************************

    2016-08-19 19:21:21 KST LOG:  database system was shut down at 2016-08-19 19:20:02 KST
    2016-08-19 19:21:21 KST LOG:  MultiXact member wraparound protections are now enabled
    2016-08-19 19:21:21 KST LOG:  database system is ready to accept connections
    2016-08-19 19:21:21 KST LOG:  autovacuum launcher started
     done
    server started
    -bash-4.1$
    ```

  * 업그레이드 상태 확인

    ```
    -bash-4.1$ psql
    Timing is on.
    psql.bin (9.5.4.9)
    Type "help" for help.
    
    edb=# \dt
                   List of relations
        Schema    |  Name   | Type  |    Owner
    --------------+---------+-------+--------------
     enterprisedb | emp2    | table | enterprisedb
     public       | dept    | table | enterprisedb
     public       | emp     | table | enterprisedb
     public       | jobhist | table | enterprisedb
    (4 rows)
    
    edb=#
    edb=# \db
              List of tablespaces
        Name    |    Owner     |  Location
    ------------+--------------+------------
     pg_default | enterprisedb |
     pg_global  | enterprisedb |
     tbs        | enterprisedb | /pg_tblspc
    (3 rows)
    
    edb=#
    edb=# \d emp2
                 Table "enterprisedb.emp2"
      Column  |            Type             | Modifiers
    ----------+-----------------------------+-----------
     empno    | numeric(4,0)                |
     ename    | character varying(10)       |
     job      | character varying(9)        |
     mgr      | numeric(4,0)                |
     hiredate | timestamp without time zone |
     sal      | numeric(7,2)                |
     comm     | numeric(7,2)                |
     deptno   | numeric(2,0)                |
    Tablespace: "tbs"
    
    edb=#
    ```

  * 테이블스페이스 디렉토리에 9.5 버전의 디렉토리가 생겨있는 것을 확인 가능

    ```
    -bash-4.1$ cd /pg_tblspc/
    -bash-4.1$
    -bash-4.1$ ls
    PG_9.4_201409291  PG_9.5_201510051
    -bash-4.1$
    ```

8. 옵티마이저 통계 생성

  pg_upgrade가 업그레이드 후 자동 생성해준 `analyze_new_cluster.sh` 스크립트를 실행하여 New 클러스터의 옵티마이저 통계 정보를 생성함

  ```
  -bash-4.1$
  -bash-4.1$ ./analyze_new_cluster.sh
  This script will generate minimal optimizer statistics rapidly
  so your system is usable, and then gather statistics twice more
  with increasing accuracy.  When it is done, your system will
  have the default level of optimizer statistics.

  If you have used ALTER TABLE to modify the statistics target for
  any tables, you might want to remove them and restore them after
  running this script because they will delay fast statistics generation.

  If you would like default statistics as quickly as possible, cancel
  this script and run:
      "/opt/PostgresPlus/9.5AS/bin/vacuumdb" --all --analyze-only

  vacuumdb: processing database "edb": Generating minimal optimizer statistics (1 target)
  vacuumdb: processing database "postgres": Generating minimal optimizer statistics (1 target)
  vacuumdb: processing database "template1": Generating minimal optimizer statistics (1 target)
  vacuumdb: processing database "edb": Generating medium optimizer statistics (10 targets)
  vacuumdb: processing database "postgres": Generating medium optimizer statistics (10 targets)
  vacuumdb: processing database "template1": Generating medium optimizer statistics (10 targets)
  vacuumdb: processing database "edb": Generating default (full) optimizer statistics
  vacuumdb: processing database "postgres": Generating default (full) optimizer statistics
  vacuumdb: processing database "template1": Generating default (full) optimizer statistics

  Done
  -bash-4.1$
  ```

9. Old 클러스터 디렉토리 삭제 (Optional)

  * 버전 클러스터 삭제

    pg_upgrade가 업그레이드 후 자동 생성해준 `delete_old_cluster.sh` 스크립트를 실행하여 이전 버전 클러스터 디렉토리 삭제

    ```
    -bash-4.1$ delete_old_cluster.sh
    ```

## Hard Link Method

Hard link 방법을 이용하여 PAS 9.4.5.12 버전의 클러스터를 PAS 9.5.4.9 로 업그레이드하는 데모

1. 사전 단계

  PAS 9.4.5.12 버전에 테이블스페이스 (tbs) 와 해당 테이블스페이스를 사용하는 테이블을 생성

  > 유저 생성 오브젝트에 대해 하드 링크를 생성하여 업그레이드하는 방식이기 때문에 유저 오브젝트의 파일 경로와 하드링크 카운트를 주의 깊게 살펴봐야함

  ```
  -bash-4.1$
  -bash-4.1$ psql
  Timing is on.
  psql.bin (9.4.5.12)
  Type "help" for help.
  
  edb=# select pg_relation_filepath('emp2');
               pg_relation_filepath
  ----------------------------------------------
   pg_tblspc/16384/PG_9.4_201409291/14570/16445
  (1 row)
  
  edb=#
  edb=# \! ls -al /opt/PostgresPlus/9.4AS/data/pg_tblspc/16384/PG_9.4_201409291/14570/16445
  -rw-------. 1 enterprisedb enterprisedb 8192 Aug 19 21:17 /opt/PostgresPlus/9.4AS/data/pg_tblspc/16384/PG_9.4_201409291/14570/16445
  edb=#
  edb=# select pg_relation_filepath('dept');
   pg_relation_filepath
  ----------------------
   base/14570/16385
  (1 row)
  
  edb=#
  edb=# \! ls -al /opt/PostgresPlus/9.4AS/data/base/14570/16385
  -rw-------. 1 enterprisedb enterprisedb 8192 Aug 19 21:17 /opt/PostgresPlus/9.4AS/data/base/14570/16385
  edb=#
  ```

  > emp2와 dept 테이블의 파일 경로를 확인하였고, 현재 하드링크 카운트는 `1` 인 것을 확인할 수 있음

2. New 클러스터 initdb, Old 클러스터 인증 설정, Old 클러스터 종료는 data copy 와 동일하기 때문에 생략

3. `pg_upgrade`

  Data copy의 모든 옵션에 추가적으로 `-k` 옵션을 추가하여 hard link 방식으로 업그레이드를 할 수 있도록 함

  * 환경 변수 세팅

    ```
    -bash-4.1$ export PGDATAOLD=/opt/PostgresPlus/9.4AS/data
    -bash-4.1$ export PGDATANEW=/opt/PostgresPlus/9.5AS/data
    -bash-4.1$ export PGBINOLD=/opt/PostgresPlus/9.4AS/bin
    -bash-4.1$ export PGBINNEW=/opt/PostgresPlus/9.5AS/bin
    -bash-4.1$ export PGPORTOLD=5432
    -bash-4.1$ export PGPORTNEW=5444
    ```

  * 업그레이드에 앞서 `--check` 옵션으로 업그레이트 준비 상태 점검

    ```
    -bash-4.1$ pg_upgrade -c -k
    Performing Consistency Checks
    -----------------------------
    Checking cluster versions                                   ok
    Checking database user is the install user                  ok
    Checking database connection settings                       ok
    Checking for prepared transactions                          ok
    Checking for reg* system OID user data types                ok
    Checking for contrib/isn with bigint-passing mismatch       ok
    Checking for presence of required libraries                 ok
    Checking database user is the install user                  ok
    Checking for prepared transactions                          ok

    *Clusters are compatible*
    -bash-4.1$
    ```

  * 실제 업그레이드 실행

    하드 링크 방식의 경우 `-k` 옵션이 추가됨

    ```
    -bash-4.1$ pg_upgrade -k
    Performing Consistency Checks
    -----------------------------
    Checking cluster versions                                   ok
    Checking database user is the install user                  ok
    Checking database connection settings                       ok
    Checking for prepared transactions                          ok
    Checking for reg* system OID user data types                ok
    Checking for contrib/isn with bigint-passing mismatch       ok
    Creating dump of global objects                             ok
    Creating dump of database schemas
                                                                ok
    Checking for presence of required libraries                 ok
    Checking database user is the install user                  ok
    Checking for prepared transactions                          ok

    If pg_upgrade fails after this point, you must re-initdb the
    new cluster before continuing.

    Performing Upgrade
    ------------------
    Analyzing all rows in the new cluster                       ok
    Freezing all rows on the new cluster                        ok
    Deleting files from new pg_clog                             ok
    Copying old pg_clog to new server                           ok
    Setting next transaction ID and epoch for new cluster       ok
    Deleting files from new pg_multixact/offsets                ok
    Copying old pg_multixact/offsets to new server              ok
    Deleting files from new pg_multixact/members                ok
    Copying old pg_multixact/members to new server              ok
    Setting next multixact ID and offset for new cluster        ok
    Resetting WAL archives                                      ok
    Setting frozenxid and minmxid counters in new cluster       ok
    Restoring global objects in the new cluster                 ok
    Restoring database schemas in the new cluster
                                                                ok
    Adding ".old" suffix to old global/pg_control               ok

    If you want to start the old cluster, you will need to remove
    the ".old" suffix from /opt/PostgresPlus/9.4AS/data/global/pg_control.old.
    Because "link" mode was used, the old cluster cannot be safely
    started once the new cluster has been started.

    Linking user relation files
                                                                ok
    Setting next OID for new cluster                            ok
    Sync data directory to disk                                 ok
    Creating script to analyze new cluster                      ok
    Creating script to delete old cluster                       ok

    Upgrade Complete
    ----------------
    Optimizer statistics are not transferred by pg_upgrade so,
    once you start the new server, consider running:
        ./analyze_new_cluster.sh

    Running this script will delete the old cluster's data files:
        ./delete_old_cluster.sh
    -bash-4.1$
    ```

4. New 클러스터 인증 설정, New 클러스터 시작은 data copy와 동일하기 때문에 생략

5. 업그레이드 상태 확인

  ```
  -bash-4.1$ psql
  Timing is on.
  psql.bin (9.5.4.9)
  Type "help" for help.

  edb=#
  edb=# select pg_relation_filepath('emp2');
               pg_relation_filepath
  ----------------------------------------------
   pg_tblspc/16420/PG_9.5_201510051/14793/16445
  (1 row)

  Time: 1.742 ms
  edb=#
  edb=# \! ls -al /opt/PostgresPlus/9.5AS/data/pg_tblspc/16420/PG_9.5_201510051/14793/16445
  -rw-------. 2 enterprisedb enterprisedb 8192 Aug 19 21:17 /opt/PostgresPlus/9.5AS/data/pg_tblspc/16420/PG_9.5_201510051/14793/16445
  edb=#
  edb=#
  edb=# select pg_relation_filepath('dept');
   pg_relation_filepath
  ----------------------
   base/14793/16385
  (1 row)

  Time: 0.366 ms
  edb=#
  edb=# \! ls -al /opt/PostgresPlus/9.5AS/data/base/14793/16385
  -rw-------. 2 enterprisedb enterprisedb 8192 Aug 19 21:20 /opt/PostgresPlus/9.5AS/data/base/14793/16385
  edb=#
  edb=#
  edb=#
  ```

  > 9.4 버전의 오브젝트가 정상적으로 확인되며, 해당 파일의 하드링크 숫자가 업그레이드 이전에 `1` 이었던 것이 업그레이드 이후 `2`로 변경되어 있는 것을 확인 가능

6. 통계 생성 및 이전 버전 디렉토리 삭제는 data copy와 동일하기 때문에 생략


## Appendix

### 업그레이드 속도 비교

| Migration Method           |      Minutes |
| -------------------------- | -----------: |
| dump / restore             |        300.0 |
| dump with parallel restore |        180.0 |
| pg_upgrade in copy mode    |         44.0 |
| pg_upgrade in link mode    | 0.7 (44 sec) |

* Database Size : 150GB, 850 tables
* 하드웨어 스펙 : N/A

> 참고자료 : https://momjian.us/main/writings/pgsql/pg_upgrade.pdf (page 17)
