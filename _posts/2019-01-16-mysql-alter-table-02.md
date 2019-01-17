---
layout: post
title:  MySQL删除超大表操作
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Alter-Table
tag: Alter Table
---

* content
{:toc}

问题原因
====================================
```

通常情况下，会使用innodb_file_per_table=ON来将每张InnoDB表数据和索引数据保存到一个独立的文件中，而当MySQL运行在Linux版本时，使用DROP TABLE删除表时，会同时删除磁盘上的数据文件来回收磁盘空间。
当删除超大表时：
1、Linux删除超大文件会在一段时间内严重消耗磁盘IO，引发磁盘性能问题。
2、MySQL删除表期间，InnoDB存储引擎需要维护一个全局锁直到删除完成，长时间持有全局锁会引发严重的阻塞。

```


优化原理
====================================
```

优化原理：
在Linux中，每个存储文件都会有指向该文件的Inode Index，多个文件名可以通过相同Inode Index指向相同一个存储文件。
当按照文件名删除文件时：
1、如果该文件名引用的Inode Index上还被其他文件名引用，则只会删除该文件名和Inode Index之间的引用
2、如果该文件名引用的Inode Index上没有被其他文件名引用，则删除该文件名和Inode Index之间的引用并删除Inode Index指向的存储文件。

```
![/styles/pic/alter_table/001.jpg]({{ '/styles/pic/alter_table/001.jpg' | prepend: site.baseurl  }})


操作步骤
====================================
假如删除mytest.erp表，其数据文件存储路径为 /data/mysql/mytest/erp.ibd

操作办法：

1、创建硬连接指向要删除表的ibd文件
```

ln /data/mysql/mytest/erp.ibd /data/mysql/mytest/erp.ibd.hdlk

```

2、使用DELETE删除表
```

DROP TABLE mytest.erp;

```

3、使用truncate命令小批量删除文件，最后用rm删除整个文件
```

##seq 2194 -10 10 表示：从2194开始循环，每次递减10，直到循环至10
TRUNCATE=/usr/local/bin/truncate  
for i in `seq 2194 -10 10 `;   
do   
  sleep 2  
  $TRUNCATE -s ${i}G /data/mysql/mytest/erp.ibd.hdlk   
done  
rm -rf /data/mysql/mytest/erp.ibd.hdlk;

```


PS1:在Window上删除表时，Window只需要删除该文件指针并将该存储区域表示为可重用即可，因此在Windows上删除超大表无需过多操作。

参考链接：
https://www.cnblogs.com/digdeep/p/9588709.html