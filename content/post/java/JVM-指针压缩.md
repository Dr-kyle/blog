---
title: "JVM 指针压缩"
date: 2021-05-22T08:46:44+08:00
lastmod: 2021-05-22T08:46:44+08:00
draft: true
keywords: ["JVM", "jvm", "指针压缩"]
description: "jvm 指针压缩"
tags: ["java"]
categories: ["java"]
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

jvm 指针压缩

<!--more-->

# JVM 指针压缩 (CompressedOops)

30 位的机器，进程能使用的最大内存是 4G（2^32）， 如果进程需要使用更多的内存，需要使用 64 位机器。

java 进程，在oop(普通对象指针，是 JVM 中用于代表引用对象的句柄),只有32位时，只能引用 4G 内存，如果需要使用更大的堆内存，需要部署 64 位 JVM

在堆中，32位的对象引用占用 4 字节， 64 位的对象引用占 8 字节

**64 位 JVM 在支持更大堆的同时，由于对象引用变大带来新的性能问题**

1. 增加 GC 开销

   64 位  对象引用占用更多的堆空间，留给其他数据的空间就会减少，加快 GC 发生

2. 降低 CPU 缓存命中率

   64 位的对象引用增大，CPU 能缓存的 oop 将会更少，降低 CPU 缓存的效率。

为了能够保留 32 位 的性能， oop 必须保留 32 位。

## 压缩指针 (CompressedOops)