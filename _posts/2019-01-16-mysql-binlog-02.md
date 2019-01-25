---
layout: post
title:  MySQL Binlog--基础知识
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Binlog
tag: MySQL-Binlog
---

* content
{:toc}


查看MySQL Binlog信息                  {#ch1}
====================================
```

##=====================================##
## 在MySQL内部查看binlog文件列表 ##
SHOW BINARY LOGS;

##=====================================##
##查看某个binglog文件中特定pos的操作
SHOW BINLOG EVENTS IN 'mysql-bin.000011' FROM 4742885 LIMIT 15;

##=====================================##
## 使用mysqlbinlog查看binlog ##
## 可以使用--start-datetime --stop-datetime  --start-position --stop-position 来过滤，
##  如果不能提供准确的pos值，则会报错，因此主要通过时间来进行过滤
##  过滤时间格式为'2004-12-25 11:25:56'
/export/servers/mysql/bin/mysqlbinlog -vv /export/data/mysql/data/mysql-bin.008735

/export/servers/mysql/bin/mysqlbinlog -vv /export/data/mysql/data/mysql-bin.008735 --start-datetime='2017-01-01 00:00:00' --stop-datetime='2017-01-02 00:00:00'

/export/servers/mysql/bin/mysqlbinlog -vv /export/data/mysql/data/mysql-bin.008735 --start-position=194 --stop-position=201

##=====================================##
## 查看binlog 文件大小和最后修改时间 ##
ll -h --time-style='+%Y-%m-%d %H:%M:%S' /export/data/mysql/data/mysql-bin*

```

```

BINLOG和REDO/UNDO LOG的区别
1、处理层次不同，REDO/UNDO LOG由Innodb存储引擎处理，而BINLOG由MySQL 服务层处理。
2、记录内容不同，REDO/UNDO LOG记录的数据页的修改情况，REDO LOG采用物理日志+逻辑日志的方式存储，UNDO LOG采用逻辑日志方式存储，用于保证数据一致性；而BINLOG日志记录的事务操作的内容，用于主从复制。
3、记录时机不同，REDO/UNDO LOG在事务的执行过程中不断生成和写入，而BINLOG在事务最终COMMIT前写入。
4、涉及到数据更新的SELECT操作会被记录到BINLOG中。
5、BINLOG刷新到磁盘的行为由参数sync_binlog决定，而REDO LOG写入磁盘的行为受参数innodb_flush_log_at_trx_commit的影响。

```


MySQL Binlog常用参数                  {#ch2}
====================================
```

log_bin
设置此参数表示启用binlog功能，并指定路径名称

log_bin_index
设置此参数是指定二进制索引文件的路径与名称

binlog_do_db
此参数表示只记录指定数据库的二进制日志

binlog_ignore_db
此参数表示不记录指定的数据库的二进制日志

max_binlog_cache_size
此参数表示binlog使用的内存最大的尺寸

binlog_cache_size
此参数表示binlog使用的内存大小，可以通过状态变量binlog_cache_use和binlog_cache_disk_use来帮助测试。

binlog_cache_use：使用二进制日志缓存的事务数量

binlog_cache_disk_use:使用二进制日志缓存但超过binlog_cache_size值并使用临时文件来保存事务中的语句的事务数量

max_binlog_size
Binlog最大值，最大和默认值是1GB，该设置并不能严格控制Binlog的大小，尤其是Binlog比较靠近最大值而又遇到一个比较大事务时，为了保证事务的完整性，不可能做切换日志的动作，只能将该事务的所有SQL都记录进当前日志，直到事务结束

sync_binlog
该参数直接影响mysql的性能和完整性

sync_binlog=0：
当事务提交后，Mysql仅仅是将binlog_cache中的数据写入Binlog文件，但不执行fsync之类的磁盘        同步指令通知文件系统将缓存刷新到磁盘，而让Filesystem自行决定什么时候来做同步，这个是性能最好的。

sync_binlog=n，在进行n次事务提交以后，Mysql将执行一次fsync之类的磁盘同步指令，同志文件系统将Binlog文件缓存刷新到磁盘。
Mysql中默认的设置是sync_binlog=0，即不作任何强制性的磁盘刷新指令，这时性能是最好的，但风险也是最大的。一旦系统绷Crash，在文件系统缓存中的所有Binlog信息都会丢失

```



会话级别log_bin参数                  {#ch3}
====================================
```

在MySQL配置文件中使用log_bin=1来设置MySQL生成binlog文件。
而如果希望当前回话执行的命令不写入binlog文件，可以使用SQL_LOG_BIN参数来设置。
SET SESSION SQL_LOG_BIN=0
如果设置GLOBAL级别SQL_LOG_BIN，会影响所有回话写binlog日志，谨慎使用！

```


binlog_rows_query_log_events参数 {#ch4}
====================================
```

默认配置下，ROW格式二进制日志只记录数据发生的变化，并不会记录什么语句导致数据发生变化，而出于审计或者处理bug的需求，需要了解导致数据变化的SQL语句，MYSQL提供了binlog_rows_query_log_events来控制是否在二进制中存放"原始SQL"。

binlog_rows_query_log_events参数默认不启用。

The binlog_rows_query_log_events system variable affects row-based logging only. When enabled, it causes the MySQL Server to write informational log events such as row query log events into its binary log. This information can be used for debugging and related purposes; such as obtaining the original query issued on the master when it cannot be reconstructed from the row updates. 

These events are normally ignored by MySQL programs reading the binary log and so cause no issues when replicating or restoring from backup.

```
![/styles/pic/mysql_binlog/001.png]({{ '/styles/pic/mysql_binlog/001.png'|prepend:site.baseurl}})
当binlog_rows_query_log_events开启后，通过解析出Binlog结构可以找到都导致数据发生变化的SQL语句。