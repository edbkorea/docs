# Lab: EDB Failover Manager Hands-on Lab

## 1. Demo Environment
* Master node
```
Hostname : pg1
IP Address : 192.168.56.101
```
* Standby node
```
Hostname : pg2
IP Address : 192.168.56.102
```
* Witness node
```
Hostname : pg3
IP Address : 192.168.56.103
```
* VIP
```
IP Address : 192.168.56.111
Up and running on master node
```

## 2. Configuration
#### 2.1. Do this in all node
```
[root@pg1 ~]# cd /etc/efm-2.0/
[root@pg1 efm-2.0]#
[root@pg1 efm-2.0]# cp efm.nodes.in efm.nodes
[root@pg1 efm-2.0]#
[root@pg1 efm-2.0]# cp efm.properties.in efm.properties
[root@pg1 efm-2.0]#
[root@pg1 efm-2.0]# export PATH=$PATH:/usr/efm-2.0/bin:$HOME/bin
```

#### 2.2. File Locations
```
- Executables: /usr/efm-2.0/bin
- Libraries: /usr/efm-2.0/lib
- Cluster configuration files: /etc/efm-2.0
- Logs: /var/log/efm-2.0
- Lock files: /var/lock/efm-2.0
- Log rotation file: /etc/logrotate.d/efm-2.0
- sudo configuration file: /etc/sudoers.d/efm-20
```

#### 2.3. efm.properties
```
 - Manual : http://www.enterprisedb.com/docs/en/2.0.3/edbfm/toc.html
 - Demo system configuration files : find attaches, please
```

## 3. EFM Agent Startup Procedures
```
1. Witness
2. Master
3. Slave
4. Cluster Status
```

#### 3.1. On witness
Start agent on witness. As this is the first node,  you don't need edit efm.nodes file.
```
[root@pg3 ~]# service efm-2.0 start
Starting local efm-2.0 service:                            [  OK  ]
[root@pg3 ~]#
```
###### - Notification Email
```
[INFO] EFM Witness agent running on 192.168.56.103 for cluster efm
EFM node:     192.168.56.103
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Witness agent is running.
```

#### 3.2. On master
##### 3.2.1. In efm.nodes add [ip of witness]:[port].
```
[root@pg1 ~]# cd /etc/efm-2.0/
[root@pg1 efm-2.0]#
[root@pg1 efm-2.0]# vi efm.nodes

# List of node address:port combinations separated by whitespace.
192.168.56.103:7800
```

##### 3.2.2. Add master node on witness using below command
###### efm add-node [cluster_name] [ip of master]
```
[root@pg3 ~]# efm add-node efm 192.168.56.101
add-node signal sent to local agent.
```

##### 3.2.3. Start agent on master
```
[root@pg1 efm-2.0]# service efm-2.0 start
Starting local efm-2.0 service:                            [  OK  ]
[root@pg1 efm-2.0]#
```
##### 3.2.4. VIP has been started as well
```
[root@pg1 efm-2.0]# ifconfig
eth2      Link encap:Ethernet  HWaddr 08:00:27:84:76:92
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe84:7692/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4675 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3060 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4630332 (4.4 MiB)  TX bytes:231234 (225.8 KiB)

eth3      Link encap:Ethernet  HWaddr 08:00:27:16:5D:93
          inet addr:192.168.56.101  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe16:5d93/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:325092 errors:0 dropped:0 overruns:0 frame:0
          TX packets:683794 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:368144803 (351.0 MiB)  TX bytes:936795744 (893.3 MiB)

eth3:1    Link encap:Ethernet  HWaddr 08:00:27:16:5D:93
          inet addr:192.168.56.111  Bcast:192.168.56.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:61699 errors:0 dropped:0 overruns:0 frame:0
          TX packets:61699 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:16045474 (15.3 MiB)  TX bytes:16045474 (15.3 MiB)

[root@pg1 efm-2.0]#
```

