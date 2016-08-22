# Lab: Security Hands-on Lab

## 1. pg_hba.conf
#### 1.1. Default Configuration
```
-bash-4.1$ cd $PGDATA
-bash-4.1$ vi pg_hba.conf

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5

```

#### 1.2. localhost에서 test 유저가 edb 데이터베이스에 인증없이 접속
```
-bash-4.1$ vi pg_hba.conf

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   edb             test                                    trust
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5

-bash-4.1$ pg_ctl reload
-bash-4.1$
-bash-4.1$ psql -d edb -U test
Password for user test:
psql.bin (9.5.0.5)
Type "help" for help.

edb=>
```

#### 1.3. localhost에서 test 유저가 edb 데이터베이스에 인증없이 접속
```
-bash-4.1$ vi pg_hba.conf

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     md5
local   edb             test                                    trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5

-bash-4.1$ pg_ctl reload
-bash-4.1$
-bash-4.1$ psql -d edb -U test
Password for user test:

```

#### 1.4. $EDBHOME/.pgpass
```
-bash-4.1$ psql -d postgres -U test
Password for user test:
-bash-4.1$
-bash-4.1$ cd $EDBHOME
-bash-4.1$
-bash-4.1$ cat .pgpass
localhost:5444:edb:enterprisedb:ppas
-bash-4.1$
-bash-4.1$ vi .pgpass

localhost:5444:edb:enterprisedb:ppas
localhost:5444:postgres:test:ppas

-bash-4.1$ psql -d postgres -U test
psql.bin (9.5.0.5)
Type "help" for help.

postgres=>
```


## 2. User & Role
#### 2.1. Create User
```sql
-bash-4.1$ psql postgres
Password:
psql.bin (9.5.0.5)
Type "help" for help.

postgres=# \du
                                     List of roles
  Role name   |                         Attributes                         | Member of
--------------+------------------------------------------------------------+-----------
 enterprisedb | Superuser, Create role, Create DB, Replication, Bypass RLS+| {}
              | Profile default                                            |
 test         | Profile default                                            | {}

postgres=#
postgres=# create user aaa password 'ppas';
CREATE ROLE
postgres=#
postgres=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 public | enterprisedb
(1 row)

postgres=#
postgres=# create user bbb identified by ppas;
CREATE ROLE
postgres=#
postgres=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 bbb    | bbb
 public | enterprisedb
(2 rows)

postgres=#
```

#### 2.2. Access Control
##### 2.2.1. Create User
```sql
-bash-4.1$ psql
psql.bin (9.5.0.5)
Type "help" for help.

edb=#
edb=# create user sales identified by ppas;
CREATE ROLE
edb=#
edb=# create user ops identified by ppas;
CREATE ROLE
edb=#
edb=# \du
                                     List of roles
  Role name   |                         Attributes                         | Member of
--------------+------------------------------------------------------------+-----------
 aaa          | Profile default                                            | {}
 bbb          | Profile default                                            | {}
 enterprisedb | Superuser, Create role, Create DB, Replication, Bypass RLS+| {}
              | Profile default                                            |
 ops          | Profile default                                            | {}
 sales        | Profile default                                            | {}
 test         | Profile default                                            | {}

edb=# \dn
          List of schemas
        Name        |    Owner
--------------------+--------------
 dbms_job_procedure | enterprisedb
 enterprisedb       | enterprisedb
 ops                | ops
 pgagent            | enterprisedb
 public             | enterprisedb
 sales              | sales
(6 rows)

edb=#
```
##### 2.2.2. Grant Privileges
###### 1. Description
```
1. edb DB에 sales 유저로 접속
2. pipeline 테이블 생성
3. ops 유저에 pipeline 테이블 조회 권한 부여
4. ops 유저가 sales의 pipeline 테이블 조회 가능 여부 확인
5. 스키마 오브젝트 조회 기본 권한인 usage 권한 부여
6. ops 유저가 sales의 pipeline 테이블 조회 가능 여부 확인
```
###### 2. Demo
```sql
edb=# \c edb sales
Password for user sales:
You are now connected to database "edb" as user "sales".
edb=>
edb=> create table pipeline (id int, name varchar);
CREATE TABLE
edb=>
edb=> grant select on pipeline to ops;
GRANT
edb=>
edb=> \dp+ pipeline
                               Access privileges
 Schema |   Name   | Type  |  Access privileges  | Column privileges | Policies
--------+----------+-------+---------------------+-------------------+----------
 sales  | pipeline | table | sales=arwdDxt/sales+|                   |
        |          |       | ops=r/sales         |                   |
(1 row)

edb=>
edb=> \c edb ops
Password for user ops:
You are now connected to database "edb" as user "ops".
edb=>
edb=> select * from sales.pipeline ;
ERROR:  permission denied for schema sales
LINE 1: select * from sales.pipeline ;
                      ^
edb=>
edb=> \c edb sales
Password for user sales:
You are now connected to database "edb" as user "sales".
edb=>
edb=> grant usage on schema sales to ops;
GRANT
edb=>
edb=> \c edb ops
Password for user ops:
You are now connected to database "edb" as user "ops".
edb=> select * from sales.pipeline ;
 id | name
----+------
(0 rows)

edb=>
```
##### 2.2.3. ALTER DEFAULT PRIVILEGES
###### 1. Description
```
1. edb DB에 ops 유저로 접속
2. ops 스키마의 이후 생성되는 모든 테이블에 대한 조회 권한을 sales 유저에게 부여
3. pipeline, deal 테이블 생성
4. ops 스키마에 대한 usage 권한을 sales 유저에 부여
5. sales 유저의 조회 여부 확인
```
###### 2. Demo
```sql
edb=> \c edb ops
edb=>
edb=> alter default privileges in schema ops grant select on tables to sales;
ALTER DEFAULT PRIVILEGES
edb=>
edb=> \ddp+
         Default access privileges
 Owner | Schema | Type  | Access privileges
-------+--------+-------+-------------------
 ops   | ops    | table | sales=r/ops
(1 row)

edb=>
edb=> create table pipeline (id int, name varchar);
CREATE TABLE
edb=>
edb=> create table deal (id int, name varchar);
CREATE TABLE
edb=>
edb=> grant usage on schema ops to sales;
GRANT
edb=>
edb=> \dp pipeline
                              Access privileges
 Schema |   Name   | Type  | Access privileges | Column privileges | Policies
--------+----------+-------+-------------------+-------------------+----------
 ops    | pipeline | table | sales=r/ops      +|                   |
        |          |       | ops=arwdDxt/ops   |                   |
(1 row)

edb=> \dp deal
                            Access privileges
 Schema | Name | Type  | Access privileges | Column privileges | Policies
--------+------+-------+-------------------+-------------------+----------
 ops    | deal | table | sales=r/ops      +|                   |
        |      |       | ops=arwdDxt/ops   |                   |
(1 row)

edb=>
edb=> \c edb sales
Password for user sales:
You are now connected to database "edb" as user "sales".
edb=>
edb=> select * from ops.pipeline ;
 id | name
----+------
(0 rows)

edb=> select * from ops.deal ;
 id | name
----+------
(0 rows)

edb=>
edb=>
```
