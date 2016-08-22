# Lab: Installation Hands-on Lab

## 1. Installation - GUI Mode
##### 1. Run install file
```
[root@ppaslab ~]# cd /opt/pkgs/
[root@ppaslab pkgs]# ls
edb_languagepack_95.bin                       instantclient-sdk-linux.x64-11.2.0.4.0.zip  pg_repack-master.zip
edb_mtk.bin                                   ojdbc6.jar                                  ppas_dbserver_as95.bin
instantclient-basic-linux.x64-11.2.0.4.0.zip  oracle_fdw-ORACLE_FDW_1_3_0.tar.gz          ppasmeta-9.5.0.5-linux-x64.tar.gz
[root@ppaslab pkgs]# tar -zxvf ppasmeta-9.5.0.5-linux-x64.tar.gz 
ppasmeta-9.5.0.5-linux-x64/
ppasmeta-9.5.0.5-linux-x64/ppasmeta-9.5.0.5-linux-x64.run
ppasmeta-9.5.0.5-linux-x64/README_FIRST_Linux64.txt
[root@ppaslab pkgs]# cd ppasmeta-9.5.0.5-linux-x64
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# ls
README_FIRST_Linux64.txt  ppasmeta-9.5.0.5-linux-x64.run
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# 
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# ./ppasmeta-9.5.0.5-linux-x64.run 
```
##### 2. Select Language
![1](./images/103_2.png)
##### 3. Error - SELinux
![1](./images/103_3.png)
##### 4. setenforce permissive
![1](./images/103_4.png)
##### 5. Welcome screen
![1](./images/103_5.png)
##### 6. Accept license terms
![1](./images/103_6.png)
##### 7. User authentication
![1](./images/103_7.png)
##### 8. Installation directory
![1](./images/103_8.png)
##### 9. Select component
![1](./images/103_9.png)
##### 10. Additional directories - Data, WAL
![1](./images/103_10.png)
##### 11. Configuration mode
![1](./images/103_11.png)
##### 12. Superuser password
![1](./images/103_12.png)
##### 13. Additional configuration - Port, Locale, Sample Schema
![1](./images/103_13.png)
##### 14. Dynatune - Server Utilization
![1](./images/103_14.png)
##### 15. Dynatune - Workload Profile
![1](./images/103_15.png)
##### 16. Advanced Configuration
![1](./images/103_16.png)
##### 17. Pre-installation summary
![1](./images/103_17.png)
##### 18. Ready to install
![1](./images/103_18.png)
##### 19. Installing
![1](./images/103_19.png)
##### 20. Completing setup and select StackBuilder run or not
![1](./images/103_20.png)
##### 21. StackBuilder wizard
![1](./images/103_21.png)
##### 22. Select component
![1](./images/103_22.png)
##### 23. User authentication
![1](./images/103_23.png)
##### 24. Download directory
![1](./images/103_24.png)
##### 25. Download directory
![1](./images/103_25.png)
##### 26. Downloading
![1](./images/103_26.png)
##### 27. Install downloaded component or not
![1](./images/103_27.png)
##### 28. Language select
![1](./images/103_28.png)
##### 29. Welcome screen
![1](./images/103_29.png)
##### 30. Accept license terms
![1](./images/103_30.png)
##### 31. User authentication
![1](./images/103_31.png)
##### 32. Existing installation
![1](./images/103_32.png)
##### 33. Ready to install
![1](./images/103_33.png)
##### 34. Completing upgrade
![1](./images/103_34.png)
##### 35. Completing StackBuilder
![1](./images/103_35.png)
##### 36. Done
```
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# ./ppasmeta-9.5.0.5-linux-x64.run 

Installing Database Server ...
 Installing pgAgent ...
 Installing Connectors ...
 Installing Migration Toolkit ...
 Installing EDB*Plus ...
 Installing Infinite Cache ...
 Installing Postgres Enterprise Manager Client ...
 Installing pgpool-II ...
 Installing pgpool-II Extensions ...
 Installing StackBuilder Plus ...
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# 
```


