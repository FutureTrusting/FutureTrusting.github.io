---
layout:     post
title:      Docker部署Redis
subtitle:   Docker部署Redis
date:       2020-05-28
author:     Square
header-img: img/wallhaven-zm3mxw.jpg
catalog: true
tags:
    - Linux
---


#### redis容器初始化
容器初始化，使用docker-compose方式，先创建一个docker-compose.yml文件，内容如下:
```
version: '3'

services:
 redis1:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8001/data:/data
  environment:
   - REDIS_PORT=8001

 redis2:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8002/data:/data
  environment:
   - REDIS_PORT=8002

 redis3:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8003/data:/data
  environment:
   - REDIS_PORT=8003

 redis4:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8004/data:/data
  environment:
   - REDIS_PORT=8004

 redis5:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8005/data:/data
  environment:
   - REDIS_PORT=8005

 redis6:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/8006/data:/data
  environment:
   - REDIS_PORT=8006
```
这里引用了别人的一个镜像publicisworldwide/redis-cluster，方便快捷。
这里使用host(主机)网络模式，把redis数据挂载到本机目录/data/redis/800*下。

若不想使用host模式，也可以把network_mode去掉，但就要加ports映射。
redis-cluster的节点端口共分为2种，一种是节点提供服务的端口，如6379；一种是节点间通信的端口，固定格式为：10000+6379。
我这里使用的是下面这种方式(后面安装的时候有问题，整合springboot获取数据的时候为null，猜想需要开通18001,18002的端口，所以我就试了下第一种，不用开端口也是可以的)
总体来讲推荐第一种
```
version: '3'

services:
 redis1:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8001/data:/data
  environment:
   - REDIS_PORT=8001
  ports:
    - '8001:8001'       #服务端口
    - '18001:18001'   #集群端口

 redis2:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8002/data:/data
  environment:
   - REDIS_PORT=8002
  ports:
    - '8002:8002'
    - '18002:18002'

 redis3:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8003/data:/data
  environment:
   - REDIS_PORT=8003
  ports:
    - '8003:8003'
    - '18003:18003'

 redis4:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8004/data:/data
  environment:
   - REDIS_PORT=8004
  ports:
    - '8004:8004'
    - '18004:18004'

 redis5:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8005/data:/data
  environment:
   - REDIS_PORT=8005
  ports:
    - '8005:8005'
    - '18005:18005'

 redis6:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /data/redis/8006/data:/data
  environment:
   - REDIS_PORT=8006
  ports:
    - '8006:8006'
    - '18006:18006'
```

