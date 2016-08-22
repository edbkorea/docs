# Monitoring

## pgbench

pgbench를 이용해 간단히 부하를 발생 시켜 보자.

* 테스트를 위해 별도의 DB를 생성한다.
  ```
  edb=# create database pgbench;
  CREATE DATABASE
  ```

* Scale factor를 100으로 하여 데이터 생성 (약 1.5G)
  ```
  $ pgbench -i -s 100 pgbench
  ```

* 4개의 Client로 60초간 select 부하 발생
  ```
  $ pgbench -c 4 -S -s 100 -T 60 pgbench
  ```

* DB 재생성
  ```
  edb=# drop database pgbench;
  DROP DATABASE
  edb=# create database pgbench;
  CREATE DATABASE
  ```

## Activity 모니터링

* 세션별 동작 상태 모니터링
	아래 쿼리를 통해 세션별 동작 상태를 모니터링 할 수 있다. 반복적으로 조회를 하면서 결과의 변화를 관찰해 보자.

	```
  edb=# \pset pager off                 
  Pager usage is off.
  edb=# select * from pg_stat_activity ;
   datid | datname | pid  | usesysid |   usename    | application_name | client_addr | client_hostname | client_port |          backend_start           |            xact_start            |           query_start            |           state_change           | waiting | state  | backend_xid | backend_xmin |                                                                                           query                                                                                           
  -------+---------+------+----------+--------------+------------------+-------------+-----------------+-------------+----------------------------------+----------------------------------+----------------------------------+----------------------------------+---------+--------+-------------+--------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   14793 | edb     | 2336 |       10 | enterprisedb |                  | ::1         |                 |       59465 | 11-FEB-16 23:28:44.829241 +09:00 |                                  | 12-FEB-16 00:19:32.112875 +09:00 | 12-FEB-16 00:19:32.11305 +09:00  | f       | idle   |             |              | SELECT J.jobid   FROM pgagent.pga_job J  WHERE jobenabled    AND jobagentid IS NULL    AND jobnextrun <= now()    AND (jobhostagent = '' OR jobhostagent = 'ppaslab') ORDER BY jobnextrun
   17500 | pgbench | 4210 |       10 | enterprisedb | pgbench          |             |                 |          -1 | 12-FEB-16 00:16:51.774201 +09:00 | 12-FEB-16 00:19:35.137392 +09:00 | 12-FEB-16 00:19:35.137392 +09:00 | 12-FEB-16 00:19:35.137394 +09:00 | f       | active |             |         2189 | SELECT abalance FROM pgbench_accounts WHERE aid = 5966891;
   17500 | pgbench | 4211 |       10 | enterprisedb | pgbench          |             |                 |          -1 | 12-FEB-16 00:16:51.775958 +09:00 | 12-FEB-16 00:19:34.987814 +09:00 | 12-FEB-16 00:19:34.987814 +09:00 | 12-FEB-16 00:19:34.987815 +09:00 | f       | active |             |         2189 | SELECT abalance FROM pgbench_accounts WHERE aid = 5444289;
   17500 | pgbench | 4212 |       10 | enterprisedb | pgbench          |             |                 |          -1 | 12-FEB-16 00:16:51.77776 +09:00  | 12-FEB-16 00:19:35.037468 +09:00 | 12-FEB-16 00:19:35.037468 +09:00 | 12-FEB-16 00:19:35.037469 +09:00 | f       | active |             |         2189 | SELECT abalance FROM pgbench_accounts WHERE aid = 2949561;
   17500 | pgbench | 4213 |       10 | enterprisedb | pgbench          |             |                 |          -1 | 12-FEB-16 00:16:51.779282 +09:00 | 12-FEB-16 00:19:35.087307 +09:00 | 12-FEB-16 00:19:35.087307 +09:00 | 12-FEB-16 00:19:35.087308 +09:00 | f       | active |             |         2189 | SELECT abalance FROM pgbench_accounts WHERE aid = 5991378;
   14793 | edb     | 4323 |       10 | enterprisedb | psql.bin         |             |                 |          -1 | 12-FEB-16 00:19:21.368266 +09:00 | 12-FEB-16 00:19:34.533137 +09:00 | 12-FEB-16 00:19:34.533137 +09:00 | 12-FEB-16 00:19:34.533139 +09:00 | f       | active |             |         2189 | select * from pg_stat_activity ;
  (6 rows)

  edb=# select * from pg_stat_activity ;
   datid | datname | pid  | usesysid |   usename    | application_name | client_addr | client_hostname | client_port |          backend_start           |            xact_start            |           query_start            |           state_change           | waiting | state  | backend_xid | backend_xmin |                                                                                           query                                                                                           
  -------+---------+------+----------+--------------+------------------+-------------+-----------------+-------------+----------------------------------+----------------------------------+----------------------------------+----------------------------------+---------+--------+-------------+--------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
   14793 | edb     | 2336 |       10 | enterprisedb |                  | ::1         |                 |       59465 | 11-FEB-16 23:28:44.829241 +09:00 |                                  | 12-FEB-16 00:19:32.112875 +09:00 | 12-FEB-16 00:19:32.11305 +09:00  | f       | idle   |             |              | SELECT J.jobid   FROM pgagent.pga_job J  WHERE jobenabled    AND jobagentid IS NULL    AND jobnextrun <= now()    AND (jobhostagent = '' OR jobhostagent = 'ppaslab') ORDER BY jobnextrun
   17500 | pgbench | 4210 |       10 | enterprisedb | pgbench          |             |                 |          -1 | 12-FEB-16 00:16:51.774201 +09:00 | 12-FEB-16 00:19:36.989329 +09:00 | 12-FEB-16 00:19:36.989329 +09:00 | 12-FEB-16 00:19:36.98933 +09:00  | f       | active |             |         2189 | SELECT abalance FROM pgbench_accounts WHERE aid = 6229996;
   17500 | pgbench | 4211 |       10 | enterprisedb | pgbench          |             |                 |          -1 | 12-FEB-16 00:16:51.775958 +09:00 | 12-FEB-16 00:19:37.049279 +09:00 | 12-FEB-16 00:19:37.049279 +09:00 | 12-FEB-16 00:19:37.04928 +09:00  | f       | active |             |         2189 | SELECT abalance FROM pgbench_accounts WHERE aid = 6668313;
   17500 | pgbench | 4212 |       10 | enterprisedb | pgbench          |             |                 |          -1 | 12-FEB-16 00:16:51.77776 +09:00  | 12-FEB-16 00:19:36.929169 +09:00 | 12-FEB-16 00:19:36.929169 +09:00 | 12-FEB-16 00:19:36.92917 +09:00  | f       | active |             |         2189 | SELECT abalance FROM pgbench_accounts WHERE aid = 3473759;
   17500 | pgbench | 4213 |       10 | enterprisedb | pgbench          |             |                 |          -1 | 12-FEB-16 00:16:51.779282 +09:00 | 12-FEB-16 00:19:37.097961 +09:00 | 12-FEB-16 00:19:37.097961 +09:00 | 12-FEB-16 00:19:37.097962 +09:00 | f       | active |             |         2189 | SELECT abalance FROM pgbench_accounts WHERE aid = 1307075;
   14793 | edb     | 4323 |       10 | enterprisedb | psql.bin         |             |                 |          -1 | 12-FEB-16 00:19:21.368266 +09:00 | 12-FEB-16 00:19:37.098168 +09:00 | 12-FEB-16 00:19:37.098168 +09:00 | 12-FEB-16 00:19:37.09819 +09:00  | f       | active |             |         2189 | select * from pg_stat_activity ;
  (6 rows)
  ```

