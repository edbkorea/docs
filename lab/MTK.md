# Lab: Oracle Migration Toolkit Hands-on Lab

## 1. Installation

```
[root@ppas ~]# cd pkgs/
[root@ppas pkgs]# ls
edb_mtk.bin                                   ojdbc6.jar            ppasmeta-9.5.0.5-linux-x64
instantclient-basic-linux.x64-11.2.0.4.0.zip  pg_repack-master.zip  ppasmeta-9.5.0.5-linux-x64.tar.gz
[root@ppas pkgs]# ./edb_mtk.bin
Language Selection

Please select the installation language
[1] English - English
[2] Japanese - ???
[3] Simplified Chinese - ????
[4] Traditional Chinese - ????
[5] Korean - ???
Please choose an option [1] : 1
----------------------------------------------------------------------------
Welcome to the Migration Toolkit Setup Wizard.

----------------------------------------------------------------------------
Please read the following License Agreement. You must accept the terms of this
agreement before continuing with the installation.

Press [Enter] to continue:
Limited Use Software License Agreement
Version 2.9

IMPORTANT - READ CAREFULLY

This Limited Use Software License Agreement ("Agreement") is a legal document
between you ("Customer") and EnterpriseDB Corporation ("EnterpriseDB"). It is
important that you read this document before using the EnterpriseDB-provided
software ("Software"). By clicking the "I ACCEPT" button, or by installing, or
otherwise using the Software, Customer agrees to be bound by the terms of this
Agreement, including, without limitation, the warranty disclaimers, limitations
of liability and termination provisions below. Customer agrees that this
Agreement is enforceable like any written agreement negotiated and signed by
Customer. If Customer does not agree with the terms and conditions of this
Agreement, Customer is not licensed to use the Software, and Customer must
destroy any downloaded copies of the Software in its possession or control. The
Software provided under this Agreement may be: (i) generally available
("Generally Available") to the customers of EnterpriseDB, or (ii) pre-release
("Pre-Release") and provided as part of an alpha, beta or evaluation test
program in which case the Software is not yet GA. This agreement will not apply
if Customer has: a) a valid, paid subscription that includes entitlement to a
Full Use Software License for the Software as specified on the applicable order,
and in that case, the terms of the Full Use Software License Agreement will
Press [Enter] to continue:
apply, or b) purchased a Full Use Software License, and in that case, the terms
of the Full Use Software License Agreement will apply.
...
중략
...

Press [Enter] to continue:

Do you accept this license? [y/n]: y

----------------------------------------------------------------------------
User Authentication

This installation requires a registration with EnterpriseDB.com. Please enter
your credentials below. If you do not have an account, Please create one now on
https://www.enterprisedb.com/user-login-registration



Email []:

Password :

----------------------------------------------------------------------------
Existing Installation

An existing Migration Toolkit installation has been found at
/opt/PostgresPlus/edbmtk. This installation will be upgraded.
Press [Enter] to continue:

----------------------------------------------------------------------------
Setup is now ready to begin installing Migration Toolkit on your computer.

Do you want to continue? [Y/n]: Y

----------------------------------------------------------------------------
Please wait while Setup installs Migration Toolkit on your computer.

 Installing Migration Toolkit
 0% ______________ 50% ______________ 100%
 #########################################

----------------------------------------------------------------------------
EnterpriseDB is the leading provider of value-added products and services for
the Postgres community.

Please visit our website at www.enterprisedb.com

[root@ppaslab pkgs]#
[root@ppaslab pkgs]# cd /opt/PostgresPlus/
[root@ppaslab PostgresPlus]#
[root@ppaslab PostgresPlus]# ll
total 7964
drwxr-xr-x. 14 enterprisedb enterprisedb    4096 Feb 18 20:53 9.5AS
drwxr-xr-x.  5 enterprisedb enterprisedb    4096 Feb 15 13:08 connectors
drwxr-xr-x.  5 root         daemon          4096 Feb 18 20:49 edbmtk
drwxr-xr-x.  8 enterprisedb enterprisedb    4096 Feb 15 13:08 pgpool-II-3.4
drwxr-xr-x.  7 enterprisedb enterprisedb    4096 Feb 15 13:08 stackbuilderplus
-rwx------.  1 enterprisedb enterprisedb 8049577 Feb 15 13:08 uninstall-ppas_9_5_complete
-rw-------.  1 enterprisedb enterprisedb   79891 Feb 15 13:08 uninstall-ppas_9_5_complete.dat
[root@ppaslab PostgresPlus]#
[root@ppaslab PostgresPlus]# chown -R enterprisedb.enterprisedb edbmtk/
[root@ppaslab PostgresPlus]#
```

## 2. Configuration

```
1. soe DB 유저 생성
2. 마이그레이션을할 mig DB 생성
3. mig DB에 soe 스키마 생성
```

#### 2.1. Initialize

```
-bash-4.1$ psql
psql.bin (9.5.0.5)
Type "help" for help.

edb=# create database mig;
CREATE DATABASE
edb=#
edb=# \c mig
You are now connected to database "mig" as user "enterprisedb".
mig=#
mig=# create user soe identified by soe;
CREATE ROLE
mig=#
mig=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 public | enterprisedb
 soe    | soe
(2 rows)

mig=>
```

#### 2.2. Copy Oracle JDBC Driver

Copy Oracle JDBC Driver to $JAVA_HOME/jre/lib/ext/

```
[root@ppas ~]# cd pkgs/
[root@ppas pkgs]# ls
instantclient-basic-linux.x64-11.2.0.4.0.zip  pg_repack-master.zip        ppasmeta-9.5.0.5-linux-x64.tar.gz
ojdbc6.jar                                    ppasmeta-9.5.0.5-linux-x64
[root@ppas pkgs]#
[root@ppas pkgs]#
[root@ppas pkgs]# cp ojdbc6.jar /usr/java/jdk1.8.0_72/jre/lib/ext/
[root@ppas pkgs]#
```

#### 2.3. MTK

##### 2.3.1. runMTK.sh

```
-bash-4.1$ cd /opt/PostgresPlus/edbmtk/bin
-bash-4.1$
-bash-4.1$ vi runMTK.sh
# ----------------------------------------------------------------------------
# --
# -- Copyright (c) 2004-2015 - EnterpriseDB Corporation.  All Rights Reserved.
# --
# ----------------------------------------------------------------------------

export base="/opt/PostgresPlus/edbmtk"

. $base/etc/sysconfig/edbmtk-49.config
. $base/etc/sysconfig/runJavaApplication.sh

runJREApplication -Dprop=$base/etc/toolkit.properties -jar $base/bin/edb-migrationtoolkit.jar "$@"
```

##### 2.3.2. edbmtk-49.config

```
-bash-4.1$ cd /opt/PostgresPlus/edbmtk/etc/sysconfig/
-bash-4.1$
-bash-4.1$ vi edbmtk-49.config
#!/bin/sh

JAVA_EXECUTABLE_PATH="/usr/bin/java"
JAVA_MINIMUM_VERSION=1.7
JAVA_BITNESS_REQUIRED=0
```

##### 2.3.3. toolkit.properties

```
-bash-4.1$ cd /opt/PostgresPlus/edbmtk/etc/
-bash-4.1$ vi toolkit.properties
SRC_DB_URL=jdbc:oracle:thin:@localhost:1521:XE
SRC_DB_USER=soe
SRC_DB_PASSWORD=soe

TARGET_DB_URL=jdbc:edb://localhost:5444/mig
TARGET_DB_USER=soe
TARGET_DB_PASSWORD=soe
```

#### 2.4.OCI Environment Setting

