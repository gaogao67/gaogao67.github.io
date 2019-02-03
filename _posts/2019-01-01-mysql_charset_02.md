---
layout: post
title:  MySQL乱码问题
date:   2019-01-16 15:00:00 +0800
categories: MySQL-CharSet
tag: MySQL-CharSet
---

* content
{:toc}




MySQL字符集参数
====================================
```

character_set_client: 表示客户端请求数据的字符集
character_set_connection：表示客户端连接到数据库后传输使用的字符集
character_set_database：默认数据库的字符集，如果创建数据库时没有指定字符集，则使用该默认字符集
character_set_filesystem：在操作系统层次上存放文件使用的字符集，默认为二进制
character_set_results：结果集使用的字符集
character_set_server：服务器使用的默认字符集
character_set_system：为存储系统元数据信息使用的字符集，总是utf8

```

MySQL参数使用
=======================================
```

在客户端与MYSQL服务器建立连接后，执行INSERT操作并返回新插入：
1>MYSQL Client使用character_set_client指定的字符集来编码请求数据并发送给MYSQL服务器;
2>MYSQL服务器接收到客户端发送来的请求数据，将数据按照character_set_connection指定的字符集进行解码;
3>MYSQL服务器执行插入请求，将步骤2中解码的数据根据对应的列的字符集进行编码，然后插入数据，写入到文件中;
4>MYSQL服务器执行查询请求，将数据从文件中读取出来，按照对应列的字符集进行解码;
5>MYSQL服务器将查询结果按照character_set_results进行编码，返回给MYSQL Client;

为避免数据库乱码问题，应该保证客户端编码/服务器character_set_client和对于表上字段的charset使用相同的编码方式。

```


影响到字符编码的设置
====================================
```

Session settings
-->character_set_server
-->character_set_client
-->character_set_connection
-->character_set_database
-->character_set_result
Schema level defaults
Table level defaults
Column charsets

```
数据库级别的字符集信息使用db.opt来存放字符集和校验字符集的信息，当该数据库下定义表时未定义字符集，将使用数据库的字符集。



SET NAMES命令
====================================
```

SET NAMES {'charset_name' [COLLATE 'collation_name'] | DEFAULT}
This statement sets the three session system variables character_set_client, character_set_connection, and character_set_results to the given character set. Setting character_set_connection to charset_name also sets collation_connection to the default collation for charset_name.

SET NAMES x相当于执行下面三条语句：
SET character_set_client = x;
SET character_set_results = x;
SET character_set_connection = x;


If you are using the mysql client with auto-reconnect enabled (which is not recommended), it is preferable to use the charset command rather than SET NAMES. 
The charset command issues a SET NAMES statement, and also changes the default character set that is used if mysql reconnects after the connection has dropped.

```

参考：
http://blog.csdn.net/y_h_t/article/details/17994335
https://dev.mysql.com/doc/refman/5.7/en/set-names.html
