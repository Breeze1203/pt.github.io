---
title: "docker运行rocketMQ"
date: 2025-06-11 19:23:38
categories: docker
tags: docker
---

### docker安装rocketMQ

##### 拉取镜像

``` shell
#拉取镜像 
docker pull rocketmqinc/rocketmq
```

##### 创建数据挂载目录

``` shell
mkdir -p  /docker/rocketmq/data/namesrv/logs   /docker/rocketmq/data/namesrv/store
mkdir -p  /docker/rocketmq/data/broker/logs   /docker/rocketmq/data/broker/store /docker/rocketmq/conf
```

##### **<span color="" fontsize="">启动NameServer</span>**

``` shell
docker run -d \
--restart=always \
--name rmqnamesrv \
-p 9876:9876 \
-v /docker/rocketmq/data/namesrv/logs:/root/logs \
-v /docker/rocketmq/data/namesrv/store:/root/store \
-e "MAX_POSSIBLE_HEAP=100000000" \
rocketmqinc/rocketmq \
sh mqnamesrv 
```

##### 编辑配置文件

``` java
vim /docker/rocketmq/conf/broker.conf
```

    brokerClusterName = DefaultCluster
    #broker名称，master和slave使用相同的名称，表明他们的主从关系
    brokerName = broker-a
    #0表示Master，大于0表示不同的slave
    brokerId = 0
    #表示几点做消息删除动作，默认是凌晨4点
    deleteWhen = 04
    #在磁盘上保留消息的时长，单位是小时
    fileReservedTime = 48
    #有三个值：SYNC_MASTER，ASYNC_MASTER，SLAVE；同步和异步表示Master和Slave之间同步数据的机制；
    brokerRole = ASYNC_MASTER
    #刷盘策略，取值为：ASYNC_FLUSH，SYNC_FLUSH表示同步刷盘和异步刷盘；SYNC_FLUSH消息写入磁盘后才返回成功状态，ASYNC_FLUSH不需要；
    flushDiskType = ASYNC_FLUSH
    #设置broker节点所在服务器的ip地址
    brokerIP1 = 39.105.18

##### 启动broker

    docker run -d  \
    --restart=always \
    --name rmqbroker \
    --link rmqnamesrv:namesrv \
    -p 10911:10911 \
    -p 10909:10909 \
    -v  /docker/rocketmq/data/broker/logs:/root/logs \
    -v  /docker/rocketmq/data/broker/store:/root/store \
    -v /docker/rocketmq/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf \
    -e "NAMESRV_ADDR=namesrv:9876" \
    -e "MAX_POSSIBLE_HEAP=200000000" \
    rocketmqinc/rocketmq \
    sh mqbroker -c /opt/rocketmq-4.4.0/conf/broker.conf

##### <span color="" fontsize="">部署RocketMQ的管理工具</span>

<span style="font-size: 14px; color: rgb(51, 51, 51)">RocketMQ提供了UI管理工具，名为rocketmq-console，我们选择docker安装</span>

    #拉取镜像 
    docker pull pangliang/rocketmq-console-ng
    #创建并启动容器 
    docker run -d \
    --restart=always \
    --name rmqadmin \
    -e "JAVA_OPTS=-Drocketmq.namesrv.addr=39.105.18:9876 \
    -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
    -p 8082:8080 \
    pangliang/rocketmq-console-ng

<span style="font-size: 14px; color: rgb(51, 51, 51)">通过浏览器进行访问：</span><a href="http://39.105.18:8082/" rel="noopener noreferrer nofollow" target="_blank"><u><span style="font-size: 14px; color: rgb(51, 51, 51)">http://39.105.18:8082/</span></u></a>
