---
title: "Logstash Deploy"
date: 2021-11-03T14:27:10+08:00
lastmod: 2021-11-03T14:27:10+08:00
draft: true
keywords: ["logstash"]
description: "logstash deploy"
tags: ["logstash"]
categories: ["logstash"]
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

Logstash start 7.15.1

<!--more-->

```sh
docker pull docker.elastic.co/logstash/logstash:7.15.1

docker run --rm -it -v ~/pipeline/:/usr/share/logstash/pipeline/ docker.elastic.co/logstash/logstash:7.15.1
```

# 配置目录

`/usr/share/logstash/config`

- jvm.options

- log4j2.properties

- logstash-sample.conf

  ```
  # Sample Logstash configuration for creating a simple
  # Beats -> Logstash -> Elasticsearch pipeline.
  
  input {
    beats {
      port => 5044
    }
  }
  
  output {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
      #user => "elastic"
      #password => "changeme"
    }
  }
  ```

- logstash.yml

  ```yaml
  http.host: "0.0.0.0"
  xpack.monitoring.elasticsearch.hosts: [ "http://elasticsearch:9200" ]
  ```

- pipelines.yml

  ```yaml
  # This file is where you define your pipelines. You can define multiple.
  # For more information on multiple pipelines, see the documentation:
  #   https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html
  
  - pipeline.id: main
    path.config: "/usr/share/logstash/pipeline"
  ```

- startup.options

`/usr/share/logstash/pipeline`

- logstash.conf

  ```
  input {
    beats {
      port => 5044
    }
  }
  
  output {
    stdout {
      codec => rubydebug
    }
  }
  ```

  

# Docker 配置方法

## 1. mounted

```sh
docker run --rm -it -v ~/settings/:/usr/share/logstash/config/ docker.elastic.co/logstash/logstash:7.15.1

docker run --rm -it -v ~/settings/logstash.yml:/usr/share/logstash/config/logstash.yml docker.elastic.co/logstash/logstash:7.15.1
```

## 2. custome images

```sh
FROM docker.elastic.co/logstash/logstash:7.15.1
RUN rm -f /usr/share/logstash/pipeline/logstash.conf
ADD pipeline/ /usr/share/logstash/pipeline/
ADD config/ /usr/share/logstash/config/
```

## 3. Environment variable

为了兼容容器编排系统，环境变量全部大写，下划线作为单词分隔符

[logstash.yml](https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html)

| **Environment Variable**                                     | **Logstash Setting**     |
| ------------------------------------------------------------ | ------------------------ |
| PIPELINE_WORKERS                                             | pipeline.workers         |
| LOG_LEVEL                                                    | log.level                |
| MONITORING_ENABLED                                           | monitoring.enabled       |
| [XPACK_MONITORING_ENABLED](https://www.elastic.co/guide/en/logstash/7.15/monitoring-internal-collection-legacy.html) | xpack.monitoring.enabled |

使用 docker 时 `logstash.yml` 的默认值

| Variable                         | Default                     |
| -------------------------------- | --------------------------- |
| `monitoring.elasticsearch.hosts` | `http://elasticsearch:9200` |
| `http.host`                      | `0.0.0.0`                   |