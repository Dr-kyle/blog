---
title: "ES Manager"
date: 2020-12-11T20:54:18+08:00
lastmod: 2020-12-11T20:54:18+08:00
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

ES Manager

<!--more-->

# ES Manager

```
第一目标代替cerebro
ES-portal 功能点   
1. 角色划分  某些集群管理员 超级集群管理员
1. 集群管理员查看集群状态，集群指标(集群名，node数，indices数，shards数，文档数，存储量)
2. 集群状态预警(发微信，发邮件)
3. 查看index分配情况， 未分配的shard, 已分配的shard 参考cerebro
4. 模板模块
5. 线程池的监控

Index module
Index 增长情况，写入量增长情况(建议所有写入的数据增加写入的时间 insetTime)

node module
统计每个节点上index占用的内存
每个节点上占用磁盘最大的index
```

## Home

1. every cluster status，documents...，kibana link
2. 所有集群总量

## Request

create index or import data from other source

## Cluster Monitor

- Index
  - Index List
    - index write speed, index search speed, index settings
    - index 在每个节点上所占的磁盘大小
    - index 在每个节点上所占的内存大小
- node
  - node list （参考cerebro）
    - 每个节点上的index 占用大小(大到小排序)
    - 每个节点上的index 占用内存大小
- cluster
  - thread pool monitor

## Cluster Manage

- cluster

  add, delete, update

- template(kibana 已有）



## Organization

- Team

  `admin`,`normal`,``

- User