* OS 모니터링

	`ps` 와 `top`을 통해 프로세스들의 동작 상태를 모니터링 해 보자.

  ```
  [root@ppaslab ~]# ps waux | grep post
  root      2065  0.0  0.0  80868  1576 ?        Ss   Feb11   0:00 /usr/libexec/postfix/master
  postfix   2071  0.0  0.0  80948  1504 ?        S    Feb11   0:00 pickup -l -t fifo -u
  postfix   2072  0.0  0.0  81116  1516 ?        S    Feb11   0:00 qmgr -l -t fifo -u
  501       2196  0.0  1.8 726852 34820 ?        S    Feb11   0:00 /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data
  501       2197  0.0  0.0 208880  1604 ?        Ss   Feb11   0:00 postgres: logger process                                                
  501       2199  0.0  0.3 727596  7540 ?        Ss   Feb11   0:00 postgres: checkpointer process                                          
  501       2200  0.0  0.6 726852 11840 ?        Ss   Feb11   0:00 postgres: writer process                                                
  501       2201  0.0  0.8 726852 16908 ?        Ss   Feb11   0:00 postgres: wal writer process                                            
  501       2202  0.0  0.1 727284  2784 ?        Ss   Feb11   0:00 postgres: autovacuum launcher process                                   
  501       2203  0.0  0.0 211116  1872 ?        Ss   Feb11   0:01 postgres: stats collector process                                       
  501       2336  0.0  0.5 731532  9720 ?        Ss   Feb11   0:00 postgres: enterprisedb edb ::1[59465] idle                              
  501       4210  9.7 25.7 727876 495276 ?       Ds   00:16   0:49 postgres: enterprisedb pgbench [local] SELECT                           
  501       4211  9.9 25.7 727876 495268 ?       Ds   00:16   0:50 postgres: enterprisedb pgbench [local] SELECT                           
  501       4212 10.0 25.7 727876 495276 ?       Ds   00:16   0:50 postgres: enterprisedb pgbench [local] SELECT                           
  501       4213  9.9 25.7 727876 495280 ?       Ds   00:16   0:50 postgres: enterprisedb pgbench [local] SELECT                           
  501       4323  0.0  0.3 728156  6900 ?        Ss   00:19   0:00 postgres: enterprisedb edb [local] idle                                 
  root      4640  0.0  0.0 107472   944 pts/2    S+   00:25   0:00 grep post
  ```

	```
  # watch 'ps waux | grep post'
  ```

