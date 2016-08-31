---
layout: post
title: MySQL Primer - Up and Running
date: 2014-09-23 15:22:18.000000000 +08:00
type: post
published: true
status: publish
categories:
- database
- MySQL
tags:
- MySQL
---

## Introduction

This article described the installation and service control of MySQL.

<!--more-->

## Table Of Contents

* TOC
{:toc}

## Before Install

### Choose Operating System

* stable
* reliable
* easy to manage
* plenty of resources available online
* allow runing MySQL without too much hassle

Recommendations

* SUSE Linux Enterprise Server
	* Live Patching
* Red Hat Enterprise Linux

### Choose Package

**OS specific packages**

* packages have been rigorously tested with other componenets of the given OS

	Ubuntu has MySQL packaging team

* simplicity of maintenance
	* auto resolv dependencies.
	* often not up to date
	* 3rd party specialized repositories provide updated versions
	* but these repos may not included by default

**Pre-bulit Binaries**

* careful with incompatible core dependencies like glibc and libaio

	which may silent corrupt your data

* flexibility of install and update

	manually installed pre-built binaries is less likely to be replaced/overwritten during an update by simply keeping package folders on unique directories

**Custom Built Binaries**

* need to alter the default behavior of MySQL

	* disabling and totally disallowing use of query cache
	* increasing maximum total number of indexes per table from 64
	* take adavntages of new hardware, kernel or core libs

* require engineering effort for continuous integration (may be a lot)

	* Google, Facebook, Twitter

## Install

### MySQL Server

Make data directory

```shell
mkdir /var/mysql
chown -R mysql:mysql /var/mysql
```

Create Configure File

```shell
vim /etc/my.cnf
```

#### package manager

#### pre-built binary

```shell
tar zxf mysql-5.7.12.tar.gz
vim my.cnf
# run as current user
./bin/mysqld --defaults-file=my.cnf --initialize
# alternative: run as mysql
sudo -u mysql ./bin/mysqld --defaults-file=my.cnf --initialize
```

```sql
alter user 'root'@'localhost' identified by 'password';
```

#### source code

#### Plugins

```shell
$ mysql_plugin -P
mysql_plugin would have been started with the following arguments:
--datadir=/var/lib/mysql
```

### MySQL Client

### Configure

back_log < (system back_log)

`my-{huge,large,medium,small}.cnf`

#### Default Configure

```shell
[will@rhel7.2.vmg]$ my_print_defaults mysqld
--datadir=/var/lib/mysql
--socket=/var/lib/mysql/mysql.sock
--symbolic-links=0
```

```shell
[will@ubuntu-14.04.4.vmg]$ my_print_defaults mysqld
--user=mysql
--pid-file=/var/run/mysqld/mysqld.pid
--socket=/var/run/mysqld/mysqld.sock
--port=3306
--basedir=/usr
--datadir=/var/lib/mysql
--tmpdir=/tmp
--lc-messages-dir=/usr/share/mysql
--skip-external-locking
--lower_case_table_names=1
--bind-address=127.0.0.1
--key_buffer=16M
--max_allowed_packet=16M
--thread_stack=192K
--thread_cache_size=8
--myisam-recover=BACKUP
--query_cache_limit=1M
--query_cache_size=16M
--log_error=/var/log/mysql/error.log
--expire_logs_days=10
--max_binlog_size=100M
```

#### Example Configure

```conf
!include /path/to/other.conf

[client]
port=3306
socket=/tmp/mysql.sock
# store password is not recommended, at lease make this file not readable by other users
password='passphrase'

[mysql]
# used for invoke of command: 'mysql'

[mysql-5.7]
# specific version
sql_mode=TRADITIONAL

[mysqld]
port=3306
socket=/tmp/mysql.sock
key_buffer_size=16M
max_allowed_packet=8M

performance_schema
performance_schema_events_waits_history_size=20
performance_schema_events_waits_history_long_size=15000

[mysqldump]
quick

[mysqladmin]
force
```

