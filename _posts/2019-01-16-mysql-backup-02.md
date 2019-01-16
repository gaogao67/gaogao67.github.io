---
layout: post
title:  MySQL备份--xtrabckup介绍
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Backup
tag: MySQL-Backup
---

* content
{:toc}


工具介绍					{#ch1}
====================================

```

由Percona公司研发的开源热备工具，支持MYSQL 5.0 以上版本。
由于Xtrabackup支持备份innodb表，实际生产环境中我们使用的工具是innobackupex，它是对xtrabackup的一层封装。innobackupex 脚本用来备份非 InnoDB 表，同时会调用 xtrabackup 命令来备份 InnoDB 表，innobackupex的基本流程如下：
1.开启redo日志拷贝线程，从最新的检查点开始顺序拷贝redo日志；
2.开启idb文件拷贝线程，拷贝innodb表的数据
3.idb文件拷贝结束，通知调用FTWRL，获取一致性位点
4.备份非innodb表(系统表)和frm文件
5.由于此时没有新事务提交，等待redo日志拷贝完成
6.最新的redo日志拷贝完成后，相当于此时的innodb表和非innodb表数据都是最新的
7.获取binlog位点，此时数据库的状态是一致的。
8.释放锁，备份结束。

参考连接：https://www.percona.com/doc/percona-xtrabackup/2.4/index.html

```


备份流程                  {#ch2}
====================================
![/styles/pic/mysql_backup/001.png]({{ '/styles/pic/mysql_backup/001.png'|prepend:site.baseurl}})
![/styles/pic/mysql_backup/002.png]({{ '/styles/pic/mysql_backup/002.png'|prepend:site.baseurl}})



安装部署                  {#ch3}
====================================
安装依赖包和rpm安装xtrabackup：
```bash

## 安装依赖包
yum -y install perl perl-devel libaio libaio-devel perl-Time-HiRes perl-DBD-MySQL rsync

## 安装libev4包
rpm -ivh libev4-4.15-7.1.x86_64.rpm

## 安装percona-xtrabackup
rpm -ivh percona-xtrabackup-24-2.4.4-1.el6.x86_64.rpm


```

创建备份账号：
```sql

## xtrabackup备份创建备份用户
CREATE USER 'backup'@'localhost' IDENTIFIED BY 'backup@123';
GRANT SELECT, RELOAD, PROCESS, SHOW DATABASES, SUPER, LOCK TABLES, REPLICATION CLIENT, SHOW VIEW, EVENT ON *.* TO 'backup'@'localhost';
FLUSH PRIVILEGES;


```

Percona Server版本优化                  {#ch4}
====================================
```

从逻辑备份和物理备份来看，无论是哪种备份工具，为了获取一致性位点，都强依赖于FTWRL。这个锁杀伤力非常大，因为持有锁的这段时间，整个数据库实质上不能对外提供写服务的。此外，由于FTWRL需要关闭表，如有大查询，会导致FTWRL等待，进而导致DML堵塞的时间变长。即使是备库，也有SQL线程在复制来源于主库的更新，上全局锁时，会导致主备库延迟。从前面的分析来看，FTWRL这把锁持有的时间主要与非innodb表的数据量有关，如果非innodb表数据量很大，备份很慢，那么持有锁的时间就会很长。即使全部是innodb表，也会因为有mysql库系统表存在，导致会锁一定的时间。为了解决这个问题，Percona公司对Mysql的Server层做了改进，引入了BACKUP LOCK，具体而言，通过"LOCK TABLES FOR BACKUP"命令来备份非innodb表数据；通过"LOCK BINLOG FOR BACKUP"来获取一致性位点，尽量减少因为数据库备份带来的服务受损。我们看看采用这两个锁与FTWRL的区别：
 
LOCK TABLES FOR BACKUP
作用：备份数据
1.禁止非innodb表更新
2.禁止所有表的ddl
优化点：
1.不会被大查询堵塞(关闭表)
2.不会堵塞innodb表的读取和更新，这点非常重要，对于业务表全部是innodb的情况，则备份过程中DML完全不受损
UNLOCK TABLES



LOCK BINLOG FOR BACKUP
作用：获取一致性位点。
1.禁止对位点更新的操作
优化点：
1.允许DDl和更新，直到写binlog为止。
UNLOCK BINLOG

```

失败案例                  {#ch4}
====================================
1、磁盘性能太差导致xtrabckup备份失败
```

xtrabackup: error: log block numbers mismatch:
xtrabackup: error: expected log block no. 201901064, but got no. 208192508 from the log file.
xtrabackup: error: it looks like InnoDB log has wrapped around before xtrabackup could process all records due to either log copying being too slow, or  log files being too small.
xtrabackup: Error: xtrabackup_copy_logfile() failed.

原因：
Innodb产生日志的速度远超于Xtrabackup复制的速度，部分Innodb日志被截断，导致备份失败。

```

2、DDL导致Xtrabackup备份失败
```

当MySQL使用xrabckup进行备份时，如果执行DDL进行表修改，会导致xrabckup备份失败。

错误类似于：
InnoDB: Last flushed lsn: 3375345258517 load_index lsn 3379255303757
InnoDB: An optimized (without redo logging) DDLoperation has been performed. All modified pages may not have been flushed to the disk yet. 
PXB will not be able take a consistent backup. Retry the backup operation.

```