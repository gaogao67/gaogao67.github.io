---
layout: post
title:  更新自增ID的潜在风险
date:   2019-01-16 15:00:00 +0800
categories: "MySQL Auto Increment"
tag: AUTO_INCREMENT
---

* content
{:toc}


测试环境
====================================
MySQL版本：MySQL 5.7.19
复制模式：ROW

模拟测试
====================================
```sql

## 创建测试表
DROP TABLE T_001;
CREATE TABLE `T_001` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `C1` int(11) NOT NULL,
  PRIMARY KEY (`ID`),
  UNIQUE KEY `UNI_C1` (`C1`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

##插入记录
INSERT INTO T_001(C1)VALUES(1);

##更新ID列
UPDATE T_001
SET ID=2,
C1=2
WHERE C1=1;

##插入新记录
INSERT INTO T_001(C1)VALUES(3);

##插入报错，ERROR 1062 (23000): Duplicate entry '2' for key 'PRIMARY'

```

问题分析
====================================
```

1、第一次INSERT语句，获得自增值为1。
2、UPDATE操作完成，将表中数据的ID列修改为2，但UPDATE操作不会触发表的自增起始值发生变化。
3、第二次INSERT语句，获得自增值为2，由于表中已存在ID=2的记录，因此插入失败，报主键重复。

```

问题总结
====================================
1、对于自增列，其自增值与业务无关，应避免对自增列数据进行更新操作，避免出现异常。
2、如果INT自增列的自增值达到INT类型最大值，应该将列升级为BIGINT，而不能重置自增初始值继续使用INT列。