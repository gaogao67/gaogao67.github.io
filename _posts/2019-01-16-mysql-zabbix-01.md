---
layout: post
title:  MySQL Zabbix--常见问题处理
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Zabbix
tag: MySQL-Zabbix
---

* content
{:toc}


No DBD::mysql found                 {#ch1}
====================================
错误日志：
```
mpm日志FromDualMySQLagent.log报错：
33754:2019-01-25 14:57:44.022 - WARN: Warning: No DBD::mysql found. Please install DBD::mysql (rc=1312).
33754:2019-01-25 14:57:44.022 - INFO: FromDual Performance Monitor for MySQL run finshed (rc=1312).

zabbix日志：
Use of uninitialized value $pLogLevel in numeric ge (>=) at /usr/local/mpm/lib/FromDualMySQLagent.pm line 674.

```

解决办法：重装perl-DBD-MySQL
```
rpm -e --nodeps perl-DBD-MySQL

yum -y install perl-DBI perl-DBD-MySQL perl-Time-HiRes perl-IO-Socket-SSL perl-TermReadKey

```


