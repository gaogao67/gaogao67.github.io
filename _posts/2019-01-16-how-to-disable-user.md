---
layout: post
title:  禁用账号和设置账号有效期
date:   2019-01-16 15:00:00 +0800
categories: MySQL-User
tag: MySQL-User
---

* content
{:toc}


MySQL5.5/5.6版本                  {#mysql55}
====================================
在MySQL 5.7版本之前，不能对账号进行锁定或设置过期，只能通过更新密码来实现。

```bash

SELECT concat('UPDATE `mysql`.`user` SET PASSWORD=''*9999999999999999999999999999999999999999'''
,' WHERE HOST=''',HOST,
''' AND USER=''',USER,''';') AS UpdateScript
FROM mysql.user WHERE USER='repl';

```


MySQL5.7版本                  {#mysql57}
====================================
在MySQL 5.7版本中，可以设置MySQL账号自动过期时间，从MySQL 5.7.10开始，参数default_password_lifetime默认值从0变更为360，即一年有效期。

```sql

## 修改默认自动过期时间：
SET GLOBAL default_password_lifetime = 0;

## 设置用户密码过期
ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE;

## 设置用户密码永不过期
ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE NEVER;

```

在MySQL 5.7 版本中，可以通过账号锁定来禁用账号。

```sql

ALTER USER 'jeffrey'@'localhost' ACCOUNT LOCK;
ALTER USER 'jeffrey'@'localhost' ACCOUNT UNLOCK;

```

参考链接：
https://dev.mysql.com/doc/refman/5.7/en/password-expiration-policy.html
https://dev.mysql.com/doc/refman/5.7/en/account-locking.html
