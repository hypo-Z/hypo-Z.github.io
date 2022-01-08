---
title: 记录一个Dockerfile书写笔记
author: hypo
top: false
cover: false
toc: true
mathjax: false
date: 2021-12-23 16:38:10
img:
coverImg:
password:
summary: 使用`新的`镜像可以节省大量空间，因为我们实际上不需要`Go`工具或其他任何东西来运行我们的编译好的程序，这可能也是`Go`在容器时代的一个优势吧。
categories: 笔记
tags:
- 笔记
- Docker
- Dockerfile
- Golang
---
## docker 制作镜像

在`Dockerfile`所在的目录下执行`docker build`构建镜像

```
➜  docker build -t 镜像名 .
```

`docker`会依据`Dockerfile`里的指令构建镜像，整个构建的过程类似下面：

### 验证镜像

这一步其实可以省略，不过为了确保制作的镜像是没有问题，我们通过`docker run`命令用这个镜像运行容器验证一下。

```
➜ docker run -d -p 本机端口:容器端口 --rm --name container名 镜像名
```

在这里，我们指示`docker`从源镜像``运行容器，将主机端口``绑定到容器的内部端口``，以后台模式（-d）运行容器，给此容器命名为`容器名`，并在容器结束运行后自动删除容器（--rm）。



### 多阶段构建

下面就来介绍一下怎么使用`Docker`的多阶段构建制作`Go`应用的镜像。

之前文章里镜像的`Dockerfile`长这样：

```
FROM golang:1.14-alpine
RUN mkdir /app
COPY . /app
WORKDIR /app
RUN go build -o main . 
CMD ["/app/main"]
复制代码
```

我们使用用多阶段构建的方式构建镜像后，`Dockerfile`会变成类似下面这样：

```
FROM golang:alpine AS build

RUN mkdir /app
COPY . /app
WORKDIR /app
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp

### 
FROM scratch as final
COPY --from=build /app/myapp .
CMD ["/myapp"]
复制代码
```


## 记录一个有sqlite3数据库和golang的Dockerfile书写笔记

### 以ubuntu为基础镜像

```dockerfile
FROM ubuntu:latest As build
# 配置环境变量
ENV GOROOT=/usr/lib/go
ENV PATH=$PATH:/usr/lib/go/bin
ENV GOPATH=/root/go
ENV PATH=$GOPATH/bin/:$PATH
ENV DEBIAN_FRONTEND=noninteractive
# 执行命令
RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update\
    && apt-get install -y $buildDeps \
    && wget https://go.dev/dl/go1.17.5.linux-amd64.tar.gz\
    && tar -xzvf go1.17.5.linux-amd64.tar.gz -C /usr/lib\
    && export GOROOT=/usr/lib/go \
    && ln -s /usr/local/go/bin/* /usr/bin/ \
    && export PATH=$PATH:/usr/lib/go/bin \
    && export GOPATH=/root/go\
    && export PATH=$GOPATH/bin/:$PATH\
    && rm go1.17.5.linux-amd64.tar.gz\
    && wget https://www.sqlite.org/2021/sqlite-autoconf-3370000.tar.gz\
    && tar -xzvf sqlite-autoconf-3370000.tar.gz -C /usr/lib\
    && rm sqlite-autoconf-3370000.tar.gz\
    && cd /usr/lib/sqlite-autoconf-3370000\
    && ./configure --prefix=/usr/local \
    && make \
    && make install \
    && cd .. \
    && mkdir /auth
COPY . /auth/
WORKDIR /auth
EXPOSE 8000
#安装库依赖项
RUN go mod tidy\
    && go build -o server\
    && apt-get purge -y --auto-remove $buildDeps

####
FROM ubuntu:latest as final
#镜像构建参数
COPY --from=build /auth/server .
CMD ["/server"]
```

`Go`项目应用的`Dockerfile`通常大概类似这样，但是每个项目的细节可能有所不同。`FROM ubuntu:latest`指定了开始阶段的基础映像（其中包含ubuntu的操作系统，无工具和库，用于构建程序），`AS build`是给这个阶段取名为`build`。

ubuntu是专门为容器设计的小型`Linux`发行版。这个`Dockerfile`中使用了两次`FROM`指令，第二条`FROM ubuntu`行，它告诉`Docker`从一个全新的，完全空的容器镜像重新开始，然后将上个阶段编译好的程序复制到其中。这个才是我们随后将用于运行的`Go`应用程序的容器镜像。

`Docker`用于`Go`程序的多阶段构建很常见，使用`新的`镜像可以节省大量空间，因为我们实际上不需要`Go`工具或其他任何东西来运行我们的编译好的程序，这可能也是`Go`在容器时代的一个优势吧。