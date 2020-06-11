---
layout:     post
title:      CentOS7安装Docker
subtitle:   CentOS7安装Docker
date:       2020-05-21
author:     Square
header-img: img/wallhaven-zm3mxw.jpg
catalog: true
tags:
    - Linux
---

#### 前言
Docker 使用越来越多，安装也很简单，本次记录一下基本的步骤。

Docker 目前支持 CentOS 7 及以后的版本，内核要求至少为 3.10。

Docker 官网有安装步骤，本文只是记录一下，您也可以参考 [Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)

#### 环境说明
CentOS 7（Minimal Install）

```
$ cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
```
#### 操作系统要求
CentOS 7 以后都可以安装 Docker 了，也可以确认一下。
```
$ uname -a
Linux localhost.localdomain 3.10.0-957.1.3.el7.x86_64 #1 SMP Thu Nov 29 14:49:43 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
Docker 需要用到 centos-extra 这个源，如果您关闭了，需要重启启用，可以参考 [Available Repositories for CentOS](https://wiki.centos.org/AdditionalResources/Repositories)

#### 卸载旧版本
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
旧版本的内容在 /var/lib/docker 下，目录中的镜像(images), 容器(containers), 存储卷(volumes), 和 网络配置（networks）都可以保留。

Docker CE 包，目前的包名为 docker-ce。

#### 安装准备
为了方便添加软件源，支持 devicemapper 存储类型，安装如下软件包

```
$ sudo yum update
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
#### 添加 yum 软件源
添加 Docker 稳定版本的 yum 软件源

```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

#### 安装指定版本
更新一下 yum 软件源的缓存，并安装 Docker。
```
$ sudo yum update
```

如果想安装指定版本的 Docker，可以查看一下版本并安装。

```
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
```
可以指定版本安装,版本号可以忽略 : 和 el7，如 docker-ce-18.09.1
```
$ sudo yum install docker-ce-19.03.6
```
至此，指定版本的 Docker 也安装完成，同样，操作系统内 docker 服务没有启动，只创建了 docker 组，而且组里没有用户。


使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS 系统上可以使用这套脚本安装：
```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```
执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 Edge 版本安装在系统中。

#### 启动 Docker
如果想添加到开机启动
```
$ sudo systemctl enable docker
```
启动 docker 服务

```
$ sudo systemctl start docker
```
#### 验证安装
验证 Docker CE 安装是否正确，可以运行 hello-world 镜像

```
$ sudo docker run hello-world
```
#### 更新 Docker CE
使用 yum 管理，更新和卸载都很方便。

```
$ sudo yum update docker-ce
```
#### 卸载 Docker CE
```
$ sudo yum remove docker-ce
```

#### 删除本地文件
注意，docker 的本地文件，包括镜像(images), 容器(containers), 存储卷(volumes)等，都需要手工删除。默认目录存储在 /var/lib/docker。
```
$ sudo rm -rf /var/lib/docker
```
#### 设置镜像
```
vi /etc/docker/daemon.json
{
  "registry-mirrors": ["https://aj2rgad5.mirror.aliyuncs.com"]
}
```
#### 重启docker
```
$ systemctl daemon-reload
$ systemctl restart docker.service
```

#### 测试docker是否正常安装和运行
```
$ docker run hello-world
```

#### 查看结果
```
$ Hello from Docker!
This message shows that your installation appears to be working correctly.
```

#### Docker ps 命令
 ```
$ docker ps           列出所有在运行的容器信息
$ docker ps -a        显示所有的容器，包括未运行的
$ docker ps -n 5      列出最近创建的5个容器信息
 ```
#### Docker kill 命令
```
$ docker ps           列出所有在运行的容器信息
6ef9b53290fa        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   9 minutes ago       Up 7 minutes                                            deploy_redis1_1
4094cdf6eb2f        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   9 minutes ago       Up 7 minutes                                            deploy_redis4_1
c6445bb1629f        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   9 minutes ago       Up 7 minutes                                            deploy_redis3_1
9e69b9811ec8        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   9 minutes ago       Up 7 minutes                                            deploy_redis2_1

$ docker rm 6ef9b53290fa        删除已经停止的容器xxx
$ docker rm -f  6ef9b53290fa    可以删除正在运行的容器xxx
```
#### 
```
docker run ****** --restart=always
```



