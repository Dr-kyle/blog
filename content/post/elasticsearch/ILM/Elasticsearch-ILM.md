---
title: "Elasticsearch 生命周期管理"
date: 2021-04-15T21:31:10+08:00
lastmod: 2021-04-15T21:31:10+08:00
draft: false
keywords: ["elasticsearch", "ILM"]
description: "elasticsearch ILM"
tags: ["elasticsearch "]
categories: ["elasticsearch "]
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

elasticsearch 生命周期介绍

<!--more-->

本文已收录至《Elastic Stack 实战手册》，欢迎和我一起解锁开发者共创书籍，系统学习 Elasticsearch

[![](/images/alibaba.png)](https://developer.aliyun.com/topic/elasticstack/playbook)

# 索引生命周期管理

Elasticsearch 在 6.7 版本正式加入索引生命周期管理，旨在管理 Elasticsearch 中的索引。

通常我们使用 elasticsearch 的时候，index 命名都是 xxx-YYYY.MM.dd 类似这样的格式，每天创建一个index，这需要我们自己创建 index，或者通过自动创建。

1. 每天创建一个 index，但是每天的数据量又非常少，这对集群来说是不利的。
2. 如果是自动创建的话，集群 index 和 shard 数过多，那么在每天的 00:00 时，大量的 index 同时创建，这时我们就会发现集群的写入速度会变慢，可能会发生 index 写入拒绝的情况。
3. 集群需要对冷热数据进行分离，性能好的机器放最近频繁查询的数据，随着时间推移，数据查询不在频繁，需要将数据迁移到性能较差的机器上。

以上这些我们可以使用 Elasticsearch 提供的索引生命周期管理功能能很好的解决，接下来我们了解一下 索引生命周期管理。

## 索引生命周期的四个阶段

- Hot

  index 正在查询和更新，一般性能好的机器会设置为 Hot 节点来进行数据的读写。

- Warm

  index不再更新，但是仍然需要查询，节点性能一般可以设置为 Warm 节点。

- Cold

  index不再被更新，且很少被查询，数据仍然可以搜索，但是能接受较慢的查询，节点性能较差，但有大量的磁盘空间。

- Delete

  数据不需要了，可以删除。

节点的类型可以通过一下两种方式设置，推荐第二种，第一种后续可能会弃用。

第一种：

```yaml
# elasticsearch.yml
# node.attr.xxx: xxx
node.attr.data: warm
```

第二种（推荐）：

```yaml
# elasticsearch.yml 
# data_content, data_hot, data_warm, data_cold
# 配置该节点既属于内容层又属于热层
node.roles: ["data_hot", "data_content"]
```

这四个阶段按照 Hot，Warm，Cold，Delete 顺序执行，上一个阶段没有执行完成是不会执行下一个阶段的，对于不存在的阶段，会跳过该阶段进入到下一个阶段。

生命周期默认每 10 分钟检测一次，可以通过集群的配置动态修改，如下

```shell
PUT _cluster/settings
{
  "transient": {
    "indices.lifecycle.poll_interval": "10m" 
  }
}
```

## 生命周期管理 API

每个阶段支持的行为会在下一章节进行介绍，此章节仅仅为了介绍 API。

1. 创建生命周期管理策略

   min_age 参数指定从 index 创建后多长时间进入到该阶段。

   以下示例是指从当 index 创建时间超过 10 天后，进入到 warm 阶段，将 segment 数量 merge 为 1，warm 阶段完成后，进入 delete 阶段，index 创建时间超过 30 天后，将 index 删除。

   ```shell
   PUT _ilm/policy/my_policy
   {
     "policy": {
       "phases": {
         "warm": {
           "min_age": "10d",
           "actions": {
             "forcemerge": {
               "max_num_segments": 1
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

2. 查看生命周期管理策略

   ```shell
   # 查看所有的生命周期管理策略
   GET _ilm/policy
   
   # 查看特定的生命周期管理策略
   # GET _ilm/policy/<policy_id>
   ```

3. 删除生命周期管理策略

   ```shell
   DELETE _ilm/policy/<policy_id>
   ```

4. 触发生命周期策略中特定步骤的执行

   current_step

   - phase 当前阶段的名称
   - action 当前行为的名称
   - name  当前步骤的名称

   next_step

   - phase 想要执行阶段的名称
   - action 想要执行行为的名称
   - name  想要执行步骤的名称

   ```shell
   POST _ilm/move/my-index-000001
   {
     "current_step": { 
       "phase": "new",
       "action": "complete",
       "name": "complete"
     },
     "next_step": { 
       "phase": "warm",
       "action": "forcemerge",
       "name": "forcemerge"
     }
   }
   ```

   5. 移除生命周期管理策略

      ```shell
      # POST <target>/_ilm/remove
      POST my-index-000001/_ilm/remove
      ```

   6. 生命周期重试

      ```shell
      # POST <index>/_ilm/retry
      POST my-index-000001/_ilm/retry
      ```

   7. 查看当前索引生命周期管理状态

      ```shell
      GET /_ilm/status
      ```

   8. 查看一个或多个索引的当前生命周期状态

      ```shell
      # GET <target>/_ilm/explain
      GET my-index-000001/_ilm/explain
      ```

   9. 启动索引生命周期管理插件

      ```shell
      POST _ilm/start
      ```

   10. 停止索引生命周期管理插件

       ```shell
       POST /_ilm/stop
       ```

   

## 四个阶段支持的行为

索引生命周期每个阶段支持的行为如下：

- Hot

  - Set Priority

  - Unfollow

  - Force Merge

  - Rollover

- Warm

  - Set Priority
  - Unfollow
  - Read only
  - Allocate
  - Shrink
  - Force Merge
  - Migrate

- Cold

  - Set Priority
  - Unfollow
  - Allocate
  - Migrate
  - Freeze
  - Searchable Snapshot

- Delete

  - Wait For Snapshot
  - Delete

### 行为

**Set Priority**

设置索引的优先级，一旦进入到某阶段，就设置索引的优先级，节点重新启动后，优先级较高的索引将会优先恢复。

参数:

- priority:  正整数。

例如：设置 warm 阶段 index 的优先级为 50

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "set_priority" : {
            "priority": 50
          }
        }
      }
    }
  }
}
```

**Rollover**

当 index 满足三个条件中的任何一个时，会将别名指向新生成的索引。

参数：

- max_age

  达到索引创建的最大时间

- max_docs

  达到指定的文档数后

- max_size

  index 达到指定的大小时，主分片的大小，不包含副本。

以上三个参数至少应该存在一个

例如：当前 index 主分片大小达到 100GB 或文档数超过 100000000 或者 index 创建超过 7天 生产新的 index

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover" : {
            "max_size": "100GB",	
			"max_docs": 100000000,
			"max_age": "7d"
          }
        }
      }
    }
  }
}
```

**Unfollow**

将 follow 索引转换为正常索引。

例如：

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "unfollow" : {}
        }
      }
    }
  }
}
```

