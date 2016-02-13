# Lab: Streaming Replication

## Prepare the Primary Server

`data/postgresql.conf`:
```ini
wal_level = hot_standby
max_wal_senders = 5
archive_mode = on
archive_command = 'cp %p /opt/PostgresPlus/arch/%f'
```

`data/pg_hba.conf`:
```
host    replication     enterprisedb        127.0.0.1/32            md5
host    replication     enterprisedb        ::1/128                 md5
```

```
[enterprisedb@ppaslab ~]$ pg_ctl restart
waiting for server to shut down.... done
server stopped
server starting
```

## Copy database to standby

* Copy database
  ```
  [enterprisedb@ppaslab ~]$ pg_basebackup -D data5446 -x -h localhost -p 5444 -U enterprisedb -W
  Password:
  ```

* `data5445/postgresql.conf`
  ```ini
  port = 5446
  archive_mode = off
  hot_standby = on
  ```

* `data5445/recovery.conf`
  ```ini
  standby_mode = 'on'
  primary_conninfo = 'host=localhost port=5444 user=enterprisedb password=ppas'
  restore_command='test -f /opt/PostgresPlus/arch/%f && cp /opt/PostgresPlus/arch/%f %p'
  trigger_file='/tmp/trigger.pg.5446'
  ```

## Start standby server

* Standby 서버를 기동한다.
  ```
  [enterprisedb@ppaslab ~]$ pg_ctl -D data5446 start
  server starting
  ```

* 로그를 확인해 보자.
  ```
  [enterprisedb@ppaslab ~]$ cat data5446/pg_log/enterprisedb-2016-02-13_162206.log 
  2016-02-13 16:22:06 KST LOG:  database system was interrupted; last known up at 2016-02-13 16:19:44 KST
  2016-02-13 16:22:06 KST LOG:  

    ** EnterpriseDB Dynamic Tuning Agent ********************************************
    *       System Utilization: 66 %                                                *
    *         Database Version: 9.5.0.5                                             *
    *            Database Size: 0.1    GB                                           *
    *                      RAM: 1.9    GB                                           *
    *            Shared Memory: 1877   MB                                           *
    *       Max DB Connections: 119                                                 *
    *               Autovacuum: on                                                  *
    *       Autovacuum Naptime: 3    Seconds                                        *
    *********************************************************************************

  2016-02-13 16:22:06 KST LOG:  entering standby mode
  2016-02-13 16:22:06 KST LOG:  restored log file "000000010000000100000000" from archive
  2016-02-13 16:22:06 KST LOG:  redo starts at 1/28
  2016-02-13 16:22:06 KST LOG:  consistent recovery state reached at 1/F8
  2016-02-13 16:22:06 KST LOG:  database system is ready to accept read only connections
  2016-02-13 16:22:06 KST LOG:  started streaming WAL from primary at 1/1000000 on timeline 1
  ```

## Test

```
         <Master Server>                        ||           <Slave Server>
                                                ||
edb=# drop table if exists test;                ||
NOTICE:  table "test" does not exist, skipping  ||
DROP TABLE                                      ||
edb=# create table test                         ||
edb-# as                                        ||
edb-# select                                    ||
edb-#   generate_series(1, 1000) as id,         ||
edb-#   gen_random_bytes(30) as val1,           ||
edb-#   gen_random_bytes(30) as val2,           ||
edb-#   varchar 'test' as val3;                 ||
SELECT 1000                                     ||
                                                || edb=# \d test
                                                ||        Table "enterprisedb.test"
                                                ||  Column |       Type        | Modifiers
                                                || --------+-------------------+-----------
                                                ||  id     | integer           |
                                                ||  val1   | bytea             |
                                                ||  val2   | bytea             |
                                                ||  val3   | character varying |
                                                ||
                                                || edb=# select count(*) from test;
                                                ||  count
                                                || -------
                                                ||   1000
                                                || (1 row)
edb=# insert into test select * from test;      ||
INSERT 0 1000                                   ||
                                                || edb=# select count(*) from test;
                                                ||  count
                                                || -------
                                                ||   2000
                                                || (1 row)

```

## Monitoring
```
edb=# select * from pg_stat_replication;
  pid  | usesysid |   usename    | application_name | client_addr | client_hostname | client_port |          backend_start           | backend_xmin |   state   | sent_location | write_location | flush_location | replay_location | sync_priority | sync_state 
-------+----------+--------------+------------------+-------------+-----------------+-------------+----------------------------------+--------------+-----------+---------------+----------------+----------------+-----------------+---------------+------------
 24595 |       10 | enterprisedb | walreceiver      | ::1         |                 |       56327 | 13-FEB-16 16:22:06.873699 +09:00 |              | streaming | 1/105E468     | 1/105E468      | 1/105E468      | 1/105E468       |             0 | async
(1 row)

edb=# select pg_xlog_location_diff(sent_location, replay_location) from pg_stat_replication;
 pg_xlog_location_diff 
-----------------------
                     0
(1 row)
```

# Replication Control
```
[enterprisedb@ppaslab ~]$ psql -p 5446
Password: 
psql.bin (9.5.0.5)
Type "help" for help.

edb=# select pg_is_xlog_replay_paused();
 pg_is_xlog_replay_paused 
--------------------------
 f
(1 row)

edb=# select pg_xlog_replay_pause();
 pg_xlog_replay_pause 
----------------------
 
(1 row)

edb=# select pg_is_xlog_replay_paused();
 pg_is_xlog_replay_paused 
--------------------------
 t
(1 row)

edb=# select pg_xlog_replay_resume();
 pg_xlog_replay_resume 
-----------------------
 
(1 row)

edb=# select pg_is_xlog_replay_paused();
 pg_is_xlog_replay_paused 
--------------------------
 f
(1 row)
```