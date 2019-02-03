---
layout: post
title:  MySQL Buffer Pool
date:   2019-01-16 15:00:00 +0800
categories: MySQL-BufferPool
tag: MySQL-BufferPool
---

* content
{:toc}


Buffer Pool基础信息
====================================
```

Innodb Buffer Pool 主要用来存放以下数据：
数据页 data page
索引页 index page
插入缓冲 insert buffer
锁信息 lock info
自适应哈希索引 adaptive hash index
数据字典信息 data dicitionary



除Innodb buffer pool 使用内存来缓冲数据外，还有重做日志缓冲 redo log buffer 和额外内存池 innodb_additional_mem_pool_size属于内存数据对象。

在InnoDB存储引擎中，缓冲池中的页默认大小为16KB。

在innodb 1.0.x版本后，允许有多个缓冲池，页面按照hash值平均分配到不同的缓冲池中，以减少数据库内部资源竞争，增减数据库的并发处理能力。

在MYSQL 5.6版本中，可以使用INFORMATION_SCHEMA下的表INNODB_BUFFER_POOL_STATUS来查看缓冲池的状态。

在MySQL 5.7版本前，一个buffer pool instance拥有一个chunk，每个chunk的大小为buffer pool size/instance个数
在MySQL 5.7版本后，一个buffer pool instance拥有多个chunk，每个chunk的大小默认为127MB, buffer pool的增加和减少必须以chunk为基本单位进行。

在MySQL 5.7版本中，buffer pool size根据instance*chunk size向上对齐，如果配置64个instance，每个chunk默认使用128MB, 并将buffer pool配置为9GB，那么部分instance便可以分配2个chunk，根据向上对其原则，每个instance都会配置2个chunk，整个buffer pool使用内存为64*2*128MB=16GB。


```


Flush列表
====================================
```

Flush列表用来管理脏页，记录自上次CheckPoint后修改的数据页。

使用show innodb status命令查看Innodb引擎的状态数据，其中Modified db pages显示的便是脏页数据量。

```


LRU(last recent used)列表
======================================
```

通常情况下，缓冲池无法将整个数据库所有数据都进行缓冲，而且不同数据的访问频率不一样，有些数据会被频繁访问，而有些数据可能数月不会被访问一次，因此数据库使用最近最少使用LRU latest Recent Used算法来管理缓冲池，其算法思想为：最近访问的数据被再次访问的概率要高于之前被访问的数据，被多次访问的数据被再次访问的概率要高于访问次数较低的数据。

Mysql使用LRU列表来记录页的访问情况，将访问频繁的页记录在LRU列表的前端，将最少访问的页记录在LRU列表的尾端，将最新插入缓冲池的数据放入LRU的中部mid(未必一定是最中间)，当缓冲池不能存放新的页面时，将首先释放LRU列表尾部的数据页。

为避免单次的表扫描或索引扫描将大量数据读取到缓冲池，导致缓冲区内热数据被交换出缓冲池，MYSQL对LRU算法进行优化，通过innodb_old_blocks_pct和innodb_old_blocks_time来限制新数据加载

innodb_old_blocks_pct指定新数据在LRU列表存放位置，默认在LRU列表唱的的5/8处，即37%的位置

innodb_old_blocks_time指定页读取到mid位置后需要等待多久才会被加入到LRU列表的热端，默认值为1000  

查看两个参数的值：
SHOW VARIABLES LIKE 'INNODB_OLD_BLOCKS_%' \G
*************************** 1. row ***************************
Variable_name: innodb_old_blocks_pct
        Value: 37
*************************** 2. row ***************************
Variable_name: innodb_old_blocks_time
        Value: 1000
2 rows in set (0.01 sec)

修改操作：
SET GLOBLE INNODB_OLD_BLOCKS_TIME=1000;
SET GLOBAL INNODB_OLD_BLOCKS_PCT=37;

```