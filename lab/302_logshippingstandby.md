# Lab: Log Shipping Standby Server

## Enable archiving

* `data/postgresql.conf`
  ```ini
  wal_level = hot_standby
  archive_mode = on
  archive_command = 'cp %p /opt/PostgresPlus/arch/%f'
  ```

* DB restart
  ```
  [enterprisedb@ppaslab ~]$ pg_ctl restart
  ```

## Copy database to standby

* Copy database

  ```
  [enterprisedb@ppaslab ~]$ pg_basebackup -D data5445 -x -h localhost -p 5444 -U enterprisedb -W
  Password:
  ```

* `data5445/postgresql.conf`
  ```ini
  port = 5445
  archive_mode = off
  hot_standby = on
  ```

* `data5445/recovery.conf`:
  ```ini
  standby_mode='on'
  restore_command='test -f /opt/PostgresPlus/arch/%f && cp /opt/PostgresPlus/arch/%f %p'
  trigger_file='/tmp/trigger.pg.5445'
  ```

* Start standby server

	```
  [enterprisedb@ppaslab ~]$ pg_ctl -D data5445 start
  server starting
  [enterprisedb@ppaslab ~]$ cat data5445/pg_log/enterprisedb-2016-02-13_154407.log 
  2016-02-13 15:44:07 KST LOG:  

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

  2016-02-13 15:44:07 KST LOG:  database system was interrupted; last known up at 2016-02-13 15:42:43 KST
  2016-02-13 15:44:07 KST LOG:  entering standby mode
  2016-02-13 15:44:07 KST LOG:  restored log file "0000000100000000000000FD" from archive
  2016-02-13 15:44:07 KST LOG:  redo starts at 0/FD000028
  2016-02-13 15:44:07 KST LOG:  consistent recovery state reached at 0/FD0000F8
  2016-02-13 15:44:07 KST LOG:  database system is ready to accept read only connections
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
                                                || Did not find any relation named "test".
                                                ||
edb=# select pg_switch_xlog();                  ||
 pg_switch_xlog                                 ||
----------------                                ||
 0/DD015E98                                     ||
(1 row)                                         ||
                                                || edb=# \d test
                                                ||        Table "enterprisedb.test"
                                                ||  Column |       Type        | Modifiers
                                                || --------+-------------------+-----------
                                                ||  id     | integer           |
                                                ||  val1   | bytea             |
                                                ||  val2   | bytea             |
                                                ||  val3   | character varying |
```
