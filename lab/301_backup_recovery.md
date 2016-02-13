# Lab: Backup and Recovery

## Logical Backup

### `pg_dump` plain text format

* `edb` 전체를 dump 받아 보자.
	```
  [enterprisedb@ppaslab ~]$ mkdir /opt/PostgresPlus/backup
  [enterprisedb@ppaslab ~]$ pg_dump -Fp -f /opt/PostgresPlus/backup/edb.sql -U enterprisedb -d edb
  ```

* 내용을 확인해 보자.

  ```sql
  [enterprisedb@ppaslab ~]$ more /opt/PostgresPlus/backup/edb.sql
  --
  -- EnterpriseDB database dump
  --

  -- Dumped from database version 9.5.0.5
  -- Dumped by pg_dump version 9.5.0.5

  SET statement_timeout = 0;
  SET lock_timeout = 0;
  SET client_encoding = 'UTF8';
  SET standard_conforming_strings = on;
  SET edb_redwood_date = off;
  SET default_with_rowids = off;
  SET check_function_bodies = false;
  SET client_min_messages = warning;
  SET row_security = off;
  ...
  ...
  ```

### `pg_dump` custom format

* Custom format
  ```
  [enterprisedb@ppaslab ~]$ pg_dump -Fc -Z 3 -f /opt/PostgresPlus/backup/edb.dump -U enterprisedb -d edb
  ```
	* `-Fc`: Custom format을 이용
	* `-Z 3`: Level 3으로 압축 하라
	* `-f xxx`: Output file 지정

* 파일의 내용을 확인해 보자
  ```
  [enterprisedb@ppaslab ~]$ pg_restore -Fc -l -v /opt/PostgresPlus/backup/edb.dump
  ;
  ; Archive created at 2016-02-12 23:44:35 KST
  ;     dbname: edb
  ;     TOC Entries: 80
  ;     Compression: 3
  ;     Dump Version: 1.12-0
  ;     Format: CUSTOM
  ;     Integer: 4 bytes
  ;     Offset: 8 bytes
  ;     Dumped from database version: 9.5.0.5
  ;     Dumped by pg_dump version: 9.5.0.5
  ;
  ;
  ; Selected TOC Entries:
  ;
  4644; 0 0 ENCODING - ENCODING 
  4645; 0 0 STDSTRINGS - STDSTRINGS 
  4646; 1262 14793 DATABASE - edb enterprisedb
  ...
  4416; 2606 16397 FK CONSTRAINT public emp_ref_dept_fk enterprisedb
  4418; 2606 16413 FK CONSTRAINT public jobhist_ref_dept_fk enterprisedb
  4417; 2606 16408 FK CONSTRAINT public jobhist_ref_emp_fk enterprisedb
  ```
	* `-l`: 복구하지 않고 파일의 내용을 요약하여 출력

* `dept`, `emp` 테이블을 `pgbench` DB에 복구해 보자.

  ```
  [enterprisedb@ppaslab ~]$ pg_restore -Fc -1 -d pgbench -t emp -t dept -v /opt/PostgresPlus/backup/edb.dump
  pg_restore: connecting to database for restore
  Password: 
  pg_restore: creating TABLE "public.dept"
  pg_restore: creating TABLE "public.emp"
  pg_restore: processing data for table "public.dept"
  pg_restore: processing data for table "public.emp"
  pg_restore: setting owner and privileges for TABLE "public.dept"
  pg_restore: setting owner and privileges for TABLE "public.emp"
  pg_restore: setting owner and privileges for TABLE DATA "public.dept"
  pg_restore: setting owner and privileges for TABLE DATA "public.emp"
  ```
	* `-Fc`: Custom format을 이용
	* `-1`: Restore 전체를 1개의 transaction으로 처리
	* `-d pgbench`: `pgbench` DB에 복구
	* `-t emp -t dept`: `emp`, `dept` 두개의 테이블을 복구하라
	* `-v`: verbose mode

  ```
  edb=# \c pgbench
  You are now connected to database "pgbench" as user "enterprisedb".
  pgbench=# \d
                  List of relations
   Schema |       Name       | Type  |    Owner     
  --------+------------------+-------+--------------
   public | dept             | table | enterprisedb
   public | emp              | table | enterprisedb
   public | pgbench_accounts | table | enterprisedb
   public | pgbench_branches | table | enterprisedb
   public | pgbench_history  | table | enterprisedb
   public | pgbench_tellers  | table | enterprisedb
   public | test             | table | copyuser
  (7 rows)
  ```