##### 2.4.1. Oracle Instant Client

```
[root@ppas ~]# cd /opt/PostgresPlus/
[root@ppas PostgresPlus]#
[root@ppas PostgresPlus]# ls ~/pkgs/
edb_mtk.bin                                   ojdbc6.jar            ppasmeta-9.5.0.5-linux-x64
instantclient-basic-linux.x64-11.2.0.4.0.zip  pg_repack-master.zip  ppasmeta-9.5.0.5-linux-x64.tar.gz
[root@ppas PostgresPlus]
[root@ppas PostgresPlus]# unzip ~/pkgs/instantclient-basic-linux.x64-11.2.0.4.0.zip
Archive:  /root/pkgs/instantclient-basic-linux.x64-11.2.0.4.0.zip
  inflating: instantclient_11_2/BASIC_README
  inflating: instantclient_11_2/adrci
  inflating: instantclient_11_2/genezi
  inflating: instantclient_11_2/libclntsh.so.11.1
  inflating: instantclient_11_2/libnnz11.so
  inflating: instantclient_11_2/libocci.so.11.1
  inflating: instantclient_11_2/libociei.so
  inflating: instantclient_11_2/libocijdbc11.so
  inflating: instantclient_11_2/ojdbc5.jar
  inflating: instantclient_11_2/ojdbc6.jar
  inflating: instantclient_11_2/uidrvci
  inflating: instantclient_11_2/xstreams.jar
[root@ppas PostgresPlus]# ls
9.5AS       edbmtk              stackbuilderplus             uninstall-ppas_9_5_complete.dat
connectors  instantclient_11_2  uninstall-ppas_9_5_complete
[root@ppas PostgresPlus]
[root@ppas PostgresPlus]# cd instantclient_11_2/
[root@ppas instantclient_11_2]# ls
BASIC_README  genezi             libnnz11.so      libociei.so      ojdbc5.jar  uidrvci
adrci         libclntsh.so.11.1  libocci.so.11.1  libocijdbc11.so  ojdbc6.jar  xstreams.jar
[root@ppas instantclient_11_2]#
[root@ppas instantclient_11_2]# ln -s libclntsh.so.11.1 libclntsh.so
[root@ppas instantclient_11_2]#
```

##### 2.4.2 PPAS Configuration

```
-bash-4.1$ cd $PGDATA
-bash-4.1$ vi postgresql.conf

oracle_home ='/opt/PostgresPlus/instantclient_11_2'     # path to the Oracle home directory;
                                        # only used by OCI Dblink; defaults
                                        # to ORACLE_HOME environment variable.

-bash-4.1$
-bash-4.1$ cd
-bash-4.1$ vi .profile
. pgplus_env.sh

export LD_LIBRARY_PATH=/opt/PostgresPlus/instantclient_11_2:$LD_LIBRARY_PATH

-bash-4.1$
-bash-4.1$ . .profile
-bash-4.1$
-bash-4.1$ pg_ctl -D $PGDATA stop -mf
waiting for server to shut down.... done
server stopped
-bash-4.1$ ps -ef|grep postgres
501      24264 23755  0 00:44 pts/0    00:00:00 grep postgres
-bash-4.1$
-bash-4.1$ pg_ctl -D $PGDATA start -w
server starting
-bash-4.1$ 2016-02-08 00:44:46 KST LOG:  redirecting log output to logging collector process
2016-02-08 00:44:46 KST HINT:  Future log output will appear in directory "pg_log".

-bash-4.1$ ps -ef|grep postgres
501      24267     1  0 00:44 pts/0    00:00:00 /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data
501      24268 24267  0 00:44 ?        00:00:00 postgres: logger process
501      24270 24267  0 00:44 ?        00:00:00 postgres: checkpointer process
501      24271 24267  0 00:44 ?        00:00:00 postgres: writer process
501      24272 24267  0 00:44 ?        00:00:00 postgres: wal writer process
501      24273 24267  0 00:44 ?        00:00:00 postgres: autovacuum launcher process
501      24274 24267  0 00:44 ?        00:00:00 postgres: stats collector process
501      24276 23755  0 00:44 pts/0    00:00:00 grep postgres
-bash-4.1$
```

## 3. Schema Migration

#### 3.1. MTK Options

