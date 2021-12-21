---
title: "Elasticsearch Upgrade"
date: 2021-03-09T10:46:28+08:00
lastmod: 2021-03-09T10:46:28+08:00
draft: true
keywords: []
description: ""
tags: []
categories: []
author: ""

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

<!--more-->

Elasticsearch Upgrade

为了降低影响，确保集群所有的 index 至少都有一个备份,采用滚动升级的方式。

## 1. **Disable shard allocation**.

因为是升级，所有没有必要让 节点上的 shard 进行 reallocation，避免 IO。设置只允许 新 index 的主分片分配。

```
all - 默认的，允许所有shards分配
primaries - 只允许主分片分配
new_primaries - 只允许新index的主分片分配
none - 所有的都不允许
```

根据集群的备份情况选择。

```console
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "new_primaries"
  }
}
```



## 2. **Stop non-essential indexing and perform a synced flush.** (Optional)

此步骤可选， 停止非必要的索引并执行同步刷新

```
POST _flush/synced
```

## 3. Stop Node

## 4. Start the upgraded node

使用以下命令查看集群节点，等节点加入集群后，执行第 5 步。

```
GET _cat/nodes
```

## 5. Reenable shard allocation

```
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
```

## 6. Wait for the node to recover

```console
GET _cat/health?v=true
```

等集群恢复正常后，重复执行以上步骤。