* 새로운 DB를 생성하고 빈 Schema만 복구해 보자.

  ```
  edb=# create database testdb;
  CREATE DATABASE
  ```

  ```
  [enterprisedb@ppaslab ~]$ pg_restore -Fc -1 -d testdb -s -v /opt/PostgresPlus/backup/edb.dump
  pg_restore: connecting to database for restore
  Password: 
  pg_restore: creating SCHEMA "pgagent"
  pg_restore: creating COMMENT "SCHEMA pgagent"
  pg_restore: creating EXTENSION "pgagent"
  ...
  pg_restore: setting owner and privileges for TRIGGER "public.user_audit_trig"
  pg_restore: setting owner and privileges for FK CONSTRAINT "public.emp_ref_dept_fk"
  pg_restore: setting owner and privileges for FK CONSTRAINT "public.jobhist_ref_dept_fk"
  pg_restore: setting owner and privileges for FK CONSTRAINT "public.jobhist_ref_emp_fk"
  ```
	* `-s`: Schema only mode

  ```
  edb=# \c testdb
  You are now connected to database "testdb" as user "enterprisedb".
  testdb=# \d+
                                     List of relations
      Schema    |        Name        |   Type   |    Owner     |    Size    | Description 
  --------------+--------------------+----------+--------------+------------+-------------
   enterprisedb | pg_buffercache     | view     | enterprisedb | 0 bytes    | 
   enterprisedb | pg_stat_statements | view     | enterprisedb | 0 bytes    | 
   public       | dept               | table    | enterprisedb | 0 bytes    | 
   public       | emp                | table    | enterprisedb | 0 bytes    | 
   public       | jobhist            | table    | enterprisedb | 0 bytes    | 
   public       | next_empno         | sequence | enterprisedb | 8192 bytes | 
   public       | salesemp           | view     | enterprisedb | 0 bytes    | 
  (7 rows)
  ```

* 방금 생성한 테이블들에 데이터를 넣어보자.

  ```
  edb=# \c testdb
  You are now connected to database "testdb" as user "enterprisedb".
  testdb=# \d+
                                     List of relations
      Schema    |        Name        |   Type   |    Owner     |    Size    | Description 
  --------------+--------------------+----------+--------------+------------+-------------
   enterprisedb | pg_buffercache     | view     | enterprisedb | 0 bytes    | 
   enterprisedb | pg_stat_statements | view     | enterprisedb | 0 bytes    | 
   public       | dept               | table    | enterprisedb | 0 bytes    | 
   public       | emp                | table    | enterprisedb | 0 bytes    | 
   public       | jobhist            | table    | enterprisedb | 0 bytes    | 
   public       | next_empno         | sequence | enterprisedb | 8192 bytes | 
   public       | salesemp           | view     | enterprisedb | 0 bytes    | 
  (7 rows)
  ```

  ```
  [enterprisedb@ppaslab ~]$ pg_restore -Fc -1 -d testdb -a -v /opt/PostgresPlus/backup/edb.dump
  pg_restore: connecting to database for restore
  Password: 
  pg_restore: processing data for table "pgagent.pga_jobagent"
  pg_restore: processing data for table "pgagent.pga_jobclass"
  pg_restore: processing data for table "pgagent.pga_job"
  pg_restore: processing data for table "pgagent.pga_schedule"
  pg_restore: processing data for table "pgagent.pga_exception"
  pg_restore: processing data for table "pgagent.pga_joblog"
  pg_restore: processing data for table "pgagent.pga_jobstep"
  pg_restore: processing data for table "pgagent.pga_jobsteplog"
  pg_restore: processing data for table "public.dept"
  pg_restore: processing data for table "public.emp"
  pg_restore: Inserting employee 7369
  pg_restore: ..New salary: 800.00
  pg_restore: Inserting employee 7499
  pg_restore: ..New salary: 1600.00
  pg_restore: Inserting employee 7521
  pg_restore: ..New salary: 1250.00
  pg_restore: Inserting employee 7566
  pg_restore: ..New salary: 2975.00
  pg_restore: Inserting employee 7654
  pg_restore: ..New salary: 1250.00
  pg_restore: Inserting employee 7698
  pg_restore: ..New salary: 2850.00
  pg_restore: Inserting employee 7782
  pg_restore: ..New salary: 2450.00
  pg_restore: Inserting employee 7788
  pg_restore: ..New salary: 3000.00
  pg_restore: Inserting employee 7839
  pg_restore: ..New salary: 5000.00
  pg_restore: Inserting employee 7844
  pg_restore: ..New salary: 1500.00
  pg_restore: Inserting employee 7876
  pg_restore: ..New salary: 1100.00
  pg_restore: Inserting employee 7900
  pg_restore: ..New salary: 950.00
  pg_restore: Inserting employee 7902
  pg_restore: ..New salary: 3000.00
  pg_restore: Inserting employee 7934
  pg_restore: ..New salary: 1300.00
  pg_restore: User enterprisedb added employee(s) on 2016-02-13
  pg_restore: processing data for table "public.jobhist"
  pg_restore: executing SEQUENCE SET next_empno
  pg_restore: setting owner and privileges for TABLE DATA "pgagent.pga_jobagent"
  pg_restore: setting owner and privileges for TABLE DATA "pgagent.pga_jobclass"
  pg_restore: setting owner and privileges for TABLE DATA "pgagent.pga_job"
  pg_restore: setting owner and privileges for TABLE DATA "pgagent.pga_schedule"
  pg_restore: setting owner and privileges for TABLE DATA "pgagent.pga_exception"
  pg_restore: setting owner and privileges for TABLE DATA "pgagent.pga_joblog"
  pg_restore: setting owner and privileges for TABLE DATA "pgagent.pga_jobstep"
  pg_restore: setting owner and privileges for TABLE DATA "pgagent.pga_jobsteplog"
  pg_restore: setting owner and privileges for TABLE DATA "public.dept"
  pg_restore: setting owner and privileges for TABLE DATA "public.emp"
  pg_restore: setting owner and privileges for TABLE DATA "public.jobhist"
  pg_restore: setting owner and privileges for SEQUENCE SET "public.next_empno"
  ```
	* `-a`: Data only mode

  ```
  testdb=# \d+
                                     List of relations
      Schema    |        Name        |   Type   |    Owner     |    Size    | Description 
  --------------+--------------------+----------+--------------+------------+-------------
   enterprisedb | pg_buffercache     | view     | enterprisedb | 0 bytes    | 
   enterprisedb | pg_stat_statements | view     | enterprisedb | 0 bytes    | 
   public       | dept               | table    | enterprisedb | 8192 bytes | 
   public       | emp                | table    | enterprisedb | 8192 bytes | 
   public       | jobhist            | table    | enterprisedb | 8192 bytes | 
   public       | next_empno         | sequence | enterprisedb | 8192 bytes | 
   public       | salesemp           | view     | enterprisedb | 0 bytes    | 
  (7 rows)
  ```

