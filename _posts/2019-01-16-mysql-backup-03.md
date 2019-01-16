---
layout: post
title:  MySQL备份--xtrabckup脚本模板
date:   2019-01-16 15:00:00 +0800
categories: MySQL Backup
tag: MySQL Backup
---

* content
{:toc}


备份语法					{#ch1}
====================================

```

innobackup [--sleep=MS] [--compress[=LEVEL]] [--include=REGEXP] [--user=NAME]
             [--password=WORD] [--port=PORT] [--socket=SOCKET] [--no-timestamp]
             [--ibbackup=IBBACKUP-BINARY] [--slave-info] [--stream=tar]
             [--defaults-file=MY.CNF]
             [--databases=LIST] [--remote-host=HOSTNAME] BACKUP-ROOT-DIR

innobackup --apply-log [--use-memory=MB] [--uncompress] [--defaults-file=MY.CNF]
             [--ibbackup=IBBACKUP-BINARY] BACKUP-DIR

innobackup --copy-back [--defaults-file=MY.CNF] BACKUP-DIR

```


常用参数                  {#ch2}
====================================
```

--slave_info
在从库上备份时，使用--slave_info参数来保存主库的binlog信息到xtrabackup_slave_info文件中。
PS: 默认会将当前备份库的binlog信息保存到xtrabackup_binlog_info文件中。

--apply-log
应用备份文件夹中名为xtrabackup_logfile的事务日志，并根据名为backup-my.cnf的配置文件来创建新的事务日志文件。

--redo-only
当需要增量还原时配合--apply-log参数使用，通过--redo-only来控制未提交事务不发生回滚，以便继续还原后续备份。（类似于SQL Server中的WITH NORECOVERY 选项）

--use-memory=1G
当使用--apply-log处理undo log时，通过参数--use-memory来控制恢复过程使用的内存大小。

--copy-back         
Copy all the files in a previously made backup from the backup directory to their original locations.

--move-back         
Move all the files in a previously made backup from the backup directory to the actual datadir location. Use with caution, as it removes backup files.

--no-timestamp
使用--no-timestamp选项来阻止命令自动创建一个以时间命名的目录,将会创建一个BACKUP-DIR目录来存储备份数据。
未指定--no-timestamp参数时，备份会创建一个以时间命名的目录，并将备份数据写入到该目录下。

--defaults-file=#
读取指定的my.cnf文件
Xtrabackup默认情况会去读my.cnf文件，读取顺序是/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf ~/.my.cnf

--tmpdir=DIRECTORY
当有指定--remote-host or --stream时, 事务日志临时存储的目录, 默认采用 MySQL 配置文件中所指定的临时目录tmpdir

--parallel=4
用于指定在copy操作时的线程数

–compress-threads
用于指定在压缩时的并发数

```



常用脚本                  {#ch3}
====================================
安装依赖包和rpm安装xtrabackup：
```bash

##======================================================================##
## 进行完整备份
innobackupex --defaults-file="/export/servers/mysql/etc/my.cnf" \
--host="localhost" \
--port=3358 \
--user="backuper" \
--password="backup@123" \
--socket="/export/data/mysql/tmp/mysql.sock" \
"/export/mysql_backup/"


##======================================================================##
## 进行完整备份并压缩
innobackupex --defaults-file="/export/servers/mysql/etc/my.cnf" \
--host="localhost" \
--port=3358 \
--user="backuper" \
--password="backup@123" \
--socket="/export/data/mysql/tmp/mysql.sock" \
--stream=tar \
"/export/mysql_backup/" > "/export/data/mysql/dumps/full_backup.tar"




##======================================================================##
### 使用parallel参数来进行多线程备份
innobackupex --defaults-file="/export/servers/mysql/etc/my.cnf" \
--host="localhost" \
--port=3358 \
--user="backup" \
--password="backup@123" \
--socket="/export/data/mysql/tmp/mysql.sock" \
--parallel=4 \
"/export/mysql_backup/"


##======================================================================##
## 对备份文件进行apply-log操作
innobackupex --defaults-file="/export/servers/mysql/etc/my.cnf" --apply-log /export/mysql_backup/2016-04-28_17-54-45


##======================================================================##
## 对备份文件进行copy-back操作
innobackupex --defaults-file="/export/servers/mysql/etc/my.cnf" --copy-back /export/mysql_backup/



参考资料：
http://www.percona.com/docs/wiki/percona-xtrabackup:xtrabackup_manual


```

远程备份                  {#ch4}
====================================
可以使用tar或xbstream方式将数据流式备份到远程服务器。

1、xbstream方式备份到远程服务器
```

##xbstream 备份到远程服务器
innobackupex \
--defaults-file="/export/servers/mysql/etc/my.cnf" \
--host="localhost" \
--port=3358 \
--user="backup" \
--password="backup@123" \
--stream=xbstream "/export/mysql_backup/" \
| ssh root@10.0.0.2 \
"gzip ->/export/mysql_backup/mysql_backup.gz"

##解压备份
gzip -d mysql_backup.gz

##使用xbstream解压
xbstream -x < mysql_backup

```

2、tar方式备份到远程服务器
```

##tar 备份到远程服务器
innobackupex \
--defaults-file="/export/servers/mysql/etc/my.cnf" \
--host="localhost" \
--port=3358 \
--user="backup" \
--password="backup@123" \
--stream=tar "/export/mysql_backup/" \
| ssh root@10.0.0.2 \
"gzip ->/export/mysql_backup/mysql_backup.tar.gz"


##使用tar解压
tar -ixzvf mysql_backup.tar.gz

```