## 2. Related Files
#### 2.1. /etc/postgres-reg.ini
```
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# vi /etc/postgres-reg.ini

[Global_machine_id]
MachineID=5086a7bc-c9be-11e5-82d0-080027089f92
[meta_9.5]
InstallationID=50d80076-c9be-11e5-8a2b-080027089f92
InstallationDirectory=/opt/PostgresPlus
Email=jihoon.kim@enterprisedb.com
[EnterpriseDB/9.5AS]
InstallationDirectory=/opt/PostgresPlus/9.5AS
Version=9.5.0.5-1
Shortcuts=1
DataDirectory=/opt/PostgresPlus/9.5AS/data
Port=5444
ServiceID=ppas-9.5
Locale=en_US.UTF-8
Superuser=enterprisedb
Serviceaccount=enterprisedb
Description=Postgres Plus Advanced Server 9.5
Branding=Postgres Plus Advanced Server 9.5
[Global_ref_counts]
sbp_components_ref_count=10
dbserver_ref_count=1
[ppas_pgAgent_9.5AS]
Description=pgAgent is a job scheduler for PostgreSQL which may be managed using pgAdmin.
InstallationDirectory=/opt/PostgresPlus/9.5AS
Version=3.4.1-1
ServiceManager=enterprisedb
PGUSER=enterprisedb
PGHOST=localhost
PGPORT=5444
PGDATABASE=edb
[ppas_connectors]
Description=ODBC drivers for Postgres Plus Advanced Server, by EnterpriseDB.
InstallationDirectory=/opt/PostgresPlus/connectors
Version=9.5.0.5-1
[ppas_edbmtk]
InstallationID=4cc82cd0-c9bf-11e5-b470-080027089f92
Description=Migration Toolkit for PostgreSQL, by EnterpriseDB.
InstallationDirectory=/opt/PostgresPlus/edbmtk
Version=49.0.3-1
Email=jihoon.kim@enterprisedb.com
[ppas_edbplus_9.5]
Description=EDB*Plus for Postgres Plus Advanced Server, by EnterpriseDB.
InstallationDirectory=/opt/PostgresPlus/9.5AS/edbplus
Version=9.5.34.0.1-1
[ppas_infinitecache]
Description=Infinite Cache for Postgres Plus Advanced Server, by EnterpriseDB.
InstallationDirectory=/opt/PostgresPlus/infinitecache
Version=2.00-1
SystemUser=enterprisedb
[PEM/client-v6]
Description=Postgres Enterprise Manager Client, by EnterpriseDB.
InstallationDirectory=/opt/PostgresPlus/9.5AS
Version=6.0.0-2
Branding=Postgres Plus Advanced Server 9.5
InstallationDate=2016-02-03
[ppas_replication_9.5]
Description=Slony Replication is a "master to multiple slaves" replication system supporting cascading and failover.
InstallationDirectory=/opt/PostgresPlus/9.5AS
Version=2.2.4-2
SystemUser=enterprisedb
[ppas_pgpool_3.4]
Description=pgpool-II is middleware that works between PostgreSQL database servers and PostgreSQL database clients. It provides the following enterprise scaling features: connection pooling, replication, load balancing, connection limitations and parallel query.
InstallationDirectory=/opt/PostgresPlus/pgpool-II-3.4
Version=3.4.3-2
SystemUser=enterprisedb
[ppas_pgpool_3.4_extension_9.5]
Description=pgpool-II Server Extensions are installed into your database server and create a set of functions like pgpool_regclass and pgpool_recovery that are accessed by pgpool-II for certain operations.
InstallationDirectory=/opt/PostgresPlus/9.5AS
Version=3.4.3-2
[pgbouncer_1.6]
Description=Connection pooler for Postgres Plus Server, packaged by EnterpriseDB.
InstallationDirectory=/opt/PostgresPlus/pgbouncer-1.6
Version=1.6.1-2
SystemUser=enterprisedb
[StackBuilderPlus]
Description=StackBuilder Plus (An advanced application stack builder)
InstallationDirectory=/opt/PostgresPlus/stackbuilderplus
Version=2.04-1
InstallationDate=2016-02-03
Branding=Postgres Plus Add-ons
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]#
```

#### 2.2. /etc/ppas-reg_9.5.ini
```
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# vi /etc/ppas-reg_9.5.ini

[server]
databaseServer=true
[sbp]
sbp=true
[connectors]
connectors=true
[edbmtk]
edbmtk=true
[infinitecache]
infinitecache=true
[pemclient]
pemclient=true
[pgpool]
pgpool=true
[pgpool_extension]
pgpool_extension=true
```

#### 2.3. /etc/init.d Service Scripts
```
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# cd /etc/init.d/
[root@ppaslab init.d]#
[root@ppaslab init.d]# ls -ltr
...
-rwxr-xr-x. 1 root root 16285 Feb  3 00:11 ppas-9.5
-rwxr-xr-x. 1 root root 15481 Feb  3 00:11 ppas-agent-9.5
-rwxr-xr-x. 1 root root 15554 Feb  3 00:12 ppas-infinitecache
-rwxr-xr-x. 1 root root 16508 Feb  3 00:12 ppas-replication-9.5
-rwxr-xr-x. 1 root root 16051 Feb  3 00:12 ppas-pgpool-3.4
-rwxr-xr-x. 1 root root 15407 Feb  3 00:12 pgbouncer-1.6
...
[root@ppaslab init.d]#

```