## Physical Backup

### Continuous Archiving

* WAL Archiving을 위한 디랙토리 생성

  ```
  [enterprisedb@ppaslab ~]$ mkdir -p /opt/PostgresPlus/arch
  ```

* `data/postgresql.conf`
  ```ini
  wal_level = archive
  archive_mode = on
  archive_command = 'cp %p /opt/PostgresPlus/arch/%f'
  ```

* DB restart
  ```
  [enterprisedb@ppaslab ~]$ pg_ctl restart
  ```

* 동작 확인

  강재로 로그 switch를 발생시켜 archived log가 발생하는지 확인해 보자.

  ```
  [enterprisedb@ppaslab ~]$ ls -l /opt/PostgresPlus/arch/
  total 0
  [enterprisedb@ppaslab ~]$ psql -c 'select pg_switch_xlog();'
   pg_switch_xlog 
  ----------------
   0/DE1A9A58
  (1 row)

  [enterprisedb@ppaslab ~]$ ls -l /opt/PostgresPlus/arch/
  total 16384
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 10:29 0000000100000000000000DE
  ```

### Manual Hot-Backup

* 아래 쿼리를 실행하여 DB에 지속적인 변경을 발생 시켜 보자.
  ```
  edb=# \c testdb
  You are now connected to database "testdb" as user "enterprisedb".
  testdb=# create table testlog (ts timestamp);
  CREATE TABLE
  testdb=# insert into testlog values (now()); \watch 1
  INSERT 0 1
  Watch every 1s	Sat Feb 13 11:11:13 2016

  INSERT 0 1
  ...
  ```
  이제 1초마다 데이터가 생성되기 때문에 DB의 시점을 확인할 수 있다.

