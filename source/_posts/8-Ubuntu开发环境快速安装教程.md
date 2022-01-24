---
title: Ubuntu开发环境快速安装教程
date: 2021-11-23 12:00:10
author: hypo
top: false
hide: false
cover: false
img: medias/featureimages/8.jpg
password:
toc: true
mathjax: false
summary: Ubuntu开发环境快速安装教程
categories: 笔记
tags:
- 笔记
- 教程
- Linux
---

# Ubuntu开发环境快速安装教程：



### ubuntu安装go：

1、下载二进制包：go1.4.linux-amd64.tar.gz。

2、将下载的二进制包解压至 /usr/local目录。

```
tar -C /usr/local -xzf go1.4.linux-amd64.tar.gz
```

3、将 /usr/local/go/bin 目录添加至PATH环境变量：

```
export PATH=$PATH:/usr/local/go/bin
```



### ubuntu安装docker：

在终端中输入以下命令：

```
sudo apt-get update
```

没有错误则成功了。

二、安装需要的包

在终端输入命令

```
sudo apt-get install apt-transport-https ca-certificates software-properties-common curl
```

没有错误则成功。
三、添加 GPG 密钥，并添加 Docker-ce 软件源

官方的软件源（不推荐，很慢）：

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg|sudo apt-key add -
```

```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

中国科技大学的 Docker-ce 源（其他源类似）：

```
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

```
sudo add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
$(lsb_release -cs) stable"
```

##### 注意：添加错了可以用以下命令删除

##### 查询keyid,下图

```
sudo apt-key list
```

##### keyid 就是90那一串

```
sudo apt-key del <keyid>
```

##### 加参数-r可以移除

```
sudo add-apt-repository -r "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```


更新软件包缓存

```
sudo apt-get update
```

四、安装 Docker-ce

```
sudo apt-get install docker-ce
```

五、测试运行

```
sudo docker run hello-world
```


六、添加当前用户到 docker 用户组，可以不用 sudo 运行 docker

将当前用户添加到 docker 组

```
sudo gpasswd -a ${USER} docker
```

重新登录或者用以下命令切换到docker组

```
newgrp - docker
```

重启docker服务

```
sudo service docker restart
```

不加sudo直接执行docker命令检查效果

```
docker ps
```



### ubuntu安装git：

##### Linux 平台上安装

Git 的工作需要调用 curl，zlib，openssl，expat，libiconv 等库的代码，所以需要先安装这些依赖工具。

在有 yum 的系统上（比如 Fedora）或者有 apt-get 的系统上（比如 Debian 体系），可以用下面的命令安装：

各 Linux 系统可以使用其安装包管理工具（apt-get、yum 等）进行安装：

##### Debian/Ubuntu

Debian/Ubuntu Git 安装命令为：

```
$ apt-get install libcurl4-gnutls-dev libexpat1-dev gettext \
  libz-dev libssl-dev

$ apt-get install git

$ git --version
git version 1.8.1.2
```



### ubuntu安装goland：

##### goland安装

1. 下载goland
   [Goland下载链接](https://www.jetbrains.com/go/download/#section=linux)
2. cd到安装包目录，移动并解压

```bash
sudo tar -zxvf goland-2020.3.4.tar.gz -C /usr/local/ #包的名字自行修改
```

1. 启动Goland

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210320104439418.png)



### ubuntu安装google chorme：

下载Chrome

```
$ wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
```

安装

```
$ sudo dpkg -i google-chrome-stable_current_amd64.deb
```

报错

```
$ sudo dpkg -i google-chrome-stable_current_amd64.deb
(Reading database ... 155736 files and directories currently installed.)
Preparing to unpack google-chrome-stable_current_amd64.deb ...
Unpacking google-chrome-stable (88.0.4324.96-1) ...
dpkg: dependency problems prevent configuration of google-chrome-stable:
 google-chrome-stable depends on fonts-liberation; however:
  Package fonts-liberation is not installed.
 google-chrome-stable depends on libatk-bridge2.0-0 (>= 2.5.3); however:
  Package libatk-bridge2.0-0 is not installed.
 google-chrome-stable depends on libatk1.0-0 (>= 2.2.0); however:
  Package libatk1.0-0 is not installed.
 google-chrome-stable depends on libatspi2.0-0 (>= 2.9.90); however:
  Package libatspi2.0-0 is not installed.
 google-chrome-stable depends on libcairo2 (>= 1.6.0); however:
  Package libcairo2 is not installed.
 google-chrome-stable depends on libcups2 (>= 1.4.0); however:
  Package libcups2 is not installed.
 google-chrome-stable depends on libgbm1 (>= 8.1~0); however:
  Package libgbm1 is not installed.
 google-chrome-stable depends on libgdk-pixbuf2.0-0 (>= 2.22.0); however:
  Package libgdk-pixbuf2.0-0 is not installed.
 google-chrome-stable depends on libgtk-3-0 (>= 3.9.10); however:
  Package libgtk-3-0 is not installed.
 google-chrome-stable depends on libnspr4 (>= 2:4.9-2~); however:
  Package libnspr4 is not installed.
 google-chrome-stable depends on libnss3 (>= 2:3.22); however:
  Package libnss3 is not installed.
 google-chrome-stable depends on libpango-1.0-0 (>= 1.14.0); however:
  Package libpango-1.0-0 is not installed.
 google-chrome-stable depends on libxkbcommon0 (>= 0.4.1); however:
  Package libxkbcommon0 is not installed.
 google-chrome-stable depends on xdg-utils (>= 1.0.2); however:
  Package xdg-utils is not installed.

dpkg: error processing package google-chrome-stable (--install):
 dependency problems - leaving unconfigured
Processing triggers for mime-support (3.64ubuntu1) ...
Processing triggers for man-db (2.9.1-1) ...
Errors were encountered while processing:
 google-chrome-stable


```