##### 3.2.5. Notification Email
```
## VIP assign notification
[INFO] EFM Assigning VIP to node 192.168.56.101
EFM node:     192.168.56.101
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Assigning VIP 192.168.56.111 to node 192.168.56.101

Results:
exit status: 0

## EFM master agent started notification
[INFO] EFM Master agent running on 192.168.56.101 for cluster efm
EFM node:     192.168.56.101
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Master agent is running and database health is being monitored.
```

#### 3.3. On slave
##### 3.3.1. In efm.nodes add [ip of witness]:[port] and [ip of master]:[port]
```
[root@pg2 ~]# cd /etc/efm-2.0/
[root@pg2 efm-2.0]#
[root@pg2 efm-2.0]# vi efm.nodes

# List of node address:port combinations separated by whitespace.
192.168.56.103:7800 192.168.56.101:7800
```

##### 3.3.2. Add slave node on witness and master node using below command:
efm add-node [cluster_name] [ip of slave]
```
[root@pg3 efm-2.0]# efm add-node efm 192.168.56.102
add-node signal sent to local agent.
[root@pg3 efm-2.0]#
```

##### 3.3.3. Start agent on slave
```
[root@pg2 efm-2.0]# /etc/init.d/efm-2.0 start
Starting local efm-2.0 service:                            [  OK  ]
[root@pg2 efm-2.0]#
```

##### 3.3.4. Notification Email
```
[INFO] EFM Standby agent running on 192.168.56.102 for cluster efm

EFM node:     192.168.56.102
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Standby agent is running and database health is being monitored.
```

#### 3.4. Cluster Status
##### 3.4.1. On witness
```
[root@pg3 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Master      192.168.56.101       UP     UP
	Witness     192.168.56.103       UP     N/A
	Standby     192.168.56.102       UP     UP

Allowed node host list:
	192.168.56.103 192.168.56.101 192.168.56.102

Standby priority host list:
	192.168.56.102

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Master      192.168.56.101       0/180012C0
	Standby     192.168.56.102       0/180012C0

	Standby database(s) in sync with master. It is safe to promote.
[root@pg3 efm-2.0]#
```

##### 3.4.2. On master
```
[root@pg1 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Standby     192.168.56.102       UP     UP
	Witness     192.168.56.103       UP     N/A
	Master      192.168.56.101       UP     UP

Allowed node host list:
	192.168.56.103 192.168.56.101 192.168.56.102

Standby priority host list:
	192.168.56.102

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Master      192.168.56.101       0/180013A0
	Standby     192.168.56.102       0/180013A0

	Standby database(s) in sync with master. It is safe to promote.
[root@pg1 efm-2.0]#
```

##### 3.4.3. On standby
```
[root@pg2 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Standby     192.168.56.102       UP     UP
	Witness     192.168.56.103       UP     N/A
	Master      192.168.56.101       UP     UP

Allowed node host list:
	192.168.56.103 192.168.56.101 192.168.56.102

Standby priority host list:
	192.168.56.102

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Master      192.168.56.101       0/180013A0
	Standby     192.168.56.102       0/180013A0

	Standby database(s) in sync with master. It is safe to promote.
[root@pg2 efm-2.0]#
```

## 4. Failure Scenarios
#### 4.1. Supported Scenarios
```
1. Master database is down
2. Master agent exits
3. Master node down
```