## 3. Uninstallation
```
[root@ppaslab init.d]# cd /opt/PostgresPlus/
[root@ppaslab PostgresPlus]#
[root@ppaslab PostgresPlus]#
[root@ppaslab PostgresPlus]# ls
9.5AS       edbmtk         pgbouncer-1.6  stackbuilderplus             uninstall-ppas_9_5_complete.dat
connectors  infinitecache  pgpool-II-3.4  uninstall-ppas_9_5_complete
[root@ppaslab PostgresPlus]#
[root@ppaslab PostgresPlus]#
[root@ppaslab PostgresPlus]# ./uninstall-ppas_9_5_complete
Do you want to uninstall Postgres Plus Advanced Server and all of its modules? [Y/n]: Y

----------------------------------------------------------------------------
Uninstall Status

-Info: The data directory (/opt/PostgresPlus/9.5AS/data) and service user account
(enterprisedb) have not been removed.
Press [Enter] to continue:
 Uninstalling Postgres Plus Advanced Server
 0% ______________ 50% ______________ 100%
 #########################################

Info: Uninstallation completed
Press [Enter] to continue:
[root@ppaslab PostgresPlus]#
[root@ppaslab PostgresPlus]# ls
9.5AS
[root@ppaslab PostgresPlus]# cd 9.5AS/
[root@ppaslab 9.5AS]# ls
data
[root@ppaslab 9.5AS]# rm -rf data
[root@ppaslab 9.5AS]#
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# cd /etc/init.d/
[root@ppaslab init.d]#
[root@ppaslab init.d]# ls -ltr
total 388
-rwxr-xr-x. 1 root root  1725 Aug 19  2010 acpid
-rwxr-xr-x. 1 root root  2261 Jun 25  2011 oddjobd
-rwxr-xr-x. 1 root root  1808 Dec 18  2011 rngd
-rwxr-xr-x. 1 root root  2062 Jan 30  2012 atd
-rwxr-xr-x. 1 root root  2023 Apr  4  2012 portreserve
...
중략
...
-rwxr-xr-x. 1 root root 15796 Feb  2 09:19 vboxadd
-rwxr-xr-x. 1 root root  4535 Feb  2 09:19 vboxadd-service
-rwxr-xr-x. 1 root root 22252 Feb  2 18:20 vboxadd-x11
```

