---
layout: post
title:  MySQL派生表临时结果集中的AutoKey
date:   2019-01-16 15:00:00 +0800
categories: MySQL-Derived-Table
tag: Derived-Table
---

* content
{:toc}


=======================================

测试环境：MySQL 5.7.19

=======================================


在某些场景中，需要对派生表生成临时结果集进行materialized，如果的该临时结果集中包含索引键，那么查询有可能通过该索引键来进行优化。

如对下面查询：
```sql

SELECT
T2.purpose_code,
T1.instance_count
FROM `assets_cluster` AS T2 
STRAIGHT_JOIN(
SELECT 
cluster_id,
COUNT(1) AS instance_count
FROM `assets_instance`
GROUP BY `cluster_id`
HAVING COUNT(1)>1
) AS T1
ON T1.cluster_id=T2.cluster_id

```
对应查询在执行计划为：
![03.png]({{ '/styles/pic/derived-table/03.png'|prepend:site.baseurl}})

查看JOSN格式的执行计划：
```bash

{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "10397.97"
    },
    "nested_loop": [
      {
        "table": {
          "table_name": "T2",
          "access_type": "index",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "idx_purpose_code",
          "used_key_parts": [
            "purpose_code"
          ],
          "key_length": "387",
          "rows_examined_per_scan": 851,
          "rows_produced_per_join": 851,
          "filtered": "100.00",
          "using_index": true,
          "cost_info": {
            "read_cost": "10.00",
            "eval_cost": "170.20",
            "prefix_cost": "180.20",
            "data_read_per_join": "15M"
          },
          "used_columns": [
            "cluster_id",
            "purpose_code"
          ]
        }
      },
      {
        "table": {
          "table_name": "T1",
          "access_type": "ref",
          "possible_keys": [
            "<auto_key0>"
          ],
          "key": "<auto_key0>",
          "used_key_parts": [
            "cluster_id"
          ],
          "key_length": "8",
          "ref": [
            "exps_db.T2.cluster_id"
          ],
          "rows_examined_per_scan": 10,
          "rows_produced_per_join": 8514,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "8514.81",
            "eval_cost": "1702.96",
            "prefix_cost": "10397.97",
            "data_read_per_join": "199K"
          },
          "used_columns": [
            "cluster_id",
            "instance_count"
          ],
          "materialized_from_subquery": {
            "using_temporary_table": true,
            "dependent": false,
            "cacheable": true,
            "query_block": {
              "select_id": 2,
              "cost_info": {
                "query_cost": "372.20"
              },
              "grouping_operation": {
                "using_filesort": false,
                "table": {
                  "table_name": "assets_instance",
                  "access_type": "index",
                  "possible_keys": [
                    "ForeignKey_cluster_id"
                  ],
                  "key": "ForeignKey_cluster_id",
                  "used_key_parts": [
                    "cluster_id"
                  ],
                  "key_length": "8",
                  "rows_examined_per_scan": 1771,
                  "rows_produced_per_join": 1771,
                  "filtered": "100.00",
                  "using_index": true,
                  "cost_info": {
                    "read_cost": "18.00",
                    "eval_cost": "354.20",
                    "prefix_cost": "372.20",
                    "data_read_per_join": "5M"
                  },
                  "used_columns": [
                    "instance_id",
                    "cluster_id"
                  ]
                }
              }
            }
          }
        }
      }
    ]
  }
}

```

由于查询中使用STRAIGHT_JOIN关键词，将`assets_cluster`作为外表，派生表结果集作为内表，由于内表上包含`cluster_id`的索引(auto_key0)，因此采用`NEST LOOP`方式循环外表的每行记录对内表做KEY LOOKUP查询。