#### 4.2. Master DB is Down
##### 4.2.1. Kill postmaster process
```
[root@pg1 efm-2.0]# ps -efH |grep postgres
root     21270 18076  0 16:47 pts/0    00:00:00         grep postgres
500      18123     1  0 13:37 pts/0    00:00:00   /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data
500      18125 18123  0 13:37 ?        00:00:00     postgres: logger process
500      18127 18123  0 13:37 ?        00:00:00     postgres: checkpointer process
500      18128 18123  0 13:37 ?        00:00:00     postgres: writer process
500      18129 18123  0 13:37 ?        00:00:00     postgres: wal writer process
500      18130 18123  0 13:37 ?        00:00:00     postgres: autovacuum launcher process
500      18131 18123  0 13:37 ?        00:00:00     postgres: archiver process   last was 000000030000000000000017.00000060.backup
500      18132 18123  0 13:37 ?        00:00:00     postgres: stats collector process
500      18468 18123  0 13:40 ?        00:00:00     postgres: wal sender process enterprisedb 192.168.56.102[45482] streaming 0/180020D8
[root@pg1 efm-2.0]#
[root@pg1 efm-2.0]# kill -9 18123
[root@pg1 efm-2.0]#
[root@pg1 efm-2.0]# ps -efH |grep postgres
root     21281 18076  0 16:48 pts/0    00:00:00         grep postgres
[root@pg1 efm-2.0]#
```

##### 4.2.2. Cluster status
```
[root@pg3 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Master      192.168.56.101       UP     UP
	Witness     192.168.56.103       UP     N/A
	Standby     192.168.56.102       UP     UP

Allowed node host list:
	192.168.56.103 192.168.56.101 192.168.56.102

Standby priority host list:
	192.168.56.102

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Unknown     192.168.56.101       UNKNOWN          Connection refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
	Standby     192.168.56.102       0/180020D8

	No master database was found.
[root@pg1 efm-2.0]#
```

##### 4.2.3. Where VIP is?
```
## on master
[root@pg1 efm-2.0]# ifconfig
eth2      Link encap:Ethernet  HWaddr 08:00:27:84:76:92
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe84:7692/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4728 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3108 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4635934 (4.4 MiB)  TX bytes:236226 (230.6 KiB)

eth3      Link encap:Ethernet  HWaddr 08:00:27:16:5D:93
          inet addr:192.168.56.101  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe16:5d93/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:329944 errors:0 dropped:0 overruns:0 frame:0
          TX packets:688828 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:368696330 (351.6 MiB)  TX bytes:937322494 (893.9 MiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:74472 errors:0 dropped:0 overruns:0 frame:0
          TX packets:74472 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:19367338 (18.4 MiB)  TX bytes:19367338 (18.4 MiB)

[root@pg1 efm-2.0]#

## on standby
[root@pg2 efm-2.0]# ifconfig
eth4      Link encap:Ethernet  HWaddr 08:00:27:B2:31:55
          inet addr:192.168.56.102  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:feb2:3155/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:279833 errors:0 dropped:0 overruns:0 frame:0
          TX packets:20326 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:407809916 (388.9 MiB)  TX bytes:1727914 (1.6 MiB)

eth4:1    Link encap:Ethernet  HWaddr 08:00:27:B2:31:55
          inet addr:192.168.56.111  Bcast:192.168.56.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

eth5      Link encap:Ethernet  HWaddr 08:00:27:14:66:E3
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe14:66e3/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:195 errors:0 dropped:0 overruns:0 frame:0
          TX packets:185 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:22201 (21.6 KiB)  TX bytes:19880 (19.4 KiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:19838 errors:0 dropped:0 overruns:0 frame:0
          TX packets:19838 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:4979564 (4.7 MiB)  TX bytes:4979564 (4.7 MiB)

[root@pg2 efm-2.0]#
```

##### 4.2.4. Cluster status
```
[root@pg3 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Witness     192.168.56.103       UP     N/A
	Idle        192.168.56.101       UP     UNKNOWN
	Master      192.168.56.102       UP     UP

Allowed node host list:
	192.168.56.103 192.168.56.101 192.168.56.102

Standby priority host list:
	(List is empty.)

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Master      192.168.56.102       0/180022C8

	No standby databases were found.

Idle Node Status (idle nodes ignored in XLog location comparisons):

	Address              XLog Loc         Info
	--------------------------------------------------------------
	192.168.56.101       UNKNOWN          Connection refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
[root@pg3 efm-2.0]#
```

