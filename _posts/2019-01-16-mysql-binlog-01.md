---
layout: post
title:  MySQL Binlog--binlog_format参数
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Binlog
tag: MySQL-Binlog
---

* content
{:toc}


binlog_format参数                  {#ch1}
====================================
```

binlog_format 
在mysql 5.1 版本前，所有二进制文件的格式都是基于SQL语句级别的，在mysql 5.1 版本后引入binlog_format参数，可以设置为STATEMENT\ROW\MIXED 

ROW 
日志中会记录成每一行数据被修改的形式，然后在 slave 端再对相同的数据进行修改。 

Statement 
每一条会修改数据的 SQL 都会记录到 master 的 bin-log 中。slave 在复制的时候 SQL 进程会解析成和原来 master 端执行过的相同的 SQL 再次执行。 

Mixed 
在 Mixed 模式下，MySQL 会根据执行的每一条具体的 SQL 语句来区分对待记录的日志形式，也就是在 statement 和 row 之间选择一种。新版本中的 statment 还是和以前一样，仅仅记录执行的语句。而新版本的 MySQL 中对 row 模式也被做了优化，并不是所有的修改都会以 row 模式来记录，比如遇到表结构变更的时候就会以 statement 模式来记录，如果 SQL 语句确实就是 update 或者 delete 等修改数据的语句，那么还是会记录所有行的变更。

binlog_format是动态参数，可以在运行环境下进行修改，除以下几种情况外，在运行时可以动态改变 binlog 的格式： 
1>存储流程或者触发器中间； 
2>启用了 NDB； 
3>当前会话使用 row 模式，并且已打开了临时表； 

```



混合格式(mixed)                  {#ch2}
====================================

```

如果 binlog 采用了 Mixed 模式，那么在以下几种情况下会自动将 binlog 的模式由 statement 模式变为 row 模式： 
1>当 DML 语句更新一个 NDB 表时； 
2>当函数中包含 UUID() 时； 
3>2个及以上包含 AUTO_INCREMENT 字段的表被更新时； 
4>执行 INSERT DELAYED 语句时； 
5>用 UDF 时； 
6>视图中必须要求运用 row 时，例如建立视图时使用了 UUID() 函数； 

```


语句格式(statement)优缺点                  {#ch3}
====================================
```

Statement 优点 
历史悠久，技术成熟； 
产生的 binlog 文件较小； 
binlog 中包含了所有数据库修改信息，可以据此来审核数据库的安全等情况； 
binlog 可以用于实时的还原(操作行为)，而不仅仅用于复制； 
主从版本可以不一样，从服务器版本可以比主服务器版本高； 


Statement 缺点： 
不是所有的 UPDATE 语句都能被复制，尤其是包含不确定操作的时候； 
调用具有不确定因素的 UDF 时复制也可能出现问题； 
运用以下函数的语句也不能被复制： 
* LOAD_FILE() 
* UUID() 
* USER() 
* FOUND_ROWS() 
* SYSDATE() (除非启动时启用了 –sysdate-is-now 选项) 
INSERT … SELECT 会产生比 RBR 更多的行级锁； 
复制须要执行全表扫描 (WHERE 语句中没有运用到索引) 的 UPDATE 时，须要比 row 请求更多的行级锁； 
对于有 AUTO_INCREMENT 字段的 InnoDB 表而言，INSERT 语句会阻塞其他 INSERT 语句； 
对于一些复杂的语句，在从服务器上的耗资源情况会更严重，而 row 模式下，只会对那个发生变化的记录产生影响； 
存储函数(不是存储流程 )在被调用的同时也会执行一次 NOW() 函数，这个可以说是坏事也可能是好事； 
确定了的 UDF 也须要在从服务器上执行； 
数据表必须几乎和主服务器保持一致才行，否则可能会导致复制出错； 
执行复杂语句如果出错的话，会消耗更多资源； 

```

行格式(row)优缺点                  {#ch4}
====================================
```

Row 优点 
任何情况都可以被复制，这对复制来说是最安全可靠的； 
和其他大多数数据库系统的复制技能一样； 
多数情况下，从服务器上的表如果有主键的话，复制就会快了很多； 
复制以下几种语句时的行锁更少： 
* INSERT … SELECT 
* 包含 AUTO_INCREMENT 字段的 INSERT 
* 没有附带条件或者并没有修改很多记录的 UPDATE 或 DELETE 语句 
执行 INSERT，UPDATE，DELETE 语句时锁更少； 
从服务器上采用多线程来执行复制成为可能； 


Row 缺点 
生成的 binlog 日志体积大了很多； 
复杂的回滚时 binlog 中会包含大量的数据； 
主服务器上执行 UPDATE 语句时，所有发生变化的记录都会写到 binlog 中，而 statement 只会写一次，这会导致频繁发生 binlog 的写并发请求； 
UDF 产生的大 BLOB 值会导致复制变慢； 
不能从 binlog 中看到都复制了写什么语句(加密过的)； 
当在非事务表上执行一段堆积的 SQL 语句时，最好采用 statement 模式，否则很容易导致主从服务器的数据不一致情况发生； 
另外，针对系统库 MySQL 里面的表发生变化时的处理准则如下： 
如果是采用 INSERT，UPDATE，DELETE 直接操作表的情况，则日志格式根据 binlog_format 的设定而记录； 
如果是采用 GRANT，REVOKE，SET PASSWORD 等管理语句来做的话，那么无论如何都要使用 statement 模式记录； 
使用 statement 模式后，能处理很多原先出现的主键重复问题 

```


事务隔离级别                 {#ch5}
====================================
```

MySQL 5.1以前，Statement是Binlog的默认格式，即依次记录系统接受的SQL请求；5.1及以后，MySQL提供了Row和Mixed两个Binlog格式。

从MySQL 5.1开始，如果打开语句级Binlog，就不支持RC和Read-Uncommited隔离级别。要想使用RC隔离级别，必须使用Mixed或Row格式。
错误消息：
ERROR 1598 (HY000): Binary logging not possible. Message: Transaction level 'READ-COMMITTED' in InnoDB is not safe for binlog mode 'STATEMENT'

由于Binlog中语句的顺序以commit为序，如果使用read commit隔离级别+语句级别binlog，主库上回话1和回话2并行执行，回话1访问数据D1，然后回话2修改数据D1为D2并提交，回话2访问D2数据，最后提交，由于binlog是串行写入，先写入回话2的BINLOG在写入回话1的BINLOG，主库BINLOG传递到从库，从库上先执行回话2的语句，再执行回话1的语句，导致回话1两次数据均为D2，因此导致主从不一致。

由于STATEMENT格式的BINLOG只记录执行SQL，而不关心具体数据变化，因此如果需要保证数据一致，就必须保证从库执行SQL时的数据状态与在主库上执行时的数据状态一致。

```


Binlog格式与数据加锁                 {#ch6}
====================================
```

在语句格式(statement)下，为保证从库执行SQL时的数据状态与在主库上执行时的数据状态一致，主要通过对数据加锁来实现。

如对于SQL语句：
INSERT INTO TB1 SELECT * FROM TB2;

虽然该SQL只是读取TB2的数据并插入到TB1中，但为防止TB2数据被其他事务修改导致主从数据差异，执行该语句时，需要对TB2的数据加锁并持续到事务提交。

```

摘抄自：http://blog.csdn.net/heizistudio/article/details/8616997 
