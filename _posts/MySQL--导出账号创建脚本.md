---
layout: post
title:  导出MySQL用户权限
date:   2019-01-16 15:00:00 +0800
categories: MySQL
tag: User
---

* content
{:toc}


MySQL5.5/5.6版本					{#mysql55}
====================================
在MySQL 5.5/5.6版本中，使用SHOW GRANTS命令可以导出用户的创建脚本和授权脚本。

```
hostname='127.0.0.1'
port=3358
username='root'
password='abc@123.com'
mysql_exe="/export/servers/mysql/bin/mysql"
echo "select concat('show grants for ''',user,'''@''',host, ''';') from mysql.user where user <>'root'" | \
${mysql_exe} --host=$hostname --user=$username --password=$password --port=$port -N | \
${mysql_exe} --host=$hostname --user=$username --password=$password --port=$port -N | \
sed "s/$/;/" > /tmp/create-users.sql
```


MySQL5.7版本                  {#mysql57}
====================================
在MySQL 5.7版本中，需要使用SHOW CRETAE USER命令导出用户创建脚本,然后使用SHOW GRANT命令导出用户授权脚本。

```
hostname='127.0.0.1'
port=3358
username='root'
password='abc@123.com'
mysql_exe="/export/servers/mysql/bin/mysql"
echo "select concat('show create user ''',user,'''@''',host, ''';'，'show grants for ''',user,'''@''',host, ''';') from mysql.user where user <>'root'" | \
${mysql_exe} --host=$hostname --user=$username --password=$password --port=$port -N | \
${mysql_exe} --host=$hostname --user=$username --password=$password --port=$port -N | \
sed "s/$/;/" > /tmp/create-users.sql
```