##### 4.2.5. Notification Email
```
1. Release VIP from master
[INFO] EFM Releasing VIP from node 192.168.56.101

EFM node:     192.168.56.101
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Releasing VIP 192.168.56.111 from node 192.168.56.101

Results:
exit status: 0

2. Master DB is down
[SEVERE] EFM Master database failure for cluster efm

EFM node:     192.168.56.101
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

The agent has detected that the database has failed on 192.168.56.101.

3. Promote standby as new master
[WARNING] EFM Promotion has started on cluster efm

EFM node:     192.168.56.102
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Promotion of standby has started on cluster efm.

4. Assign VIP to new master
[INFO] EFM Assigning VIP to node 192.168.56.102

EFM node:     192.168.56.102
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Assigning VIP 192.168.56.111 to node 192.168.56.102

Results:
exit status: 0

5. Complete failover
[WARNING] EFM Failover has completed on cluster efm

EFM node:     192.168.56.102
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Failover has completed on cluster efm.
```

##### 4.2.6. Remove old master node from cluster
###### 1. Stop agent on old master
```
[root@pg1 efm-2.0]# service efm-2.0 stop
Stopping local efm-2.0 service:                            [  OK  ]
[root@pg1 efm-2.0]#
```
###### 2. Remove node from cluster on witness
```
[root@pg3 efm-2.0]# efm remove-node efm 192.168.56.101
remove-node signal sent to local agent.
[root@pg3 efm-2.0]#
```

##### 4.2.7. Add new standby node to cluster
###### 1. Configure stremaing replication
```
[root@pg1 efm-2.0]# su - enterprisedb
-bash-4.1$
-bash-4.1$ ps -ef|grep postgres
500      21678 21652  0 17:08 pts/0    00:00:00 grep postgres
-bash-4.1$
-bash-4.1$
-bash-4.1$ ls
arch       installer                       server_license.txt
bin        lib                             share
client-v6  pgagent_3rd_party_licenses.txt  uninstall-db_server
data       pgagent_license.txt             uninstall-db_server.dat
doc        pgplus_env.sh                   uninstall-edbpgagent
etc        scripts                         uninstall-edbpgagent.dat
include    server_3rd_party_licenses.txt
-bash-4.1$ rm -rf data
-bash-4.1$
-bash-4.1$ pg_basebackup -h pg2 -R -D data
NOTICE:  pg_stop_backup complete, all required WAL segments have been archived
-bash-4.1$
-bash-4.1$ cd data
-bash-4.1$
-bash-4.1$ vi postgresql.conf
-bash-4.1$
-bash-4.1$ ls
backup_label      pg_dynshmem    pg_replslot   pg_twophase
backup_label.old  pg_hba.conf    pg_serial     PG_VERSION
base              pg_ident.conf  pg_snapshots  pg_xlog
dbms_pipe         pg_log         pg_stat       postgresql.auto.conf
global            pg_logical     pg_stat_tmp   postgresql.conf
pg_clog           pg_multixact   pg_subtrans   recovery.conf
pg_commit_ts      pg_notify      pg_tblspc     recovery.done
-bash-4.1$
-bash-4.1$ rm -f recovery.done
-bash-4.1$
-bash-4.1$ pg_ctl -D ~/data start -w
waiting for server to start....2016-02-13 17:20:57 KST LOG:  redirecting log output to logging collector process
2016-02-13 17:20:57 KST HINT:  Future log output will appear in directory "pg_log".
.. done
server started
-bash-4.1$
-bash-4.1$ ps -efH|grep postgres
500      22246 22183  0 17:21 pts/2    00:00:00             grep postgres
500      22234     1  0 17:20 pts/2    00:00:00   /opt/PostgresPlus/9.5AS/bin/edb-postgres -D /opt/PostgresPlus/9.5AS/data
500      22236 22234  0 17:20 ?        00:00:00     postgres: logger process
500      22237 22234  0 17:20 ?        00:00:00     postgres: startup process   recovering 00000004000000000000001C
500      22240 22234  1 17:20 ?        00:00:00     postgres: wal receiver process   streaming 0/1C000060
500      22241 22234  0 17:20 ?        00:00:00     postgres: checkpointer process
500      22242 22234  0 17:20 ?        00:00:00     postgres: writer process
500      22243 22234  0 17:20 ?        00:00:00     postgres: stats collector process
-bash-4.1$
```
###### 2. In efm.nodes add [ip of witness]:[port] and [ip of master]:[port]
```
[root@pg1 efm-2.0]# vi efm.nodes

# List of node address:port combinations separated by whitespace.
192.168.56.103:7800 192.168.56.102:7800
```
###### 3. Add new standby node on witness using below command
###### efm add-node [cluster_name] [ip of new standby]
```
[root@pg3 efm-2.0]# efm add-node efm 192.168.56.101
add-node signal sent to local agent.
[root@pg3 efm-2.0]#
```
###### 4. Start agent
```
[root@pg1 ~]# service efm-2.0 start
Starting local efm-2.0 service:                            [  OK  ]
[root@pg1 ~]#
```
###### 5. Cluster status
```
[root@pg3 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Master      192.168.56.102       UP     UP
	Witness     192.168.56.103       UP     N/A
	Standby     192.168.56.101       UP     UP

Allowed node host list:
	192.168.56.103 192.168.56.102 192.168.56.101

Standby priority host list:
	192.168.56.101

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Master      192.168.56.102       0/1C000140
	Standby     192.168.56.101       0/1C000140

	Standby database(s) in sync with master. It is safe to promote.
[root@pg3 efm-2.0]#
```

