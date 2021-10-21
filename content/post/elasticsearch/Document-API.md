---
title: "Document API"
date: 2021-10-21T08:50:26+08:00
lastmod: 2021-10-21T08:50:26+08:00
draft: false
keywords: ["elasticsearch", "document", "api"]
description: "elasticsearch document api"
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

<!--more-->

# Document API

es: localhost:9200， index: kyle_test

## 单条新增

1. 不指定 ID，建议的方式

   ```
   POST http://localhost:9200/kyle_test/_doc
   {
   	"name": "kyle",
   	"age": 10,
   	"@timestamp": 1634609435199
   }
   ```

2. 指定 ID

   ```
   // id 存在则覆盖
   POST http://localhost:9200/kyle_test/_doc/id111
   {
   	"name": "kyle",
   	"age": 10,
   	"@timestamp": 1634609435199
   }
   
   # id 存在会报错，data stream 如果需要指定ID， 只能用这种方式
   POST http://localhost:9200/kyle_test/_create/id111
   {
   	"name": "kyle",
   	"age": 10,
   	"@timestamp": 1634609435199
   }
   ```

## 批量新增

1. 建议的方式

   ```
   POST http://localhost:9200/kyle_test/_bulk
   {"create":{}}
   {"name":"name1"}
   {"create":{"_id":"1"}}  // 指定 id, 如果id存在 会报错
   {"name":"name2"}
   ```

2. index

   ```
   POST http://localhost:9200/kyle_test/_bulk
   {"index":{}}
   {"name":"name1"}
   {"index":{"_id":"1"}}  // 指定 id，id存在，直接覆盖
   {"name":"name2"}
   ```

## 单条更新

1. 更新部分字段

   ```
   POST kyletest/_update/id111
   {
     "doc": {
       "name": 2
     }
   }
   ```

2. 覆盖

   ```
   POST http://localhost:9200/kyle_test/_doc/id111
   {
   	"name": "kyle",
   	"age": 10,
   	"@timestamp": 1634609435199
   }
   ```

## 查询

```
POST http://localhost:9200/kyle_test/_search
 {
   "query": {
     "match_all": {}
   }
 }
```