* 수동으로 hot backup을 수행 해 보자.

  ```
  [enterprisedb@ppaslab ~]$ psql -c "select pg_start_backup('hot-backup test');"
   pg_start_backup 
  -----------------
   0/E2000028
  (1 row)
  [enterprisedb@ppaslab ~]$ cp -R data data5445
  [enterprisedb@ppaslab ~]$ psql -c "select pg_stop_backup();"
  NOTICE:  pg_stop_backup complete, all required WAL segments have been archived
   pg_stop_backup 
  ----------------
   0/E2004770
  (1 row)
  [enterprisedb@ppaslab ~]$ rm -f data5445/postmaster.pid 
  [enterprisedb@ppaslab ~]$ rm -f data5445/pg_log/*
  ```

* Backup 정보를 확인해 보자.
  ```
  [enterprisedb@ppaslab ~]$ cat /opt/PostgresPlus/arch/0000000100000000000000E6.00000028.backup
  START WAL LOCATION: 0/E6000028 (file 0000000100000000000000E6)
  STOP WAL LOCATION: 0/E6000810 (file 0000000100000000000000E6)
  CHECKPOINT LOCATION: 0/E6000300
  BACKUP METHOD: pg_start_backup
  BACKUP FROM: master
  START TIME: 2016-02-13 11:29:46 KST
  LABEL: hot-backup test
  STOP TIME: 2016-02-13 11:29:58 KST
  ```

  지금은 DB의 크기가 작고 변경량이 많지 않기 때문에 `START WAL LOCATION`과, `STOP WAL LOCATION`이 모두 동일하다. 그리고 이 파일은 online WAL 즉 `pg_xlog`에도 동일하게 존제 한다.

  ```
  [enterprisedb@ppaslab ~]$ ls data5445/pg_xlog/0000000100000000000000E6
  data5445/pg_xlog/0000000100000000000000E6
  ```

  따라서 지금의 테스트는 완전 복구를 위해 별도의 archived log backup이 필요하지 않다.

* 이후 테스트를 위해 방금 백업 받은 데이터를 별도로 보관해 두자.

  편의상 `data5445/postgresql.conf`의 포트 설정을 미리 변경해 둔다.

  ```
  port = 5445
  archive_mode = off
  ```

	```
  [enterprisedb@ppaslab ~]$ tar -zcvf data5445.tar.gz data5445
  ```

### 완전 복구

지금 이 시간에도 원본 DB는 계속 `testlog` 테이블에 데이터를 발생시키고 있을 것이다. Continuous Archiving을 이용하면 앞서 받은 과거의 backup으로 지금 이시점 까지의 DB로 복구해 낼 수 있다.

* Log switch

  아직 archiving 되지 않은 WAL로그가 있을 것이기 때문에 강제로 log switch 시켜 archiving 되도록 한다.

  ```
  [enterprisedb@ppaslab ~]$ psql -c "select now(), pg_switch_xlog();"

                 now                | pg_switch_xlog 
  ----------------------------------+----------------
   13-FEB-16 11:52:36.560382 +09:00 | 0/E7036ED8
  (1 row)

  [enterprisedb@ppaslab ~]$ psql -c "select now(), pg_switch_xlog();"

                 now                | pg_switch_xlog 
  ----------------------------------+----------------
   13-FEB-16 11:52:40.960382 +09:00 | 0/E8000178
  (1 row)

  [enterprisedb@ppaslab ~]$ ls -l /opt/PostgresPlus/arch/
  total 98308
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 11:27 0000000100000000000000E3
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 11:27 0000000100000000000000E4
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 11:29 0000000100000000000000E5
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 11:29 0000000100000000000000E6
  -rw-------. 1 enterprisedb enterprisedb      302 Feb 13 11:29 0000000100000000000000E6.00000028.backup
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 11:52 0000000100000000000000E7
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 11:52 0000000100000000000000E8
  ```

  위의 예에서는 `0000000100000000000000E7`, `0000000100000000000000E8` 두개의 파일이 백업 이후에 생성된 파일이다. 우리는 같은 장비에서 테스트를 하기 때문에 별도의 백업이 필요없지만 운영 환경이라면 이 파일들을 주기적으로 백업하여야 하고, 복구시에 복구 대상 장비의 적절한 위치에 가져다 놓아야 한다.
  *++지금은 이 절차는 완료된 것으로 가정하고 진행하겠다.++*

* Post를 5445로 변경해서 완전 복구를 한 다음 DB를 기동해 보자.

  * `data5445/recovery.conf`
    ```
    restore_command = 'cp /opt/PostgresPlus/arch/%f %p'
    ```
    `recovery.conf`파일에 `recovery_target_time`이 없으므로 완전 복구가 실행 된다. 이 경우 `restore_command`에 의해서 찾을 수 있는 archived wal log 전부를 복구한다.

  * DB 기동
    ```
    [enterprisedb@ppaslab ~]$ pg_ctl -D data5445 start
    server starting
    ```

