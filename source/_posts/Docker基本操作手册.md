---
title: Docker基本操作手册
author: hypo
top: false
cover: false
toc: true
mathjax: false
date: 2022-02-17 16:12:19
img: medias/featureimages/46.jpeg
coverImg:
password:
summary: docker镜像和容器常用命令
categories: 笔记
tags:
- Docker
---
# docker镜像常用命令

attach   将本地标准输入、输出和错误流附加到正在运行的容器中

build    从Dockerfile构建一个映像

commit   从容器的更改中创建一个新映像

cp     在容器和本地文件系统之间复制文件/文件夹

create   创建一个新容器

diff    检查容器文件系统上文件或目录的更改

events   从服务器获取实时事件

exec    在正在运行的容器中运行命令

export   将容器的文件系统导出为tar存档文件

history   显示图像的历史

images   图片列表

import   从tarball导入内容以创建文件系统映像

info    显示整个系统的信息

inspect   返回Docker对象的底层信息

kill    杀死一个或多个正在运行的容器

load    从tar存档或STDIN加载图像

login    登录到Docker注册表

logout   从Docker注册表注销

logs    获取容器的日志

pause    暂停一个或多个容器中的所有进程

port    列出容器的端口映射或特定映射

ps     列表容器

pull    从注册表中提取映像或存储库

push    将映像或存储库推入注册表

rename   重命名一个容器

restart   重新启动一个或多个容器

rm     移除一个或多个容器

rmi     删除一个或多个图像

run     在新容器中运行命令
常用设置参数：
- rm 即运行结束后自动删除容器
- d 后台运行
- p 运行端口
- l 连接另一个容器
save    将一个或多个图像保存到tar存档文件(默认情况下流到STDOUT)

search   在Docker集线器中搜索图像

start    启动一个或多个停止的容器

stats    显示容器资源使用统计数据的实时流

stop    停止一个或多个正在运行的容器

tag     创建一个引用SOURCE_IMAGE的标记TARGET_IMAGE

top     显示容器的运行进程

unpause   在一个或多个容器中暂停所有进程

update   更新一个或多个容器的配置

version   显示Docker版本信息

wait    阻塞，直到一个或多个容器停止，然后打印它们的退出代码

# docker容器基本操作

一.宿主机的内容拷贝到启动的docker容器中
1.查看docker容器状态

```
docker ps-a 
```


2.如果没有启动docker容器则启动docker容器

```
docker start 容器名或者容器ID
```

3.进入容器

```
docker exec -it 容器名/容器ID  /bin/bash
```


4.克隆宿主机会话并在宿主机创立文件
(1)进入容器的根目录

```
cd /
```


(2)创建jhj文件,并写入内容123456

```
echo 123456 > jhj
```


(3)拷贝到docker容器中

```
docker cp 要拷贝的宿主机文件或目录 容器名称:容器文件或目录
```

二、查看容器详情
1.查看容器运行内部细节，比如可看容器的IP

```
docker inspect 容器名：容器ID
```


2.查看容器IP

```
docker inspect --format='{{.NetworkSettings.IPAddress}}` 容器名
```


3.删除容器注意只能删除停止的容器

```
docker rm `docker ps -a -q`
```

4.清理所有处于终止状态的容器:

```
docker container prune
```

# push至本地镜像仓库

1.查看本地镜像

```
docker images
```

2.给镜像打tag标签

```
docker tag 原镜像名:版本号 本地仓库名/仓库管理者/原镜像名:版本号 
```

3.验证镜像标签
```
docker images
```

4.推送

```
docker push 本地仓库名/仓库管理者/原镜像名:版本号 
```





