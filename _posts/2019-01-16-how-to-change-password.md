---
layout: post
title:  修改MySQL账号密码
date:   2019-01-16 15:00:00 +0800
categories: MySQL-User
tag: MySQL-User
---

* content
{:toc}


使用mysqladmin进行修改
====================================
```sql

mysqladmin -u username -h hostname password 'new password';

```

使用set命令进行修改
====================================
```sql

SET PASSWORD FOR 'username'@'host' =PASSWORD('new password')

```


使用update命令进行修改
====================================
MySQL 5.7版本：
```sql

UPDATE mysql.user 
SET authentication_string=PASSWORD('abc@123')
WHERE user='root';
FLUSH PRIVILEGES;

```
MySQL 5.6版本：
```sql

UPDATE mysql.user 
SET password=PASSWORD('abc@123')
WHERE user='root';
FLUSH PRIVILEGES;

```

使用grant命令进行修改
====================================
使用明文密码：
```sql

## 使用明文密码
GRANT USAGE ON *.* TO 'gga2'@'localhost' IDENTIFIED BY '123@456';

```

使用加密后密码：
```sql

SELECT PASSWORD('123@456');
GRANT USAGE ON *.* TO 'gga3'@'localhost' IDENTIFIED BY PASSWORD '*864CE19166BD4846244E41F8EA56377F18C3EFAA';

```