## 4. Installation - CLI Mode
```
[root@ppaslab ~]# cd /opt/pkgs/ppasmeta-9.5.0.5-linux-x64
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]#
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]#
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]#
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# ./ppasmeta-9.5.0.5-linux-x64.run --mode text
Language Selection

Please select the installation language
[1] English - English
[2] Japanese - ???
[3] Simplified Chinese - ????
[4] Traditional Chinese - ????
[5] Korean - ???
Please choose an option [1] :
----------------------------------------------------------------------------
Welcome to the Postgres Plus Advanced Server Setup Wizard.

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


1. Scope of License. Subject to the terms and conditions of this Agreement,
EnterpriseDB grants to Customer a non-exclusive, non-transferable right to
install a single instance of the Software on no more than five (5) Servers. The
Software may only be used solely for internal evaluation purposes ("Authorized
Use") during the time that Customer is current in the payment of the applicable
subscription fees or if no subscription fees are due, then this license will
remain in effect until this Agreement is terminated as set forth in Section 9.
Evaluation purposes do not include the right to use the Software for production
use, sublicensing, resale, training or distribution, including without
limitation, operation on a time sharing, software as a service or service bureau
basis or distributing the Software as part of an ASP, VAR, OEM, distributor or
reseller arrangement. "Server" is a single machine which processes data using
one or more CPUs. If a machine includes server blades or virtual servers, each
such server blade or virtual server is considered a separate Server.


2. License Restrictions. Customer agrees not to, or allow any third party to:
(a) copy or use the Software in any manner except as expressly permitted in this
Agreement; (b) transfer, sell, rent, lease, distribute, or sublicense the
Press [Enter] to continue:
Software; (c) use the Software for providing time-sharing services, service
bureau services or as part of an application services provider or software as a
service offering; (d) reverse engineer, disassemble, decompile the Software; (e)
alter or remove any proprietary notices in the Software; and (f) make available
to any third party any analysis of the results of operation of the Software,
including benchmarking results, without the prior written consent of
EnterpriseDB. Customer may make one additional copy of the Software for backup
or archival purposes provided that the Authorized Use is not exceeded. If
Customer would like to change the level of Authorized Use, Customer will need to
enter into the appropriate EnterpriseDB license and pay the applicable fees.


3. Ownership. EnterpriseDB and its licensors retain all right, title and
interest in and to the Software and any modifications and enhancements to the
Software and all upgrades, including all intellectual property rights that are
not expressly granted in this Agreement. Customer may provide EnterpriseDB with
feedback regarding the Software ("Feedback"), including bugs, errors and feature
requests. EnterpriseDB owns all right, title and interest in and to all
Feedback.


4. Open Source Programs. The Software may be distributed with third party open
source software programs as described in the licenses directory of the Software.
Press [Enter] to continue:
These open source programs are distributed under open source licenses and not
this Agreement.


5. Verification. Customer acknowledges that the Software may include
functionality that notifies Customer of the availability of updates and collects
and reports certain information about the use of the Software to EnterpriseDB.
Customer will provide EnterpriseDB with documentation concerning transactions
related to the Software within thirty (30) days after written request. In
addition, upon at least thirty (30) days prior written notice, EnterpriseDB or
its designated agent may inspect and review Customer's facilities and records in
order to verify Customer's compliance with this Agreement.


6. DISCLAIMER OF WARRANTIES. ENTERPRISEDB PROVIDES THE SOFTWARE TO CUSTOMER "AS
IS". ENTERPRISEDB DISCLAIMS ALL WARRANTIES OF ANY KIND, WHETHER EXPRESS OR
IMPLIED, INCLUDING, WITHOUT LIMITATION, THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE, TITLE AND NONINFRINGEMENT, AND ALL WARRANTIES
THAT MAY ARISE FROM COURSE OF PERFORMANCE, COURSE OF DEALING OR USAGE OF TRADE.
WITH RESPECT TO PRE-RELEASE SOFTWARE, CUSTOMER ACKNOWLEDGES THAT THE SOFTWARE
CONTAINS UNTESTED AND EXPERIMENTAL SOFTWARE OF ENTERPRISEDB AND THAT THE TESTING
AND QUALITY ASSURANCE OF THE SOFTWARE OR PARTS THEREOF HAVE NOT YET BEEN
COMPLETED.
Press [Enter] to continue:


7. LIMITATIONS OF LIABILITY. Notwithstanding any other clause in this Agreement,
in no event will EnterpriseDB be liable for any special, indirect, incidental,
punitive or consequential damages (including, without limitation, any failure to
realize savings or other benefits; any loss of use; or any claims made by or any
payments made to any third person), any loss of revenue or profits, any loss
and/or damage arising from or in connection with a virus, or any loss of data
and/or damage arising there from or relating thereto, in each case arising from
or in connection with this Agreement or the use or performance of any Software
whether in an action based on contract, tort or any other legal theory, whether
or not EnterpriseDB has been notified of the possibility thereof.
Notwithstanding any other clause in this Agreement, in no event will
EnterpriseDB's total aggregate liability for any damages arising from or in
connection with this Agreement or the use or performance of any Software whether
in actions based on contract, tort or any other legal theory, and whether or not
EnterpriseDB has been notified of the possibility thereof, exceed the amount
paid under this Agreement during the twelve (12) month period preceding the date
of the claim. The foregoing limitations, exclusions and disclaimers are an
allocation of the risk between the parties and will apply to the maximum extent
permitted by applicable law, even if any remedy fails in its essential purpose.


Press [Enter] to continue:
8. Government Rights. The Software under this Agreement is "commercial computer
software" as that term is described in DFAR 252.227-7014(a) (1). If acquired by
or on behalf of a civilian agency, the U.S. Government acquires this commercial
computer software and/or commercial computer software documentation subject to
the terms and this Agreement as specified in 48C.F.R. 12.212 (Computer Software)
and 12.11 (Technical Data) of the Federal Acquisition Regulations ("FAR") and
its successors. If acquired by or on behalf of any agency within the Department
of Defense ("DOD"), the U.S. Government acquires this commercial computer
software and/or commercial computer software documentation subject to the terms
of this Agreement as specified in 48 C.F.R. 227.7202 of the DOD FAR Supplement
and its successors.


9. Term and Termination. This Agreement is effective as of the date this
Agreement is accepted by Customer and will continue: (i) with respect to
Generally Available Software for sixty (60) days unless earlier terminated by
either party giving written notice to the other of its intent to terminate, and
(ii) with respect to Pre-Release Software, until the end of the applicable
testing program unless the Software earlier becomes Generally Available. In the
event of a termination of this Agreement, Customer must de-install all Software
and cease all use of the Software. Sections 2, 3, 4, 5, 6, 7, 8, 9 and 10 will
survive the termination of this Agreement. In addition, Customer will pay
EnterpriseDB all monies that become due prior to the date of termination.
Press [Enter] to continue:


10. Miscellaneous.

10.1 Entire Agreement. This Agreement constitutes the entire agreement between
the parties concerning the subject matter hereof, notwithstanding any different
or additional terms that may be contained in the form of purchase order or other
document used by Customer to place orders or otherwise effect transactions
hereunder. This Agreement supersedes all prior or contemporaneous discussions,
proposals and agreements between the parties relating to the subject matter of
this Agreement. No amendment, modification or waiver of any provision of this
Agreement will be effective unless in writing and signed by both parties.

10.2 Severability. If any provision of this Agreement is held to be invalid or
unenforceable, the remaining portions will remain in full force and effect and
such provision will be enforced to the maximum extent possible so as to effect
the intent of the parties and will be reformed to the extent necessary to make
such provision valid and enforceable provided, however, that if Sections 6 and 7
cannot be modified to be valid and enforceable, this Agreement will be deemed
invalid in its entirety.

10.3 Force Majeure. Neither party will be liable or deemed to be in breach for
any delay or failure in performance of this Agreement (except for the payment of
Press [Enter] to continue:
money) or interruption of services resulting directly or indirectly from acts of
God, civil or military authority, war, riots, civil disturbances, accidents,
fire, earthquake, floods, strikes, lock-outs, labor disturbances, foreign or
governmental order, or any other cause beyond the reasonable control of such
party.

10.4 Governing Law and Venue. This Agreement will be governed by the laws of New
York without regard for its choice of law provisions. All disputes arising out
of or relating to this Agreement will be submitted to the exclusive jurisdiction
of the state or federal courts of New York, and each party irrevocably consents
to such personal jurisdiction and waives all objections to this venue.

10.5 Export Regulations. Customer will comply fully with all export control laws
and regulations of the United States and all other jurisdictions.

10.6 Assignment. Neither party may assign this Agreement without the prior
written consent of the other party, which consent will not be unreasonably
withheld, provided that no consent will be necessary if this Agreement is being
assigned by a party to an acquirer of all or substantially all of the party's
assets (or the assets of the party's applicable business unit), whether by
merger, sale or exchange of stock, sale of assets or otherwise and in this case,
the party may assign this Agreement by providing written notice to the other
party.
Press [Enter] to continue:

10.7 Marketing. EnterpriseDB may use Customer's name and company logo on its
customer list and web site, and link to Customer's web site.

10.8 Independent Contractor. The relationship of the parties is that of
independent contractors. Neither party will be deemed to be the legal
representative of the other nor will it have any right to bind the other party
to any contract or commitment. This Agreement does not, and will not, be
construed to create an employer-employee, agency, joint venture or partnership
relationship between the parties. Each party agrees to assume complete
responsibility for its own employees regarding federal or state laws, including
employers' liability and tax withholding, worker's compensation, social
security, unemployment insurance, and OSHA requirements.

10.9 Notice. All notices and other communications herein permitted or required
under this Agreement will be sent by postage prepaid, via registered or
certified mail or overnight courier, return receipt requested, or delivered
personally to the parties at their respective addresses, or to such other
address as either party will give to the other party in the manner provided
herein for giving notice. Notice will be considered given upon receipt.

10.10 Confidentiality. "Confidential Information" means the Software, the
Feedback and all information in any form disclosed by EnterpriseDB to Customer
Press [Enter] to continue:
that, due to its nature or the circumstances surrounding disclosure would be
considered confidential by a reasonable person. Confidential Information does
not include any information that is: (i) or becomes available to the general
public due to no fault of Customer, (ii) in Customer's possession prior to the
disclosure and under no obligation to hold such information in confidence, (iii)
disclosed to Customer by a third party who is under no obligation to hold that
information in confidence, (iv) is independently developed by Customer without
use of Confidential Information, or (v) is approved for release or disclosure by
written authorization of EnterpriseDB. Customer agrees to use Confidential
Information solely for the purpose of evaluation and testing of the Software.
Customer agrees (a) to hold the Confidential Information in confidence, (b) to
protect and store it consistently with its own confidential information, but in
no event to use less than a reasonable standard of care, and (c) not to copy,
duplicate, disclose or deliver all or any portion of the Confidential
Information to any third party (except as expressly permitted in this
Agreement). Customer may share the Confidential Information only with those
employees and consultants with a specific need to review the Confidential
Information in the course of the evaluation and testing of the Software.
Customer will have executed or shall execute appropriate written agreements with
its employees and consultants sufficient to enable Customer to enforce all the
provisions of this Agreement. Customer is liable for all acts and omissions of
its employees and consultants to the extent that such act or omission would be a
breach of this Agreement if it had been done by Customer. Notwithstanding the
Press [Enter] to continue:
foregoing restrictions, the Customer may use and disclose any information to the
extent required by an order of any court or other governmental authority, but
only after EnterpriseDB has been so notified and has had the opportunity, if
possible, to obtain reasonable protection for such information in connection
with such disclosure. If EnterpriseDB is not successful in precluding the
requesting legal body from requiring the disclosure of the Confidential
Information, the Customer will furnish only that portion of the Confidential
Information which is legally required.

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
Please specify the directory where Postgres Plus Advanced Server will be
installed.

Installation Directory [/opt/PostgresPlus]:

----------------------------------------------------------------------------
Select the components you want to install.

Database Server [Y/n] :Y

Connectors [Y/n] :Y

Infinite Cache [Y/n] :Y

Migration Toolkit [Y/n] :Y

Postgres Enterprise Manager Client [Y/n] :Y

pgpool-II [Y/n] :Y

pgpool-II Extensions [Y/n] :Y

EDB*Plus [Y/n] :Y

Slony Replication [Y/n] :n

PgBouncer [Y/n] :n

Is the selection above correct? [Y/n]: Y

----------------------------------------------------------------------------
Additional Directories

Please select a directory under which to store your data.

Data Directory [/opt/PostgresPlus/9.5AS/data]:

Please select a directory under which to store your Write-Ahead Logs.

Write-Ahead Log (WAL) Directory [/opt/PostgresPlus/9.5AS/data/pg_xlog]:

----------------------------------------------------------------------------
Configuration Mode

Postgres Plus Advanced Server always installs with database compatibility features for Oracle(R) and maintains full PostgreSQL compliance. Select your style preference for installation defaults and samples.

The Oracle configuration will cause the use of certain objects  (e.g. DATE data types, string operations, etc.) to produce results compatible with Oracle, create the same Oracle sample tables, and have the database match Oracle examples used in the documentation.

Configuration Mode

[1] Compatible with Oracle
[2] Compatible with PostgreSQL
Please choose an option [1] :

----------------------------------------------------------------------------
Please provide a password for the database superuser (enterprisedb). A locked
Unix user account (enterprisedb) will be created if not present.

Password :
Retype Password :
----------------------------------------------------------------------------
Additional Configuration

Please select the port number the server should listen on.

Port [5444]:

Select the locale to be used by the new database cluster.

Locale

[1] [Default locale]
[2] C
[3] POSIX
[4] aa_DJ
[5] aa_DJ.iso88591
[6] aa_DJ.utf8
[7] aa_ER
[8] aa_ER.utf8
[9] aa_ER.utf8@saaho
[10] aa_ER@saaho
...
[286] es_US
[287] es_US.iso88591
[288] es_US.utf8
...
[442] ko_KR
[443] ko_KR.euckr
[444] ko_KR.utf8
...
[717] zu_ZA.iso88591
[718] zu_ZA.utf8
Please choose an option [1] :



Install sample tables and procedures. [Y/n]: Y


----------------------------------------------------------------------------
Dynatune Dynamic Tuning:
Server Utilization

Please select the type of server to determine the amount of system resources
that may be utilized:



[1] Development (e.g. a developer's laptop)
[2] General Purpose (e.g. a web or application server)
[3] Dedicated (a server running only Postgres Plus)
Please choose an option [2] :

----------------------------------------------------------------------------
Dynatune Dynamic Tuning:
Workload Profile

Please select the type of workload this server will be used for:



[1] Transaction Processing (OLTP systems)
[2] General Purpose (OLTP and reporting workloads)
[3] Reporting (Complex queries or OLAP workloads)
Please choose an option [1] :

----------------------------------------------------------------------------
Advanced Configuration

----------------------------------------------------------------------------
Service Configuration



Autostart pgAgent Service [Y/n]: Y




Update Notification Service [Y/n]: n


The Update Notification Service informs, downloads and installs whenever
security patches and other updates are available for your Postgres Plus Advanced
Server installation.



----------------------------------------------------------------------------
Pre Installation Summary

Following settings will be used for installation:

Installation Directory: /opt/PostgresPlus
Data Directory: /opt/PostgresPlus/9.5AS/data
WAL Directory: /opt/PostgresPlus/9.5AS/data/pg_xlog
Database Port: 5444
Database Superuser: enterprisedb
Operating System Account: enterprisedb
Database Service: ppas-9.5

Press [Enter] to continue:

----------------------------------------------------------------------------
Setup is now ready to begin installing Postgres Plus Advanced Server on your
computer.

Do you want to continue? [Y/n]: Y

----------------------------------------------------------------------------
Please wait while Setup installs Postgres Plus Advanced Server on your computer.

 Installing Postgres Plus Advanced Server
 0% ______________ 50% ______________ 100%
 ########################################
 Installing Database Server ...
 Installing pgAgent ...
 Installing Connectors ...
 Installing Migration Toolkit ...
 Installing EDB*Plus ...
 Installing Infinite Cache ...
 Installing Postgres Enterprise Manager Client ...
 Installing pgpool-II ...
 Installing pgpool-II Extensions ...
 Installing StackBuilder Plus ...
 #

----------------------------------------------------------------------------
Setup has finished installing Postgres Plus Advanced Server on your computer.

[root@ppaslab ppasmeta-9.5.0.5-linux-x64]#
```

