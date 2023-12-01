---
title: "Flink Study"
date: 2023-11-24T13:21:10+08:00
lastmod: 2023-11-24T13:21:10+08:00
draft: true
keywords: [“flink"]
description: "flink learn"
tags: [”flink“]
categories: [”flink“]
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

Flink 流批一体，能够同时支持批处理任务和流处理任务，flink 版本为 1.18.0

<!--more-->

# Flink

## 运行模式

### Standalone

Standalone 运行模式不依赖任何外部的资源管理平台，如果资源不足，或者出现故障，没有自动扩展或重分配资源的能力，必须手动处理。

### Yarn 运行模式

### K8S 运行模式

## 部署模式

![](/images/flink/deployment_modes.svg)

### 会话模式 Session Mode

多个job 运行到同一个集群，共享资源

### Per-Job 

Per-job mode is only supported by YARN and has been deprecated in Flink 1.15.

每一个 job 启动一个集群，需要借助资源管理平台，yarn 

### 应用模式

应用模式不会提前创建集群，不能调用 start-cluster.sh 脚本，可以使用 standalone-job.sh 来创建一个 JobManager。

## 高可用

### [Zookeeper](https://nightlies.apache.org/flink/flink-docs-release-1.18/docs/deployment/ha/zookeeper_ha/)

```
// 高可用的类型  required
high-availability.type: zookeeper
// required JobManager metadata 存储的位置，zookeeper 中只保存指向此状态的指针
high-availability.storageDir: hdfs:///flink/recovery
// required zookeeper 的地址
high-availability.zookeeper.quorum: address1:2181[,...],addressX:2181
// recommended
high-availability.zookeeper.path.root: /flink
// recommended   The cluster-id ZooKeeper node
high-availability.cluster-id: /default_ns # important: customize per cluster
```



## Docker 部署



```
jobmanager.rpc.address: localhost
jobmanager.rpc.port: 6123
jobmanager.bind-host: 0.0.0.0
jobmanager.memory.process.size: 1600m
taskmanager.bind-host: 0.0.0.0
taskmanager.host: localhost
taskmanager.memory.process.size: 1728m
# 仅用来隔离内存，建议设置为 CPU 核心数
taskmanager.numberOfTaskSlots: 4
parallelism.default: 1
jobmanager.execution.failover-strategy: region
rest.port: 8081
rest.address: localhost
rest.bind-address: 0.0.0.0

```





```
jobmanager
sudo docker run -d --name jobmanager -v /opt/app/flink-docker/jobmanager:/opt/flink/conf --network host -e JOB_MANAGER_RPC_ADDRESS="hadoop01" a.newegg.org/newegg-docker/flink:1.18.0-scala_2.12 jobmanager
Taskmanager
sudo docker run -d --name taskmanager -v /opt/app/flink-docker/taskmanager:/opt/flink/conf --network host -e JOB_MANAGER_RPC_ADDRESS="hadoop01" a.newegg.org/newegg-docker/flink:1.18.0-scala_2.12 taskmanager


 -e FLINK_PROPERTIES="jobmanager.rpc.address: hadoop01"
JOB_MANAGER_RPC_ADDRESS
TASK_MANAGER_NUMBER_OF_TASK_SLOTS

```





## DataStream API

- 获取执行环境 Environment
- 读取数据源 Source
- 转换操作 Transform
- 输出 Sink
- 触发执行



## 提交参数

> 指定执行模式

- -Dexecution.runtime-mode=BATCH
- env.setRuntimeMode(RuntimeExecutionMode.BATCH)

## Maven

```
		<dependency>
			<groupId>org.apache.flink</groupId>
			<artifactId>flink-streaming-java</artifactId>
			<version>${flink.version}</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.flink</groupId>
			<artifactId>flink-clients</artifactId>
			<version>${flink.version}</version>
			<scope>provided</scope>
		</dependency>
		
```

