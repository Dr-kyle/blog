---
title: "Elasticsearch配置文件"
date: 2021-10-30T21:14:04+08:00
lastmod: 2021-10-30T21:14:04+08:00
draft: false
keywords: ["elasticsearch"]
description: "elasticsearch config"
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

elasticsearch 配置

<!--more-->

# important system configuration

`firewall`

根据需要设置防火墙，此示例直接关闭

```
# 停止
sudo systemctl stop firewalld.service
# 禁止开机启动
sudo systemctl disable firewalld.service
```

`ulimit`

```
sudo ulimit -n 65535
```

`/etc/security/limits.conf`

```
*  -  nofile  65535
* soft memlock unlimited
* hard memlock unlimited
* hard nproc unlimited
* soft nproc unlimited
```

`Disable all swap files`

```
sudo swapoff -a
# sudo vim /etc/fstab  注释掉包含 swap 的行
```

`swappiness`

```
sudo sysctl -w vm.swappiness=1
sudo sysctl -w vm.max_map_count=262144
sudo vim /etc/sysctl.conf
vm.max_map_count=262144
vm.swappiness=1
```



```
sudo vim /etc/systemd/system.conf
DefaultLimitNOFILE=65536
DefaultLimitNPROC=infinity
DefaultLimitMEMLOCK=infinity
```

重启机器。

# elasticsearch.yml

```yaml
# 可以动态调整的配置，推荐使用 _cluster/settings 进行设置
bootstrap.memory_lock: true
# PATH
path:
  # 数据目录，不要在目录上运行病毒扫描程序
  data: /var/data/elasticsearch
  # 日志目录
  logs: /var/log/elasticsearch
# Cluster name
cluster.name: logging-prod
# 环境变量
# node.name: ${HOSTNAME}
# defaults to the hostname of the machine 
node.name: prod-data-2
# default 127.0.0.1 and [::1], 生产模式，启动时警告变成error
network.host: 192.168.1.10


# Discovery, 没有配置将扫描本地端口9300 - 9305
discovery.seed_hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 
   - seeds.mydomain.com 
   - [0:0:0:0:0:ffff:c0a8:10c]:9301
# 在第一次成功形成集群时，删除掉该配置，重新启动集群或向现有集群添加新节点时，请不要使用此设置
cluster.initial_master_nodes: 
   - master-node-a
   - master-node-b
   - master-node-c

# Path circuit breaker avoid OutOfMemoryError
# static, default: true, 确定父父断路器是否考虑实际内存使用情况，还是只考虑子断路器
indices.breaker.total.use_real_memory: true
# Dynamic, if indices.breaker.total.use_real_memory: true default: JVM heap 95%, else 70%
indices.breaker.total.limit: 95%

# Field data circuit breaker, 预估将字段加载到 field data cache 所需的堆内存大小，如果超过限制停止操作，并返回错误
# Dynamic, default: JVM heap 40%
indices.breaker.fielddata.limit: 40%
# Dynamic, 预估大小和该因子相乘确定最终大小
indices.breaker.fielddata.overhead: 1.03

# Request, for example, memory used for calculating aggregations during a request
# Dynamic, default: JVM heap 60%
indices.breaker.request.limit: 60%
indices.breaker.request.overhead: 1

# In flight requests circuit breaker
# Dynamic, defalult: JVM heap 100% ,意味会受到 path circuit breaker 的限制
network.breaker.inflight_requests.limit: 100%
network.breaker.inflight_requests.overhead: 2

# Accounting requests circuit breaker
# Dynamic
indices.breaker.accounting.limit: 100%
indices.breaker.accounting.overhead: 1

# Script compilation circuit breaker
# Dynamic, 脚本编译断路器限制了一段时间内内联脚本编译的次数, 默认 75/5m, 每 5 分钟 75次。
script.context.$CONTEXT.max_compilations_rate: 75/5m

# Regex circuit breaker， Static， 写得不好的正则表达式会降低集群的稳定性和性能
# limited 启动正则表达式，使用 script.painless.regex.limit-factor 集群设置限制复杂性。 
# true 启用没有复杂性限制的正则表达式。禁用正则表达式断路器。
# false 禁用正则表达式。任何包含正则表达式的 Painless 脚本都会返回错误。
script.painless.regex.enabled: limited
# 限制 Painless 脚本中的正则表达式可以考虑的字符数。 Elasticsearch 通过将设置值乘以脚本输入的字符长度来计算此限制, 例如 input `foobarbaz` 长度为9， 9 * 6 = 54， 如果表达式超过此限制，断路器返回错误。
script.painless.regex.limit-factor: 6

```

https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html

