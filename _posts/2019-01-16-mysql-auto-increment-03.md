---
layout: post
title:  使用REPLACE INTO导致主从差异
date:   2019-01-16 15:00:00 +0800
categories: "MySQL Auto Increment"
tag: AUTO_INCREMENT
---

* content
{:toc}


问题说明
====================================
在主库上使用REPLACE INTO更新数据时，表存在唯一索引且REPALCE INTO更新自增列值，在ROW模式下，主从数据能保持一致，但从库自增列初始值不会发生变化，当主从切换后，会存在问题。


测试场景
====================================
在主库上执行下面SQL:
```

CREATE TABLE T_AUTO_TEST
(
    ID INT AUTO_INCREMENT PRIMARY KEY,
    C1 INT NOT NULL,
    UNIQUE KEY UNI_C1(C1)
);

INSERT INTO T_AUTO_TEST(ID,C1)VALUES(99,99);
REPLACE INTO T_AUTO_TEST(ID,C1)VALUES(101,99);

```



在主库上，由于会话1的INSERT比会话2的INSERT要早，因此会话1插入数据获得的内部自增ID要比会话2插入数据获得的ID要小。

在从库上，由于会话2的事务先于会话1的会话提交，会话2生成的binlog会先于会话1生成的binlog执行，因此会话1插入数据获得的内部自增ID要比会话2插入数据获得的ID要大。

由于上面的操作导致的数据差异，在执行ALTER TABLE操作后，“相同数据”获得不同的自增ID，导致复制异常。


解决办法
====================================
新建一个相同表结构的临时表，修改临时表增加自增列，将原表数据复制插入到临时表中，然后交换表名。

对于主库上临时表中插入的每条数据，会包含自增ID值一起传递到从库，保证主从数据完全一致。
