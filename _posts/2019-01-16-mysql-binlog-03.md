---
layout: post
title:  MySQL Binlog--Binlog和Relaylog的生成与删除
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Binlog
tag: MySQL-Binlog
---

* content
{:toc}


Binlog和Relaylog的生成与删除                 {#ch1}
====================================
```

##=====================================================================##
binlog文件生成：
在每条二进制日志写入到日志文件后，会判断该文件是否超过max_binlog_size，如果超过则生成一个新的binlog

##=====================================================================##
binlog文件删除：
1>当使用RESET MASTER命令后，会清空全部二进制日志
命令：RESET MASTER;

2>当执行PURGE MASTER LOG TO命令后，会删除指定binlog以及之前的二进制日志
命令：PURGE MASTER LOGS TO 'binlog file name';

3>当执行PURGE MASTER LOG BEFORE 命令后，会删除指定时间前的所有二进制
命令：PURGE MASTER LOGS TO 'datetime';

4>当实例启动或执行flush logs时，按照expire_logs_days设置，如果超过该参数指定天数的二进制会被全部删除
命令：mysqladmin flush-log

##=======================================================##
清理binlog文件顺序：
先从文件系统中清理文件,再修改索引文件。


##=======================================================##
Relay Log rotate 机制：
Rotate：每从Master fetch一个events后，判断当前文件是否超过max_relay_log_size 如果超过则自动生成一个新的relay-log-file
Delete： purge-relay-log 在SQL Thread每执行完一个events时判断，如果该relay-log 已经不再需要则自动删除
Delete： expire-logs-days 只在 实例启动时 和 flush logs 时判断，如果文件访问时间早于设定值，则purge file （同Binlog file） (updated: expire-logs-days和relaylog的purge没有关系) 
PS： 因此还是建议配置 expire-logs-days ， 否则当我们的外部脚本因意外而停止时，还能有一层保障。

##=======================================================##

```

