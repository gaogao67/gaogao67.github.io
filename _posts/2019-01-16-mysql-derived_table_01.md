---
layout: post
title:  MySQL派生表Condition Pushdown优化
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Derived-Table
tag: Derived-Table
---

* content
{:toc}


=======================================

测试环境：MySQL 5.7.19

=======================================


如果派生表外部过滤条件可以下推到派生表内部，可以有效减少派生表内部扫描数据量和派生表使用内存甚至避免使用派生表。

如对下面查询：
```sql

SELECT * FROM (
SELECT 
cluster_id,
COUNT(1) AS instance_count
FROM `assets_instance`
GROUP BY `cluster_id`
HAVING COUNT(1)>1
) AS T1
WHERE T1.cluster_id=1487

```
对应查询在执行计划为：
![01.png]({{ '/styles/pic/derived-table/01.png'|prepend:site.baseurl}})

将派生表外部的查询条件下pushdown到派生表内部：
```sql

SELECT 
cluster_id,
COUNT(1) AS instance_count
FROM `assets_instance`
WHERE cluster_id=1487
GROUP BY `cluster_id`
HAVING COUNT(1)>1

```
对于执行计划为：
![02.png]({{ '/styles/pic/derived-table/02.png'|prepend:site.baseurl}})

<img src="https://gaogao67.github.io//styles/pic/derived-table/02.png"/>

优化前需要扫描`instance_count`表上`ForeignKey_cluster_id`索引全部数据然后进行分组计算，再按照`HAVING`条件进行过滤，得到派生表数据，再根据派生表外部条件`cluster_id=1487`进行过滤得到最终结果。

优化后仅需要对`instance_count`表上`ForeignKey_cluster_id`按照`cluster_id=1487`条件进行范围查找，然后进行`GROUP BY`+`HAVING`计算。

如果表中存在10000个`cluster_id`,那么优化后仅需要访问1/10000的数据，性能提升10000倍。


PS1: 在MariaDB中有类似优化。