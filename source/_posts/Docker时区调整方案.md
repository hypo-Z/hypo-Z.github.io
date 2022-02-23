---
title: Docker时区调整方案
author: hypo
top: false
cover: false
toc: true
mathjax: false
date: 2022-02-23 12:13:52
img: medias/featureimages/50.png
coverImg:
password:
summary: 对于经常使用Docker的人来说，可能会碰到一个问题：时区问题。
categories: 笔记
tags:
- Docker
---
# Docker 时区调整方案

对于经常使用 Docker 的人来说，可能会碰到一个问题：时区问题。

大部分 Docker 镜像都是基于 Alpine，Ubuntu，Debian，CentOS 等基础镜像制作而成。

基本上都采用 UTC 时间，默认时区为零时区。

```javascript
docker run --name test --rm -ti alpine /bin/sh
/ # date
Fri Nov 29 08:14:49 UTC 2019
```

而我们主要用的是 CST 时间，北京时间，位于东八区。时区代号： Asia/Shanghai

```javascript
docker run --name test --rm -ti -v /etc/timezone:/etc/timezone:ro -v /etc/localtime:/etc/localtime:ro alpine /bin/sh
/ # date
Fri Nov 29 16:13:55 CST 201
```

对比一下，我们会发现，时间上相差 8 小时。

经过一系列探索实践，我们总结了一些 Docker 时区调整方案。

## 一、运行 Docker 容器时调整时区

在 Linux 系统中，控制时区和时间的主要是两个地方：

- `/etc/timezone` 主要代表当前时区设置，一般链接指向`/usr/share/zoneinfo`目录下的具体时区。
- `/etc/localtime` 主要代表当前时区设置下的本地时间。

### 1. 通用 docker 时区修改方案

#### [宿主机](https://cloud.tencent.com/product/cdh?from=10680)为 Linux 系统

当宿主机为 Linux 系统时，我们可以直接将宿主机上的`/etc/timezone`和`/etc/localtime`挂载到容器中，这样可以保持容器和宿主机时区和时间一致。

```javascript
 -v /etc/timezone:/etc/timezone:ro -v /etc/localtime:/etc/localtime:ro
```

使用示例如下：

```javascript
docker run --name test --rm -ti -v /etc/timezone:/etc/timezone:ro -v /etc/localtime:/etc/localtime:ro alpine /bin/sh
/ # date
Fri Nov 29 16:13:55 CST 2019
```

### 2. 通过传递环境变量改变容器时区

- 适用于基于 Debian 基础镜像, CentOS 基础镜像 制作的 Docker 镜像
- 不适用于基于 Alpine 基础镜像, Ubuntu 基础镜像 制作的 Docker 镜像

对于基于 Debian 基础镜像，CentOS 基础镜像制作的 Docker 镜像，在运行 Docker 容器时，传递环境变量`-e TZ=Asia/Shanghai`进去，能修改 docker 容器时区。

```javascript
-e TZ=Asia/Shanghai
```

使用示例如下：

```javascript
docker run --name test -e TZ=Asia/Shanghai --rm -ti debian /bin/bash
/# date
Fri Nov 29 18:46:18 CST 2019
```

## 二、制作 Docker 镜像时调整时区

通过编写 Dockerfile,构建自己的 Docker 镜像，可以永久解决时区问题。

### 1. Alpine

根据[《Setting the timezone》](https://wiki.alpinelinux.org/wiki/Setting_the_timezone)提示，我们可以将以下代码添加到 Dockerfile 中：

```dockerfile
ENV TZ Asia/Shanghai

RUN apk add tzdata && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && apk del tzdata
```

### 2. Debian

Debian 基础镜像 中已经安装了 tzdata 包，我们可以将以下代码添加到 Dockerfile 中：

```dockerfile
ENV TZ=Asia/Shanghai \
    DEBIAN_FRONTEND=noninteractive

RUN ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && dpkg-reconfigure --frontend noninteractive tzdata \
    && rm -rf /var/lib/apt/lists/*
```

### 3. Ubuntu

Ubuntu 基础镜像中没有安装了 tzdata 包，因此我们需要先安装 tzdata 包。

我们可以将以下代码添加到 Dockerfile 中。

```dockerfile
ENV TZ=Asia/Shanghai \
    DEBIAN_FRONTEND=noninteractive

RUN apt update \
    && apt install -y tzdata \
    && ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && dpkg-reconfigure --frontend noninteractive tzdata \
    && rm -rf /var/lib/apt/lists/*
```

### 4. CentOS

CentOS 基础镜像 中已经安装了 tzdata 包，我们可以将以下代码添加到 Dockerfile 中。

```dockerfile
ENV TZ Asia/Shanghai

RUN ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone
```

## 总结

时区问题是大问题。

时间没统一好，业务会乱套。

希望通过上面的内容，能够帮助大家解决好 Docker 方面的时区问题。