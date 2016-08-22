# Lab: Create Cluster & Database Hands-on Lab

## 1. initdb
#### 1.1. initdb options
```
-bash-4.1$ initdb --help
initdb initializes a PostgreSQL database cluster.

Usage:
  initdb [OPTION]... [DATADIR]

Options:
  -A, --auth=METHOD         default authentication method for local connections
      --auth-host=METHOD    default authentication method for local TCP/IP connections
      --auth-local=METHOD   default authentication method for local-socket connections
 [-D, --pgdata=]DATADIR     location for this database cluster
  -E, --encoding=ENCODING   set default encoding for new databases
      --locale=LOCALE       set default locale for new databases
      --lc-collate=, --lc-ctype=, --lc-messages=LOCALE
      --lc-monetary=, --lc-numeric=, --lc-time=LOCALE
                            set default locale in the respective category for
                            new databases (default taken from environment)
      --no-locale           equivalent to --locale=C
      --icu-short-form=CFG  set default ICU short form string for new databases
      --pwfile=FILE         read password for the new superuser from file
  -T, --text-search-config=CFG
                            default text search configuration
  -U, --username=NAME       database superuser name
  -W, --pwprompt            prompt for a password for the new superuser
  -X, --xlogdir=XLOGDIR     location for the transaction log directory

Less commonly used options:
  -d, --debug               generate lots of debugging output
  -k, --data-checksums      use data page checksums
  -L DIRECTORY              where to find the input files
  -n, --noclean             do not clean up after errors
  --no-redwood-compat       do not install Redwood-compatibility casts and views
  --redwood-like            use Redwood-compatible LIKE behavior
  -N, --nosync              do not wait for changes to be written safely to disk
  -s, --show                show internal settings
  -S, --sync-only           only sync data directory

Other options:
  -V, --version             output version information, then exit
  -?, --help                show this help, then exit

If the data directory is not specified, the environment variable PGDATA
is used.

Report bugs to <support@enterprisedb.com>.
-bash-4.1$
```

#### 1.2. initdb without option
Execute initdb without any optios
```
-bash-4.1$ initdb -D ~/data2
The files belonging to this database system will be owned by user "enterprisedb".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /opt/PostgresPlus/9.5AS/data2 ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
creating template1 database in /opt/PostgresPlus/9.5AS/data2/base/1 ... ok
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

    pg_ctl -D /opt/PostgresPlus/9.5AS/data2 -l logfile start

-bash-4.1$
```

#### 1.3. initdb with options (Encoding, Locale, User, Password)
Execute initdb with various options like encoding=utf8, locale=ko_KR.utf8, superuser=edb
```
-bash-4.1$ initdb -D ~/data3 --encoding=utf8 --locale=ko_KR.utf8 -U edb -W
The files belonging to this database system will be owned by user "enterprisedb".
This user must also own the server process.

The database cluster will be initialized with locale "ko_KR.utf8".
initdb: could not find suitable text search configuration for locale "ko_KR.utf8"
The default text search configuration will be set to "simple".

Data page checksums are disabled.

creating directory /opt/PostgresPlus/9.5AS/data3 ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
creating template1 database in /opt/PostgresPlus/9.5AS/data3/base/1 ... ok
initializing pg_authid ... ok
Enter new superuser password:
Enter it again:
setting password ... ok
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

    pg_ctl -D /opt/PostgresPlus/9.5AS/data3 -l logfile start

-bash-4.1$
```

