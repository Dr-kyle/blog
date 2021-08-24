---
title: "AppSearch"
date: 2021-08-24T16:23:34+08:00
lastmod: 2021-08-24T16:23:34+08:00
draft: true
keywords: []
description: ""
tags: []
categories: []
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

App Search instruction

<!--more-->

# Elastic App Search

Elastic App Search 是  Elastic Enterprise Search 的一部分，和 Elasticsearch 一起使用，是免费使用的。

![](https://play.vidyard.com/zKjiM8zVMi13fnecZHAFhy.jpg)

App Search 可以轻松实现高级搜索，通过完善的 API 集合以及直观的仪表板，Elastic App Search 为您的公司网站、电子商务网站或应用带来了 Elasticsearch 的强大功能。获得无缝的可扩展性、可调的相关性控制、详尽的文档、经过良好维护的客户端和强大的分析能力，为您的客户打造领先的搜索体验。

## 亮点

- 轻松完成索引

  不限平台，不限数据，不限位置，通过不区分模式的数据采集方式以及直观的仪表板，您马上就能完成准备工作并开始查询。只需使用网络爬虫或经过良好维护的 API 客户端之一对您的数据加以索引，然后便可开始查询啦。

- 功能强大的搜索

  相关性高，容错性优异，Elastic App Search 由 Elasticsearch 提供支持，其相关性模型已针对实际搜索情况进行优化。“允许拼写错误”、词干提取、双连词等功能均可直接使用。

- 搜索分析

  实时数据，可付诸实践的见解，Elastic App Search 的分析结果能够让您洞悉用户的全部行为，并为您提供改进建议以及达到目标的方式。

- 相关性调整

  定义您自己的搜索方式，使用直观界面调校搜索的相关性。创建同义词组，针对特定查询对结果进行重排序，以及分配权重和系数，这些都有助于对整体精准度进行微调。

- 爬取过程便捷

  高效索引所有公开的网站内容。只需点击鼠标——无需代码——用户便可自定义网络爬虫规则，从而指定入口点，并且/或者将特定页面、内容或词汇排除在外。

## 部署

因为 App Search 是 Enterprise Search 的一部分，所有只需要部署 Enterprise Search 即可使用 App Search。

以 Docker 部署举例

1. 拉取镜像

   ```sh
   docker pull docker.elastic.co/enterprise-search/enterprise-search:7.12.0
   ```

2. 用 Docker Cli 启动

   ```sh
   docker run \
     -p 3002:3002 \
     -e elasticsearch.host='http://host.docker.internal:9200' \
     -e elasticsearch.username=elastic \
     -e elasticsearch.password=changeme \
     -e allow_es_settings_modification=true \
     -e secret_management.encryption_keys='[4a2cd3f81d39bf28738c10db0ca782095ffac07279561809eecc722e0c20eb09]' \
   docker.elastic.co/enterprise-search/enterprise-search:7.12.0
   ```

   `allow_es_settings_modification` 和 `secret_management.encryption_keys` 必须配置。

3. 用 Docker Compose 启动

   ```sh
   docker compose up --remove-orphans
   ```

   `docker-compose.yml` 文件内容如下:

   ```sh
   version: "7.12.0"
   
   networks:
     elastic:
       driver: bridge
   
   volumes:
     elasticsearch:
       driver: local
   
   services:
     elasticsearch:
       image: docker.elastic.co/elasticsearch/elasticsearch:{version}
       restart: unless-stopped
       environment:
         - "discovery.type=single-node"
         - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
         - "xpack.security.enabled=true"
         - "xpack.security.authc.api_key.enabled=true"
         - "ELASTIC_PASSWORD=changeme"
       ulimits:
         memlock:
           soft: -1
           hard: -1
       volumes:
         - elasticsearch:/usr/share/elasticsearch/data
       ports:
         - 127.0.0.1:9200:9200
       networks:
         - elastic
   
     ent-search:
       image: docker.elastic.co/enterprise-search/enterprise-search:{version}
       restart: unless-stopped
       depends_on:
         - "elasticsearch"
       environment:
         - "JAVA_OPTS=-Xms512m -Xmx512m"
         - "ENT_SEARCH_DEFAULT_PASSWORD=changeme"
         - "elasticsearch.username=elastic"
         - "elasticsearch.password=changeme"
         - "elasticsearch.host=http://elasticsearch:9200"
         - "allow_es_settings_modification=true"
         - "secret_management.encryption_keys=[4a2cd3f81d39bf28738c10db0ca782095ffac07279561809eecc722e0c20eb09]"
         - "elasticsearch.startup_retry.interval=15"
       ports:
         - 127.0.0.1:3002:3002
       networks:
         - elastic
   
     kibana:
       image: docker.elastic.co/kibana/kibana:{version}
       restart: unless-stopped
       depends_on:
         - "elasticsearch"
         - "ent-search"
       ports:
         - 127.0.0.1:5601:5601
       environment:
         ELASTICSEARCH_HOSTS: http://elasticsearch:9200
         ENTERPRISESEARCH_HOST: http://ent-search:3002
         ELASTICSEARCH_USERNAME: elastic
         ELASTICSEARCH_PASSWORD: changeme
       networks:
         - elastic
   ```

4. 启动成功后，可以通过 kibana -> Enterprise Search -> App Search 访问，或者访问单独的 UI ， [http://localhost:3002](http://localhost:3002/)

   

## 使用

1. 创建 Engine

   每个 Engine 在 Elasticsearch 中都会有对应的一个 index

   

2. 导入数据

   数据导入方式有以下四种

   - Paste JSON

     直接粘贴 JSON 内容

   - Upload a JSON file

     上传 json 文件，将 文件内容写入

   - Use the web crawler(BETA)

     使用爬虫器进行爬取

   - Index by API

     使用 Rest API 写入数据

   我们根据场景选择不同的数据导入方式，以下以 Rest API 举例

   




