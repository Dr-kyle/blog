---
title: "Data Streams"
date: 2021-04-28T15:38:36+08:00
lastmod: 2021-04-28T15:38:36+08:00
draft: true
keywords: ["data", "streams", "data streams"]
description: "elasticsearch data streams"
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
author: "Kyle Zhao"

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

Elasticsearch Data Streams

<!--more-->

Data stream （数据流）是 Elastic Stack 7.9 的一个新的功能。Data stream 可以跨多个索引存储只追加时序性数据，同时为查询写入等请求提供唯一的一个命名资源。 data stream 非常适合日志，事件，指标以及其他持续生成的数据。

简单来说，data stream根据模板生成存储数据的后备索引，然后自动将搜索或者索引请求路由到存储流数据的后备索引。 而这些后备索引则根据索引生命周期管理（ILM）来自动管理。 例如，你可以使用 ILM 自动将较旧的后备索引移动到较便宜的硬件上（冷热数据处理），根据索引大小自动rollover出新的后备索引，或者删除到时间限制的索引。

## Data stream 组成

数据流在elasticsearch集群中由一个或多个隐藏的、自动生成的后备索引组成

![Data stream](https://tcs.teambition.net/storage/312400e3e42b9d0901c4704d52c050bf75db?Signature=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJBcHBJRCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9hcHBJZCI6IjU5Mzc3MGZmODM5NjMyMDAyZTAzNThmMSIsIl9vcmdhbml6YXRpb25JZCI6IiIsImV4cCI6MTYyMDE5OTQ3MywiaWF0IjoxNjE5NTk0NjczLCJyZXNvdXJjZSI6Ii9zdG9yYWdlLzMxMjQwMGUzZTQyYjlkMDkwMWM0NzA0ZDUyYzA1MGJmNzVkYiJ9.xYVI3T5E50WvavP9y2kSLBxlenipDxG98AaNcub5YZM)

数据流依靠索引模板来设定数据流实体的后备索引。

- 模板包含用于配置流的后备索引的映射和设置。
- 同一个索引模板可用于多个数据流。
- 不能删除数据流正在使用的索引模板。

每个索引到数据流的文档必须包含一个`@timestamp` 字段，映射为date或date_nanos字段类型。如果索引模板没有为@timestamp字段指定映射，Elasticsearch将@timestamp映射为带有默认选项的日期字段。

## Read Index

数据流提交读取请求时，该流会将请求路由到其所有后备索引

![Read](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/data-streams/data-streams-search-request.svg)

## Write Index

新的文档仅添加到最新的索引，不能将新文档添加到旧索引，即使直接将请求发送到旧索引。

![request](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/data-streams/data-streams-index-request.svg)

## Rollover

在data stream的使用中，rollover是必不可少的条件。

创建数据流时，Elasticsearch 会自动为该data stream根据template创建一个后备索引。 该索引还充当流的第一个写入索引。当满足一定条件时， rollover 会创建一个新的后备索引，该后备索引将成为data stream的新写入索引。

## Append-Only

由于时序性数据的特征，data stream的设计场景中，数据是只追加的，极少需要修改删除。如果实际需要修改删除，则可以考虑以下操作：

- 对于数据流只能通过update by query或者delete by query操作，不能进行update或者delete文档。

- 需要delete或者update文档，则直接对后备索引操作。

如果经常更新或删除现有文档，请使用索引别名和索引模板，数据流不适合这种场景。

## 使用

在没有数据流 之前，我们要使用 ILM 都是通过三个步骤

1. 创建生命周期策略

   ```
   PUT _ilm/policy/logx_policy
   {
     "policy": {
       "phases": {
         "hot": {
           "min_age": "0ms",
           "actions": {
             "rollover": {
               "max_size": "20gb",
               "max_age": "1d"
             }
           }
         },
         "delete": {
           "min_age": "30d",
           "actions": {
             "delete": {}
           }
         }
       }
     }
   }
   ```

2. 将生命周期策略应用到模板中并指定滚动别名。

   ```
   PUT _template/logx-kyle-template
   {
     "index_patterns": ["logx-kyle-*"],
     "settings": {
       "index" : {
        "refresh_interval" : "60s",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "lifecycle.name": "logx_policy",
        "lifecycle.rollover_alias": "logx-kyle"
      }
     }
   }
   ```

3. 创建第一个index,以下两种形式任选一种即可， index 格式必须满足该正则 *^.\*-\d+$* , example：logs-000001

   ```
   PUT logx-kyle-000001
   {
     "aliases": {
       "logx-kyle": {
         "is_write_index": true
       }
     }
   }
   OR
   # PUT /<logx-kyle-{now/d}-1> with URI encoding:
    PUT /%3Clogx-kyle-%7Bnow%2Fd%7D-1%3E 
    {
      "aliases": {
        "logx-kyle": {
          "is_write_index": true
        }
      }
    } 
   ```

   每一种日志业务我们都至少需要执行 2 ，3  步骤，较为麻烦。

   使用数据流只需要创建一次模板，后续直接创建 index 或者 直接写入数据就可以开始使用。

### 创建数据流

1. 创建生命周期策略

   ```
   PUT _ilm/policy/logx_policy
   {
     "policy": {
       "phases": {
         "hot": {
           "min_age": "0ms",
           "actions": {
             "rollover": {
               "max_size": "20gb",
               "max_age": "1d"
             }
           }
         },
         "delete": {
           "min_age": "30d",
           "actions": {
             "delete": {}
           }
         }
       }
     }
   }
   ```

2. 创建模板

   ```
   PUT _index_template/logx-template
   {
     "index_patterns" : ["logx-*"],
     "priority" : 1,
     "data_stream": { },
     "template": {
       "settings" : {
         "index" : {
   		"number_of_shards" : "1",
   		"number_of_replicas" : "1",
   		"lifecycle.name": "logx_policy"
   	  }
       }
     }
   }
   ```

3. 创建一个 data stream: logx-business，要符合模板正则

   ```
   POST /logx-business/_doc/
   {
   	"@timestamp":"2021-04-13T11:04:05.000Z",
   	"message":"Loginattemptfailed"
   }
   # OR
   PUT /_data_stream/logx-business
   ```

   后续的数据写入读取都使用 logx-business 即可。

   新的业务数据流直接执行 3 步骤就可以，不需要每种业务都要创建单独的模板。