#### 1.4. Run New Clusters
##### 1.4.1. Cluster directory : data2
Set port as 5445
```
-bash-4.1$ cd data2/
-bash-4.1$ vi postgresql.conf

port = 5445                             # (change requires restart)

-bash-4.1$ pg_ctl -D ~/data2/ start -w
waiting for server to start....2016-02-10 19:07:00 KST LOG:

	** EnterpriseDB Dynamic Tuning Agent ********************************************
	*       System Utilization: 66 %                                                *
	*         Database Version: 9.5.0.5                                             *
	*            Database Size: 0.1    GB                                           *
	*                      RAM: 1.9    GB                                           *
	*            Shared Memory: 1877   MB                                           *
	*       Max DB Connections: 112                                                 *
	*               Autovacuum: on                                                  *
	*       Autovacuum Naptime: 60   Seconds                                        *
	*********************************************************************************

2016-02-10 19:07:00 KST LOG:  database system was shut down at 2016-02-10 18:56:21 KST
2016-02-10 19:07:00 KST LOG:  MultiXact member wraparound protections are now enabled
2016-02-10 19:07:00 KST LOG:  database system is ready to accept connections
2016-02-10 19:07:00 KST LOG:  autovacuum launcher started
 done
server started
-bash-4.1$
-bash-4.1$
-bash-4.1$ ps -efH|grep postgres
501      13856 10067  0 00:38 pts/1    00:00:00             grep postgres
501      10139     1  0 Feb10 pts/1    00:00:00   /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data
501      10140 10139  0 Feb10 ?        00:00:00     postgres: logger process
501      10142 10139  0 Feb10 ?        00:00:00     postgres: checkpointer process
501      10143 10139  0 Feb10 ?        00:00:00     postgres: writer process
501      10144 10139  0 Feb10 ?        00:00:00     postgres: wal writer process
501      10145 10139  0 Feb10 ?        00:00:00     postgres: autovacuum launcher process
501      10146 10139  0 Feb10 ?        00:00:00     postgres: stats collector process
501      13201     1  0 Feb10 pts/1    00:00:00   /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data2
501      13203 13201  0 Feb10 ?        00:00:00     postgres: checkpointer process
501      13204 13201  0 Feb10 ?        00:00:00     postgres: writer process
501      13205 13201  0 Feb10 ?        00:00:00     postgres: wal writer process
501      13206 13201  0 Feb10 ?        00:00:00     postgres: autovacuum launcher process
501      13207 13201  0 Feb10 ?        00:00:00     postgres: stats collector process
-bash-4.1$
-bash-4.1$
-bash-4.1$ psql -p 5445
psql.bin (9.5.0.5)
Type "help" for help.

edb=#
edb=# \l
                                           List of databases
   Name    |    Owner     | Encoding |   Collate   |    Ctype    | ICU |       Access privileges
-----------+--------------+----------+-------------+-------------+-----+-------------------------------
 edb       | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     |
 postgres  | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     |
 template0 | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     | =c/enterprisedb              +
           |              |          |             |             |     | enterprisedb=CTc/enterprisedb
 template1 | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     | =c/enterprisedb              +
           |              |          |             |             |     | enterprisedb=CTc/enterprisedb
(4 rows)

edb=#
```

