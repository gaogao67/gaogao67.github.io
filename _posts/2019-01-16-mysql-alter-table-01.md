---
layout: post
title:  MySQL Alter Table注意事项
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Alter-Table
tag: MySQL-AlterTable
---

* content
{:toc}

ALTER TABLE 和FLUSH TABLE导致的间接等待
====================================
```

场景：
1、会话A执行耗时较长的操作；
2、会话B执行ALTER TABLE 或FLUSH TABLES等操作时，会向其他会话(线程)发送表变更通知，要求其他会话关闭再重新打开相关表；
3、会话A执行过程中收到会话B的变更通知，在会话A执行结束前，会话A阻塞会话B执行；
4、会话C收到会话B的变更通知，等待会话B完成，形成等待链：会话C>>会话B>>会话A
5、会话B因为其他原因执行失败或被关闭,ALTER TABLE或FLSUH TABLE等操作被取消
6、会话C转为等待会话A执行完成。

官方文档：
The thread got a notification that the underlying structure for a table has changed and it needs to reopen the table to get the new structure. However, to reopen the table, it must wait until all other threads have closed the table in question.

This notification takes place if another thread has used FLUSH TABLES or one of the following statements on the table in question: FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, orOPTIMIZE TABLE.

```


ALTER TABLE过程报主键重复
====================================
```

在MySQL 5.6版本中引入Online DDL特性，很多ALTER TABLE操作可以联机修改，该特性使用ROW LOG来保存ALTER操作期间发生的数据变化，并回放到新表中，保证数据一致性。如果在业务高峰期执行Online DDL操作，可能报下面错误：
ERROR 1062 (23000) at line 13: Duplicate entry 'xxx' for key 'PRIMARY'

解决办法：
1、将DDL操作移到业务低峰期执行，降低错误出现概率
2、如果业务允许阻塞，修改ALTER TABLE语句使用阻塞方式执行
如ALTER命令为：
ALTER TABLE TB001 ADD C2 INT;
修改为：
ALTER TABLE TB001 ADD C2 INT, ALGORITHM =COPY;

```


修改表名
====================================
```
修改表名有两种语法：
RENAME TABLE old_table TO new_table;
或
ALTER TABLE old_table RENAME new_table;


使用RENAME TABLE方式可以一次修改多个表的表名，MySQL保证RENAME操作的原子性。
When you execute RENAME TABLE, you cannot have any locked tables or active transactions. With that condition satisfied, the rename operation is done atomically; no other session can access any of the tables while the rename is in progress. 

```

DROP TABLE与MySQL版本
====================================
```
MySQL在5.5版本中引入自适应hash索引，用于提升经常访问的数据页的性能，在删除表时，需要先通过扫描LRU链表找到该表在自适应hash索引使用的数据页，将这些数据从自适应hash索引中删除。如果为MySQL实例配置较多的物理内存，扫描自适应hash索引的LRU链表可能会导致数据库性能异常甚至数据库Crash。

MySQL 5.7版本的官方文档如下描述：
On a system with a large InnoDB buffer pool and innodb_adaptive_hash_index enabled,TRUNCATE TABLE operations may cause a temporary drop in system performance due to an LRU scan that occurs when removing an InnoDB table's adaptive hash index entries.
The problem was addressed for DROP TABLE in MySQL 5.5.23 (Bug #13704145, Bug #64284) but remains a known issue for TRUNCATE TABLE (Bug #68184).


对MySQL 5.6之前版本，建议使用TRUNCATE TABLE+DROP TABLE来代替直接DROP TABLE。
对MySQL 5.6及之后版本，建议直接使用DROP TABLE来删除表。


参考链接：
https://dev.mysql.com/doc/refman/5.7/en/truncate-table.html
https://yq.aliyun.com/articles/570032?spm=a2c4e.11155435.0.0.234e998bdGFhGB 


```
