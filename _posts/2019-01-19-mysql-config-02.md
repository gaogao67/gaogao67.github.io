---
layout: post
title:  MySQL参数配置之read-only
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Config
tag: MySQL参数配置
---

* content
{:toc}


参数说明				{#ch1}
====================================
```

参数super_read_only和read_only以及innodb-read-only

参数innodb-read-only：主要用来设置将MySQL运行在只读介质如DVD上，设置--innodb-read-only时需将--pid-file设置到可读写的介质上，并将--event-scheduler设置为禁用。innodb-read-only参数仅用于服务器启动时。

参数read_only：用来设置数据对普通用户只读，但拥有SUPER权限用户依旧可以修改数据

参数super_read_only：当super_read_only启用时，即使客户端的用户拥有SUPER权限，也不能修改数据。

当super_read_only启用时，read_only也会默认启用
当read_only关闭时，super_read_only也会被默认关闭

当read_only开启时，允许以下操作：
1、复制IO线程和SQL线程可以正常进行复制
2、正常使用ANALYZE TABLE和 OPTIMIZE TABLE语句，ANALYZE TABLE和 OPTIMIZE TABLE并不导致数据变化
3、操作临时表，普通用户查询也可能产生临时表
4、向日志表如mysql.general_log 和 mysql.slow_log写数据
5、对Performance Schema的表进行DML操作


```


PS1:在5.6.22及之前版本中，super_read_only和Event Scheduler会产生冲突并导致MySQL服务启动

PS2:在主库设置read_only和super_read_only并不会传递到从库

PS3:使用GRANT ALL PRIVILEGES ON *.* 或GRANT SUPER ON *.*来授予用户SUPER管理员账户权限，拥有SUPER权限的用户可以KILL其他用户的查询，并能修改全局变量，使用CHANGE MASTER/PURGE MASTER LOGS等命令。