* PEM & pgAdmin III
	pgBench를 실행한 다음 PEM client의 `메뉴 > Tools > Server Status`를 실행하여 각종 항목들을 비교해 본다.

## DRITA

### 설정

`postgresql.conf` 파일의 아래 설정을 변경한 뒤 DB를 재기동 하여야 한다.
```
timed_statistics = on
```

```
[enterprisedb@ppaslab ~]$ pg_ctl restart
waiting for server to shut down.... done
server stopped
server starting
```

### Snapshot 관리

* Snapshot 생성

  ```
  edb=# select * from edbsnap();
         edbsnap        
  ----------------------
   Statement processed.
  (1 row)
  ```

* Snapshot 목록

  ```
  edb=# select * from get_snaps();
            get_snaps          
  -----------------------------
   1 12-FEB-16 00:30:53.830153
  (1 row)
  ```

* Snapshot 삭제

  삭제하고자 하는 snapshot의 범위를 지정한다.
  ```
  edb=# select * from purgesnap(1, 1);
               purgesnap              
  ------------------------------------
   Snapshots in range 1 to 1 deleted.
  (1 row)
  ```

### Reporting

* Snapshot 생성

  ```
  edb=# select * from edbsnap();      
         edbsnap        
  ----------------------
   Statement processed.
  (1 row)
  ```

* pgbench 부하 발생

  ```
  $ pgbench -c 4 -S -s 100 -T 60 pgbench
  ```

* Snapshot 생성

  ```
  edb=# select * from edbsnap();      
         edbsnap        
  ----------------------
   Statement processed.
  (1 row)
  ```

* 리포트 생성

  ```
  edb=# select * from get_snaps();    
            get_snaps          
  -----------------------------
   2 12-FEB-16 00:34:41.695159
   3 12-FEB-16 00:36:10.255261
  (2 rows)

  edb=# \pset pager off
  Pager usage is off.
  edb=# select * from edbreport(2, 3);
                                                                         edbreport                                                                       
  -------------------------------------------------------------------------------------------------------------------------------------------------------
      EnterpriseDB Report for database edb        12-FEB-16
   Version: EnterpriseDB 9.5.0.5 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.1.2 20080704 (Red Hat 4.1.2-55), 64-bit

        Begin snapshot: 2 at 12-FEB-16 00:34:41.695159
        End snapshot:   3 at 12-FEB-16 00:36:10.255261

   Size of database edb is 24 MB
        Tablespace: pg_default Size: 1561 MB Owner: enterprisedb
        Tablespace: pg_global Size: 608 kB Owner: enterprisedb

        ...
        ...
  ```

## 기타 DB 모니터링 쿼리