```
-bash-4.1$ cd /opt/PostgresPlus/edbmtk/bin
-bash-4.1$
-bash-4.1$ ./runMTK.sh -help
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...

EnterpriseDB Migration Toolkit (Build 49.0.3)

Usage: runMTK [-options] SCHEMA

If no option is specified, the complete schema will be imported.

where options include:
-help		Display the application command-line usage.
-version	Display the application version information.
-verbose [on|off] Display application log messages on standard output (default: on).

-schemaOnly	Import the schema object definitions only.
-dataOnly	Import the table data only. When -tables is in place, it imports data only for the selected tables. Note: If there are any FK constraints defined on target tables, use -truncLoad option along with this option.

-sourcedbtype db_type The -sourcedbtype option specifies the source database type. db_type may be one of the following values: mysql, oracle, sqlserver, sybase, postgresql, enterprisedb. db_type is case-insensitive. By default, db_type is oracle.
-targetdbtype db_type The -targetdbtype option specifies the target database type. db_type may be one of the following values: oracle, sqlserver, postgresql, enterprisedb. db_type is case-insensitive. By default, db_type is enterprisedb.

-allTables	Import all tables.
-tables LIST	Import comma-separated list of tables.
-constraints	Import the table constraints.
-indexes	Import the table indexes.
-triggers	Import the table triggers.
-allViews	Import all Views.
-views LIST	Import comma-separated list of Views.
-allProcs	Import all stored procedures.
-procs LIST	Import comma-separated list of stored procedures.
-allFuncs	Import all functions.
-funcs LIST	Import comma-separated list of functions.
-allPackages	Import all packages.
-packages LIST Import comma-separated list of packages.
-allSequences	Import all sequences.
-sequences LIST Import comma-separated list of sequences.
-targetSchema NAME Name of the target schema (default: target schema is named after source schema).
-allDBLinks	Import all Database Links.
-allSynonyms	It enables the migration of all public and private synonyms from an Oracle database to an Advanced Server database.  If a synonym with the same name already exists in the target database, the existing synonym will be replaced with the migrated version.
-allPublicSynonyms	It enables the migration of all public synonyms from an Oracle database to an Advanced Server database.  If a synonym with the same name already exists in the target database, the existing synonym will be replaced with the migrated version.
-allPrivateSynonyms	It enables the migration of all private synonyms from an Oracle database to an Advanced Server database.  If a synonym with the same name already exists in the target database, the existing synonym will be replaced with the migrated version.

-dropSchema [true|false] Drop the schema if it already exists in the target database (default: false).
-truncLoad	It disables any constraints on target table and truncates the data from the table before importing new data. This option can only be used with -dataOnly.
-safeMode	Transfer data in safe mode using plain SQL statements.
-copyDelimiter	Specify a single character to be used as delimiter in copy command when loading table data. Default is \t
-batchSize	Specify the Batch Size to be used by the bulk inserts. Valid values are  1-1000, default batch size is 1000, reduce if you run into Out of Memory exception
-cpBatchSize	 Specify the Batch Size in MB, to be used in the Copy Command. Valid value is > 0, default batch size is 8 MB
-fetchSize	 Specify fetch size in terms of number of rows should be fetched in result set at a time. This option can be used when tables contain millions of rows and you want to avoid out of memory errors.
-filterProp	The properties file that contains table where clause.
-skipFKConst	Skip migration of FK constraints.
-skipCKConst	Skip migration of Check constraints.
-ignoreCheckConstFilter	By default MTK does not migrate Check constraints and Default clauses from Sybase, use this option to turn off this filter.
-fastCopy	Bypass WAL logging to perform the COPY operation in an optimized way, default disabled.
-customColTypeMapping LIST	Use custom type mapping represented by a semi-colon separated list, where each entry is specified using COL_NAME_REG_EXPR=TYPE pair. e.g. .*ID=INTEGER
-customColTypeMappingFile PROP_FILE	The custom type mapping represented by a properties file, where each entry is specified using COL_NAME_REG_EXPR=TYPE pair. e.g. .*ID=INTEGER
-offlineMigration [PATH] This performs offline migration and saves the DDL/DML scripts in files for a later execution. By default the script files will be saved under user home folder, if required follow -offlineMigration option with a custom path.
-logDir LOG_PATH Specify a custom path to save the log file. By default, on Linux the logs will be saved under folder $HOME/.enterprisedb/migration-toolkit/logs. In case of Windows logs will be saved under folder %HOMEDRIVE%%HOMEPATH%\.enterprisedb\migration-toolkit\logs.
-copyViaDBLinkOra This option can be used to copy data using dblink_ora COPY command. This option can only be used in Oracle to EnterpriseDB migration mode.
-singleDataFile	Use single SQL file for offline data storage for all tables. This option cannot be used in COPY format.
-allUsers Import all users and roles from the source database.
-users LIST Import the selected users/roles from the source database. LIST is a comma-separated list of user/role names e.g. -users MTK,SAMPLE
-allProfiles Import all profiles from the source database.
-profiles LIST Import the selected profiles from the source database. LIST is a comma-separated list of profile names e.g. -profiles USER_PROFILE,ADMIN_PROFILE
-allRules Import all rules from the source database.
-rules LIST Import the selected rules from the source database. LIST is a comma-separated list of rule names e.g. -rules high_sal_emp,low_sal_emp
-allGroups Import all groups from the source database.
-groups LIST Import the selected groups from the source database. LIST is a comma-separated list of group names e.g. -groups acct_emp,mkt_emp
-allDomains Import all domain, enumeration and composite types from the source database.
-domains LIST Import the selected domain, enumeration and composite types from the source database. LIST is a comma-separated list of domain names e.g. -domains d_email,d_dob, mood
-objecttypes	Import the user-defined object types.
-replaceNullChar <CHAR> If null character is part of a column value, the data migration fails over JDBC protocol. This option can be used to replace null character with a user-specified character.
-importPartitionAsTable [LIST] Use this option to import Oracle Partitioned table as a normal table in EnterpriseDB. To apply the rule on a selected set of tables, follow the option by a comma-separated list of table names.
-enableConstBeforeDataLoad Use this option to re-enable constraints (and triggers) before data load. This is useful in the scenario when the migrated table is mapped to a partition table in EnterpriseDB.
-checkFunctionBodies [true|false] When set to false, it disables validation of the function body during function creation, this is to avoid errors if function contains forward references. Applicable when target database is Postgres/EnterpriseDB, default is true.
-retryCount VALUE	Specify the number of re-attempts performed by MTK to migrate objects that failed due to cross-schema dependencies. The VALUE parameter should be greater than 0, default is 2.
-analyze 	It invokes ANALYZE operation against a target Postgres or Postgres Plus Advanced Server database. The ANALYZE collects statistics for the migrated tables that are utilized for efficient query plans.
-vacuumAnalyze 	It invokes VACUUM and ANALYZE operations against a target Postgres or Postgres Plus Advanced Server database. The VACUUM reclaims dead tuple storage whereas ANALYZE collects statistics for the migrated tables that are utilized for efficient query plans.
-loaderCount VALUE	Specify the number of jobs (threads) to perform data load in parallel. The VALUE parameter should be greater than 0, default is 1.
-logFileSize VALUE	It represents the maximum file size limit (in MB) before rotating to a new log file, defaults to 50MB.
-logFileCount VALUE	It represents the number of files to maintain in log file rotation history, defaults to 20. Specify a value of zero to disable log file rotation.
-useOraCase	It preserves the identifier case while migrating from Oracle, except for functions, procedures and packages unless identifier names are given in quotes.
-logBadSQL	It saves the DDL scripts for the objects that fail to migrate, in a .sql file in log folder.
-targetDBVersion	It represents the major.minor version of the target database. This option is applicable for offline migration mode and is used to validate certain migration options as per target db version [default is 9.5 for EnterpriseDB database].

Database Connection Information:
The application will read the connectivity information for the source and target database servers from toolkit.properties file.
Refer to MTK readme document for more information.

-bash-4.1$
```

#### 3.2. Schema Only Migration