#### 4.1. Post Installation
##### 4.1.1. Change Directory Ownership
```
[root@ppaslab ppasmeta-9.5.0.5-linux-x64]# cd /opt/
[root@ppaslab opt]#
[root@ppaslab opt]# ls
PostgresPlus  VBoxGuestAdditions-5.0.14  rh
[root@ppaslab opt]# ll
total 12
drwxr-xr-x. 8 root daemon 4096 Feb 13 22:18 PostgresPlus
drwxr-xr-x. 9 root root   4096 Feb  2 18:49 VBoxGuestAdditions-5.0.14
drwxr-xr-x. 2 root root   4096 Mar 26  2015 rh
[root@ppaslab opt]#
[root@ppaslab opt]# chown -R enterprisedb.enterprisedb PostgresPlus/
[root@ppaslab opt]#
[root@ppaslab opt]# ll
total 12
drwxr-xr-x. 8 enterprisedb enterprisedb 4096 Feb 13 22:18 PostgresPlus
drwxr-xr-x. 9 root         root         4096 Feb  2 18:49 VBoxGuestAdditions-5.0.14
drwxr-xr-x. 2 root         root         4096 Mar 26  2015 rh
[root@ppaslab opt]#
```
##### 4.1.2. pgplus_env.sh
```
[root@ppaslab ~]# su - enterprisedb
-bash-4.1$ ls
bin        installer                       scripts                        uninstall-edbpgagent
client-v6  lib                             server_3rd_party_licenses.txt  uninstall-edbpgagent.dat
data       pgagent_3rd_party_licenses.txt  server_license.txt             uninstall-pgpool_extension
doc        pgagent_license.txt             share                          uninstall-pgpool_extension.dat
etc        pgplus_env.sh                   uninstall-db_server
include    pgpool_ext_license.txt          uninstall-db_server.dat
-bash-4.1$
-bash-4.1$ cat pgplus_env.sh
# EnterpriseDB shell environment loader
#
#  Instructions:
#   This file contains additions to the user environment
#  that make accessing Postgres Plus Advanced Server
#  executables easier.
#
#  To load the environment for a single user:
#     cp pgplus_env.sh /home/<username>
#     chown <username> /home/<username>/pgplus_env.sh
#     vi /home/<username>/.bash_profile
#      At the bottom, add the line:
#       . /home/<username>/pgplus_env.sh
#          ( Note the '.' followed by a space )

#  To load the environment for all users:
#     cp pgplus_env.sh /etc
#     vi /etc/profile
#     At the bottom, add the line:
#       . /etc/pgplus_env.sh
#        ( Note the '.' followed by a space )

# Environment

export PATH=/opt/PostgresPlus/9.5AS/bin:$PATH
export EDBHOME=/opt/PostgresPlus/9.5AS
export PGDATA=/opt/PostgresPlus/9.5AS/data
export PGDATABASE=edb
# export PGUSER=enterprisedb
export PGPORT=5444
export PGLOCALEDIR=/opt/PostgresPlus/9.5AS/share/locale
-bash-4.1$
```
##### 4.1.2. Create .profile
```
-bash-4.1$ vi .profile

. pgplus_env.sh

-bash-4.1$
```