##### 1.4.2. Cluster directory : data3
Set port as 5446
```
-bash-4.1$ cd ~/data3/
-bash-4.1$
-bash-4.1$ vi postgresql.conf

port = 5446                             # (change requires restart)

-bash-4.1$ pg_ctl -D ~/data3/ start -w
waiting for server to start....2016-02-11 00:36:17 KST LOG:

	** EnterpriseDB Dynamic Tuning Agent ********************************************
	*       System Utilization: 66 %                                                *
	*         Database Version: 9.5.0.5                                             *
	*            Database Size: 0.1    GB                                           *
	*                      RAM: 1.9    GB                                           *
	*            Shared Memory: 1877   MB                                           *
	*       Max DB Connections: 112                                                 *
	*               Autovacuum: on                                                  *
	*       Autovacuum Naptime: 60   Seconds                                        *
	*********************************************************************************

2016-02-11 00:36:17 KST LOG:  database system was shut down at 2016-02-10 19:03:18 KST
2016-02-11 00:36:17 KST LOG:  MultiXact member wraparound protections are now enabled
2016-02-11 00:36:17 KST LOG:  database system is ready to accept connections
2016-02-11 00:36:17 KST LOG:  autovacuum launcher started
2016-02-11 00:36:18 KST FATAL:  role "enterprisedb" does not exist
 done
server started
-bash-4.1$
-bash-4.1$
-bash-4.1$ ps -efH|grep postgres
501      13856 10067  0 00:38 pts/1    00:00:00             grep postgres
501      10139     1  0 Feb10 pts/1    00:00:00   /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data
501      10140 10139  0 Feb10 ?        00:00:00     postgres: logger process
501      10142 10139  0 Feb10 ?        00:00:00     postgres: checkpointer process
501      10143 10139  0 Feb10 ?        00:00:00     postgres: writer process
501      10144 10139  0 Feb10 ?        00:00:00     postgres: wal writer process
501      10145 10139  0 Feb10 ?        00:00:00     postgres: autovacuum launcher process
501      10146 10139  0 Feb10 ?        00:00:00     postgres: stats collector process
501      13201     1  0 Feb10 pts/1    00:00:00   /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data2
501      13203 13201  0 Feb10 ?        00:00:00     postgres: checkpointer process
501      13204 13201  0 Feb10 ?        00:00:00     postgres: writer process
501      13205 13201  0 Feb10 ?        00:00:00     postgres: wal writer process
501      13206 13201  0 Feb10 ?        00:00:00     postgres: autovacuum launcher process
501      13207 13201  0 Feb10 ?        00:00:00     postgres: stats collector process
501      13816     1  0 00:36 pts/1    00:00:00   /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data3
501      13818 13816  0 00:36 ?        00:00:00     postgres: checkpointer process
501      13819 13816  0 00:36 ?        00:00:00     postgres: writer process
501      13820 13816  0 00:36 ?        00:00:00     postgres: wal writer process
501      13821 13816  0 00:36 ?        00:00:00     postgres: autovacuum launcher process
501      13822 13816  0 00:36 ?        00:00:00     postgres: stats collector process
-bash-4.1$
-bash-4.1$
-bash-4.1$ psql -p 5446
2016-02-11 00:36:43 KST FATAL:  role "enterprisedb" does not exist
psql.bin: FATAL:  role "enterprisedb" does not exist
-bash-4.1$
-bash-4.1$ psql -p 5446 -U edb
psql.bin (9.5.0.5)
Type "help" for help.

edb=# \l
                                List of databases
   Name    | Owner | Encoding |  Collate   |   Ctype    | ICU | Access privileges
-----------+-------+----------+------------+------------+-----+-------------------
 edb       | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     |
 postgres  | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     |
 template0 | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     | =c/edb           +
           |       |          |            |            |     | edb=CTc/edb
 template1 | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     | =c/edb           +
           |       |          |            |            |     | edb=CTc/edb
(4 rows)

edb=#
edb=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 edb       | Superuser, Create role, Create DB, Replication, Bypass RLS+| {}
           | Profile default                                            |

edb=#
```

## 2. Create Database
#### 2.1. Create database
##### 2.1.1. createdb utility
```
-bash-4.1$ createdb --help
createdb creates a PostgreSQL database.

Usage:
  createdb [OPTION]... [DBNAME] [DESCRIPTION]

Options:
  -D, --tablespace=TABLESPACE  default tablespace for the database
  -e, --echo                   show the commands being sent to the server
  -E, --encoding=ENCODING      encoding for the database
  -l, --locale=LOCALE          locale settings for the database
      --lc-collate=LOCALE      LC_COLLATE setting for the database
      --lc-ctype=LOCALE        LC_CTYPE setting for the database
  -O, --owner=OWNER            database user to own the new database
  -T, --template=TEMPLATE      template database to copy
  -V, --version                output version information, then exit
  -?, --help                   show this help, then exit

Connection options:
  -h, --host=HOSTNAME          database server host or socket directory
  -p, --port=PORT              database server port
  -U, --username=USERNAME      user name to connect as
  -w, --no-password            never prompt for password
  -W, --password               force password prompt
  --maintenance-db=DBNAME      alternate maintenance database

By default, a database with the same name as the current user is created.

Report bugs to <support@enterprisedb.com>.
-bash-4.1$
-bash-4.1$ createdb -p 5445 ppaslab
-bash-4.1$
-bash-4.1$ psql -p 5445 -c "\l"
                                           List of databases
   Name    |    Owner     | Encoding |   Collate   |    Ctype    | ICU |       Access privileges
-----------+--------------+----------+-------------+-------------+-----+-------------------------------
 edb       | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     |
 postgres  | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     |
 ppaslab   | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     |
 template0 | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     | =c/enterprisedb              +
           |              |          |             |             |     | enterprisedb=CTc/enterprisedb
 template1 | enterprisedb | UTF8     | en_US.UTF-8 | en_US.UTF-8 |     | =c/enterprisedb              +
           |              |          |             |             |     | enterprisedb=CTc/enterprisedb
(5 rows)

-bash-4.1$
```

