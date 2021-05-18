---
title: "Elasticsearch Shard"
date: 2021-03-29T11:20:22+08:00
lastmod: 2021-03-29T11:20:22+08:00
draft: false
keywords: ["elasticsearch", "ILM"]
description: "elasticsearch ILM"
tags: ["elasticsearch"]
categories: ["elasticsearch"]
author: "kyle zhao"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

Elasticsearch ILM 介绍
<!--more-->

# Elasticsearch Cluster Shard and ILM

## 索引优化点
ES每个索引由一个或多个分片组成，每个分片都是一个Lucene 索引实例。

1. 在集群中应该避免分片过大，分片过大可能会对集群从故障中恢复造成不利影响，通常分片上限为50GB。
2. 尽量使用时序型索引来管理具有数据保留期要求的数据，根据保留期限对数据分组，存储到索引中。通过时序型索引用户能随着时间推移轻松调整主分片和副本分片的数量。
3. 索引和分片都会产生一定的资源开销，分片过小会导致段过小，致使开销增加，尽量将分片的平均大小控制在至少几GB到几十GB之间，对于时序型索引，分片大小通常介于20GB至40GB之间。
4. .每个节点上可以存储的分片数量与可用的堆内存大小成正比关系，但是elasticsearch并未强制规定，如果某个节点拥有30GB的堆内存，则该节点最多可有600个分片(每GB 20 个分片)，分片数量越少，效果越好。

## 检测频率 默认10m
```sh
PUT _cluster/settings
{
  "transient": {
    "indices.lifecycle.poll_interval": "1m" 
  }
}
```

## ILM(index生命周期管理)
对于以往每天生成一个index的形式有如下缺点：

1. 一天的数据量可能太小，每天使用一个index造成资源浪费
2. index创建集中在00:00，造成该时间点集群响应变慢

对于以上问题我们可以使用ILM解决，以如下场景举例：

- 当索引达到50GB或文档数量超过500000000或index创建超过一天，切换到新的index写入
- 将旧索引标记为只读，缩小分片数
- 索引达到保留期7天后，自动删除

具体步骤如下：

1. 创建生命周期策略

   ```
   PUT _ilm/policy/kyle_policy
   {
     "policy": {
       "phases": {
         "hot": {
           "actions": {
             "rollover": {
               "max_docs": 500000000,
               "max_age": "1d",
               "max_size": "50gb"
             }
           }
         },
         "warm": {
           "actions": {
             "shrink": {
               "number_of_shards": 1
             }
           }
         },
         "delete": {
           "min_age": "7d",
           "actions": {
             "delete": {}
           }
         }
       }
     }
   }
   ```

2. 应用ILM策略到index

   ```
   PUT _template/kyletest-template
   {
     "index_patterns": ["kyletest-*"],
     "settings": {
       "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_ip" : "127.1.1.*"
            }
          }
        },
        "refresh_interval" : "60s",
        "number_of_shards" : "2",
        "number_of_replicas" : "1",
        "lifecycle.name": "kyle_policy",
        "lifecycle.rollover_alias": "kyle_data"
      }
       
     }
   }
   ```

3. 创建第一个index,以下两种形式任选一种即可， index 格式必须满足该正则 *^.\*-\d+$* , example：logs-000001

   ```
   PUT kyletest-000001
   {
     "aliases": {
       "kyle_data": {
         "is_write_index": true
       }
     }
   }
   OR
   # PUT /<kyletest-{now/d}-1> with URI encoding:
    PUT /%3Ckyletest-%7Bnow%2Fd%7D-1%3E 
    {
      "aliases": {
        "kyle_data": {
          "is_write_index": true
        }
      }
    } 
   ```

4. 检查 index生命周期策略是否正常工作

   ```
   GET kyletest-*/_ilm/explain
   RESPONSE：
   {
     "indices" : {
       "kyletest-2018-08-01" : {
         "index" : "kyletest-2018-08-01",
         "managed" : true,
         "policy" : "kyle_policy",
         "lifecycle_date_millis" : 1565156982695,
         "phase" : "hot", //what phase the index is currently in
         "phase_time_millis" : 1565156982795,
         "action" : "rollover", //what action the index is currently on
         "action_time_millis" : 1565156990814,
         "step" : "check-rollover-ready", //what step the index is currently on
         "step_time_millis" : 1565156990814,
         "phase_execution" : {
           "policy" : "kyle_policy",
           "phase_definition" : { //the definition of the phase (in this case, the "hot" phase) that the index is currently on
             "min_age" : "0ms",
             "actions" : {
               "rollover" : {
                 "max_docs" : 5
               }
             }
           },
           "version" : 6,
           "modified_date_in_millis" : 1565156951804
         }
       }
     }
   }
   ```

5. 使用别名写入数据

   ```
   POST kyle_data/_doc
   {
   	"name":"kyle",
   	"time":"2019-08-14T06:51:37.000Z"
   }
   ```

6. 查询数据

   ```
   GET kyle_data/_search
   ```
   

### Note(读者请忽略，我只是为了记录这个操作)
1. 手动生成下一个index

   ```
   POST /kyletest/_rollover/kyletest-000003
   {
     "conditions": {
       "max_age":   "1m"
     }
   }
   
   PUT kyletest-000002/_settings
    {
      "index.lifecycle.indexing_complete": true
    }
   ```