* `relfronzenxid` 모니터링

  ```
  edb=# select datname, age(datfrozenxid) from pg_database;
    datname  | age
  -----------+-----
   template1 |  24
   template0 |  24
   postgres  |  24
   edb       |  24
  (4 rows)

  edb=# select max(age(relfrozenxid)) from pg_class where relkind in ('r', 't');
   max
  -----
    24
  (1 row)

  edb=# vacuum freeze;    
  VACUUM
  edb=# select datname, age(datfrozenxid) from pg_database;
    datname  | age
  -----------+-----
   template1 |  25
   template0 |  25
   postgres  |  25
   edb       |   1
  (4 rows)
  ```

* 설정값 확인

	```sql
  show all;
  또는 select * from pg_settings;
  ```

* Long running 쿼리 확인

	```sql
  select pid,waiting,current_timestamp - least(query_start,xact_start) as runtime, query, 
         extract(epoch from ((timeofday()::timestamp)-query_start)) as secs
    from pg_stat_activity
   where pid <> pg_backend_pid() and state <> 'idle'
   order by secs desc ;
  ```

* 사용 가능한 connection 수

  ```sql
  select (select to_number(setting,'999999') from pg_settings where name = 'max_connections')
         - (select count(*) from pg_stat_activity); 
  ```

* 현재 Session의 backend pid 확인

  ```sql
  select pg_backend_pid(); 
  ```

* Blocking and blocked activity

  ```sql
  SELECT blocked_locks.pid     AS blocked_pid,
           blocked_activity.usename  AS blocked_user,
           blocking_locks.pid     AS blocking_pid,
           blocking_activity.usename AS blocking_user,
           blocked_activity.query    AS blocked_statement,
           blocking_activity.query   AS blocking_statement
     FROM  pg_catalog.pg_locks         blocked_locks
      JOIN pg_catalog.pg_stat_activity blocked_activity  ON blocked_activity.pid = blocked_locks.pid
      JOIN pg_catalog.pg_locks         blocking_locks 
          ON blocking_locks.locktype = blocked_locks.locktype
          AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
          AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
          AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
          AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
          AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
          AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
          AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
          AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
          AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
          AND blocking_locks.pid != blocked_locks.pid
       JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
     WHERE NOT blocked_locks.GRANTED;
  ```

* Table별 cache hit ratio 정보

  ```sql
  SELECT relname, (cast(heap_blks_hit as numeric) / ( heap_blks_hit + heap_blks_read)) * 100 AS hit_pct,
         heap_blks_hit as "heap from cache", heap_blks_read as "heap from disc" 
    FROM pg_statio_user_tables
   WHERE relname = 'customers’;
  ```

* Index별 cache hit ratio 정보

  ```sql
  SELECT indexrelname,cast(idx_blks_hit as numeric) / (idx_blks_hit + idx_blks_read) * 100 AS hit_pct,
         idx_blks_hit, idx_blks_read
    FROM pg_statio_user_indexes
   WHERE relname = 'customers’;
  ```

