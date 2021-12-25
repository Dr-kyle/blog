---
title: "Elasticsearch Hdfs"
date: 2021-12-25T14:59:30+08:00
lastmod: 2021-12-25T14:59:30+08:00
draft: true
keywords: ["Elasticsearch", "hdfs"]
description: "Elasticsearch hdfs"
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

Elasticsearch 集群数据备份到 HDFS

<!--more-->

1. 安装 hdfs 插件

   ```
   sudo bin/elasticsearch-plugin install file:///data/repository-hdfs-7.16.1.zip
   ```

2. [注册存储库](https://www.elastic.co/guide/en/elasticsearch/plugins/7.16/repository-hdfs-config.html)

   ```shell
   PUT _snapshot/my_repository
   {
     "type": "hdfs",
     "settings": {
       "uri": "hdfs://namenode:8020/",
       "path": "elasticsearch/repositories/my_hdfs_repository",
       "conf.dfs.client.read.shortcircuit": "true"
     }
   }
   ```



## 手动创建快照

1. 创建快照

   ```
   # 在后台运行
   # PUT _snapshot/my_repository/<my_snapshot_{now/d}>
   PUT _snapshot/my_repository/%3Cmy_snapshot_%7Bnow%2Fd%7D%3E
   
   # 非后台运行
   PUT _snapshot/my_repository/%3Cmy_snapshot_%7Bnow%2Fd%7D%3E?wait_for_completion=true
   ```

2. 查看运行快照

   ```
   GET _snapshot/my_repository/_current
   GET _snapshot/_status
   ```

3. 删除 或 取消快照

   ```
   DELETE _snapshot/my_repository/my_snapshot_2099.05.06
   ```

## 自动创建快照

1. 创建快照

   ```
   # PUT _snapshot/my_repository/<my_index_snapshot_{now/d}>
   PUT _snapshot/my_repository/%3Cmy_index_snapshot_%7Bnow%2Fd%7D%3E
   ```

2. 创建 SLM policy

   ```
   PUT _slm/policy/nightly-snapshots
   {
     "schedule": "0 30 1 * * ?",       
     "name": "<my_index_snapshot_{now/d}>", 
     "repository": "my_repository",    
     "config": {
       "indices": "*",                 
       "include_global_state": true    
     },
     "retention": {                    
       "expire_after": "30d",
       "min_count": 5,
       "max_count": 50
     }
   }
   ```

3. 手动执行 SLM policy

   ```
   POST _slm/policy/nightly-snapshots/_execute
   ```

   