```
-bash-4.1$ ./runMTK.sh -schemaOnly soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:XE
user =soe
password=******
Target database connectivity info...
conn =jdbc:edb://localhost:5444/mig
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Connecting with target EnterpriseDB database server...
Connected to EnterpriseDB, version '9.5.0.5'
Importing redwood schema soe...
Creating Schema...soe
Creating Sequence: ADDRESS_SEQ
Creating Sequence: CARD_DETAILS_SEQ
Creating Sequence: CUSTOMER_SEQ
Creating Sequence: LOGON_SEQ
Creating Sequence: ORDERS_SEQ
Creating Tables...
Creating Table: ADDRESSES
Creating Table: CARD_DETAILS
Creating Table: CUSTOMERS
Creating Table: INVENTORIES
Creating Table: LOGON
Creating Table: ORDERENTRY_METADATA
Creating Table: ORDERS
Creating Table: ORDER_ITEMS
Creating Table: PRODUCT_DESCRIPTIONS
Creating Table: PRODUCT_INFORMATION
Creating Table: WAREHOUSES
Created 11 tables.
Creating Constraint: CUSTOMERS_PK
Creating Constraint: ADDRESS_PK
Creating Constraint: CARD_DETAILS_PK
Creating Constraint: WAREHOUSES_PK
Creating Constraint: ORDER_ITEMS_PK
Creating Constraint: ORDER_PK
Creating Constraint: PRODUCT_INFORMATION_PK
Creating Constraint: PRODUCT_DESCRIPTIONS_PK
Creating Constraint: INVENTORY_PK
Creating Constraint: PRODUCT_STATUS_LOV
Creating Constraint: ORDER_ITEMS_ORDER_ID_FK
Creating Constraint: ORDER_ITEMS_PRODUCT_ID_FK
Creating Constraint: ORDERS_CUSTOMER_ID_FK
Creating Constraint: ORDER_MODE_LOV
Creating Constraint: ORDER_TOTAL_MIN
Creating Constraint: INVENTORIES_WAREHOUSES_FK
Creating Constraint: INVENTORIES_PRODUCT_ID_FK
Creating Constraint: CUSTOMER_CREDIT_LIMIT_MAX
Creating Constraint: CUSTOMER_ID_MIN
Creating Constraint: ADD_CUST_FK
Creating Index: ADDRESS_CUST_IX
Creating Index: CARDDETAILS_CUST_IX
Creating Index: CUST_ACCOUNT_MANAGER_IX
Creating Index: CUST_DOB_IX
Creating Index: CUST_EMAIL_IX
Creating Index: CUST_FUNC_LOWER_NAME_IX
Creating Index: INV_PRODUCT_IX
Creating Index: INV_WAREHOUSE_IX
Creating Index: ORD_SALES_REP_IX
Creating Index: ORD_CUSTOMER_IX
Creating Index: ORD_ORDER_DATE_IX
Creating Index: ORD_WAREHOUSE_IX
Creating Index: ITEM_ORDER_IX
Creating Index: ITEM_PRODUCT_IX
Creating Index: PRD_DESC_PK
Creating Index: PROD_NAME_IX
Creating Index: PROD_SUPPLIER_IX
Creating Index: PROD_CATEGORY_IX
Creating Index: WHS_LOCATION_IX
Creating View: PRODUCT_PRICES
Creating View: PRODUCTS
MTK-15008: Error Creating View: PRODUCTS
DB-42601: ERROR: syntax error at or near "USING" at position 351
-- CREATE VIEW PRODUCTS (PRODUCT_ID,LANGUAGE_ID,PRODUCT_NAME,CATEGORY_ID,PRODUCT_DESCRIPTION,WEIGHT_CLASS,WARRANTY_PERIOD,SUPPLIER_ID,PRODUCT_STATUS,LIST_PRICE,MIN_PRICE,CATALOG_URL)  AS
-- SELECT i.product_id
-- ,      d.language_id
-- ,      CASE WHEN d.language_id IS NOT NULL
--             THEN d.translated_name
-- Line 6:             ELSE TRANSLATE(i.product_name USING NCHAR_CS)
--                                                   ^

Creating Package: ORDERENTRY
MTK-16003: Error Creating Package Spec ORDERENTRY
DB-42601: ERROR: syntax error at or near "CR" at position 1
-- Line 1: CR;
--         ^

MTK-16004: Error Creating Package Body ORDERENTRY
DB-3F000: com.edb.util.PSQLException: ERROR: package "orderentry" does not exist

Schema soe imported with errors.

java.sql.SQLSyntaxErrorException: ORA-00942: table or view does not exist

	at oracle.jdbc.driver.T4CTTIoer.processError(T4CTTIoer.java:447)
	at oracle.jdbc.driver.T4CTTIoer.processError(T4CTTIoer.java:396)
	at oracle.jdbc.driver.T4C8Oall.processError(T4C8Oall.java:951)
	at oracle.jdbc.driver.T4CTTIfun.receive(T4CTTIfun.java:513)
	at oracle.jdbc.driver.T4CTTIfun.doRPC(T4CTTIfun.java:227)
	at oracle.jdbc.driver.T4C8Oall.doOALL(T4C8Oall.java:531)
	at oracle.jdbc.driver.T4CPreparedStatement.doOall8(T4CPreparedStatement.java:208)
	at oracle.jdbc.driver.T4CPreparedStatement.executeForDescribe(T4CPreparedStatement.java:886)
	at oracle.jdbc.driver.OracleStatement.executeMaybeDescribe(OracleStatement.java:1175)
	at oracle.jdbc.driver.OracleStatement.doExecuteWithTimeout(OracleStatement.java:1296)
	at oracle.jdbc.driver.OraclePreparedStatement.executeInternal(OraclePreparedStatement.java:3613)
	at oracle.jdbc.driver.OraclePreparedStatement.executeQuery(OraclePreparedStatement.java:3657)
	at oracle.jdbc.driver.OraclePreparedStatementWrapper.executeQuery(OraclePreparedStatementWrapper.java:1495)
	at com.edb.dbhandler.redwood.MetaData.addUserReferencedProfileNames(MetaData.java:2349)
	at com.edb.dbhandler.redwood.MetaData.initProfileStatement(MetaData.java:2289)
	at com.edb.common.MTKMetaData.getProfiles(MTKMetaData.java:581)
	at com.edb.MigrationToolkit.migrateProfiles(MigrationToolkit.java:3525)
	at com.edb.MigrationToolkit.main(MigrationToolkit.java:1969)
MTK-12001: The user/role migration failed due to insufficient privileges.
Grant the user SELECT privilege on the following Oracle catalogs:
DBA_ROLES
DBA_USERS
DBA_TAB_PRIVS
DBA_PROFILES
DBA_ROLE_PRIVS
ROLE_ROLE_PRIVS
DBA_SYS_PRIVS

One or more schema objects could not be imported during the migration process. Please review the migration output for more details.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs

******************** Migration Summary ********************
Sequences: 5 out of 5
Tables: 11 out of 11
Constraints: 20 out of 20
Indexes: 19 out of 19
Views: 1 out of 2
Packages: 0 out of 1

Total objects: 59
Successful count: 56
Failed count: 2
Invalid count: 0

List of failed objects
======================
Views
--------------------
1. SOE.PRODUCTS

Packages
--------------------
1. SOE.ORDERENTRY


*************************************************************
-bash-4.1$
```

#### 3.3. Schema Only Migration with Offline Option

```
-bash-4.1$ mkdir -p ~/work/soe
-bash-4.1$
-bash-4.1$ ./runMTK.sh -schemaOnly -offlineMigration ~/work/soe/ soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:xe
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Importing redwood schema soe...
Creating Sequence: ADDRESS_SEQ
Creating Sequence: CARD_DETAILS_SEQ
Creating Sequence: CUSTOMER_SEQ
Creating Sequence: LOGON_SEQ
Creating Sequence: ORDERS_SEQ
Creating Tables...
Creating Table: ADDRESSES
Creating Table: CARD_DETAILS
Creating Table: CUSTOMERS
Creating Table: INVENTORIES
Creating Table: LOGON
Creating Table: ORDERENTRY_METADATA
Creating Table: ORDERS
Creating Table: ORDER_ITEMS
Creating Table: PRODUCT_DESCRIPTIONS
Creating Table: PRODUCT_INFORMATION
Creating Table: WAREHOUSES
Created 11 tables.
Creating Constraint: INVENTORY_PK
Creating Constraint: PRODUCT_DESCRIPTIONS_PK
Creating Constraint: PRODUCT_INFORMATION_PK
Creating Constraint: ORDER_PK
Creating Constraint: ORDER_ITEMS_PK
Creating Constraint: WAREHOUSES_PK
Creating Constraint: CARD_DETAILS_PK
Creating Constraint: CUSTOMERS_PK
Creating Constraint: ADDRESS_PK
Creating Constraint: ORDER_TOTAL_MIN
Creating Constraint: ORDER_MODE_LOV
Creating Constraint: CUSTOMER_ID_MIN
Creating Constraint: CUSTOMER_CREDIT_LIMIT_MAX
Creating Constraint: PRODUCT_STATUS_LOV
Creating Constraint: ORDER_ITEMS_PRODUCT_ID_FK
Creating Constraint: INVENTORIES_PRODUCT_ID_FK
Creating Constraint: ORDER_ITEMS_ORDER_ID_FK
Creating Constraint: INVENTORIES_WAREHOUSES_FK
Creating Constraint: ORDERS_CUSTOMER_ID_FK
Creating Constraint: ADD_CUST_FK
Creating Index: ADDRESS_CUST_IX
Creating Index: CARDDETAILS_CUST_IX
Creating Index: CUST_ACCOUNT_MANAGER_IX
Creating Index: CUST_DOB_IX
Creating Index: CUST_EMAIL_IX
Creating Index: CUST_FUNC_LOWER_NAME_IX
Creating Index: INV_PRODUCT_IX
Creating Index: INV_WAREHOUSE_IX
Creating Index: ORD_SALES_REP_IX
Creating Index: ORD_CUSTOMER_IX
Creating Index: ORD_ORDER_DATE_IX
Creating Index: ORD_WAREHOUSE_IX
Creating Index: ITEM_ORDER_IX
Creating Index: ITEM_PRODUCT_IX
Creating Index: PRD_DESC_PK
Creating Index: PROD_NAME_IX
Creating Index: PROD_SUPPLIER_IX
Creating Index: PROD_CATEGORY_IX
Creating Index: WHS_LOCATION_IX
Creating View: PRODUCT_PRICES
Creating View: PRODUCTS
Creating Package: ORDERENTRY

Schema soe imported successfully.

MTK-12001: The user/role migration failed due to insufficient privileges.
Grant the user SELECT privilege on the following Oracle catalogs:
DBA_ROLES
DBA_USERS
DBA_TAB_PRIVS
DBA_PROFILES
DBA_ROLE_PRIVS
ROLE_ROLE_PRIVS
DBA_SYS_PRIVS

One or more schema objects could not be imported during the migration process. Please review the migration output for more details.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs
-bash-4.1$
-bash-4.1$
```