#### 创建文件后，直接启动服务
```
# 窗口模式
$ docker-compose up
# 后台进程
$ docker-compose up -d
```
#### 运行docker-compose 命令报错
```
$ docker-compose up -d
-bash: docker-compose: command not found

$ pip -V 
-bash: pip: command not found

$ yum -y install epel-release

$ yum -y install python-pip
# 升级
$ pip install --upgrade pip

```
#### 运行pip install docker-compose 命令报错
```
$ pip install docker-compose

compilation terminated.
   error: command 'gcc' failed with exit status 1
   ----------------------------------------
ERROR: Command errored out with exit status 1: /usr/bin/python2 -u -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-ZnU75z/subprocess32/setup.py'"'"'; __file__='"'"'/tmp/pip-install-ZnU75z/subprocess32/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' install --record /tmp/pip-record-HQUs7W/install-record.txt --single-version-externally-managed --compile --install-headers /usr/include/python2.7/subprocess32 Check the logs for full command output.

$ sudo yum install gcc libffi-devel python-devel openssl-devel -y

```
#### 再次运行pip install docker-compose 命令报错
```
$ pip install docker-compose

ERROR: Cannot uninstall 'requests'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.

$ sudo pip install --ignore-installed requests
$ pip install docker-compose

$ docker-compose -version

docker-compose version 1.25.5, build unknown
```
#### 查看启动的进程
```
# 再次运行 docker-compose
[root@localhost redis-cluster]# docker-compose ps
        Name                       Command               State   Ports
----------------------------------------------------------------------
rediscluster_redis1_1   /usr/local/bin/entrypoint. ...   Up           
rediscluster_redis2_1   /usr/local/bin/entrypoint. ...   Up           
rediscluster_redis3_1   /usr/local/bin/entrypoint. ...   Up           
rediscluster_redis4_1   /usr/local/bin/entrypoint. ...   Up           
rediscluster_redis5_1   /usr/local/bin/entrypoint. ...   Up           
rediscluster_redis6_1   /usr/local/bin/entrypoint. ...   Up           
```
状态为Up，说明服务均已启动，镜像无问题。
<Strong>注意：以上镜像不能设置永久密码，其实redis一般是内网访问，可以不需密码。</Strong>
#### redis容器集群配置
上面只是启动了6个redis容器，并没有设置集群，通过下面的命令可以设置集群,注意修改ip地址
```
$ docker run --rm -it inem0o/redis-trib create --replicas 1 106.55.13.2:8001 106.55.13.2:8002 106.55.13.2:8003 106.55.13.2:8004 106.55.13.2:8005 106.55.13.2:8006

```
这里同样使用了另一个镜像inem0o/redis-trib，执行时会自动下载。    
日志如下：
```
[root@localhost disconf]# docker run --rm -it inem0o/redis-trib create --replicas 1 106.55.13.2:8001 106.55.13.2:8002 106.55.13.2:8003 106.55.13.2:8004 106.55.13.2:8005 106.55.13.2:8006
Unable to find image 'inem0o/redis-trib:latest' locally
latest: Pulling from inem0o/redis-trib
a2b2998a36ab: Pull complete 
a3ed95caeb02: Pull complete 
46ab6b64c08e: Pull complete 
3d82c3ac2025: Pull complete 
Digest: sha256:0b89d25b387f70ef1c54605bdf061dd86e0833dbc0e2149390570b8b372278f8
Status: Downloaded newer image for inem0o/redis-trib:latest
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
172.19.165.222:8001
172.19.165.222:8002
172.19.165.222:8003
Adding replica 172.19.165.222:8004 to 172.19.165.222:8001
Adding replica 172.19.165.222:8005 to 172.19.165.222:8002
Adding replica 172.19.165.222:8006 to 172.19.165.222:8003
M: 67d9a6bb6875f3a0f9a53e5bb05ddeca8e656950 172.19.165.222:8001
   slots:0-5460 (5461 slots) master
M: 206626063f31dcd7e69010ce13c258e786197f1e 172.19.165.222:8002
   slots:5461-10922 (5462 slots) master
M: e9924018d95772b8ff535f6bc0605a6630837069 172.19.165.222:8003
   slots:10923-16383 (5461 slots) master
S: 548f4e65fbab8dcde8aac187849d50983d68599d 172.19.165.222:8004
   replicates 67d9a6bb6875f3a0f9a53e5bb05ddeca8e656950
S: 0a5c799c1f8fed083c50902639fc354e4c25aa8c 172.19.165.222:8005
   replicates 206626063f31dcd7e69010ce13c258e786197f1e
S: 94e2530ddd05b0e9eb3e71a9616342bd6647a5e6 172.19.165.222:8006
   replicates e9924018d95772b8ff535f6bc0605a6630837069
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join....
>>> Performing Cluster Check (using node 172.19.165.222:8001)
M: 67d9a6bb6875f3a0f9a53e5bb05ddeca8e656950 172.19.165.222:8001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 94e2530ddd05b0e9eb3e71a9616342bd6647a5e6 172.19.165.222:8006@18006
   slots: (0 slots) slave
   replicates e9924018d95772b8ff535f6bc0605a6630837069
S: 0a5c799c1f8fed083c50902639fc354e4c25aa8c 172.19.165.222:8005@18005
   slots: (0 slots) slave
   replicates 206626063f31dcd7e69010ce13c258e786197f1e
M: e9924018d95772b8ff535f6bc0605a6630837069 172.19.165.222:8003@18003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: 206626063f31dcd7e69010ce13c258e786197f1e 172.19.165.222:8002@18002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 548f4e65fbab8dcde8aac187849d50983d68599d 172.19.165.222:8004@18004
   slots: (0 slots) slave
   replicates 67d9a6bb6875f3a0f9a53e5bb05ddeca8e656950
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
#### 若创建成功，可以使用命令登录并查看集群信息。
```
$ docker ps

CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS              PORTS                               NAMES
df0d49474423        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   28 minutes ago      Up 28 minutes                                           deploy_redis6_1
800da782aefa        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   28 minutes ago      Up 28 minutes                                           deploy_redis1_1
c5ac53febd7e        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   28 minutes ago      Up 28 minutes                                           deploy_redis3_1
5306b6f054a6        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   28 minutes ago      Up 28 minutes                                           deploy_redis5_1
3e66efae6103        publicisworldwide/redis-cluster   "/usr/local/bin/entr…"   28 minutes ago      Up 28 minutes                                           deploy_redis4_1

# 进入其中一个容器
$ docker  exec -it df0d49474423  redis-cli -h 106.55.13.2 -p 8001

```
与上面相同，重新执行上面的第二步，再次查看结果如下：
以下信息显示集群状态ok。
```
127.0.0.1:8003> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:3
cluster_stats_messages_ping_sent:39
cluster_stats_messages_pong_sent:37
cluster_stats_messages_meet_sent:4
cluster_stats_messages_sent:80
cluster_stats_messages_ping_received:34
cluster_stats_messages_pong_received:43
cluster_stats_messages_meet_received:3
cluster_stats_messages_received:80
```
#### 设置集群密码
上面提供了publicisworldwide/redis-cluster的镜像不能设置永久密码，是因为它没有给redis.conf写入权限，在我们重创建的镜像里是有写入权限的，所以可以创建永久密码。
集群构建完毕后再通过config set + config rewrite命令逐个机器设置密码
在之前的镜像上设置密码会有权限问题，如下：
```
127.0.0.1:8001> config set masterauth 12345
OK
127.0.0.1:8001> config set requirepass 12345
OK
127.0.0.1:8001> auth
(error) ERR wrong number of arguments for 'auth' command
127.0.0.1:8001> auth 12345
OK
127.0.0.1:8001> config rewrite
(error) ERR Rewriting config file: Permission denied
```

#### 云服务器问题
这个问题看了这个[博客](https://www.jianshu.com/p/7fec6d0d0ae0)才想到怎么解决
```
#以集群方式进入任意一个节点，密码为前面配置文件所配置密码
$ docker  exec -it b38c1d873732  redis-cli -h 106.55.13.2 -p 8002 -a 12345
#进入后查看Redis集群主从信息
CLUSTER NODES
#上面命令作者结果如下，master表示主库，slave表示从库，前面字母和数字表示节点ID，slave后面的字母和数字表示所属主库的节点ID,后面还有连接状态等
50dceba04d0fa0fd0080a5f842496db07f9366d9 公网IP:8005@18005 slave 54e44f550328562d3cf3984426dc9b163aa9e2b1 0 1581404581690 9 connected
7273d970b66d2dcb0257f4d7555309f5c2404a09 公网IP:8003@18003 master - 0 1581404579686 12 connected 10923-16383
f2dbad625bd52b102a916e5ae211cdc999a1d781 公网IP:8002@18002 master - 0 1581404580688 14 connected 5461-10922
54e44f550328562d3cf3984426dc9b163aa9e2b1 内网IP:8001@18001 myself,master - 0 1581404580000 9 connected 0-5460
420299acf743549c19cab5ecf0b6cb9e0f482b9b 公网IP:8004@18004 slave 7273d970b66d2dcb0257f4d7555309f5c2404a09 0 1581404581000 12 connected
a4a884ed02f0da555020cb2e5fb5bbfbe2148a05 公网IP:8006@18006 slave f2dbad625bd52b102a916e5ae211cdc999a1d781 0 1581404579000 14 connected
```
接下来是解决本章前面提出的问题。前面我们说到整合Spring Boot后，它会用任意节点通过CLUSTER SLOTS命令去获取集群中的槽点信息，
我们可以看到上面的结果中8001节点返回的是内网IP，其实当我们用8002节点执行CLUSTER SLOTS命令时会发现7002节点对应的IP地址变为了内网IP，
而其他节点包括之前的7001节点都是公网IP，也就是说用哪个节点获取集群信息则那个节点会变成内网IP，
这是因为每个节点的集群信息是从之前每个节点的配置文件里面的这一行cluster-config-file nodes_8001.conf指定文件里面读取的信息，
前面说过第一次启动后会在cd ./cluster/data/相对应节点目录下生成该文件，打开该文件会发现每个节点自己本身的IP地址均为内网IP，其他的均为公网IP，这是因为生成时自己节点的IP是通过读取网卡IP作为地址的，由于云服务器的网卡地址是内网，所以这里是内网IP。那么知道了它是怎么获取信息的那就好解决了，以修改8001节点为例，
整合Spring Boot报如下错误：
```
Caused by: org.redisson.client.RedisConnectionException: Not all slots are covered! Only 10923 slots are avaliable
	at org.redisson.cluster.ClusterConnectionManager.<init>(ClusterConnectionManager.java:174)
	at org.redisson.config.ConfigSupport.createConnectionManager(ConfigSupport.java:200)
	at org.redisson.Redisson.<init>(Redisson.java:120)
	at org.redisson.Redisson.create(Redisson.java:160)
	at com.ms.redisdemo.redisson.RedissonClusterUtil.createClient(RedissonClusterUtil.java:51)
	at com.ms.redisdemo.redisson.RedissonClusterUtil.<clinit>(RedissonClusterUtil.java:39)
	... 26 more