* Bloated table 확인

  ```sql
  SELECT current_database(), schemaname, tblname, bs*tblpages AS real_size,
    (tblpages-est_tblpages)*bs AS extra_size,
    CASE WHEN tblpages - est_tblpages > 0
      THEN 100 * (tblpages - est_tblpages)/tblpages::float
      ELSE 0
    END AS extra_ratio, fillfactor, (tblpages-est_tblpages_ff)*bs AS bloat_size,
    CASE WHEN tblpages - est_tblpages_ff > 0
      THEN 100 * (tblpages - est_tblpages_ff)/tblpages::float
      ELSE 0
    END AS bloat_ratio, is_na
    -- , (pst).free_percent + (pst).dead_tuple_percent AS real_frag
  FROM (
    SELECT ceil( reltuples / ( (bs-page_hdr)/tpl_size ) ) + ceil( toasttuples / 4 ) AS est_tblpages,
      ceil( reltuples / ( (bs-page_hdr)*fillfactor/(tpl_size*100) ) ) + ceil( toasttuples / 4 ) AS est_tblpages_ff,
      tblpages, fillfactor, bs, tblid, schemaname, tblname, heappages, toastpages, is_na
      -- , stattuple.pgstattuple(tblid) AS pst
    FROM (
      SELECT
        ( 4 + tpl_hdr_size + tpl_data_size + (2*ma)
          - CASE WHEN tpl_hdr_size%ma = 0 THEN ma ELSE tpl_hdr_size%ma END
          - CASE WHEN ceil(tpl_data_size)::int%ma = 0 THEN ma ELSE ceil(tpl_data_size)::int%ma END
        ) AS tpl_size, bs - page_hdr AS size_per_block, (heappages + toastpages) AS tblpages, heappages,
        toastpages, reltuples, toasttuples, bs, page_hdr, tblid, schemaname, tblname, fillfactor, is_na
      FROM (
        SELECT
          tbl.oid AS tblid, ns.nspname AS schemaname, tbl.relname AS tblname, tbl.reltuples,
          tbl.relpages AS heappages, coalesce(toast.relpages, 0) AS toastpages,
          coalesce(toast.reltuples, 0) AS toasttuples,
          coalesce(substring(
            array_to_string(tbl.reloptions, ' ')
            FROM '%fillfactor=#"__#"%' FOR '#')::smallint, 100) AS fillfactor,
          current_setting('block_size')::numeric AS bs,
          CASE WHEN version()~'mingw32' OR version()~'64-bit|x86_64|ppc64|ia64|amd64' THEN 8 ELSE 4 END AS ma,
          24 AS page_hdr,
          23 + CASE WHEN MAX(coalesce(null_frac,0)) > 0 THEN ( 7 + count(*) ) / 8 ELSE 0::int END
            + CASE WHEN tbl.relhasoids THEN 4 ELSE 0 END AS tpl_hdr_size,
          sum( (1-coalesce(s.null_frac, 0)) * coalesce(s.avg_width, 1024) ) AS tpl_data_size,
          bool_or(att.atttypid = 'pg_catalog.name'::regtype) AS is_na
        FROM pg_attribute AS att
          JOIN pg_class AS tbl ON att.attrelid = tbl.oid
          JOIN pg_namespace AS ns ON ns.oid = tbl.relnamespace
          JOIN pg_stats AS s ON s.schemaname=ns.nspname
            AND s.tablename = tbl.relname AND s.inherited=false AND s.attname=att.attname
          LEFT JOIN pg_class AS toast ON tbl.reltoastrelid = toast.oid
        WHERE att.attnum > 0 AND NOT att.attisdropped
          AND tbl.relkind = 'r'
        GROUP BY 1,2,3,4,5,6,7,8,9,10, tbl.relhasoids
        ORDER BY 2,3
      ) AS s
    ) AS s2
  ) AS s3
  ```

* 가장 큰 relation 찾기

  ```sql
  SELECT nspname || '.' || relname AS "relation",
      pg_size_pretty(pg_relation_size(C.oid)) AS "size"
    FROM pg_class C
    LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
    WHERE nspname NOT IN ('pg_catalog', 'information_schema')
    ORDER BY pg_relation_size(C.oid) DESC
    LIMIT 20;
  ```

## Dictionary

### Dictionary 조회

* Search path 상의 스키마 목록을 조회해 보자.
    ```
    edb=# show search_path;
       search_path   
    -----------------
     "$user", public
    (1 row)

    edb=# select current_schemas(true);
                 current_schemas              
    ------------------------------------------
     {sys,dbo,pg_catalog,enterprisedb,public}
    (1 row)

    edb=# select current_schemas(false);
        current_schemas    
    -----------------------
     {enterprisedb,public}
    (1 row)
    ```

* `public` 스키마의 View 목록을 조회 해 보자.
    ```
    edb=# select * from pg_views where schemaname='public';
     schemaname | viewname |  viewowner   |                  definition
    ------------+----------+--------------+-----------------------------------------------
     public     | salesemp | enterprisedb |  SELECT emp.empno,                           +
                |          |              |     emp.ename,                               +
                |          |              |     emp.hiredate,                            +
                |          |              |     emp.sal,                                 +
                |          |              |     emp.comm                                 +
                |          |              |    FROM emp                                  +
                |          |              |   WHERE ((emp.job)::text = 'SALESMAN'::text);
    (1 row)
    ```

* `pg_catalog` 스키마의 모든 dictionary view들의 목록을 확인 해 보자.

    ```
    edb=# \d pg_catalog.<TAB>
    Display all 258 possibilities? (y or n) y
    ...
    ...
    ```

### Session 관리

* 임의의 세션을 하나 실행 시킨다.
    ```
    [enterprisedb@ppaslab ~]$ psql
    psql.bin (9.5.0.5)
    Type "help" for help.

    edb=# 
    ```

