---
layout: post
title:  MySQL--SemiJoin优化策略
date:   2019-01-16 15:00:00 +0800
categories: MySQL-SemiJoin
tag: MySQL-SemiJoin
---

* content
{:toc}


Semi-join(半连接)				{#ch1}
====================================
半连接主要场景：检查一个结果集（外表）的记录是否在另外一个结果集（字表）中存在匹配记录，半连接仅关注"子表是否存在匹配记录”，而并不考虑"子表存在多少条匹配记录"，半连接的返回结果集仅使用外表的数据集，查询语句中IN或EXISTS语句常使用半连接来处理。

MySQL支持5中Semi-join策略:

1、DuplicateWeedout

2、FirstMatch

3、LooseScan

4、Materializelookup

5、MaterializeScan



DuplicateWeedout   		{#ch2}
====================================
DuplicateWeedout: 使用临时表对semi-join产生的结果集去重。

Duplicate Weedout: Run the semi-join as if it was a join and remove duplicate records using a temporary table.

![01.png]({{ '/styles/pic/semi-join/01.png'|prepend:site.baseurl}})



FirstMatch   				{#ch3}
====================================
FirstMatch: 只选用内部表的第1条与外表匹配的记录。

FirstMatch: When scanning the inner tables for row combinations and there are multiple instances of a given value group, choose one rather than returning them all. This "shortcuts" scanning and eliminates production of unnecessary rows.

![02.png]({{ '/styles/pic/semi-join/02.png'|prepend:site.baseurl}})



LooseScan   				{#ch4}
====================================
LooseScan: 把inner-table数据基于索引进行分组，取每组第一条数据进行匹配。

LooseScan: Scan a subquery table using an index that enables a single value to be chosen from each subquery's value group.


![03.png]({{ '/styles/pic/semi-join/03.png'|prepend:site.baseurl}})



Materializelookup   				{#ch5}
====================================
Materializelookup: 将inner-table去重固化成临时表，遍历outer-table，然后在固化表上去寻找匹配。

Materialize the subquery into a temporary table with an index and use the temporary table to perform a join. The index is used to remove duplicates. The index might also be used later for lookups when joining the temporary table with the outer tables; if not, the table is scanned.


MaterializeScan   				{#ch6}
====================================
MaterializeScan: 将inner-table去重固化成临时表，遍历固化表，然后在outer-table上寻找匹配。

![05.png]({{ '/styles/pic/semi-join/05.png'|prepend:site.baseurl}})


在MySQL中优化器开关optimizer_switch中，以下参数影响Semi-join的选择：
```

semijoin={on|off}
materialization={on|off}
loosescan={on|off}
subquery_materialization_cost_based={on|off}

```



总结   				{#ch7}
====================================
在SemiJoin中5中优化策略中，影响策略的最关键的因素:

1、inner-table和outer-table上的数据量。

2、inner-table和outer-table上是否有能快速定位数据的索引。