Caused by: io.netty.channel.ConnectTimeoutException: connection timed out: 172.16.30.41/172.16.30.41:8001
	at io.netty.channel.nio.AbstractNioChannel$AbstractNioUnsafe$1.run(AbstractNioChannel.java:263)
	at io.netty.util.concurrent.PromiseTask$RunnableAdapter.call(PromiseTask.java:38)
	at io.netty.util.concurrent.ScheduledFutureTask.run(ScheduledFutureTask.java:127)
	at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:163)
	at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:416)
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:515)
	at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918)
	at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
	at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
	at java.lang.Thread.run(Thread.java:745)
```
解决步骤如下：
```
#停止容器进程
$ docker stop b38c1d873732
#进入配置目录
$ cd /data/redis/8001/data/
# 依次修改集群文件
$ vim nodes.conf 

013c59dceb1bf2262385106760f36b9239f7ef55 外网:8005@18005 slave 31603ac0097d0958970d43a01db571761b61828b 0 1590716333000 8 connected
8829af563e092f9dcf36f49c015f5b22838b9f1f 内网:8001@18001 myself,master - 0 1590716333000 11 connected 0-5460
31603ac0097d0958970d43a01db571761b61828b 外网:8002@18002 master - 0 1590716333000 8 connected 5461-10922
a7a7e90c814d0bc70757a78f4a7421b2937fb949 外网:8006@18006 master - 0 1590716333000 9 connected 10923-16383
9ff0099fefde6a4e60a85f7ebf4fc0cf82210ae1 外网:8004@18004 slave 8829af563e092f9dcf36f49c015f5b22838b9f1f 0 1590716334708 11 connected
9c88658767c94dca36d7e4594fb944064cc7edf2 外网:8003@18003 slave a7a7e90c814d0bc70757a78f4a7421b2937fb949 0 1590716333909 9 connected

# 启动容器服务
$ docker start b38c1d873732
```