### Environment Variables

```shell
MYSQL_TCP_PORT=3306
EXPORT MYSQL_TCP_PORT
```

## Post Install

`mysql_install_db`

### First Run

```shell
sudo -u mysql /usr/local/mysql/bin/mysqld_safe &
```

#### Update Password

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'passphrase';
```

#### Shutdown

```shell
/usr/local/mysql/bin/mysqladmin shutdown -u root -p
```

Invoke `mysql_secure_installation` to:

0. set root password
0. remove test databases
0. restrict access

### Setup Users

```sql
# show current user
SELECT USER();
# set root password
mysqladmin -u root -password passphrase`
# update password
UPDATE mysql.user SET password = PASSWORD('passphrase') WHERE user = 'root';
FLUSH PRIVILEGES;
```

### Grant Accesses

Grant **Minimal Access** Only

`help grant`

**Syntax** `grant on <access> testdb.* to developer@'192.168.0.%';`

access:

* create
* alter
* drop
* references
* execute
* create temporary tables
* index
* create view
* show view
* create routine (can show procedure status)
* alter routine (can drop a procedure)

``` sql
with_option:
    GRANT OPTION
  | MAX_QUERIES_PER_HOUR count
  | MAX_UPDATES_PER_HOUR count
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count
```

## Running

* mysqld — The MySQL Server
* mysqld_safe — MySQL Server Startup Script
* mysql.server — MySQL Server Startup Script
* mysqld_multi — Manage Multiple MySQL Servers

### Partitioning

#### Partitioning Types

* RANGE
* LIST
* COLUMNS
* HASH
* KEY
* subpartitioning

#### Restrictions & Limitations

* Query cache not supported
* Foreign keys not supported for partitioned InnoDB tables
* Partitioned tables do not support FULLTEXT indexes or searches
* Temporary tables cannot be partitioned
* A partitioning key must be either an integer or an expression that resolves to an integer

#### Performance Considerations

* Filesystem operations
* MyISAM and partition file descriptor usage
	* MySQL use 2 file descriptor for each partition
* Table locks
* Storage engine
	* generally tend to be faster with MyISAM tables than with InnoDB or NDB tables
* Indexes; partition pruning
* Performance with LOAD DATA

