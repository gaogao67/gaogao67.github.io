---
layout: post
title:  MySQL CheckPoint基础
date:   2019-01-16 15:00:00 +0800
categories: MySQL-CheckPoint
tag: MySQL-CheckPoint
---

* content
{:toc}




CheckPoint基础
====================================
```

===================================
Checkpint 分两种：
Sharp Checkpoint : 在服务器正常关闭时，将所有脏页都写入到磁盘中，默认配置参数 innodb_fast_shutdown=1
Fuzzy Checkpoint: 在服务正常运行过程中，由各种条件触发进行Fuzzy Checkpoint, 只将部分脏页刷新写入到磁盘中。

===================================
触发Fuzzy Checkpoint的可能情况：
1> Master Thread Checkpoint
2> FLUSH_LRU_LIST Checkpoint
3> Async/Sync Flush Checkpoint
4> Dirty Page Too Much Checkpoint

===================================
Master Thread Checkpoint
Master Thread 会以一定频率(每秒或每10秒)从缓冲池的脏页列表中刷新一定比例的页到磁盘中，该操作为异步模式，不会阻塞用户查询。

===================================
FLUSH_LRU_LIST Checkpoint
由于InnoDB存储引擎需要保证LRU列表中有一定数量的空闲页空使用，当FLUSH_LRU_LIST列表中没有足够的空闲页时，InnoDB会将LRU列表尾部的页移除，，如果要移除的数据页中有脏页，就需要进行checkpoint将数据写入到磁盘上。

在Innodb 1.1.x 版本前，检查FLUSH_LRU_LIST列表的操作在用户线程中执行，因此checkpoint操作会阻塞用户查询操作
在Innodb 1.1.x 版本后，检查FLUSH_LRU_LIST列表的操作在独立线程Page Cleaner中执行，因此checkpoint操作不会阻塞用户查询操作

在Innodb 1.2.x 版本前，FLUSH_LRU_LIST列表控制在100个空闲页可用。
在Innodb 1.2.x 版本后，用户可用通过参数innodb_lru_scan_depth来控制列表可用页的数量，该参数默认值为1024.


===================================
Async/Sync Flush Checkpoint
Async/Sync Flush Checkpoint为了保证重做日志的循环使用的可用性，根据最近一次checkoint和当前redo lsn计算出checkpoint_page 数，再按照checkpoint_page 数占重做日志空间的比例来选择进行Async/Sync Flush Checkpoint。

checkpoint_page =redb_lsn - checkpoint_lsn
async_water_mark =75%* total_redo_log_file_size
sync_water_mark =90%* total_redo_log_file_size

当checkpoint_page<async_water_mark时，不需要刷新任何脏页到磁盘
当async_water_mark<checkpoint_page  <sync_water_mark 时，使用async flush checkpoint 将脏页数据写入到磁盘中
当checkpoint_page  >sync_water_mark 时，使用 sync flush checkpoint 将脏页数据写入到磁盘中

在Innodb 1.2.X版本前，Async Flush Checkpoint操作会阻塞发现问题的用户线程查询，Sync Flush Checkpoint 会阻塞所有用户的查询线程。

在Innodb 1.2.X版本后，这两类checkpoint 操作被放入到Page Cleaner Thread中，故不会阻塞用户查询线程。


===================================
Dirty Page Too Much Checkpoint
为保障缓冲池有足够可用的页，当脏页数量到达缓冲池特定比例的时候，会导致Innodb存储引擎强制进行check point, 该比例由参数innodb_max_dirty_page_pct控制：
show variables like 'innodb_max_dirty_pages_pct' \G
*************************** 1. row ***************************
Variable_name: innodb_max_dirty_pages_pct
        Value: 75
1 row in set (0.00 sec)

在innodb 1.0.x 版本之前，该默认值为90，在innodb 1.0.x 版本之后，默认值为75


===================================
Undo 日志在Checkpoint时会写入磁盘。


```

