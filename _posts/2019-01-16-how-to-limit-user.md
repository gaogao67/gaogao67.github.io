---
layout: post
title:  限制用户使用资源
date:   2019-01-16 15:00:00 +0800
categories: MySQL-User
tag: MySQL-User
---

* content
{:toc}



在MySQL 5.7及后续版本中，可以按照账号来限制每个账号实际具有的资源限制。
语法：
GRANT  WITH option

如：
```sql

GRANT SELECT ON test.* 
TO user1@localhost 
WITH MAX_QUERIES_PER_HOUR 3
MAX_USER_CONNECTIONS 5;

```
可设选项：
MAX_QUERIES_PER_HOUR count : 每小时最大查询次数
MAX_UPDATES_PER_HOUR count ：每小时最大更新次数
MAX_CONNECTIONS_PER_HOUR count ：每小时最大连接次数
MAX_USER_CONNECTIONS count ：最大用户连接数


MAX_USER_CONNECTIONS 指的是瞬间的并发连接数，而MAX_CONNECTIONS_PER_HOUR指的是每小时累计的最大连接次数，
如果MAX_USER_CONNECTIONS count的值为0，那么用户的实际值为全局的参数值MAX_USER_CONNECTIONS，否则按照用户的MAX_USER_CONNECTIONS count来设置。

资源限制是对某一账号进行累计的，而不是对账号的一次连接进行累计的，当资源限制到达后，账号的任何一次相关操作都会被拒绝。

系统默认调用的一些隐式查询也会被记录到MAX_QUERIES_PER_HOUR的值中。