* 로그를 확인해 보자.
  ```
  [enterprisedb@ppaslab ~]$ cat data5445/pg_log/enterprisedb-2016-02-13_120342.log 
  앞부분 생략
  ...
  2016-02-13 12:03:42 KST LOG:  consistent recovery state reached at 0/E6000810
  2016-02-13 12:03:42 KST LOG:  restored log file "0000000100000000000000E7" from archive
  2016-02-13 12:03:42 KST LOG:  restored log file "0000000100000000000000E8" from archive
  cp: cannot stat `/opt/PostgresPlus/arch/0000000100000000000000E9': No such file or directory
  2016-02-13 12:03:42 KST LOG:  unexpected pageaddr 0/C4000000 in log segment 0000000100000000000000E9, offset 0
  2016-02-13 12:03:42 KST LOG:  redo done at 0/E8000160
  2016-02-13 12:03:42 KST LOG:  last completed transaction was at log time 2016-02-13 11:52:40.714872+09
  2016-02-13 12:03:42 KST LOG:  restored log file "0000000100000000000000E8" from archive
  cp: cannot stat `/opt/PostgresPlus/arch/00000002.history': No such file or directory
  2016-02-13 12:03:42 KST LOG:  selected new timeline ID: 2
  cp: cannot stat `/opt/PostgresPlus/arch/00000001.history': No such file or directory
  2016-02-13 12:03:42 KST LOG:  archive recovery complete
  2016-02-13 12:03:42 KST LOG:  MultiXact member wraparound protections are now enabled
  2016-02-13 12:03:42 KST LOG:  database system is ready to accept connections
  2016-02-13 12:03:42 KST LOG:  autovacuum launcher started
  ```

  로그를 보면 앞서 확인한 최종 WAL log (여기서는 `0000000100000000000000E8`) 까지 복구가 되었으며 새로운 timeline이 시작됨을 알 수 있다.

  복구가 완료되면 DB가 open 되며 `recovery.conf` 파일은 `recovery.done`으로 변경된다.

  ```
  [enterprisedb@ppaslab ~]$ ls -l data5445/recovery.*
  -rw-rw-r--. 1 enterprisedb enterprisedb 52 Feb 13 11:33 data5445/recovery.done
  ```

  쿼리를 통해서도 마지막 log switch 시점까지의 데이터가 모두 복구되어 있음을 알 수 있다.

  ```
  [enterprisedb@ppaslab ~]$ psql -p 5445 testdb
  Password: 
  psql.bin (9.5.0.5)
  Type "help" for help.

  testdb=# select max(ts) from testlog;
             max           
  -------------------------
   13-FEB-16 11:52:40.7147
  (1 row)
  ```

### 불완전 복구 (PITR - Point In Time Recovery)

불완전 복구란 소스 DB의 최종 상태가 아니라 full 백업 시점 이후의 특정 시점 까지만 복구를 하는 기능이다. 전부가 아닌 일부만 복구한다는 의미에서 '불완전'복구라고 부른다.

* 초기화

  먼저 앞서 했던 작업을 초기화 하자.

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl -D data5445 stop
  waiting for server to shut down.... done
  server stopped
  [enterprisedb@ppaslab ~]$ rm -rf data5445
  [enterprisedb@ppaslab ~]$ tar -zxvf data5445.tar.gz 
  ```

* 복구 설정

  `recovry.conf` 파일에 원하는 복구 시점을 지정한다. Full backup을 한 시점과 마지막 log switch 시점 중간의 적당한 시간을 지정해 준다.

  ```
  restore_command = 'cp /opt/PostgresPlus/arch/%f %p'
  recovery_target_time = '13-FEB-16 11:50:00'
  ```

