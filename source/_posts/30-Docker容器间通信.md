---
title: Docker容器间通信
author: hypo
top: false
cover: false
toc: true
mathjax: false
date: 2022-01-08 17:56:32
img:
coverImg:
password:
summary: Bridge网桥双向通信
categories: 笔记
tags:
- Docker
---
# Docker容器间通信

Bridge网桥双向通信

**浏览器<==>物理网卡<=网桥=>Docker环境下的容器**


将指定的容器都绑定到同一个网桥上，这些容器就会天然的互联互通

```
docker run -d --name server app
docker run -d --name database mysql
docker network create -d bridge my-bridge #创建一个新的网桥
[root@VM-0-2-centos ~]# docker network ls #列出docker服务网络底层明细
NETWORK ID          NAME                DRIVER              SCOPE
0dfca8776e29        bridge              bridge              local
dec8ce58f992        host                host                local
522173622fe8        my-bridge           bridge              local
36e1a4828e03        none                null                local
```

## 创建新的网桥

```
docker network create my-bridge
```



## 容器和网桥绑定

```
docker network connect my-bridge web
docker network connect my-bridge database
```

## 网桥实现原理

```
外网---->物理网卡（192.168.0.117）----->docker里的虚拟网卡（172.17.0.1）|----->redis(172.17.0.2:6375)  |---->server(172.17.0.3:8080)  
```



![](https://hypo-pictrue-1308430808.cos.ap-shanghai.myqcloud.com/hypo.ltd-%E6%96%87%E4%BB%B6%E8%AE%BF%E9%97%AE%E5%AD%98%E5%82%A8/%E7%BD%91%E6%A1%A5.png)



## 容器和网桥解除绑定

```
docker network disconnect my-bridge server
docker network disconnect my-bridge database
```