[Partitioning](http://dev.mysql.com/doc/refman/5.7/en/partitioning.html)

## Backup and Recovery

### InnoDB Backup and Recovery

#### Backup

Type            | Tool
----------------|-----
hot backup      | MySQL Enterprise Backup
cold backup     | copying files when sql server is down
physical backup | fast operation (esp. restore)
logical backup  | mysqldump

**Notes**

`mysqldump` utility is usually used under two circumstances:

* smaller data volumes
* record schema obj structure

##### Cold Backup

0. slow shutdown
0. copy ibdata files and .ibd files
0. copy .frm files
0. copy ib_logfile files
0. copy my.cnf configuration files

#### InnoDB Recovery Process

0. apply redo log
0. rolling back incomplete transactions
0. chagne buffer merge
0. perge

## High Availability

See [High Availability] for more details.

### MySQL Replication

#### Performance

* Multi-Threaded Slaves
* Binary Log Group Commit
* Optimized Row-Based Replication

#### Failover & Recovery

* Global Transaction Identifers
* Replication Failover & Admin Utilities
* Crash Safe Slaves & Binlogs

#### Data Integrity

* Replication Event Checksums

#### Dev/Ops Agility

* Replication Utilities
* Time-Delayed Replication
	* To protect against user mistakes on the master (DBA rollback to the time just before the disaster)
	* To test how the system behaves when there is a lag
	* To inspect what the database looked like long ago, without having to reload a backup. ( 1 week ago)
* Remote Binlog Backup
* Informational Log Events
* Server UUIDs

## Migration

### Inspect Table Size

```sql
SELECT table_name,
  data_length/1024/1024 AS 'data_length(MB)',
  index_length/1024/1024 AS 'index_length(MB)',
  (data_length + index_length)/1024/1024 AS 'total(MB)'
FROM information_schema.tables
WHERE table_schema='test' AND table_name = 't1';
```

### Tools

* logical migrate
	* mysqldump
	* mysqlpump (5.7)
	* mysqldumper
* MySQL Utilities
	* mysqldbexport
	* mysqldbimport
* SELECT * INTO OUTFILE * FROM table
* xtrabackup

It is recommond to use logical migration when the data size is small, otherwise phsical migration would performs better.

### mysqlpump

* Parallel processing of databases, and of objects within databases, to speed up the dump process
* Dumping of user accounts as account-management statements (CREATE USER, GRANT) rather than as inserts into the mysql system database
* Capability of creating compressed output
* Progress indicator (the values are estimates)
* For dump file reloading, faster secondary index creation for InnoDB tables by adding indexes after rows are inserted

### Copy *.ibd Files

* Large amount of data
* Temporary interrupt of service is acceptable

Prerequisite

innodb_file_per_table = 1 && engine = InnoDB

Create new table the same as old one:

```sql
CREATE DATABASE db2;
USE db2;
CREATE TABLE tbl LIKE db1.tbl;
```

Discard tablespace:

```sql
ALTER TABLE tbl DISCARD TABLESPACe;
```

Lock for export:

```sql
FLUSH TABLES tbl FOR EXPORT;
```

Copy *.ibd and *.cfg files

Release lock:

```sql
UNLOCK TABLES;
```

Import data:

```sql
ALTER TABLE tbl2 IMPORT TABLESPACE;
```

## MySQL Replication

### Master/Slave Replication

Either

MySQL Enterprise Backup (without taking down server)

or

Cold Backup.

MySQL Replication is based on binlog.

Transactions that fails on the master do not affect replication at all.

**Note**

Replication and CASCADE foreign key cautious

### Master/Master Replication

my.cnf

```conf
[mysqld]
server-id = 1 # 2 for backup
log-bin = mysql-bin
auto-increment-increment = 2
auto-increment-offset = 1
slave-skip-errors = all
```

```sql
show master status;
+-----------------+----------+--------------+------------------+-------------------+
| File            | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------+----------+--------------+------------------+-------------------+
| rmbp-bin.000001 |      154 |              |                  |                   |
+-----------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

setup master

```sql
GRANT  REPLICATION SLAVE ON *.* TO 'replication'@'192.168.0.%' IDENTIFIED  BY 'replication';
flush  privileges;
change  master to
    master_host='192.168.0.17{2,4}',
    master_user='replication',
    master_password='replication',
    master_log_file='rmbp-bin.000001',
    master_log_pos=106;
start  slave;
```

Verify
: check if Slave_IO_Running/Slave_SQL_Running is both YES

```sql
show slave status;
```

Keepalived
: setup hot standby backup

## Upgrade

### In-place Upgrade

0. shut down the old MySQL version
0. replace the old MySQL binaries with the new ones
0. restart MySQL on the existing data directory
0. run `mysql_upgrade`

### Logical Upgrade

0. export existing data from the old MySQL version using `mysqldump`

   ```sql
   mysqldump -u root -p database > database.sql
   ```

0. install the new MySQL version
0. loading the dump file into the new MySQL version

   ```sql
   mysql -u root -p database < database.sql
   ```

0. run `mysql_upgrade`

   ```sql
   mysql_upgrade
   ```

## Downgrade

## References

* [OLTP vs OLAP](http://datawarehouse4u.info/OLTP-vs-OLAP.html)
* [option files](https://dev.mysql.com/doc/refman/5.7/en/option-files.html)
* [Overview of MySQL Programs](https://dev.mysql.com/doc/refman/5.7/en/programs-overview.html)
* [MySQL Server and Server-Startup Programs](https://dev.mysql.com/doc/refman/5.7/en/programs-server.html)

[High Availability]: 