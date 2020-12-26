---
title: "Elasticsearch Basic Security"
date: 2020-12-26T09:13:21+08:00
lastmod: 2020-12-26T09:13:21+08:00
draft: true
keywords: ["elasticsearch","security"]
description: "elasticsearch security"
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

Elasticsearch Basic Security Module

<!--more-->

# Elasticsearch Basic Security

[不同版本支持的功能](https://www.elastic.co/cn/subscriptions)

> 免费的功能

- 安全设置
- 加密通信
- 基于角色的访问控制
- 文件和原生身份验证
- Kibana Spaces
- Kibana 功能控制
- API 密钥管理

> 收费的功能

- 审计日志
- IP 筛选
- LDAP、PKI 和活动目录身份验证
- Elasticsearch 令牌服务
- 单点登录 (SAML、OpenID Connect 和 Kerberos)
- 基于属性的访问控制
- 字段和文档级别安全性
- 自定义身份验证和授权 Realm
- 数据静态加密支持
- FIPS 140-2 模式

## 配置安全性

1. 设置 xpack.security.enabled: true 默认false

   启用Elasticsearch安全功能时，默认情况下启用基本身份验证。要与集群通信，必须指定用户名和密码。除非您[启用匿名访问](https://www.elastic.co/guide/en/elasticsearch/reference/current/anonymous-access.html)，否则所有不包含用户名和密码的请求都将被拒绝

