---
layout: post
title:  MySQL数据库连接信息
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Connection
tag: MySQL-Connection
---

* content
{:toc}


查看当前连接到数据库的用户和Host
====================================
```sql
## 查看当前连接到数据库的用户和Host ##

SELECT DISTINCT 
USER,HOST 
FROM `information_schema`.`PROCESSLIST` P 
WHERE P.USER NOT IN('root','repl','system user')   \G

```

查看每个host的当前连接数和总连接数
====================================
```sql

SELECT * FROM performance_schema.hosts;

```
PS1: 系统表performance_schema.hosts在MySQL 5.6.3版本中引入，用来保存MySQL服务器启动后的连接情况。


按照登录用户+登录服务器查看登录信息
====================================
```sql

SELECT 
USER as login_user,
LEFT(HOST,POSITION(':' IN HOST)-1) AS login_ip,
count(1) as login_count
FROM `information_schema`.`PROCESSLIST` P 
WHERE P.USER NOT IN('root','repl','system user') 
GROUP BY USER,LEFT(HOST,POSITION(':' IN HOST)-1) \G

```

按照登录用户+数据库+登录服务器查看登录信息
====================================
```sql

SELECT 
DB as database_name,
USER as login_user,
LEFT(HOST,POSITION(':' IN HOST)-1) AS login_ip,
count(1) as login_count
FROM `information_schema`.`PROCESSLIST` P 
WHERE P.USER NOT IN('root','repl','system user') 
GROUP BY DB,USER,LEFT(HOST,POSITION(':' IN HOST)-1);


```