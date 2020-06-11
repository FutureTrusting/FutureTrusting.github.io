---
layout:     post
title:      Docker部署jar
subtitle:   Docker部署jar
date:       2020-06-02
author:     Square
header-img: img/wallhaven-zm3mxw.jpg
catalog: true
tags:
    - Linux
---


#### 一、 上传jar到服务器的指定目录
##### 2. 在该目录下创建Dockerfile 文件
```
vi Dockerfile

FROM java:8
MAINTAINER bingo
ADD demo-0.0.1-SNAPSHOT.jar demo.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","demo.jar"]

```
> from java:8              拉取一个jdk为1.8的docker image
> maintainer               作者是bingo
> demo-0.0.1-SNAPSHOT.jar  就是你上传的jar包，替换为jar包的名称
> demo.jar                 是你将该jar包重新命名为什么名称，在容器中运行
>  expose                  该容器暴露的端口是多少，就是jar在容器中以多少端口运行
> entrypoint               容器启动之后执行的命令，java -jar demo.jar  即启动jar

##### 4. 创建好Dockerfile文件之后，执行命令 构建镜像：
```
   docker build -t square-root  .
```
注意最后的 .  表示 Dockerfile 文件在当前目录下
square-root  构建之后镜像名称
##### 5. 镜像构建成功运行容器
```
    docker run -d --restart=always --name square -p 18091:18091  square-root  --restart=always
```
 --restart=always 这个表示docker容器在停止或服务器开机之后会自动重新启动
##### 6. 查看启动日志
```
### 查看启动日志
docker logs --tail -300f square  
```
如果docker run 的时候没有加 --restart=always ，然后已经运行的docker容器怎么设置自动重启？ 执行下面命令：
```
docker container update --restart=always square
```
square : 你的容器名称

#### 二、 配置连接docker部署
##### 1.修改docker远程连接配置
添加如下内容
-H tcp://0.0.0.0:2375  -H unix:///var/run/docker.sock
```
$ vi  /usr/lib/systemd/system/docker.service 

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
BindsTo=containerd.service
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375  -H unix:///var/run/docker.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

```
##### 2. 保存配置重启Docker
```
$ systemctl daemon-reload    
$ service docker restart 
```
##### 3. 安装Docker插件
点击 File->Settings->Plugins->Browse Repositories 如下
![](https://alwaysfaith.github.io/img/hash/v2-3b5d910a510a359c0a0c69cb135b4996_720w.jpg)
打开 File->Settings->Build,Execution,Deployment->Docker ，然后配置一下 Docker 的远程连接地址：
![](https://alwaysfaith.github.io/img/hash/v2-3b5d910a510a359c0a0c69cb135b4996_720w.jpg)
##### 4. 配置 Dockerfile
```
FROM hub.c.163.com/library/java:latest
VOLUME /tmp
ADD target/docker-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
> Spring Boot 项目的运行依赖 Java 环境，所以我自己的镜像基于 Java 镜像来构建。
> 考虑到 Docker 官方镜像下载较慢，我这里使用了网易提供的 Docker 镜像。
> 由于 Spring Boot 运行时需要 tmp 目录，这里数据卷配置一个 /tmp 目录出来。
> 将本地 target 目录中打包好的 .jar 文件复制一份新的 到 /app.jar。
> 最后就是配置一下启动命令，由于我打包的 jar 已经成为 app.jar 了，所以启动命令也是启动 app.jar。

##### 5.配置 Maven 插件
```
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.2.0</version>
    <executions>
        <execution>
            <id>build-image</id>
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <dockerHost>http://192.168.66.131:2375</dockerHost>
        <imageName>javaboy/${project.artifactId}</imageName>
        <imageTags>
            <imageTag>${project.version}</imageTag>
        </imageTags>
        <forceTags>true</forceTags>
        <dockerDirectory>${project.basedir}</dockerDirectory>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>

```
> 首先在 execution 节点中配置当执行 mvn package 的时候，顺便也执行一下 docker:build
> 然后在 configuration 中分别配置 Docker 的主机地址，镜像的名称，镜像的 tags，其中 dockerDirectory 表示指定 Dockerfile 的位置。
> 最后 resource 节点中再配置一下 jar 的位置和名称即可。

##### 6.打包运行

