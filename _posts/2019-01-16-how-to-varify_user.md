---
layout: post
title:  用户权限验证顺序
date:   2019-01-16 15:00:00 +0800
categories: MySQL-User
tag: MySQL-User
---

* content
{:toc}

用户权限验证顺序
====================================
用户使用host、user、password 三项来验证登录，然后按照user>>db>>tables_priv>>columns_priv来得到用户权限。


对于相同用户名的登录用户，在进行匹配hostname时，按照最全匹配方式来匹配，如假设分别有以下授权用户：
user1@'192.168.1.1'
user1@'192.168.1.%'
user1@'192.%'
当user1用户登录时，先看登录IP是否匹配192.168.1.1，再看登录IP是否匹配192.168.1.%，最后看登录IP是否匹配192.%


实例级别授权
====================================
```sql

##授予用户所有数据库上的SELECT 权限
##授权后在mysql.user表的Select_priv=Y
GRANT SELECT ON *.* TO gga@localhost;
SELECT * FROM mysql.user where user='gga' \G

##取消上面授权
REVOKE SELECT ON *.* FROM gga@localhost;

```
![/styles/pic/mysql_user/180802161726.png]({{ '/styles/pic/mysql_user/180802161726.png' | prepend: site.baseurl  }})



数据库级别级别授权
====================================
```
##授予用户数据库上test的所有表的SELECT权限
##授权后在mysql.db表的Select_priv=Y
GRANT SELECT ON testdb1.* TO gga@localhost;
SELECT * FROM  mysql.db WHERE User='gga' \G

##取消上面授权
REVOKE SELECT ON testdb1.* FROM gga@localhost;

```
![/styles/pic/mysql_user/180802161648.png]({{ '/styles/pic/mysql_user/180802161648.png' | prepend: site.baseurl  }})

表级别级别授权
====================================
```
##授予用户数据库上test的tb001表的SELECT权限
##授权后在mysql.db表的Select_priv=Y
GRANT SELECT ON testdb1.tb001 TO gga@localhost;
SELECT * FROM mysql.tables_priv WHERE User='gga' \G

##取消上面授权
REVOKE SELECT ON testdb1.tb001 FROM gga@localhost;

```

![/styles/pic/mysql_user/JdOnline20180802161607.png]({{ '/styles/pic/mysql_user/JdOnline20180802161607.png' | prepend: site.baseurl  }})


MySQL特殊权限
====================================
```
USAGE 权限只用用于数据库登录，不能执行任何操作,usage 权限不能被回收。
ALL: 允许做任何事(和root一样)。

```

MySQL数据库/数据表/数据列权限
====================================
```
Alter: 修改已存在的数据表(例如增加/删除列)和索引。
Create: 建立新的数据库或数据表。
Delete: 删除表的记录。
Drop: 删除数据表或数据库。
INDEX: 建立或删除索引。
Insert: 增加表的记录。
Select: 显示/搜索表的记录。
Update: 修改表中已存在的记录。

```

MySQL全局管理权限
====================================
```
FILE: 在MySQL服务器上读写文件。
PROCESS: 显示或杀死属于其它用户的服务线程。
RELOAD: 重载访问控制表，刷新日志等。
SHUTDOWN: 关闭MySQL服务。

```