##### 3.3.1. Grant select any dictionary to soe;

```
[oracle@ppas ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.2.0 Production on Mon Feb 8 22:07:36 2016

Copyright (c) 1982, 2011, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL>
SQL>
SQL> grant select any dictionary to soe;

Grant succeeded.

SQL>
```

##### 3.3.2. Re-run MTK

```
-bash-4.1$
-bash-4.1$ ./runMTK.sh -schemaOnly -offlineMigration ~/work/soe/ soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:xe
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Importing redwood schema soe...
Creating Sequence: ADDRESS_SEQ
Creating Sequence: CARD_DETAILS_SEQ
Creating Sequence: CUSTOMER_SEQ
Creating Sequence: LOGON_SEQ
Creating Sequence: ORDERS_SEQ
Creating Tables...
Creating Table: ADDRESSES
Creating Table: CARD_DETAILS
Creating Table: CUSTOMERS
Creating Table: INVENTORIES
Creating Table: LOGON
Creating Table: ORDERENTRY_METADATA
Creating Table: ORDERS
Creating Table: ORDER_ITEMS
Creating Table: PRODUCT_DESCRIPTIONS
Creating Table: PRODUCT_INFORMATION
Creating Table: WAREHOUSES
Created 11 tables.
Creating Constraint: INVENTORY_PK
Creating Constraint: PRODUCT_DESCRIPTIONS_PK
Creating Constraint: PRODUCT_INFORMATION_PK
Creating Constraint: ORDER_PK
Creating Constraint: ORDER_ITEMS_PK
Creating Constraint: WAREHOUSES_PK
Creating Constraint: CARD_DETAILS_PK
Creating Constraint: CUSTOMERS_PK
Creating Constraint: ADDRESS_PK
Creating Constraint: ORDER_TOTAL_MIN
Creating Constraint: ORDER_MODE_LOV
Creating Constraint: CUSTOMER_ID_MIN
Creating Constraint: CUSTOMER_CREDIT_LIMIT_MAX
Creating Constraint: PRODUCT_STATUS_LOV
Creating Constraint: ORDER_ITEMS_PRODUCT_ID_FK
Creating Constraint: INVENTORIES_PRODUCT_ID_FK
Creating Constraint: ORDER_ITEMS_ORDER_ID_FK
Creating Constraint: INVENTORIES_WAREHOUSES_FK
Creating Constraint: ORDERS_CUSTOMER_ID_FK
Creating Constraint: ADD_CUST_FK
Creating Index: ADDRESS_CUST_IX
Creating Index: CARDDETAILS_CUST_IX
Creating Index: CUST_ACCOUNT_MANAGER_IX
Creating Index: CUST_DOB_IX
Creating Index: CUST_EMAIL_IX
Creating Index: CUST_FUNC_LOWER_NAME_IX
Creating Index: INV_PRODUCT_IX
Creating Index: INV_WAREHOUSE_IX
Creating Index: ORD_SALES_REP_IX
Creating Index: ORD_CUSTOMER_IX
Creating Index: ORD_ORDER_DATE_IX
Creating Index: ORD_WAREHOUSE_IX
Creating Index: ITEM_ORDER_IX
Creating Index: ITEM_PRODUCT_IX
Creating Index: PRD_DESC_PK
Creating Index: PROD_NAME_IX
Creating Index: PROD_SUPPLIER_IX
Creating Index: PROD_CATEGORY_IX
Creating Index: WHS_LOCATION_IX
Creating View: PRODUCT_PRICES
Creating View: PRODUCTS
Creating Package: ORDERENTRY

Schema soe imported successfully.

Creating User: SOE

Migration process completed successfully.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs
-bash-4.1$
-bash-4.1$ ll ~/work/soe/
total 140
-rw-rw-r--. 1 enterprisedb enterprisedb  2392 Feb  8 22:09 mtk_soe_constraint_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb 59193 Feb  8 22:10 mtk_soe_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb  1355 Feb  8 22:09 mtk_soe_index_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb 50513 Feb  8 22:10 mtk_soe_package_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb    23 Feb  8 22:09 mtk_soe_schema_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb   660 Feb  8 22:09 mtk_soe_sequence_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb  3184 Feb  8 22:09 mtk_soe_table_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb  1171 Feb  8 22:09 mtk_soe_view_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb    61 Feb  8 22:10 mtk_user_ddl.sql
```

#### 3.4. 비호환 구문 Workaround 적용

##### 3.4.1. SOE.PRODUCTS Views

```
-bash-4.1$
-bash-4.1$ cd ~/work/soe/
-bash-4.1$
-bash-4.1$ vi mtk_soe_view_ddl.sql
            ELSE TRANSLATE(i.product_name USING NCHAR_CS)
    ==>
            ELSE i.product_description

            ELSE TRANSLATE(i.product_description USING NCHAR_CS)
    ==>
            ELSE i.product_description
```

##### 3.4.2. SOE.ORDERENTRY Packages

###### CONNECT BY (Line 1200)

```
-bash-4.1$ vi mtk_soe_package_ddl.sql
    cursor c1 is WITH stage1 AS -- get 12 rows of 5mins
      (SELECT
        /*+ materialize CARDINALITY(12) */
        (rownum*(1/288)) offset
      FROM dual
        CONNECT BY rownum <= 12
      ),
    ==>
    cursor c1 is WITH stage1 AS -- get 12 rows of 5mins
      (SELECT (x*(1/288)) offsets from generate_series(1, 12) x),
```

###### Reserved keyword - offset

```
      (SELECT
        /*+ materialize CARDINALITY(12) */
        lag(offset, 1, 0) over (order by rownum) ostart,
        offset oend
      FROM stage1
      ),
    ==>
      (SELECT
        /*+ materialize CARDINALITY(12) */
        lag(offsets, 1, 0) over (order by rownum) ostart,
        offsets oend
      FROM stage1
      ),
```

#### 3.5. Executing Offline Migration Scripts

##### 3.5.1. Overall Steps

```
1.	Create the database which we will restore the migrated database objects:
-bash-4.1$ createdb -U enterprisedb mig
or
edb=# create database mig;

2.	Create schema authorization
-bash-4.1$ psql mig
Password:
psql.bin (9.5.0.5)
Type "help" for help.

mig=# create schema authorization soe;
CREATE SCHEMA
mig=#
mig=# \dn
    List of schemas
  Name  |    Owner
--------+--------------
 public | enterprisedb
 soe    | soe
(2 rows)

3.	Connect to the new database with psql:
-bash-4.1$ psql -U soe mig

4.	Use the \i meta-command to invoke the migration script that creates the object definitions:
mig=# \i ./mtk_soe_ddl.sql
```

##### 3.5.2. 실습 환경 Steps

```
1. Connect to mig DB using enterprisedb user
2. Drop schema soe
3. Create schema soe
4. Connect mig DB using soe user
5. Run DDL scripts with psql CLI
  a. Sequence
  b. Table
  c. Constraints
  d. Index
  e. View
  f. Package
```  