##### 2.1.2. CREATE DATABASE statement in psql
```sql
-bash-4.1$ psql -p 5446 -U edb
psql.bin (9.5.0.5)
Type "help" for help.

edb=#
edb=# \h create database
Command:     CREATE DATABASE
Description: create a new database
Syntax:
CREATE DATABASE name
    [ [ WITH ] [ OWNER [=] user_name ]
           [ TEMPLATE [=] template ]
           [ ENCODING [=] encoding ]
           [ LC_COLLATE [=] lc_collate ]
           [ LC_CTYPE [=] lc_ctype ]
           [ ICU_SHORT_FORM [=] icu_short_form ]
           [ TABLESPACE [=] tablespace_name ]
           [ ALLOW_CONNECTIONS [=] allowconn ]
           [ CONNECTION LIMIT [=] connlimit ] ]
           [ IS_TEMPLATE [=] istemplate ]

edb=#
edb=# create database ppaslab template template0;
CREATE DATABASE
edb=#
edb=# \l
                                List of databases
   Name    | Owner | Encoding |  Collate   |   Ctype    | ICU | Access privileges
-----------+-------+----------+------------+------------+-----+-------------------
 edb       | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     |
 postgres  | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     |
 ppaslab   | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     |
 template0 | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     | =c/edb           +
           |       |          |            |            |     | edb=CTc/edb
 template1 | edb   | UTF8     | ko_KR.utf8 | ko_KR.utf8 |     | =c/edb           +
           |       |          |            |            |     | edb=CTc/edb
(5 rows)

edb=#
```

#### 2.2. Create user
##### 2.2.1. createuser utility
```
-bash-4.1$ createuser --help
createuser creates a new PostgreSQL role.

Usage:
  createuser [OPTION]... [ROLENAME]

Options:
  -c, --connection-limit=N  connection limit for role (default: no limit)
  -d, --createdb            role can create new databases
  -D, --no-createdb         role cannot create databases (default)
  -e, --echo                show the commands being sent to the server
  -E, --encrypted           encrypt stored password
  -g, --role=ROLE           new role will be a member of this role
  -i, --inherit             role inherits privileges of roles it is a
                            member of (default)
  -I, --no-inherit          role does not inherit privileges
  -l, --login               role can login (default)
  -L, --no-login            role cannot login
  -N, --unencrypted         do not encrypt stored password
  -P, --pwprompt            assign a password to new role
  -r, --createrole          role can create new roles
  -R, --no-createrole       role cannot create roles (default)
  -s, --superuser           role will be superuser
  -S, --no-superuser        role will not be superuser (default)
  -V, --version             output version information, then exit
  --interactive             prompt for missing role name and attributes rather
                            than using defaults
  --replication             role can initiate replication
  --no-replication          role cannot initiate replication
  -?, --help                show this help, then exit

Connection options:
  -h, --host=HOSTNAME       database server host or socket directory
  -p, --port=PORT           database server port
  -U, --username=USERNAME   user name to connect as (not the one to create)
  -w, --no-password         never prompt for password
  -W, --password            force password prompt

Report bugs to <support@enterprisedb.com>.
-bash-4.1$
-bash-4.1$ createuser -p 5445 -P lab
Enter password for new role:
Enter it again:
-bash-4.1$
-bash-4.1$
-bash-4.1$ psql -p 5445 -U lab
psql.bin (9.5.0.5)
Type "help" for help.

edb=> \du
                                     List of roles
  Role name   |                         Attributes                         | Member of
--------------+------------------------------------------------------------+-----------
 enterprisedb | Superuser, Create role, Create DB, Replication, Bypass RLS+| {}
              | Profile default                                            |
 lab          | Profile default                                            | {}

edb=>
```

