---
layout: post
title:  MySQL数据库连接超时
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Connection
tag: MySQL-Connection
---

* content
{:toc}


connect_timeout
====================================
connect_timeout用在client和server之间建立连接时等待的握手超时时间，仅在登录时有效。在网络不好或网络丢包导致重试的情况下可适当增加该值，否则保留默认值即可。
```

The number of seconds that the mysqld server waits for a connect packet before responding with Bad handshake. The default value is 10 seconds as of MySQL 5.1.23 and 5 seconds before that.
Increasing the connect_timeout value might help if clients frequently encounter errors of the form Lost connection to MySQL server at ‘XXX’, system error: errno.

```


delayed_insert_timeout
====================================
作用于MyISAM引擎表，在INSERT DELAY中止前等待INSERT语句的时间
```

How many seconds an INSERT DELAYED handler thread should wait for INSERT statements before terminating.


```



innodb_lock_wait_timeout
====================================
innodb_lock_wait_timeout仅作用于innodb的行锁，对表锁的等待无效。当执行的语句等待行锁时间超过innodb_lock_wait_timeout阀值后，会回滚该语句，但不会回滚整个事务。在高并发的OLTP系统中，可以适当降低该阀值来降低锁等待的时间。可以在线修改全局或回话级别的innodb_lock_wait_timeout参数。

```

The length of time in seconds an InnoDB transaction waits for a row lock before giving up. The default value is 50 seconds. A transaction that tries to access a row that is locked by another InnoDB transaction waits at most this many seconds for write access to the row before issuing the following error:
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
When a lock wait timeout occurs, the current statement is rolled back (not the entire transaction). To have the entire transaction roll back, start the server with the --innodb_rollback_on_timeout option.


```



innodb_rollback_on_timeout
====================================
```

InnoDB rolls back only the last statement on a transaction timeout by default. If --innodb_rollback_on_timeout is specified, a transaction timeout causes InnoDB to abort and roll back the entire transaction.

```
默认innodb_rollback_on_timeout被关闭，当innodb_rollback_on_timeout被指定时，最后一条语句超时会导致整个事务发生回滚。



wait_timeout
====================================
```

The number of seconds the server waits for activity on a noninteractive connection before closing it.
On thread startup, the session wait_timeout value is initialized from the global wait_timeout value or from the global interactive_timeout value, depending on the type of client (as defined by the CLIENT_INTERACTIVE connect option to mysql_real_connect()). 

```



interactive_timeout
====================================
```

The number of seconds the server waits for activity on an interactive connection before closing it. An interactive client is defined as a client that uses the CLIENT_INTERACTIVE option to mysql_real_connect().

```
wait_timeout和interactive_timeout在MySQL 5.7版本默认值为28800秒=8小时，用来设置连接处于不活动状态后多长时间未激活则关闭该连接。



net_write_timeout和net_read_timeout
====================================
```

net_write_timeout
The number of seconds to wait for a block to be written to a connection before aborting the write. 

net_read_timeout
The number of seconds to wait for more data from a connection before aborting the read. When the server is reading from the client, net_read_timeout is the timeout value controlling when to abort. When the server is writing to the client, net_write_timeout is the timeout value controlling when to abort.

```


slave_net_timeout
====================================
```

The number of seconds to wait for more data from a master/slave connection before aborting the read. Setting this variable has no immediate effect. The state of the variable applies on all subsequent START SLAVE commands.
当从库在slave_net_timeout阀值时间内未获得主库的同步信息，会断定与主库的连接出现问题，并尝试关闭当前连接并重新建立新连接。

```
