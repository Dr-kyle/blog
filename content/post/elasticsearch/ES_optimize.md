---
title: "Es Settings"
date: 2021-08-07T22:15:40+08:00
lastmod: 2021-08-07T22:15:40+08:00
draft: false
keywords: ["elasticsearch"]
description: "elasticsearch"
tags: ["elasticsearch"]
categories: ["elasticsearch"]
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

Elasticsearch settings

<!--more-->

# Index Settings

设置 _routing 参数为 true，CURD 必须包含 _routing 参数，否则报错`routing_missing_exception`

为了 避免数据不均衡，可以路由到分片子集，而不是单个分片

```sh
PUT my-index-000002
{
  "mappings": {
    "_routing": {
      "required": true 
    }
  },
  "settings": {
    // shard_num = hash(_routing) % num_primary_shards
    // shard_num = (hash(_routing) + hash(_id) % routing_partition_size) % num_primary_shards
    // 此值小于 index.number_of_shards 的值，缺点：不能使用 join 字段类型，所有的请求 _routing 参数为必须
    "index.routing_partition_size": 1,
    // 默认 10000
    "index.max_result_window": 5000
  }
}
```



# 集群设置

[http.max_content_length](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#http-settings)

默认 100MB，可以适当调大，但是 lucene 限制为 2GB。

`cluster.publish.timeout`

默认为`30s`，从发布开始的时间开始计算。如果在提交新集群状态之前达到这个时间，那么集群状态更改将被拒绝，并且主节点认为自己已经失败。它停止并开始尝试选举一个新的主人。

`cluster.follower_lag.timeout`

默认为`90s`. 如果节点在这段时间内仍未成功应用集群状态更新，则认为它已失败并从集群中删除

[`monitor.fs.health`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-settings.html)

每个节点通过将一个小文件写入磁盘然后再次删除它来定期验证其数据路径是否健康。如果节点发现其数据路径不正常，则将其从集群中删除，直到数据路径恢复。您可以使用[`monitor.fs.health`设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-settings.html)来控制此行为 。

# 集群优化点

1. 不要返回太大的数据集

   Elasticsearch 被设计为一个搜索引擎，这使得它非常擅长获取与查询匹配的顶级文档，例如检索与特定查询匹配的所有文档，则效果不佳。

2. 避免大的文档

   大文档对网络、内存使用和磁盘的压力更大，即使不获取 _source 字段，但是大多数情况需要获取文档 _id, 由于文件系统缓存的工作方式，获取该字段的成本更大，索引此文档可能使用的内存量是文档原始大小的乘数，短语查询和高亮查询也会更耗资源。

3. 使用 bulk 批量提交

   过大的批量请求可能会使集群承受内存压力，因此建议避免每个请求超过几十兆字节

4. 使用多线程将数据发送到 elasticsearch

   除了更好地利用集群的资源之外，有助于降低每个 fsync 的成本。

5. 取消设置或增加 refresh interval

   正在进行索引时经常调用它会影响索引速度，默认情况下，Elasticsearch 每秒都会定期刷新索引，但仅限于在过去 30 秒内收到一个或多个搜索请求的索引，如果数据不经常被搜索，不建议设置此值。

6. 留内存给文件系统

   文件系统缓存将用于缓冲 I/O 操作。您应该确保至少将运行 Elasticsearch 的机器的一半内存分配给文件系统缓存。

7. 使用自动生成的ID

   显示指定 ID ，Elasticsearch  需要检查 id 的文档是否已经存在，自动生成的 ID 可以跳过该步骤。

8. 使用 SSD

9. 索引缓存区大小

   如果节点仅执行大量索引，请确保 [`indices.memory.index_buffer_size`](https://www.elastic.co/guide/en/elasticsearch/reference/current/indexing-buffer.html)足够大，以便为每个分片提供最多 512 MB 的索引缓冲区进行大量索引（除此之外，索引性能通常不会提高），默认堆内存的 10%

10. 使用跨集群复制(该功能收费)

    在单个集群中，索引和搜索可能会争夺资源。通过设置两个集群，配置[跨集群复制](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-ccr.html)以将数据从一个[集群复制](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-ccr.html)到另一个集群，并将所有搜索路由到具有跟随者索引的集群

11. 为文档创建映射

     joins 应该避免使用，[`nested`](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)可以使查询慢几倍，[父子](https://www.elastic.co/guide/en/elasticsearch/reference/current/parent-join.html)关系可以使查询慢数百倍

12. 尽可能搜索少的字段

    如果要经常搜索 a,b,c 三个字段，可以考虑使用 copy_to

13. 预处理数据

    ```
    PUT index/_doc/1
    {
      "designation": "spoon",
      "price": 13
    }
    
    POST index/_search
    {
      "aggs": {
        "price_ranges": {
          "range": {
            "field": "price",
            "ranges": [
              { "to": 10 },
              { "from": 10, "to": 100 },
              { "from": 100 }
            ]
          }
        }
      }
    }
       
    ```
    
    使用以下方式会更好
    
    ```
    PUT index/_doc/1
        {
          "designation": "spoon",
          "price": 13,
          "price_range": "10-100"
        }
        POST index/_search
        {
          "aggs": {
            "price_ranges": {
              "terms": {
                "field": "price_range"
              }
            }
          }
        }
    ```
    
    

14. 优先使用 keyword 类型

    `term`对`keyword`字段的查询搜索通常比`term`对数字字段的搜索快

15. 避免使用 script

    如果可能，请避免使用基于[脚本](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)的排序、聚合中的脚本和[`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-score-query.html)查询

16. 搜索四舍五入的日期

    对使用的日期字段的查询`now`通常不可缓存，因为匹配的范围一直在变化。但是，从用户体验的角度来看，切换到舍入日期通常是可以接受的，并且可以更好地利用查询缓存。

    ```sh
    PUT index/_doc/1
    {
      "my_date": "2016-05-11T16:30:55.328Z"
    }
    
    GET index/_search
    {
      "query": {
        "constant_score": {
          "filter": {
            "range": {
              "my_date": {
                "gte": "now-1h",
                "lte": "now"
              }
            }
          }
        }
      }
    }
    ```

    可以使用如下查询代替
    

    ```sh
    GET index/_search
    {
      "query": {
        "constant_score": {
          "filter": {
            "range": {
              "my_date": {
                // 四舍五入到分钟 
                "gte": "now-1h/m",
                "lte": "now/m"
              }
            }
          }
        }
      }
    }
    ```

17. force merge 只读 index

18. 预热全局序数

    [全局序数](https://www.elastic.co/guide/en/elasticsearch/reference/current/eager-global-ordinals.html)是一种用于优化聚合性能的数据结构，它们被延迟计算并作为[字段数据缓存的](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-fielddata.html)一部分存储在 JVM 堆中。对于大量用于分桶聚合的字段，您可以告诉 Elasticsearch 在收到请求之前构造和缓存全局序数。这应该小心完成，因为它会增加堆使用量，并使[刷新](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-refresh.html) 需要更长的时间, 经常需要根据某字段聚合 可以开启该设置。

    ```sh
    PUT index
    {
      "mappings": {
        "properties": {
          "foo": {
            "type": "keyword",
            "eager_global_ordinals": true
          }
        }
      }
    }
    ```

19. 减少集群shard数量

    每个分片都使用内存。在大多数情况下，一小组大分片比许多小分片使用更少的资源

20. 避免使用昂贵的查询

    昂贵的搜索会占用大量内存

    ```
    PUT _settings
    {
      "index.max_result_window": 5000
    }
    
    PUT _cluster/settings
    {
      "persistent": {
        "search.max_buckets": 20000,
        "search.allow_expensive_queries": false
      }
    }
    ```

    

# 磁盘优化

1. 禁用不需要的字段

   默认情况下，文本字段还存储索引中的频率和位置，频率用于计算分数，位置用于运行短语查询。如果您不需要运行短语查询 "index_options": "freqs"

   ```
   PUT index
   {
     "mappings": {
       "properties": {
         "foo": {
           "type": "integer",
           "index": false
         },
         "foo1": {
           "type": "text",
           // 如果不关心分数，可以将此值设置为 false
           "norms": false,
           "index_options": "freqs"
         }
       }
     }
   }
   ```

2. 不要使用默认的 动态 字符串映射

   默认字符串动态映射 会 生成 keyword 和 text 两个字段

   ```
   PUT index
   {
     "mappings": {
       "dynamic_templates": [
         {
           "strings": {
             "match_mapping_type": "string",
             "mapping": {
               "type": "keyword"
             }
           }
         }
       ]
     }
   }
   ```

3. 关注分片大小

4. 如果不需要 _source 字段，可以禁用，但是 update 和 reindex 将不可用

5. 使用 `best_compression` [codec](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-codec).

6. 在文档中以相同的顺序存放字段

   由于多个文档被一起压缩成块，`_source`如果字段总是以相同的顺序出现，则更有可能在这些文档中找到更长的重复字符串。

7. 汇总历史数据

   保留较旧的数据对以后的分析很有用，但由于存储成本，通常会避免。您可以使用数据汇总以原始数据存储成本的一小部分来汇总和存储历史数据。