```
-bash-4.1$ psql -U enterprisedb mig
Password for user enterprisedb:
psql.bin (9.5.0.5)
Type "help" for help.

mig=>
mig=> drop schema soe cascade;
NOTICE:  drop cascades to 17 other objects
DETAIL:  drop cascades to sequence address_seq
drop cascades to sequence card_details_seq
drop cascades to sequence customer_seq
drop cascades to sequence logon_seq
drop cascades to sequence orders_seq
drop cascades to table addresses
drop cascades to table card_details
drop cascades to table customers
drop cascades to table inventories
drop cascades to table logon
drop cascades to table orderentry_metadata
drop cascades to table orders
drop cascades to table order_items
drop cascades to table product_descriptions
drop cascades to table product_information
drop cascades to table warehouses
drop cascades to view product_prices
DROP SCHEMA
mig=>
mig=# create schema authorization soe;
CREATE SCHEMA
mig=#
mig=# \c mig soe
Password for user soe:
You are now connected to database "mig" as user "soe".
mig=>
mig=> \i mtk_soe_sequence_ddl.sql
mig=>
mig=> \i mtk_soe_table_ddl.sql
mig=>
mig=> \i mtk_soe_constraint_ddl.sql
mig=>
mig=> \i mtk_soe_index_ddl.sql
mig=>
mig=> \i mtk_soe_view_ddl.sql
mig=>
mig=> \i mtk_soe_package_ddl.sql
mig=>
```

## 4. Data Migration

#### 4.0. Initialization

```
1. Drop soe schema
2. Create soe schema
3. Create table only (for data loading purpose)
```

###### Initialize

```
-bash-4.1$ psql -U enterprisedb mig
Password for user enterprisedb:
psql.bin (9.5.0.5)
Type "help" for help.

mig=>
mig=> drop schema soe cascade;
NOTICE:  drop cascades to 17 other objects
DETAIL:  drop cascades to sequence address_seq
drop cascades to sequence card_details_seq
drop cascades to sequence customer_seq
drop cascades to sequence logon_seq
drop cascades to sequence orders_seq
drop cascades to table addresses
drop cascades to table card_details
drop cascades to table customers
drop cascades to table inventories
drop cascades to table logon
drop cascades to table orderentry_metadata
drop cascades to table orders
drop cascades to table order_items
drop cascades to table product_descriptions
drop cascades to table product_information
drop cascades to table warehouses
drop cascades to view product_prices
DROP SCHEMA
mig=>
mig=# create schema authorization soe;
CREATE SCHEMA
mig=#
mig=# \c mig soe
Password for user soe:
You are now connected to database "mig" as user "soe".
mig=>
mig=> \i mtk_soe_table_ddl.sql
SET
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
mig=>
mig=> \dt
               List of relations
 Schema |         Name         | Type  | Owner
--------+----------------------+-------+-------
 soe    | addresses            | table | soe
 soe    | card_details         | table | soe
 soe    | customers            | table | soe
 soe    | inventories          | table | soe
 soe    | logon                | table | soe
 soe    | order_items          | table | soe
 soe    | orderentry_metadata  | table | soe
 soe    | orders               | table | soe
 soe    | product_descriptions | table | soe
 soe    | product_information  | table | soe
 soe    | warehouses           | table | soe
(11 rows)

mig=>
```

###### 데이터 이관 대상 선정 - Oracle Segment Size 확인

```
SQL> select segment_name, sum(bytes/1024/1024) mb from dba_segments
  2  where owner='SOE' and segment_type='TABLE'
  3  group by segment_name
  4  order by 2;

SEGMENT_NAME			       MB
------------------------------ ----------
PRODUCT_DESCRIPTIONS            1
WAREHOUSES                      1
ORDERENTRY_METADATA             1
PRODUCT_INFORMATION             1
LOGON                          32
CARD_DETAILS                   39
CUSTOMERS                      66
ADDRESSES                      67
ORDERS                         78
ORDER_ITEMS                   138
INVENTORIES                   176

11 rows selected.

SQL>
SQL> select table_name, num_rows from dba_tables
  2  where owner='SOE'
  3  order by 2;

TABLE_NAME                      NUM_ROWS
------------------------------ ----------
ORDERENTRY_METADATA                    4
PRODUCT_DESCRIPTIONS                1000
PRODUCT_INFORMATION                 1000
WAREHOUSES                          1000
CUSTOMERS                         500000
ORDERS                            714895
CARD_DETAILS                      750000
ADDRESSES                         750000
INVENTORIES                       901894
LOGON                            1191492
ORDER_ITEMS                      2145019

11 rows selected.

```

#### 4.1. Data Only Migration

##### 4.1.1. Data Only without option (JDBC)

###### Single Table Migration

```
-bash-4.1$ ./runMTK.sh -dataOnly -tables CUSTOMERS soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:XE
user =soe
password=******
Target database connectivity info...
conn =jdbc:edb://localhost:5444/mig
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Connecting with target EnterpriseDB database server...
Connected to EnterpriseDB, version '9.5.0.5'
Importing redwood schema soe...
Table List: 'CUSTOMERS'
Loading Table Data in 8 MB batches...
Loading Table: CUSTOMERS ...
[CUSTOMERS] Migrated 58521 rows.
[CUSTOMERS] Migrated 116865 rows.
[CUSTOMERS] Migrated 174914 rows.
[CUSTOMERS] Migrated 232964 rows.
[CUSTOMERS] Migrated 291041 rows.
[CUSTOMERS] Migrated 349090 rows.
[CUSTOMERS] Migrated 407111 rows.
[CUSTOMERS] Migrated 465140 rows.
[CUSTOMERS] Migrated 500000 rows.
[CUSTOMERS] Table Data Load Summary: Total Time(s): 16.804 Total Rows: 500000 Total Size(MB): 68.7958984375
Data Load Summary: Total Time (sec): 16.877 Total Rows: 500000 Total Size(MB): 70.447

Schema soe imported successfully.


Migration process completed successfully.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs

******************** Migration Summary ********************
Tables: 1 out of 1

Total objects: 2
Successful count: 1
Failed count: 0
Invalid count: 1

List of invalid objects
=======================
1. SOE.ORDERENTRY (PACKAGE)

*************************************************************
-bash-4.1$
```
###### Multiple Tables Migration
```
-bash-4.1$ ./runMTK.sh -dataOnly -tables PRODUCT_DESCRIPTIONS,PRODUCT_INFORMATION soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:XE
user =soe
password=******
Target database connectivity info...
conn =jdbc:edb://localhost:5444/mig
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Connecting with target EnterpriseDB database server...
Connected to EnterpriseDB, version '9.5.0.5'
Importing redwood schema soe...
Table List: 'PRODUCT_DESCRIPTIONS','PRODUCT_INFORMATION'
Loading Table Data in 8 MB batches...
Loading Table: PRODUCT_DESCRIPTIONS ...
[PRODUCT_DESCRIPTIONS] Migrated 1000 rows.
[PRODUCT_DESCRIPTIONS] Table Data Load Summary: Total Time(s): 0.219 Total Rows: 1000 Total Size(MB): 0.1064453125
Loading Table: PRODUCT_INFORMATION ...
[PRODUCT_INFORMATION] Migrated 1000 rows.
[PRODUCT_INFORMATION] Table Data Load Summary: Total Time(s): 0.205 Total Rows: 1000 Total Size(MB): 0.2080078125
Data Load Summary: Total Time (sec): 0.476 Total Rows: 2000 Total Size(MB): 0.322

Schema soe imported successfully.


Migration process completed successfully.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs

******************** Migration Summary ********************
Tables: 2 out of 2

Total objects: 3
Successful count: 2
Failed count: 0
Invalid count: 1

List of invalid objects
=======================
1. SOE.ORDERENTRY (PACKAGE)

*************************************************************
-bash-4.1$
```

