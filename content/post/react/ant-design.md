---
title: "Ant Design"
date: 2020-12-12T19:35:35+08:00
lastmod: 2020-12-12T19:35:35+08:00
draft: true
keywords: ["ant-design","react"]
description: "ant-design study"
tags: ["react"]
categories: ["react","ant-design"]
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

ant-design study

<!--more-->

# Ant Design

适用ant design 进行项目的创建

### 准备

本地需要安装 `yarn`,`node`,`git`,技术栈 [ES2015+](http://es6.ruanyifeng.com/), [React](http://facebook.github.io/react/), [UmiJS](https://umijs.org/), [dva](https://github.com/dvajs/dva),[g2](https://g2.antv.vision/zh)和 [antd](https://ant.design/docs/react/introduce-cn)。

### 安装

新建空文件夹，执行

```sh
yarn create umi
```

or

```sh
npm create umi
```

选择`ant-design-pro`

```sh
 Select the boilerplate type (Use arrow keys)
❯ ant-design-pro  - Create project with an layout-only ant-design-pro boilerplate, use together with umi block.
  app             - Create project with a simple boilerplate, support typescript.
  block           - Create a umi block.
  library         - Create a library with umi.
  plugin          - Create a umi plugin.
```

### 目录结构

```sh
├── config                   # umi 配置，包含路由，构建等配置
├── mock                     # 本地模拟数据
├── public
│   └── favicon.png          # Favicon
├── src
│   ├── assets               # 本地静态资源
│   ├── components           # 业务通用组件
│   ├── e2e                  # 集成测试用例
│   ├── layouts              # 通用布局
│   ├── models               # 全局 dva model
│   ├── pages                # 业务页面入口和常用模板
│   ├── services             # 后台接口服务
│   ├── utils                # 工具库
│   ├── locales              # 国际化资源
│   ├── global.less          # 全局样式
│   └── global.ts            # 全局 JS
├── tests                    # 测试工具
├── README.md
└── package.json
```

### 本地开发

安装依赖

```sh
npm install
npm start
```

打开浏览器访问 [http://localhost:8000](http://localhost:8000)

### 部署

[https://pro.ant.design/docs/deploy-cn](https://pro.ant.design/docs/deploy-cn)

## [UmiJS](https://umijs.org/zh-CN/docs)

Umi以路由为基础，同时支持配置式路由和约定式路由，保证路由的功能完备，并以此进行功能扩展。