**Allocate**

指定 index 的副本数，迁移 index 到某些节点，冷热节点数据迁移依赖此步骤。

参数：

- number_of_replicas

  指定 index 的副本数

- include

  将 index 迁移到具有指定属性之一的节点

- exclude

  将 index 迁移到不包含指定属性的节点

- require

  将 index 迁移到具有所有指定属性的节点

Note: include 满足其中一个就可以， require 必须全部满足。

例如：到达 warm 阶段将 index 的备份数设置为 2，并且将 index 迁移至属性 box_type 包含 hot, warm 且不包含 cold 的节点。

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "allocate" : {
            "number_of_replicas" : 2,
			"include" : {
		        "box_type": "hot"
            },
			"exclude" : {
		        "box_type": "cold"
            },
			"require" : {
		        "box_type": "hot,warm"
            }
          }
        }
      }
    }
  }
}
```

**Force Merge**

指定 index 合并后 segment 数量，在 hot 阶段使用时，必须包含 rollover ，merge 时会将 index 设置为只读。

参数：

- max_num_segments

  segment 最大数量

- index_codec

  压缩文件存储， default: LZ4

例如：warm 阶段将 index 的 segments 合并为 1。

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "forcemerge" : {
            "max_num_segments": 1
          }
        }
      }
    }
  }
}
```