## 5. Start and Stop Server
#### 5.1. Stop Server
##### 5.1.1. pg_ctl
```
-bash-4.1$ pg_ctl -D $PGDATA stop -mf
waiting for server to shut down.... done
server stopped
-bash-4.1$
```
##### 5.1.2. PPAS-9.5 Service
```
[root@ppaslab ~]# /etc/init.d/ppas-9.5 stop -mf
INFO: [Stopping dependent service: ppas-agent-9.5]
INFO: [PID: 5586]
INFO: [CMD: /opt/PostgresPlus/9.5AS/bin/pgagent -l 1 -s /var/log/ppas-agent-9.5/ppas-agent-9.5.log hostaddr=localhost port=5444 dbname=edb user=enterprisedb]

Stopping ppas-agent-9.5                                    [  OK  ]

MSG:  [ppas-agent-9.5 stopped]

INFO: [Please see service script file /var/log/ppas-agent-9.5/ppas-agent-9.5_script.log for details]

INFO: [PID: 5167]
INFO: [CMD:]

Stopping ppas-9.5
                                                           [  OK  ]

MSG:  [ppas-9.5 stopped]

INFO: [Please see service script file /var/log/ppas-9.5/ppas-9.5_script.log for details]

[root@ppaslab ~]#
```
#### 5.2. Start Server
##### 5.2.1. pg_ctl
```
-bash-4.1$ pg_ctl -D $PGDATA start -w
waiting for server to start....2016-02-10 12:19:35 KST LOG:  redirecting log output to logging collector process
2016-02-10 12:19:35 KST HINT:  Future log output will appear in directory "pg_log".
 done
server started
-bash-4.1$
```
##### 5.2.2. PPAS-9.5 Service
```
[root@ppaslab ~]# /etc/init.d/ppas-9.5 start
Starting ppas-9.5                                          [  OK  ]

INFO: [PID: 9617]
INFO: [CMD:]
MSG:  [ppas-9.5 started]

INFO: [Please see service script file /var/log/ppas-9.5/ppas-9.5_script.log for details]

[root@ppaslab ~]#
```
#### 5.3. Restart Server
##### 5.3.1. pg_ctl
```
-bash-4.1$ pg_ctl -D $PGDATA restart -mf -w
waiting for server to shut down.... done
server stopped
waiting for server to start....2016-02-10 12:20:19 KST LOG:  redirecting log output to logging collector process
2016-02-10 12:20:19 KST HINT:  Future log output will appear in directory "pg_log".
 done
server started
-bash-4.1$
```
##### 5.3.2. PPAS-9.5 Service
```
[root@ppaslab ~]# /etc/init.d/ppas-9.5 restart -mf
Restarting ppas-9.5                                        [  OK  ]

INFO: [PID: 9855]
INFO: [CMD:]
MSG:  [ppas-9.5 restarted]

INFO: [Please see service script file /var/log/ppas-9.5/ppas-9.5_script.log for details]

[root@ppaslab ~]#
```
#### 5.4. Reload Server
##### 5.4.1. pg_ctl
```
-bash-4.1$ pg_ctl -D $PGDATA reload
server signaled
-bash-4.1$
```
##### 5.4.2. PPAS-9.5 Service
```
[root@ppaslab ~]# /etc/init.d/ppas-9.5 reload
INFO: [PID: 9855]
INFO: [CMD:]

Reloading ppas-9.5                                         [  OK  ]

MSG:  [ppas-9.5 service configuration file reloaded]

INFO: [Please see service script file /var/log/ppas-9.5/ppas-9.5_script.log for details]

[root@ppaslab ~]#
```
#### 5.5. Status Server
##### 5.5.1. pg_ctl
```
-bash-4.1$ pg_ctl -D $PGDATA status
pg_ctl: server is running (PID: 5167)
/opt/PostgresPlus/9.5AS/bin/edb-postgres "-D" "/opt/PostgresPlus/9.5AS/data"
-bash-4.1$
```
##### 5.5.2. PPAS-9.5 Service
```
[root@ppaslab ~]#
[root@ppaslab ~]# /etc/init.d/ppas-9.5 status
INFO: [PID: 5167]
INFO: [CMD:]
MSG:  [ppas-9.5 is running]
INFO: [Please see service script file /var/log/ppas-9.5/ppas-9.5_script.log for details]

[root@ppaslab ~]#
```
