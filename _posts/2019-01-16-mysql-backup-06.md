---
layout: post
title:  MySQL备份--mydumper介绍
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Backup
tag: MySQL-Backup
---

* content
{:toc}



多线程备份原理                  {#ch1}
====================================
```bash

1、主线程使用FTWRL(FLUSH TABLES WITH READ LOCK)锁定整个实例
2、主线程保存当前时间点二进制日志文件的信息(文件名和位置点)
3、创建N个线程，每个线程使用START TRANSACTION WITH CONSISTENT SNAPSHOT开启事务，由于整实例数据被锁定，新线程获得和主线程一样的ReadView。
4、使用N个线程备份非事务引擎表（如MyISAM表），由于整实例数据被锁定，非事务引擎表无法被更新，能获得多表数据一致性。
5、非事务引擎表备份结束，主线程执行UNLOCK TABLES释放FTWRL获得的全局锁
6、使用N个线程备份事务引擎表，由于线程已开启可重复读事务隔离级别的事务，因此在整个备份区间对事务引擎表获得多表数据一致性。
7、所有数据表备份完成，子线程结束，主线程执行完成。

```
![/styles/pic/mysql_backup/006.png]({{ '/styles/pic/mysql_backup/006.png'|prepend:site.baseurl}})

备份文件结构                  {#ch2}
====================================
```

mydumper将所有备份文件存放到指定目录中，其中metadata文件存放备份时间点SHOW MASTER STATUS的binlog信息和SHOW SLAVE STATUS的主库执行进度信息。
mydumper对每个备份表创建两个备份文件，database.table-schema.sql存放表结构信息，database.table.sql存放表数据文件。如果对表文件进行分片，会生产多个表数据文件。

```

脚本模板                  {#ch3}
====================================
```bash

## 备份脚本指定数据库
mydumper_exe="/usr/bin/mydumper"
mysql_host="127.0.0.1"
mysql_port=3358
mysql_user="backup"
mysql_password="XXXXXXXXXXXXXX"
mydumper_data_dir="/data/mysql_backup/data/"
mydumper_log="/data/mysql_backup/log/mydumper.log"
database='test1'


${mydumper_exe} \
--host="${mysql_host}" \
--port=${mysql_port} \
--user="${mysql_user}" \
--password="${mysql_password}" \
--events --routines --triggers \
--trx-consistency-only \
--compress --verbose=3 \
--complete-insert \
--logfile="${mydumper_log}" \
--database $database \
--outputdir="${mydumper_data_dir}"

```


下载地址                  {#ch4}
====================================
源码地址：https://github.com/maxbube/mydumper
rpm地址：https://github.com/maxbube/mydumper/releases
rpm包安装路径：/usr/bin/mydumper