##### 2.2.2. CREATE USER statement in psql
```sql
-bash-4.1$ psql -p 5445
psql.bin (9.5.0.5)
Type "help" for help.

edb=#
edb=# create user test with password 'ppas' password expire at '2016-03-01 00:00:00';
CREATE ROLE
edb=#
```

## 3. Schema and search_path
#### 3.1. Create schema
```sql
-bash-4.1$ psql -p 5445
psql.bin (9.5.0.5)
Type "help" for help.

edb=#
edb=# \h create schema
Command:     CREATE SCHEMA
Description: define a new schema
Syntax:
CREATE SCHEMA schema_name [ AUTHORIZATION role_specification ] [ schema_element [ ... ] ]
CREATE SCHEMA AUTHORIZATION role_specification [ schema_element [ ... ] ]
CREATE SCHEMA IF NOT EXISTS schema_name [ AUTHORIZATION role_specification ]
CREATE SCHEMA IF NOT EXISTS AUTHORIZATION role_specification

where role_specification can be:

    [ GROUP ] user_name
  | CURRENT_USER
  | SESSION_USER

edb=#
edb=# create schema a;
CREATE SCHEMA
edb=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 a      | enterprisedb
 public | enterprisedb
(2 rows)

edb=#
edb=# create schema authorization test;
CREATE SCHEMA
edb=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 a      | enterprisedb
 public | enterprisedb
 test   | test
(3 rows)

edb=#
edb=# create schema if not exists b;
CREATE SCHEMA
edb=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 a      | enterprisedb
 b      | enterprisedb
 public | enterprisedb
 test   | test
(4 rows)

edb=#
edb=# create schema if not exists authorization lab;
CREATE SCHEMA
edb=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 a      | enterprisedb
 b      | enterprisedb
 lab    | lab
 public | enterprisedb
 test   | test
(5 rows)

edb=#
```

#### 3.2. search_path
```sql
-bash-4.1$ psql -p 5445
psql.bin (9.5.0.5)
Type "help" for help.

edb=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 a      | enterprisedb
 b      | enterprisedb
 lab    | lab
 public | enterprisedb
 test   | test
(5 rows)

edb=#
edb=# \dt
No relations found.
edb=#
edb=# show search_path ;
   search_path
-----------------
 "$user", public
(1 row)

edb=#
edb=# create table tbl (name varchar);
CREATE TABLE
edb=# create table a.tbl (name varchar);
CREATE TABLE
edb=# create table b.tbl (name varchar);
CREATE TABLE
edb=#
edb=# insert into tbl values ('public');
INSERT 0 1
edb=# insert into a.tbl values ('a');
INSERT 0 1
edb=# insert into b.tbl values ('b');
INSERT 0 1
edb=#
edb=# select name from tbl ;
  name
--------
 public
(1 row)

edb=#
edb=# \dt
          List of relations
 Schema | Name | Type  |    Owner
--------+------+-------+--------------
 public | tbl  | table | enterprisedb
(1 row)

edb=#
edb=#
edb=# set search_path to "$user",a,b,public;
SET
edb=#
edb=# select name from tbl ;
 name
------
 a
(1 row)

edb=# select name from public.tbl ;
  name
--------
 public
(1 row)

edb=#
edb=# drop table tbl;
DROP TABLE
edb=#
edb=# select name from tbl;
 name
------
 b
(1 row)

edb=#
edb=# drop table tbl;
DROP TABLE
edb=# select name from tbl;
  name
--------
 public
(1 row)

edb=#
edb=# drop table tbl;
DROP TABLE
edb=# select name from tbl;
2016-02-11 01:50:13 KST ERROR:  relation "tbl" does not exist at character 18
2016-02-11 01:50:13 KST STATEMENT:  select name from tbl;
ERROR:  relation "tbl" does not exist
LINE 1: select name from tbl;
                         ^
edb=#
```