#### 4.3. Master Agent Exits
##### 4.3.1. Kill agent process
```
[root@pg2 efm-2.0]# ps -ef|grep efm
efm       3841     1  0 15:27 ?        00:00:16 /usr/lib/jvm/java-1.7.0-openjdk-1.7.0.65.x86_64/jre/bin/java -cp /usr/efm-2.0/lib/EFM-2.0.3.jar com.enterprisedb.hal.main.ServiceCommand __int_start /etc/efm-2.0/efm.properties
root      5654  2925  0 17:38 pts/1    00:00:00 grep efm
[root@pg2 efm-2.0]#
[root@pg2 efm-2.0]# kill -9 3841
[root@pg2 efm-2.0]#
```
##### 4.3.2. Cluster Status
```
[root@pg3 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Standby     192.168.56.101       UP     UP
	Witness     192.168.56.103       UP     N/A

Allowed node host list:
	192.168.56.103 192.168.56.102 192.168.56.101

Standby priority host list:
	192.168.56.101

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Standby     192.168.56.101       0/1C000300

	No master database was found.
[root@pg3 efm-2.0]#
```
##### 4.3.3. Notification Email
```
[WARNING] EFM Standby agent tried to promote, but master DB is still running

EFM node:     192.168.56.103
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

The standby EFM agent tried to promote itself, but detected that
the master DB is still running on 192.168.56.102.

This usually indicates that the master EFM agent has exited.

Failover has NOT occurred.
```
##### 4.3.4. Re-start Agent
###### 1. Delete EFM lock file
```
[root@pg2 efm-2.0]# rm -f /var/lock/efm-2.0/efm.lock
[root@pg2 efm-2.0]#
```
###### 2. Start agent
```
[root@pg2 efm-2.0]# service efm-2.0 start
Starting local efm-2.0 service:                            [  OK  ]
[root@pg2 efm-2.0]#
```
##### 4.3.5. Cluster Status
```
[root@pg3 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Master      192.168.56.102       UP     UP
	Witness     192.168.56.103       UP     N/A
	Standby     192.168.56.101       UP     UP

Allowed node host list:
	192.168.56.103 192.168.56.102 192.168.56.101

Standby priority host list:
	192.168.56.101

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Master      192.168.56.102       0/1C0003E0
	Standby     192.168.56.101       0/1C0003E0

	Standby database(s) in sync with master. It is safe to promote.
[root@pg3 efm-2.0]#
```