* DB 기동

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl -D data5445 start
  server starting
  ```

* 로그 확인

  ```
  [enterprisedb@ppaslab ~]$ cat data5445/pg_log/enterprisedb-2016-02-13_121533.log 
  ...
  2016-02-13 12:15:34 KST LOG:  starting point-in-time recovery to 2016-02-13 11:50:00+09
  2016-02-13 12:15:34 KST LOG:  restored log file "0000000100000000000000E6" from archive
  2016-02-13 12:15:34 KST LOG:  redo starts at 0/E6000028
  2016-02-13 12:15:34 KST LOG:  consistent recovery state reached at 0/E6000810
  2016-02-13 12:15:34 KST LOG:  restored log file "0000000100000000000000E7" from archive
  2016-02-13 12:15:34 KST LOG:  recovery stopping before commit of transaction 4391, time 2016-02-13 11:50:00.10177+09
  2016-02-13 12:15:34 KST LOG:  redo done at 0/E70300E0
  2016-02-13 12:15:34 KST LOG:  last completed transaction was at log time 2016-02-13 11:49:59.096903+09
  cp: cannot stat `/opt/PostgresPlus/arch/00000002.history': No such file or directory
  2016-02-13 12:15:34 KST LOG:  selected new timeline ID: 2
  cp: cannot stat `/opt/PostgresPlus/arch/00000001.history': No such file or directory
  2016-02-13 12:15:34 KST LOG:  archive recovery complete
  2016-02-13 12:15:34 KST LOG:  MultiXact member wraparound protections are now enabled
  2016-02-13 12:15:34 KST LOG:  database system is ready to accept connections
  2016-02-13 12:15:34 KST LOG:  autovacuum launcher started
  ```

* 결과 확인

  ```
  [enterprisedb@ppaslab ~]$ psql -p 5445 testdb
  Password: 
  psql.bin (9.5.0.5)
  Type "help" for help.

  testdb=# select max(ts) from testlog;
              max            
  ---------------------------
   13-FEB-16 11:49:59.096749
  (1 row)
  ```

### `pg_basebackup`

* 초기화 

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl -D data5445 stop
  waiting for server to shut down.... done
  server stopped
  [enterprisedb@ppaslab ~]$ rm -rf data5445
  ```

* 설정

  `pg_basebackup`은 replication protocol을 사용한다. 아래 내용을 `pg_hba.conf`에 추가해 준다.

  ```
  host    replication     enterprisedb        127.0.0.1/32            md5
  host    replication     enterprisedb        ::1/128                 md5
  ```

  replication protocol은 WAL Sender를 통해 서비스 된다. `postgresql.conf`에 아래 내용을 변경해 준다.

  ```
  max_wal_senders = 5
  ```

  ```
  [enterprisedb@ppaslab ~]$ pg_ctl restart
  waiting for server to shut down.... done
  server stopped
  server starting
  ```

* Plain format으로 백업을 받아 보자.
  ```
  [enterprisedb@ppaslab ~]$ pg_basebackup -D data2 -h localhost -p 5444 -U enterprisedb -W
  Password: 
  NOTICE:  pg_stop_backup complete, all required WAL segments have been archived
  ```

* `-x` 옵션을 이용해서 백업을 받아 보자.

  ```
  [enterprisedb@ppaslab ~]$ pg_basebackup -D data3 -x -h localhost -p 5444 -U enterprisedb -W
  Password: 
  ```

  Hot backup시 start backup과 end backup 사이의 WAL log를 반드시 보관해야 한다. End backup시 자동으로 log switch가 발생하고 archive 된다. 따라서 이 파일을 별도로 복사해 둬야 이 백업이 유효한 백업이 된다.

  ```
  [enterprisedb@ppaslab ~]$ ls -l data2/pg_xlog
  total 4
  drwx------. 2 enterprisedb enterprisedb 4096 Feb 13 15:14 archive_status
  [enterprisedb@ppaslab ~]$ ls -l /opt/PostgresPlus/arch/
  total 16388
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 15:15 0000000100000000000000F7
  -rw-------. 1 enterprisedb enterprisedb      305 Feb 13 15:15 0000000100000000000000F7.00000028.backup
  ```

  `-x` 옵션을 이용하면 복구에 필요한 WAL log가 pg_xlog에 포함된 상태로 백업 된다. 따라서 이 백업 자체로 유효한 백업이 된다.

  ```
  [enterprisedb@ppaslab ~]$ ls -l data3/pg_xlog
  total 16388
  -rw-------. 1 enterprisedb enterprisedb 16777216 Feb 13 15:15 0000000100000000000000F7
  drwx------. 2 enterprisedb enterprisedb     4096 Feb 13 15:15 archive_status
  ```

* `tar.gz` 포멧으로 백업을 받아 보자.
  ```
  [enterprisedb@ppaslab ~]$ pg_basebackup -h localhost -p 5444 -U enterprisedb -W -D /opt/PostgresPlus/backup160213/ -Ft -z -P
  Password: 
  74053/74053 kB (100%), 1/1 tablespace
  NOTICE:  pg_stop_backup complete, all required WAL segments have been archived
  ```