* 다른 세션을 하나 더 열고 idle 상태의 세션 목록을 조회해 보자.
    ```
    edb=# select pid, usename, now() - state_change as "Idle Time", state from pg_stat_activity where state = 'idle';
      pid  |   usename    |    Idle Time    | state 
    -------+--------------+-----------------+-------
     12814 | enterprisedb | 00:02:36.437632 | idle
      9820 | enterprisedb | 00:00:01.998137 | idle
    (2 rows)
    ```

* Idle Time을 기준으로 앞서 생성한 세션의 pid을 이용해 ps 명령으로 확인한다.
    ```
    [enterprisedb@ppaslab ~]$ ps -aef |grep 12814
    501      12814  4856  0 16:39 ?        00:00:00 postgres: enterprisedb edb [local] idle
    501      12966  7177  0 16:42 pts/5    00:00:00 grep 12814
    ```

* 해당 session의 backend를 종료 시킨다.
    ```
    edb=# select pg_terminate_backend(12814);
     pg_terminate_backend 
    ----------------------
     t
    (1 row)

    edb=# select pid, usename, now() - state_change as "Idle Time", state from pg_stat_activity where state = 'idle';
     pid  |   usename    |    Idle Time    | state 
    ------+--------------+-----------------+-------
     9820 | enterprisedb | 00:00:05.483133 | idle
    (1 row)
    ```

### Oracle 호환 뷰

* all_tables 뷰의 정보를 확인해 보자.
    ```
    edb=# select * from all_tables; 
        owner     | schema_name  |        table_name         | tablespace_name | status | temporary 
    --------------+--------------+---------------------------+-----------------+--------+-----------
     ENTERPRISEDB | PUBLIC       | DEPT                      |                 | VALID  | N
     ENTERPRISEDB | PUBLIC       | EMP                       |                 | VALID  | N
     ENTERPRISEDB | PUBLIC       | JOBHIST                   |                 | VALID  | N
     ENTERPRISEDB | ENTERPRISEDB | TEST                      |                 | VALID  | N
     ENTERPRISEDB | PGAGENT      | PGA_JOBCLASS              |                 | VALID  | N
     ENTERPRISEDB | PGAGENT      | PGA_JOBAGENT              |                 | VALID  | N
     ENTERPRISEDB | PGAGENT      | PGA_SCHEDULE              |                 | VALID  | N
     ENTERPRISEDB | PGAGENT      | PGA_JOBLOG                |                 | VALID  | N
     ENTERPRISEDB | PGAGENT      | PGA_JOB                   |                 | VALID  | N
     ENTERPRISEDB | PGAGENT      | PGA_EXCEPTION             |                 | VALID  | N
     ENTERPRISEDB | PGAGENT      | PGA_JOBSTEP               |                 | VALID  | N
     ENTERPRISEDB | PGAGENT      | PGA_JOBSTEPLOG            |                 | VALID  | N
     ENTERPRISEDB | SYS          | PRODUCT_COMPONENT_VERSION |                 | VALID  | N
     ENTERPRISEDB | SYS          | DUAL                      |                 | VALID  | N
     ENTERPRISEDB | SYS          | PLSQL_PROFILER_RUNS       |                 | VALID  | N
     ENTERPRISEDB | SYS          | PLSQL_PROFILER_UNITS      |                 | VALID  | N
     ENTERPRISEDB | SYS          | PLSQL_PROFILER_RAWDATA    |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$SYSTEM_WAITS          |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$SESSION_WAITS         |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$SNAP                  |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$STATIO_ALL_INDEXES    |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$SESSION_WAIT_HISTORY  |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$STAT_ALL_INDEXES      |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$STAT_ALL_TABLES       |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$STAT_DATABASE         |                 | VALID  | N
     ENTERPRISEDB | SYS          | EDB$STATIO_ALL_TABLES     |                 | VALID  | N
    (26 rows)
    ```