#### 4.4. Master Node is Down
##### 4.4.1. Halt master node
```
[root@pg2 efm-2.0]# halt

Broadcast message from root@pg2
	(/dev/pts/1) at 17:42 ...

The system is going down for halt NOW!
[root@pg2 efm-2.0]# Connection to 192.168.56.102 closed by remote host.
Connection to 192.168.56.102 closed.
```
##### 4.4.2. Cluster status
```
[root@pg3 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Witness     192.168.56.103       UP     N/A

Allowed node host list:
	192.168.56.103 192.168.56.102 192.168.56.101

Standby priority host list:
	192.168.56.101

Promote Status:

Did not find XLog location for any nodes.
[root@pg3 efm-2.0]#
```
##### 4.4.3. Where is VIP?
```
[root@pg1 ~]# ifconfig
eth2      Link encap:Ethernet  HWaddr 08:00:27:84:76:92
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe84:7692/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4965 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3325 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4661546 (4.4 MiB)  TX bytes:258949 (252.8 KiB)

eth3      Link encap:Ethernet  HWaddr 08:00:27:16:5D:93
          inet addr:192.168.56.101  Bcast:192.168.56.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:fe16:5d93/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:884755 errors:0 dropped:0 overruns:0 frame:0
          TX packets:738994 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:1183369019 (1.1 GiB)  TX bytes:940823810 (897.2 MiB)

eth3:1    Link encap:Ethernet  HWaddr 08:00:27:16:5D:93
          inet addr:192.168.56.111  Bcast:192.168.56.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:78000 errors:0 dropped:0 overruns:0 frame:0
          TX packets:78000 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:20236191 (19.2 MiB)  TX bytes:20236191 (19.2 MiB)

[root@pg1 ~]#
```
##### 4.4.4. Cluster status
```
[root@pg3 efm-2.0]# efm cluster-status efm
Cluster Status: efm

	Agent Type  Address              Agent  DB       Info
	--------------------------------------------------------------
	Master      192.168.56.101       UP     UP
	Witness     192.168.56.103       UP     N/A

Allowed node host list:
	192.168.56.103 192.168.56.102 192.168.56.101

Standby priority host list:
	(List is empty.)

Promote Status:

	DB Type     Address              XLog Loc         Info
	--------------------------------------------------------------
	Master      192.168.56.101       0/1C0004F0

	No standby databases were found.
[root@pg3 efm-2.0]#
```
##### 4.4.5. Notification Email
```
1. Node isolated
[WARNING] EFM One or more nodes isolated from network for cluster efm

EFM node:     192.168.56.102
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

This node appears to be isolated from the network. Other members
seen in the cluster are: 192.168.56.103 192.168.56.101 192.168.56.102.

2. There is something wrong
[SEVERE] EFM An unexpected error has occurred for cluster efm

EFM node:     192.168.56.103
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

An unexpected error has occurred on this node. Please
check the agent log for more information. Error:
Node 192.168.56.101 has signalled that it will not attempt promotion.

3. Start promotion
[WARNING] EFM Promotion has started on cluster efm

EFM node:     192.168.56.101
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Promotion of standby has started on cluster efm.

4. Assign VIP
[INFO] EFM Assigning VIP to node 192.168.56.101

EFM node:     192.168.56.101
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Assigning VIP 192.168.56.111 to node 192.168.56.101

Results:
exit status: 0

5. Complete failover
[WARNING] EFM Failover has completed on cluster efm

EFM node:     192.168.56.101
Cluster name:  efm
Database name: edb
VIP support:   ENABLED

Failover has completed on cluster efm.
```
