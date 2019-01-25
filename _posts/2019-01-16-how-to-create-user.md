---
layout: post
title:  创建MySQL用户
date:   2019-01-16 15:00:00 +0800
categories: MySQL-User
tag: MySQL-User
---

* content
{:toc}


创建语法					{#ch1}
========================
## CREATE USER语法
```bash
CREATE USER 'username'@'hostname' IDENTIFIED BY 'password'; 
```

## GRANT USER语法
```bash
GRANT priv_type
ON object_type
TO 'username'@'hostname'
IDENTIFIED BY [PASSWORD] 'password'
[WITH GRANT OPTION]
```

注意[PASSWORD]是可选参数，IDENTIFIED BY PASSWORD 后跟的是加密后的密码，IDENTIFIED BY 后跟的是未加密的密码
```
GRANT USAGE ON *.* TO 'cx'@'%' IDENTIFIED BY PASSWORD '*67CF1635475C64DE1C9608FEEE1AD307C4C2BC2F';
```
和
```
GRANT USAGE ON *.* TO 'cx'@'%' IDENTIFIED BY 'abc@123.com';
```
上面两条SQL效果一样，但推荐使用第一种方式，第二种方式的明文密码会被记录到binlog中，存在安全风险。


WITH GRANT OPTION 允许用户将自身拥有的权限授予他人。
The WITH GRANT OPTION clause gives the user the ability to give to other users any privileges the user has at the specified privilege level.


对于hostname，可以使用%和_两种通配符，如：
```bash
GRANT ALL PRIVILEGES
ON *.* TO 'gga1'@'%.mysql.com'
IDENTIFIED BY '123@456'
WITH GRANT OPTION;
```


## 设置用户过期时间为60天过期
```bash
ALTER USER 'gga'@'localhost' PASSWORD EXPIRE INTERVAL 60 DAY;
```


创建语法					{#ch2}
========================
在MySQL中，有5种控制数据访问的权限，12种控制数据结构的权限，11种管理数据库系统的权限
![/styles/pic/mysql_user/001.png]({{ '/styles/pic/mysql_user/001.png'|prepend:site.baseurl}})