* `all_indexes`와 `all_ind_columns`이 제공하는 정보를 확인해 보자.
    ```
    edb=# \d all_indexes           
               View "sys.all_indexes"
         Column      |     Type     | Modifiers 
    -----------------+--------------+-----------
     owner           | text         | 
     schema_name     | text         | 
     index_name      | text         | 
     index_type      | text         | 
     table_owner     | text         | 
     table_name      | text         | 
     table_type      | text         | 
     uniqueness      | text         | 
     compression     | character(1) | 
     tablespace_name | text         | 
     logging         | text         | 
     status          | text         | 
     partitioned     | character(3) | 
     temporary       | character(1) | 
     secondary       | character(1) | 
     join_index      | character(3) | 
     dropped         | character(3) | 

    edb=# \d all_ind_columns
             View "sys.all_ind_columns"
         Column      |     Type     | Modifiers 
    -----------------+--------------+-----------
     index_owner     | text         | 
     schema_name     | text         | 
     index_name      | text         | 
     table_owner     | text         | 
     table_name      | text         | 
     column_name     | text         | 
     column_position | smallint     | 
     column_length   | smallint     | 
     char_length     | numeric      | 
     descend         | character(1) | 
    ```

* `sys` 스키마에서 제공하는 뷰의 목록을 확인해 보자.

    ```
    edb=# \d sys.<TAB>
    Display all 112 possibilities? (y or n)
    sys.all_all_tables                        sys.dba_sequences                         sys.scheduler_0250_program_argument_type
    sys.all_cons_columns                      sys.dba_source                            sys.scheduler_0300_schedule_type
    sys.all_constraints                       sys.dba_subpart_key_columns               sys.scheduler_0400_job_type
    sys.all_db_links                          sys.dba_synonyms                          sys.scheduler_0450_job_argument_type
    sys.all_ind_columns                       sys.dba_tab_columns                       sys.session_waits_hist_pk
    sys.all_indexes                           sys.dba_tables                            sys.session_waits_pk
    sys.all_jobs                              sys.dba_tab_partitions                    sys.snap_pk
    sys.all_objects                           sys.dba_tab_subpartitions                 sys.snapshot_num_seq
    sys.all_part_key_columns                  sys.dba_triggers                          sys.system_waits_pk
    sys.all_part_tables                       sys.dba_types                             sys.user_all_tables
    sys.all_policies                          sys.dba_users                             sys.user_cons_columns
    sys.all_sequences                         sys.dba_view_columns                      sys.user_constraints
    sys.all_source                            sys.dba_views                             sys.user_db_links
    sys.all_subpart_key_columns               sys.dual                                  sys.user_ind_columns
    sys.all_synonyms                          sys."edb$session_wait_history"            sys.user_indexes
    sys.all_tab_columns                       sys."edb$session_waits"                   sys.user_jobs
    sys.all_tables                            sys."edb$snap"                            sys.user_objects
    sys.all_tab_partitions                    sys."edb$stat_all_indexes"                sys.user_part_key_columns
    sys.all_tab_subpartitions                 sys."edb$stat_all_tables"                 sys.user_part_tables
    sys.all_triggers                          sys."edb$stat_database"                   sys.user_policies
    sys.all_types                             sys."edb$stat_db_pk"                      sys.user_role_privs
    sys.all_users                             sys."edb$stat_idx_pk"                     sys.user_sequences
    sys.all_view_columns                      sys."edb$statio_all_indexes"              sys.user_source
    sys.all_views                             sys."edb$statio_all_tables"               sys.user_subpart_key_columns
    sys.dba_all_tables                        sys."edb$statio_idx_pk"                   sys.user_synonyms
    sys.dba_cons_columns                      sys."edb$statio_tab_pk"                   sys.user_tab_columns
    sys.dba_constraints                       sys."edb$stat_tab_pk"                     sys.user_tables
    sys.dba_db_links                          sys."edb$system_waits"                    sys.user_tab_partitions
    sys.dba_ind_columns                       sys.lineno_text                           sys.user_tab_subpartitions
    sys.dba_indexes                           sys.plsql_profiler_data                   sys.user_triggers
    sys.dba_jobs                              sys.plsql_profiler_rawdata                sys.user_types
    sys.dba_objects                           sys.plsql_profiler_runid                  sys.user_users
    sys.dba_part_key_columns                  sys.plsql_profiler_runs                   sys.user_view_columns
    sys.dba_part_tables                       sys.plsql_profiler_runs_pkey              sys.user_views
    sys.dba_policies                          sys.plsql_profiler_units                  sys._utl_file_dir
    sys.dba_profiles                          sys.product_component_version             sys."v$version"
    sys.dba_role_privs                        sys.scheduler_0100_component_name_type    
    sys.dba_roles                             sys.scheduler_0200_program_type           

    ```