---
title: "Springcloudalibaba"
date: 2023-10-21T13:09:48+08:00
lastmod: 2023-10-21T13:09:48+08:00
draft: true
keywords: ["spring cloud","alibaba"]
description: "springcloud alibaba"
tags: ["springboot"]
categories: ["springboot", "springcloud"]
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

Spring Cloud Alibaba

为分布式应用开发提供一站式解决方案。

<!--more-->

 # Spring Cloud Alibaba 

- 流量控制和业务降级, flow control and service degradation   组件 [Alibaba Sentinel](https://github.com/alibaba/Sentinel/)
- 服务注册与发现， 支持 Ribbon， 通过 Spring Cloud Netflix 客户端负载均衡 Nacos
- 配置中心  分布式配置， [Nacos ](https://nacos.io/zh-cn/docs/quick-start.html)
- 事件驱动 Spring Cloud Stream RocketMQ Binder 连接的高度可扩展的事件驱动微服务
- 消息总线 Spring Cloud Bus RocketMQ
- 分布式事务  Seata 
- Dubbo RPC 通过 [Apache Dubbo RPC](https://dubbo.apache.org/en/) 扩展 Spring 云服务到服务调用的通信协议