**Read only**

将 index 设置为只读。

例如：

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "readonly" : { }
        }
      }
    }
  }
}
```

**Shrink**

index 设置为只读，然后将 index 缩小为具有更少的的 shard， 缩小后的 index 名称为 shrink-\<original-index-name> 

参数：

- number_of_shards

  合并后的主分片数

例如：warm 阶段将 index 的 shard 数合并为 1 个。

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "shrink" : {
            "number_of_shards": 1
          }
        }
      }
    }
  }
}
```

**Freeze**

最大程度减少 index 的内存占用。

例如：cold 阶段将 index freeze，释放内存。

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "cold": {
        "actions": {
          "freeze" : { }
        }
      }
    }
  }
}
```

**Migrate**

通过更新 index.routing.allocation.include._tier_preference 设置，将 index 移动到对应的数据层，如果指定了 allocate，会在迁移前先将副本数减少。如果在热阶段和冷阶段没有指定 allocate 分配选项，ILM 会自动注入迁移操作，如果要禁用可以将 enabled 设置为 false。

参数：

- enabled

  default: true, 控制 ILM 在此阶段是否自动迁移索引

例如：warm 阶段禁用迁移操作， 主动将 index 备份数设置为 1，并且将 index 迁移至属性 rack_id 为 one 或者 two 的节点。

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "warm": {
        "actions": {
          "migrate" : {
           "enabled": false
          },
          "allocate": {
			"number_of_replicas": 1,
            "include" : {
              "rack_id": "one,two"
            }
          }
        }
      }
    }
  }
}
```

**Searchable Snapshot**

生成可搜索快照，在 7.10 版本还处于 beta，在新版可能会有所更改。

在 delete action 步骤中默认会删除快照，如果需要保留，在 delete action 中将 delete_searchable_snapshot 设置 false

参数：

- snapshot_repository

  Required，指定存储快照的位置

- force_merge_index

  Boolean，default: true,  如果索引在先前的操作中已经使用了 force merge， 则可搜索快照操作不会执行强制合并。

例如：在 cold 阶段生成快照。

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "cold": {
        "actions": {
          "searchable_snapshot" : {
            "snapshot_repository" : "backing_repo"
          }
        }
      }
    }
  }
}
```

**Wait For Snapshot**

等待制定的 SLM 策略执行，然后在删除索引，为了确保删除的索引快照是可用的。

参数：

- policy

  reqired， SML 策略的名字

例如：delete 阶段等待 SLM 策略执行，然后删除索引。

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "delete": {
        "actions": {
          "wait_for_snapshot" : {
            "policy": "slm-policy-name"
          }
        }
      }
    }
  }
}
```

**Delete**

删除 index 

参数：

- delete_searchable_snapshot

  boolean, default: true,  是否删除 cold 阶段创建的 searchable snapshot。

例如：index 创建 90 天后，删除 index

```shell
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "delete": {
		"min_age" : "90d",
        "actions": {
          "delete" : { }
        }
      }
    }
  }
}
```

## 通过 alias 使用 ILM