报错原因：缺少依赖软件包

```
dpkg: error processing package google-chrome-stable (--install):
 dependency problems - leaving unconfigured
```

解决办法：修复依赖关系

```
$ sudo apt-get install -f
```

如果系统中有某个软件包不满足依赖条件,这个命令就会自动修复,将要安装那个软件包依赖的软件包。

查看Chrome版本信息

```
$ google-chrome --version
```



### ubuntu安装teams：

1、去官网下载二进制包teams.tar.gz。

2、将下载的二进制包解压至 /usr/local目录。

```
tar -C /usr/local -xzf teams.tar.gz
```



### Ubuntu安装typora:

在终端中输入以下命令：

```
# sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -

# add Typora's repository
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update

# install typora
sudo apt-get install typora
```



### ubuntu安装cloudmusic：

先到网站下载安装包，网址：https://music.163.com/#/download , 然后下载客户端，点击下载全部客户端，选择Linux版，选择Ubuntu18.04，其实我的20.04 版的没找到，不过这个用起来还不错。至于是32还是64，自己看吧。
然后将安装包下载到：~/下载 目录下，然后在这个目录下面打开终端。
在终端里面输入：sudo dpkg -i netease-cloud-music_1.2.1_amd64_ubuntu_20190428.deb 就行了

### ubuntu安装vim：

自带的vi不好用，所以自己重新下了一下

```
sudo apt-get install vim
```

安装完成后进入/etc/vim目录下配置文件，使用会舒服一点

```
sudo vim vimrc
```

在末尾添加代码

    set number   
    set history=1000   
    set autoindent
    set smartindent 
    set tabstop=4 
    set shiftwidth=4 
    set showmatch



### ubuntu 安装PostMan:

```test
sudo snap install postman
```



#### ubuntu 安装nodejs和npm:

一.下载最新版本的nodejs包（最新版本的nodejs包里默认含有npm）
可以去nodejs官网去查看最新版本的nodejs
网址如下：https://nodejs.org/en/download/current/
目前最新版本为：v17.1.0

```
wget https://nodejs.org/dist/v17.1.0/node-v17.1.0-linux-x64.tar.xz    
tar xf  node-v17.1.0-linux-x64.tar.xz       // 解压
cd node-v17.1.0-linux-x64/                  // 进入解压目录
 ./bin/node -v                               // 执行node命令 查看版本
// bin目录下有执行文件npm和node 做软链接
```


下载到你当前的目录,假如是：/home/root/

第二步 创建软连接 可以在任意路径下执行npm node
注意：在创建软连接的时候要写 绝对路径,软连接到/usr/local/bin/

```
ln -s  /home/root/node-v17.1.0-linux-x64/bin/node  /usr/local/bin/
ln -s /home/root/node-v17.1.0-linux-x64/bin/npm  /usr/local/bin/
```

// 注意要写文件的绝对路径

如果你在创建软连的时候，出现npm已经存在,node 已经存在
解决方案：
删除 /usr/local/bin/目录下的node，npm

rt@ubuntu:~$ sudo rm -rf /usr/local/bin/node
rt@ubuntu:~$ sudo rm -rf /usr/local/bin/npm

之后再创建软连接，完成。

```
rbt@ubuntu:~$ node -v
v17.1.0
rt@ubuntu:~$ npm -v
8.4.1
```



这样node和npm就同时安装成功了！



#### ubuntu安装python3：

ubuntu本身是有Python2.7版本的，但是不同版本的ubuntu中的Python3的版本是不同的，我现在用的是14版本就是python3.4，我想把它升级为3.6版本。当然，如果需要，你可以改为任意版本。

1. 安装python3.6(非必需)
   在终端中输入下面的命令（不要怀疑，每行都是一个命令）

```
wget http://www.python.org/ftp/python/3.6.4/Python-3.6.4.tgz  
tar -xvzf Python-3.6.4.tgz  
cd Python-3.6.4  
./configure --with-ssl  
make  
sudo make install
```

这些命令会使你的ubuntu下载python3.6.4，并替换你现在的python3版本。

2. 安装python运行环境
   输入sudo passwd 输入root相关密码，输入su，进入超级管理员（如果你没设置过，需要设置root用户密码），也许你在安装时还需要升级你的apt-get，命令行下输入apt-get update

```
sudo apt-get install python
sudo apt-get install python-dev(编译外部模块文件使用的)
sudo apt-get install python-pip
sudo apt-get install libxml*
sudo apt-get install net-tools
sudo apt-get install lsof
```

python3的话安装pip，命令为sudo apt-get install python3-pip
执行之后，输入python3来确定你是否安装成功，如下图所示显示python3.6.4即安装成功：

3.更新pip版本

```
sudo pip install --upgrade pip
```

4.安装SSH

```
sudo apt-get install openssh-server
```

5.安装 Nginx

```
sudo apt-get install nginx
```

6.安装 uwsgi

```
sudo pip install uwsgi
```

部署django项目前输入以下命令开启8000端口

```
uwsgi --http :8000  --chdir 项目路径 -w  项目名称.wsgi
```


解除端口被占用的命令：

```
sudo fuser -k 8000/tcp
```



##### 至此ubuntu的基本的软件开发环境基本搭建完成，本文只做快速搭建环境的流程，具体问题网上都有解决方案，还有就是为什么没有数据库的安装？废话，都有Docker了，谁还安装数据库呀！！！