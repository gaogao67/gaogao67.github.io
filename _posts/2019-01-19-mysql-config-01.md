---
layout: post
title:  MySQL参数配置之innodb_force_recovery
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Config
tag: MySQL参数配置
---

* content
{:toc}


参数说明				{#ch1}
====================================
```

当MySQL因为异常关机后，数据库无法正常恢复，可以通过修改innodb_force_recovery参数，使mysqld跳过恢复步骤，将mysqld 启动,将数据导出来然后重建数据库。
innodb_force_recovery可以设置为1-6,大的数字包含前面所有数字的影响。
1. (SRV_FORCE_IGNORE_CORRUPT):忽略检查到的corrupt页。
2. (SRV_FORCE_NO_BACKGROUND):阻止主线程的运行，如主线程需要执行full purge操作，会导致crash。
3. (SRV_FORCE_NO_TRX_UNDO):不执行事务回滚操作。
4. (SRV_FORCE_NO_IBUF_MERGE):不执行插入缓冲的合并操作。
5. (SRV_FORCE_NO_UNDO_LOG_SCAN):不查看重做日志，InnoDB存储引擎会将未提交的事务视为已提交。
6. (SRV_FORCE_NO_LOG_REDO):不执行前滚的操作。

PS:
1、当设置参数值大于0后，可以对表进行select,create,drop操作,但insert,update或者delete这类操作是不允许的。
2、当innodb_purge_threads 和 innodb_force_recovery一起设置会出现一种loop现象.


```

摘抄自：http://blog.itpub.net/22664653/viewspace-1441389/  


