---
title: "Win10 Ubuntu Docker"
date: 2020-04-13T17:52:14+08:00
lastmod: 2020-04-13T17:52:14+08:00
draft: false
keywords: ["windows10 Ubuntu Docker"]
description: ""
tags: ["Docker"]
categories: ["software"]
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."hugo 
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

Windows10 子系统Ubuntu安装Docker详细步骤

<!--more-->

# Windows 10 子系统安装docker，并解决Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running? 问题。

1. 搜索框中搜索开发者设置  ![image.png](https://upload-images.jianshu.io/upload_images/10995160-6e2322a2cf88604c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

2. windows 设置打开开发者选项  ![image.png](https://upload-images.jianshu.io/upload_images/10995160-772ecc0a2cc3e2e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

3. 搜索框中搜索windows功能  ![image.png](https://upload-images.jianshu.io/upload_images/10995160-e62555308135f5c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

4. 勾选使用于Linux的Windwos子系统  ![image.png](https://upload-images.jianshu.io/upload_images/10995160-16c3ace05b766473.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

5. 安装 Docker Desktop for Windows

   下载地址：[https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

   安装成功后打开 Expose daemon on tcp://localhost:2375 without TLS
     ![image.png](https://upload-images.jianshu.io/upload_images/10995160-11f62d551e446aa6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


6. 在Microsoft Store中搜索Ubuntu，然后安装就行  ![image.png](https://upload-images.jianshu.io/upload_images/10995160-5f95f839f5062cca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

7. Ubuntu安装成功后打开(第一次打开需要设置用户名和密码)  ![image.png](https://upload-images.jianshu.io/upload_images/10995160-a193f5abac4557aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

8. 然后依次执行以下命令：

   Docker 官网安装步骤: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/) ，不想看官网的直接按照以下命令执行即可

```
    $ sudo apt-get update
    $ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    $ sudo apt-key fingerprint 0EBFCD88
    $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    $ sudo apt-get update
    $ sudo apt-get install docker-ce
    $ docker -H localhost:2375 images
    $ export DOCKER_HOST=localhost:2375
    $ echo "export DOCKER_HOST=localhost:2375" >> ~/.bashrc</pre>
```

到此docker就安装就成功了，可以愉快的玩耍了。