##### 4.1.1. Data Only with Various Options - fastCopy, copyViaDBLinkOra (OCI)

###### Single Table Migration

* 주의사항 : copyViaDBLinkOra 옵션 사용시 EDB Postgres 스키마 유저는 반드시 SUPERUSER 권한이 있어야함!

```
-bash-4.1$ ./runMTK.sh -dataOnly -tables CUSTOMERS -truncLoad -fastCopy -copyViaDBLinkOra soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:XE
user =soe
password=******
Target database connectivity info...
conn =jdbc:edb://localhost:5444/mig
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Connecting with target EnterpriseDB database server...
Connected to EnterpriseDB, version '9.5.0.5'
Importing redwood schema soe...
Table List: 'CUSTOMERS'
MTK-11014: Error connecting to DBLinkOra
DB-42501: com.edb.util.PSQLException: ERROR: must be superuser to use dblink_ora

One or more schema objects could not be imported during the migration process. Please review the migration output for more details.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs

******************** Migration Summary ********************

Total objects: 0
Successful count: 0
Failed count: 0
Invalid count: 0

*************************************************************
-bash-4.1$
-bash-4.1$ psql -c "alter user soe superuser"
ALTER ROLE
-bash-4.1$
-bash-4.1$ ./runMTK.sh -dataOnly -tables CUSTOMERS -truncLoad -fastCopy -copyViaDBLinkOra soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:XE
user =soe
password=******
Target database connectivity info...
conn =jdbc:edb://localhost:5444/mig
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Connecting with target EnterpriseDB database server...
Connected to EnterpriseDB, version '9.5.0.5'
Importing redwood schema soe...
Table List: 'CUSTOMERS'
Disabling FK constraints & triggers on soe.customers before truncate...
Truncating table CUSTOMERS before data load...
Disabling indexes on soe.customers before data load...
Loading Table: CUSTOMERS ...
Migrated 5000 rows.
Migrated 10000 rows.
Migrated 15000 rows.
Migrated 20000 rows.
Migrated 25000 rows.
...
...
Migrated 495000 rows.
Migrated 500000 rows.
Migrated 500000 rows.
[CUSTOMERS] Table Data Load Summary: Total Time(s): 5.982 Total Rows: 500000
Enabling FK constraints & triggers on soe.customers...
Enabling indexes on soe.customers after data load...
Data Load Summary: Total Time (sec): 6.161 Total Rows: 500000 Total Size(MB): 0.0

Schema soe imported successfully.


Migration process completed successfully.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs

******************** Migration Summary ********************
Tables: 1 out of 1

Total objects: 2
Successful count: 1
Failed count: 0
Invalid count: 1

List of invalid objects
=======================
1. SOE.ORDERENTRY (PACKAGE)

*************************************************************
-bash-4.1$
```

###### Multiple Tables Migration

```
-bash-4.1$ ./runMTK.sh -dataOnly -tables CARD_DETAILS,ADDRESSES -loaderCount 2 -fastCopy -copyViaDBLinkOra soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:XE
user =soe
password=******
Target database connectivity info...
conn =jdbc:edb://localhost:5444/mig
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Connecting with target EnterpriseDB database server...
Connected to EnterpriseDB, version '9.5.0.5'
Importing redwood schema soe...
Table List: 'CARD_DETAILS','ADDRESSES'
Loading Table: ADDRESSES ...
Loading Table: CARD_DETAILS ...
Migrated 5000 rows.
Migrated 10000 rows.
Migrated 5000 rows.
Migrated 15000 rows.
Migrated 10000 rows.
Migrated 20000 rows.
Migrated 15000 rows.
Migrated 25000 rows.
Migrated 20000 rows.
Migrated 30000 rows.
...
Migrated 670000 rows.
Migrated 750000 rows.
Migrated 750000 rows.
[CARD_DETAILS] Table Data Load Summary: Total Time(s): 12.407 Total Rows: 750000
Migrated 675000 rows.
Migrated 680000 rows.
...
Migrated 745000 rows.
Migrated 750000 rows.
Migrated 750000 rows.
[ADDRESSES] Table Data Load Summary: Total Time(s): 13.12 Total Rows: 750000
Data Load Summary: Total Time (sec): 13.359 Total Rows: 1500000 Total Size(MB): 0.0

Schema soe imported successfully.


Migration process completed successfully.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs

******************** Migration Summary ********************
Tables: 2 out of 2

Total objects: 3
Successful count: 2
Failed count: 0
Invalid count: 1

List of invalid objects
=======================
1. SOE.ORDERENTRY (PACKAGE)

*************************************************************
-bash-4.1$
```

#### 4.2. Data Only Migration with Offline option

###### Copy Command Type

```
-bash-4.1$ ./runMTK.sh -dataOnly -offlineMigration ~/work/soe/ -tables WAREHOUSES,ORDERENTRY_METADATA soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:xe
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Importing redwood schema soe...
Table List: 'WAREHOUSES','ORDERENTRY_METADATA'
Loading Table Data in 8 MB batches...
Loading Table: ORDERENTRY_METADATA ...
[ORDERENTRY_METADATA] Migrated 4 rows.
[ORDERENTRY_METADATA] Table Data Load Summary: Total Time(s): 0.026 Total Rows: 4
Loading Table: WAREHOUSES ...
[WAREHOUSES] Migrated 1000 rows.
[WAREHOUSES] Table Data Load Summary: Total Time(s): 0.144 Total Rows: 1000 Total Size(MB): 0.0263671875
Data Load Summary: Total Time (sec): 0.202 Total Rows: 1004 Total Size(MB): 0.027

Schema soe imported successfully.


Migration process completed successfully.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs
-bash-4.1$
-bash-4.1$ ls -al ~/work/soe/mtk_*.cpy
-rw-rw-r--. 1 enterprisedb enterprisedb    94 Jan 31 19:17 /opt/PostgresPlus/9.5AS/work/soe/mtk_soe_orderentry_metadata_data.cpy
-rw-rw-r--. 1 enterprisedb enterprisedb 27470 Jan 31 19:17 /opt/PostgresPlus/9.5AS/work/soe/mtk_soe_warehouses_data.cpy

-bash-4.1$
-bash-4.1$ psql mig soe
Password for user soe:
psql.bin (9.5.0.5)
Type "help" for help.

mig=#
mig=# copy warehouses from '/opt/PostgresPlus/9.5AS/work/soe/mtk_soe_warehouses_data.cpy';
COPY 1000
mig=#
mig=# copy orderentry_metadata from '/opt/PostgresPlus/9.5AS/work/soe/mtk_soe_orderentry_metadata_data.cpy';
COPY 4
mig=#
```

###### INSERT Statement Type