1. 创建生命周期策略  

   warm 阶段将 index 分配给节点属性 data 为 warm 的节点， cold 阶段将 index 分配给节点属性 data 为 cold 的节点。

   节点属性可以通过 elasticsearch.yml 进行配置或环境变量设置。

   ```yaml
   # 启动命令
   bin/elasticsearch -Enode.attr.data=warm
   
   # elasticsearch.yml
   # node.attr.xxx: xxx
   # 建议使用 node.roles 进行配置, 可以参考 通过 data tiers 使用 ILM 这一章节
   # node.attr 后续版本可能不在使用
   
   node.attr.data: warm
   ```

   创建生命周期策略, 在 index 创建 1 天后进入 hot 阶段，设置优先级为 100， 当 index 主分片大小超过 50gb 或者 index 文档数超过 500000000 或者 index 创建超过 2 天生成新的 index

   warm 阶段将 index 迁移至属性 data 为 warm 的节点

   cold 阶段将 index 副本数设置为 1 并将 index 迁移至属性 data 为 cold 的节点

   当 hot ，warm，cold 阶段的动作都完成并且 index 创建达到 7 天，删除 index。

   ```shell
   
   PUT _ilm/policy/logx_policy
   {
     "policy": {
       "phases": {
         "hot": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 100
             },
             "rollover": {
               "max_age": "2d",
               "max_docs": 500000000,
               "max_size": "50gb"
             }
           }
         },
         "warm": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 50
             },
             "allocate": {
               "include": {
                 "data" : "warm"
               }
             }
           }
         },
         "cold": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 0
             },
             "allocate": {
               "number_of_replicas": 1,
               "include" : {
                 "data": "cold"
               }
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

2. 创建索引模板，将生命周期应用到 index

   设置 shard 数为 2， 备份数 为 1， 生命周期策略为 logx_policy，滚动别名为 logx

   ```shell
   PUT _index_template/logx-template
   {
     "index_patterns" : ["logx-*"],
     "priority" : 1,
     "template": {
       "settings" : {
         "index" : {
   		"number_of_shards" : "2",
   		"number_of_replicas" : "1",
   		"lifecycle.name": "logx_policy",
   		"lifecycle.rollover_alias": "logx"
   	  }
       }
     }
   }
   ```

3. 创建第一个 index，以下两种形式任选一种即可， index 格式必须满足该正则 *^.\*-\d+$* , example：logs-000001

   ```shell
   PUT logx-000001
   {
     "aliases": {
       "logx": {
         "is_write_index": true
       }
     }
   }
   # OR 带创建日期的 index
   # PUT /<logx-{now/d}-1> with URI encoding:
   PUT /%3Clogx-%7Bnow%2Fd%7D-1%3E 
   {
     "aliases": {
       "logx": {
         "is_write_index": true
       }
     }
   } 
   ```

后续的数据读写均使用固定别名 logx

## 通过 Data stream 使用 ILM

1. 创建生命周期策略  

   创建生命周期策略，在 index 创建 1 天后进入 hot 阶段，设置优先级为 100， 当 index 主分片大小超过 50gb 或者 index 文档数超过 500000000 或者 index 创建超过 2 天生成新的 index，warm 阶段将 index 迁移至属性 data 为 warm 的节点，cold 阶段将 index 副本数设置为 1 并将 index 迁移至属性 data 为 cold 的节点，当 hot ，warm，cold 阶段的动作都完成并且 index 创建达到 7 天，删除 index。

   ```shell
   
   PUT _ilm/policy/logx_policy
   {
     "policy": {
       "phases": {
         "hot": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 100
             },
             "rollover": {
               "max_age": "2d",
               "max_docs": 500000000,
               "max_size": "50gb"
             }
           }
         },
         "warm": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 50
             },
             "allocate": {
               "include": {
                 "data" : "warm"
               }
             }
           }
         },
         "cold": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 0
             },
             "allocate": {
               "number_of_replicas": 1,
               "include" : {
                 "data": "cold"
               }
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

2. 创建索引模板，将生命周期应用到 index

   与通过 alias 的形式区别: 模板不需要指定 index.rollover_alias，也不需要手动创建第一个index，直接将数据写入符合模板的 index 即可，至于这个 index 在 Elasticsearch 中对应几个index，我们无需关注。

   ```shell
   PUT _index_template/logx-template
   {
     "index_patterns" : ["logx-*"],
     "priority" : 1,
     "data_stream": { },
     "template": {
       "settings" : {
         "index" : {
   		"number_of_shards" : "2",
   		"number_of_replicas" : "1",
   		"lifecycle.name": "logx_policy"
   	  }
       }
     }
   }
   ```

3. 创建 data stream

   ```shell
   POST /logx-business/_doc/
   {
   	"@timestamp":"2021-04-13T11:04:05.000Z",
   	"message":"Loginattemptfailed"
   }
   # OR 
   PUT /_data_stream/logx-business
   ```

后续的数据读写均使用固定 index:  logx-business

## 通过 Data tiers 使用 ILM

data tiers （ 数据层 ）是具有相同数据角色的节点的集合

- Content tier （ 内容层 ）节点处理诸如产品目录之类的内容的索引和查询负载。
- Hot tier （ 热层 ） 节点处理诸如日志或指标之类的时间序列数据的索引负载，并保存你最近，最常访问的数据。
- Warm tier （ 温层 ）节点保存的时间序列数据访问频率较低，并且很少需要更新。
- Cold tier （ 冷层 ）节点保存时间序列数据，这些数据偶尔会被访问，并且通常不会更新。

推荐冷热分离采用 data tiers 这种方式，节点可以通过如下配置方式配置：

```yaml
# elasticsearch.yml 
# data_content, data_hot, data_warm, data_cold
# 配置该节点既属于内容层又属于热层
node.roles: ["data_hot", "data_content"]
```

1. 创建生命周期策略 

   warm 阶段将 index 迁移至 warm 节点，cold 阶段禁用 migrate，将 index 分配给 rack_id 为 one 或 two 的节点。

   创建生命周期策略，在 index 创建 1 天后进入 hot 阶段，设置优先级为 100， 当 index 主分片大小超过 50gb 或者 index 文档数超过 500000000 或者 index 创建超过 2 天生成新的 index，warm 阶段将 index 迁移至 warm 节点，cold 阶段将 index 副本数设置为 1，禁用 migrate， 并将 index 迁移至属性 data 为 cold 的节点，当 hot ，warm，cold 阶段的动作都完成并且 index 创建达到 7 天，删除 index。

   ```shell
   PUT _ilm/policy/logx_policy
   {
     "policy": {
       "phases": {
         "hot": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 100
             },
             "rollover": {
               "max_age": "2d",
               "max_docs": 500000000,
               "max_size": "50gb"
             }
           }
         },
         "warm": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 50
             },
             "migrate" : {
             }
           }
         },
         "cold": {
           "min_age": "1d",
           "actions": {
             "set_priority": {
               "priority": 0
             },
             "allocate": {
               "number_of_replicas": 1,
               "include" : {
                 "data": "cold"
               }
             }, 
             "migrate" : {
               "enabled": false
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

2. 创建索引模板，将生命周期应用到 index

   设置 shard 数为 2， 备份数 为 1， 生命周期策略为 logx_policy

   ```shell
   PUT _index_template/logx-template
   {
     "index_patterns" : ["logx-*"],
     "priority" : 1,
     "data_stream": { },
     "template": {
       "settings" : {
         "index" : {
   		"number_of_shards" : "2",
   		"number_of_replicas" : "1",
   		"lifecycle.name": "logx_policy"
   	  }
       }
     }
   }
   ```

3. 创建 data stream

   ```shell
   POST /logx-business/_doc/
   {
   	"@timestamp":"2021-04-13T11:04:05.000Z",
   	"message":"Loginattemptfailed"
   }
   # OR 
   PUT /_data_stream/logx-business
   ```

后续的数据读写均使用固定 index:  logx-business