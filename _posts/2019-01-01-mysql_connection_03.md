---
layout: post
title:  MySQL数据库连接异常问题汇总
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Connection
tag: Connection
---

* content
{:toc}


Name or service not known
====================================
错误消息：
```

[Warning] IP address 'xxx.xxx.xx.xxx' could not be resolved: Name or service not known

```
错误原因：
```

MySQL数据库服务器上没有配置/ect/hosts，也没有DNS服务，导致MySQL服务线程解析IP对应的主机名时发生失败。

```

解决办法：
```

使用参数--skip-name-resolve来禁用DNS的主机名解析功能，禁用该功能后，在MySQL授权表里面，只能使用IP地址。
配置my.cnf中参数为：
skip_host_cache
skip-name-resolve=1


```



Aborted Connections类错误
====================================
错误消息：
```

[Note] Aborted connection 854 to db: 'employees' user: 'josh'

```
问题原因：
```

客户端无法正常连接MySQL数据库，主要原因：
1、用户账号不正确
2、用户权限不足
3、连接包存在问题（网络丢包等问题）
4、建立连接时间超过 connect_timeout 参数的阀值


```


Aborted_clients类错误
====================================
错误原因：
```

客户端成功连接MySQL数据库但非正常断开或者异常中止，主要原因有：
1、客户端应用关闭前没有正常调用mysql close()方法
2、客户端建立连接后长时间未向MySQL发出请求，休眠时间超过 wait_timeout 或 interactive_timeout 两个参数
3、客户端在数据传输过程中非正常关闭。

```
其他异常：https://dev.mysql.com/doc/refman/5.7/en/communication-errors.html



No operations allowed after connection closed.
====================================
异常信息：
```

Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: No operations allowed after connection closed.

```
异常原因：
```

应用服务器连接数据库服务器后，长时间未使用而导致连接空闲时间超过数据库配置“wait_timeout”的阈值，MySQL自动将这些连接断开，但应用服务器并不知晓该连接失效，再次使用这些连接时就可以报错。

```

解决办法：
```

1、增到MySQL参数“wait_timeout”的值
2、减少 Connection pools 中 connection 的 lifetime
3、应用服务器定期检查连接池中各连接的有效性。

```



Cannot assign requested address
=====================================
问题描述：
```

使用python+MySQLdb来收集几百个MySQL数据库节点的数据库信息，每个MySQL数据库节点进行1000+次的数据库查询，并将查询结果插入的本地数据库中上（数据库服务器与程序运行在同一服务器中），同样造成1000+的数据库插入，在程序运行中，不定期出现连接失败的异常，异常信息为：
2003,Can’t connect to MySQL server on ‘XXX.XXX.XXX.XXX′(99)

错误代码99含义为：
 OS error code  99:  Cannot assign requested address
表示无法分配本地地址资源，socket无法创建。

在Linux级别使用命令 netstat -anp |grep TIME_WAIT 可以发现有大量的TIME_WAIT，超过1W+的等待：

```

问题原因：
```

在程序运行过程中，运行程序的服务器与其他MySQL数据库服务器频繁建立连接并执行MySQL命令，当MySQL命令执行完成后，TCP连接被关闭后处于TIME_WAIT状态，TCP连接未被及时释放而导致TCP连接端口占满不可用。

```

解决办法：
```

配置TCP连接可以重用和快速回收，在文件/etc/sysctl.conf中加入以下代码：
net.ipv4.tcp_syncookies = 1 
net.ipv4.tcp_tw_reuse = 1 
net.ipv4.tcp_tw_recycle = 1 

然后使用/sbin/sysctl -p 命令使配置文件生效。

配置完成后，在程序运行时仍存在大量（超过1W+）处于TIME_WAIT状态的TCP连接，但未再出现连接失败的情况。

```

相关知识：
```

net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。

```