```
-bash-4.1$ ./runMTK.sh -dataOnly -offlineMigration ~/work/soe/ -tables WAREHOUSES,ORDERENTRY_METADATA -singleDataFile soe
MTK-10036:Cannot use singleDataFile option for offline data migration in COPY format. Use -safeMode option to store plain SQL statements in a single file.
-bash-4.1$
-bash-4.1$
-bash-4.1$ ./runMTK.sh -dataOnly -offlineMigration ~/work/soe/ -tables WAREHOUSES,ORDERENTRY_METADATA -singleDataFile -safeMode soe
Running EnterpriseDB Migration Toolkit (Build 49.0.3) ...
Source database connectivity info...
conn =jdbc:oracle:thin:@localhost:1521:xe
user =soe
password=******
Connecting with source Oracle database server...
Connected to Oracle, version 'Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production'
Importing redwood schema soe...
Table List: 'WAREHOUSES','ORDERENTRY_METADATA'
Loading table data in safe mode...
Loading Table: ORDERENTRY_METADATA ...
[ORDERENTRY_METADATA] Table Data Load Summary: Total Time(s): 0.004 Total Rows: 4
Loading Table: WAREHOUSES ...
[WAREHOUSES] Table Data Load Summary: Total Time(s): 0.086 Total Rows: 1000
Data Load Summary: Total Time (sec): 0.09 Total Rows: 1004 Total Size(MB): 0.0

Schema soe imported successfully.


Migration process completed successfully.

Migration logs have been saved to /opt/PostgresPlus/9.5AS/.enterprisedb/migration-toolkit/logs
-bash-4.1$ ls -ltr ~/work/soe/
total 176
-rw-rw-r--. 1 enterprisedb enterprisedb   277 Jan 31 18:52 mtk_soe_sequence_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb  5098 Jan 31 18:52 mtk_soe_table_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb  2098 Jan 31 18:52 mtk_soe_constraint_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb  1228 Jan 31 18:52 mtk_soe_index_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb  1229 Jan 31 18:52 mtk_soe_view_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb 34708 Jan 31 18:52 mtk_soe_package_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb    30 Jan 31 18:52 mtk_user_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb    94 Jan 31 19:17 mtk_soe_orderentry_metadata_data.cpy
-rw-rw-r--. 1 enterprisedb enterprisedb 27470 Jan 31 19:17 mtk_soe_warehouses_data.cpy
-rw-rw-r--. 1 enterprisedb enterprisedb    23 Jan 31 19:19 mtk_soe_schema_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb    44 Jan 31 19:19 mtk_soe_ddl.sql
-rw-rw-r--. 1 enterprisedb enterprisedb 70772 Jan 31 19:19 mtk_soe_data.sql
-bash-4.1$
-bash-4.1$ vi ~/work/soe/mtk_soe_data.sql
INSERT INTO soe.ORDERENTRY_METADATA VALUES ('SOE_MIN_ORDER_ID' , '1');
INSERT INTO soe.ORDERENTRY_METADATA VALUES ('SOE_MAX_ORDER_ID' , '9000000');
INSERT INTO soe.ORDERENTRY_METADATA VALUES ('SOE_MIN_CUSTOMER_ID' , '1');
INSERT INTO soe.ORDERENTRY_METADATA VALUES ('SOE_MAX_CUSTOMER_ID' , '8000000');
INSERT INTO soe.WAREHOUSES VALUES (1 , 'RsLwJsjjTUi3OLoQo9LY9LTI' , 168);
INSERT INTO soe.WAREHOUSES VALUES (2 , 'FPk7oux7SVMRKHeefzEqfrhRc7v' , 6049);
INSERT INTO soe.WAREHOUSES VALUES (3 , 'S918FD3qdUeQTJAOyrN54YDV' , 6218);
INSERT INTO soe.WAREHOUSES VALUES (4 , 'OTX4Sk' , 5078);
INSERT INTO soe.WAREHOUSES VALUES (5 , 'J1zRQzCWGieTbXD5H' , 6289);
INSERT INTO soe.WAREHOUSES VALUES (6 , 'kkID8OzR3yPUxfH6o1Hgrgv' , 4602);
INSERT INTO soe.WAREHOUSES VALUES (7 , 'eyf6BcauJrXJ' , 9869);
INSERT INTO soe.WAREHOUSES VALUES (8 , 'Bemln8H9' , 5615);
INSERT INTO soe.WAREHOUSES VALUES (9 , 'qg2c130mm1QyfZa9' , 9795);
INSERT INTO soe.WAREHOUSES VALUES (10 , 'EA' , 7352);
~~~

-bash-4.1$ psql mig soe
Password for user soe:
psql.bin (9.5.0.5)
Type "help" for help.

mig=# truncate table warehouses;
TRUNCATE TABLE
mig=# truncate table orderentry_metadata;
TRUNCATE TABLE
mig=#
mig=# \i ~/work/soe/mtk_soe_data.sql
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
INSERT 0 1
...
mig=#
mig=# select count(*) from warehouses ;
 count
-------
  1000
(1 row)

mig=# select count(*) from orderentry_metadata ;
 count
-------
     4
(1 row)

```

#### 4.3. Using DB Link

```
-bash-4.1$
-bash-4.1$ psql mig soe
Password for user soe:
psql.bin (9.5.0.5)
Type "help" for help.

mig=# create database link soelink connect to soe identified by 'soe' using '//localhost:1521/XE';
CREATE DATABASE LINK
mig=#
mig=# select * from edb_dblink ;
 lnkname | lnkowner | lnktype | lnkispublic | lnkuser | lnkpassword |     lnkconnstr      | lnkacl |  oid
---------+----------+---------+-------------+---------+-------------+---------------------+--------+-------
 soelink |    16875 |       0 | f           | soe     | ********    | //localhost:1521/XE |        | 17791
(1 row)

mig=#
mig=# truncate table customers ;
TRUNCATE TABLE
mig=#
mig=# \timing
Timing is on.
mig=#
mig=# insert into customers select * from customers@soelink;
INSERT 0 500000
Time: 52229.318 ms
mig=#
```

#### 4.4. Using dblink_ora Function

##### 4.4.1. dblink_ora Function Overview

```
-bash-4.1$ psql mig soe
Password for user soe:
psql.bin (9.5.0.5)
Type "help" for help.

mig=# \df dblink_ora*
                                                List of functions
   Schema   |         Name          | Result data type |              Argument data types               |  Type
------------+-----------------------+------------------+------------------------------------------------+--------
 pg_catalog | dblink_ora_call       | SETOF record     | text, text, numeric                            | normal
 pg_catalog | dblink_ora_connect    | text             | text                                           | normal
 pg_catalog | dblink_ora_connect    | text             | text, boolean                                  | normal
 pg_catalog | dblink_ora_connect    | text             | text, text, text, text, text, integer          | normal
 pg_catalog | dblink_ora_connect    | text             | text, text, text, text, text, integer, boolean | normal
 pg_catalog | dblink_ora_copy       | bigint           | text, text, text, text                         | normal
 pg_catalog | dblink_ora_copy       | bigint           | text, text, text, text, boolean                | normal
 pg_catalog | dblink_ora_copy       | bigint           | text, text, text, text, boolean, integer       | normal
 pg_catalog | dblink_ora_disconnect | text             | text                                           | normal
 pg_catalog | dblink_ora_exec       | void             | text, text                                     | normal
 pg_catalog | dblink_ora_record     | SETOF record     | text, text                                     | normal
 pg_catalog | dblink_ora_status     | text             | text                                           | normal
(12 rows)

* 참고 : /opt/PostgresPlus/9.5AS/share/contrib/dblink_ora.sql
```

#### 4.4.2. How to dblink_ora Function

```
-bash-4.1$ psql mig soe
Password for user soe:
psql.bin (9.5.0.5)
Type "help" for help.

mig=#
mig=# select dblink_ora_connect ('oralink', '127.0.0.1', 'XE', 'soe', 'soe', 1521, false);
 dblink_ora_connect
--------------------
 OK
(1 row)

mig=# \timing
Timing is on.
mig=#
mig=# select dblink_ora_copy ('oralink', 'select * from customers', 'soe', 'customers', true);
 dblink_ora_copy
-----------------
          500000
(1 row)

Time: 5015.913 ms
mig=#
mig=# select dblink_ora_disconnect ('oralink');
 dblink_ora_disconnect
-----------------------
 OK
(1 row)
```
