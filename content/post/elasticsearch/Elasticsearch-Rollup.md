---
title: "Elasticsearch Rollup"
date: 2021-03-14T15:43:10+08:00
lastmod: 2021-03-14T15:43:10+08:00
draft: true
keywords: ["elasticsearch Rollup", "Rollup"]
description: "elasticsearch Rollup"
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

Elasticsearch Rollup 

<!--more-->

# [Elasticsearch Rollup](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/data-rollup-transform.html)

elasticsearch 提供了两个操作数据的方法，rolling up 和 transforming。

## [Rolling up your historical data](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/xpack-rollup.html)

通常我们保留历史数据，会考虑成本问题，保留的数据越多需要花费的代价也就越大，Elastic Stack提供了数据汇总历史数据的功能，这样我们就可以降低存储成本。

适用场景：

1. 假设系统每天会生成4300万份文档，当前的数据对实时分析可能很有用，但是十年前的数据可能就需要以较大的间隔(例如每天或每月或每年)进行分析。如果将过去的数据按较大的时间间隔进行汇总，就可以节省大量的空间，并且汇总历史数据过程是自动化的。

2. 系统可能保留一个月的原始数据，一个月后，将历史数据进行汇总，并删除原始数据。

## [Transforming your data](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/xpack-rollup.html)

