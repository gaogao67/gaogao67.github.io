---
layout: post
title:  MySQL备份--mysqldump中single-transaction参数
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Backup
tag: MySQL-Backup
---

* content
{:toc}


官网说明					{#ch1}
====================================

```

Creates a consistent snapshot by dumping all tables in a
single transaction. Works ONLY for tables stored in
storage engines which support multiversioning (currently
only InnoDB does); the dump is NOT guaranteed to be
consistent for other storage engines. While a
--single-transaction dump is in process, to ensure a
valid dump file (correct table contents and binary log
position), no other connection should use the following
statements: ALTER TABLE, DROP TABLE, RENAME TABLE,
TRUNCATE TABLE, as consistent snapshot is not isolated
from them. Option automatically turns off --lock-tables.

```


操作步骤                  {#ch2}
====================================
在mysqldump中指定single-transaction时，mysqldump的操作步骤如下：
```sql

FLUSH TABLES;
FLUSH TABLES WITH READ LOCK;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION WITH CONSISTENT SNAPSHOT;
SHOW MASTER STATUS;
UNLOCK TABLES;

SHOW TABLES LIKE 'xxx'
SET OPTION SQL_QUOTE_SHOW_CREATE=1
SHWO CREATE TABLE 'xxx'
SHOW FIELDS FROM 'xxx'
SHOW TABLE STATUS LIKE 'xxx'
SELECT /*!40001 SQL_NO_CACHE */ * FROM  xxx

QUIT

```

通过FLUSH TABLES WITH READ LOCK来锁定所有表，然后开启事务，由于外部事务不能对表数据进行修改，因此SHOW MASTER STATUS的数据不会发生变化，而由于事务隔离级别为REPEATABLE READ，因此在整个mysqldump过程中，获取到的数据为开始事务时的数据，因此可以保证mysqldump出来的数据一致性，并可以结合SHOW MASTER STATUS出来的数据进行恢复。


参考连接：http://imysql.cn/2008_10_24_deep_into_mysqldump_options


DLL操作影响               {#ch3}
====================================
single-transaction依赖InnoDB事务引擎的多版本控制机制来实现数据一致性，而DDL操作不属于事务操作，会破坏事务一致性，因此在使用mysqldump --single-transaction进行备份过程中其他会话执行DDL操作，会导致备份出错或被阻塞。




在MySQL 5.6版本中，测试场景1：mysqldump开始但尚未备份到表tb001时，另外回话对表tb001进行alter操作，然后mysqldump对表tb001进行导出：
![/styles/pic/mysql_backup/004.png]({{ '/styles/pic/mysql_backup/004.png'|prepend:site.baseurl}})


在MySQL 5.6版本中，测试场景2：mysqldump开始备份并完成tb001的导出，在对其他表进行导出过程中，其他回话对表进行alter操作：
![/styles/pic/mysql_backup/005.png]({{ '/styles/pic/mysql_backup/005.png'|prepend:site.baseurl}})


在MySQL 5.6版本中，测试场景3：mysqldump开始但尚未备份到表tb001时，另外回话对表tb001进行alter操作，然后mysqldump对表tb001进行导出：
![/styles/pic/mysql_backup/003.png]({{ '/styles/pic/mysql_backup/003.png'|prepend:site.baseurl}})


MySQL 5.5版本进行mysqldump备份和DLL操作：
```

1、如果修改表操作在 ”mysqldump开启后但还未导出修改表数据前“ 的时间段内开始，则修改表操作成功完成，而mysqldump不会执行失败，但是无法正常导出修改表的数据；

2、如果修改表操作在 “mysqldum已导出修改表数据但还未结束mysqldump操作前”的时间段内开始，则修改表操作被阻塞，mysqldum能成功完成，在mysqldump操作完成后修改表操作方可正常执行。
 
对于MySQL 5.5版本，应该避免mysqldump和修改表操作同时进行，以避免备份显示正常但实际数据丢失的问题。

```

MySQL 5.6版本进行mysqldump备份和DLL操作：
```

1、如果修改表操作在 ”mysqldump开启后但还未导出修改表数据前“ 的时间段内开始，则修改表操作成功完成，而mysqldump会执行失败；

2、如果修改表操作在 “mysqldum已导出修改表数据但还未结束mysqldump操作前”的时间段内开始，则修改表操作被阻塞，mysqldum能成功完成，在mysqldump操作完成后修改表操作方可正常执行。

对于MySQL 5.6版本，DDL操作和mysqldump备份能并行执行且正常完成，但为避免相互阻塞，也建议调整DDL操作和mysqldump备份的执行窗口。

```



