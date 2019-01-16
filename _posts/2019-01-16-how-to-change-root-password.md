---
layout: post
title:  管理员密码遗失处理
date:   2019-01-16 15:00:00 +0800
categories: MySQL-User
tag: MySQL-User
---

* content
{:toc}


忘记MYSQL管理员密码如root用户密码，可以按照以下方式来修改

## STEP1: 停止MySQL服务
```bash
ps -ef | grep -v 'grep' | grep 'mysqld' | awk '{print $2}' | xargs kill -9
```


## STEP2: 以忽略权限方式启动MYSQL服务
```bash
/export/servers/mysql/bin/mysqld --skip-grant-tables --explicit_defaults_for_timestamp --user=mysql &
```


## STEP3: 更新管理员密码
```sql
##MySQL5.6版本更新密码
update mysql.user set password=password("abc@123.com") where user="root";
##MySQL5.7版本更新密码
update mysql.user set authentication_string=password("new_pass") where user="root";
##刷新权限
flush privileges;
```


## 停止MySQL服务，重新正常启动
```bash
/export/servers/mysql/bin/mysqld_safe --defaults-file=/export/servers/mysql/etc/my.cnf &
```
