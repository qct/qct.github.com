---
layout: post
title: "ApacheMQ LevelDB 集群 安装配置"
description: "1. 安装keepalived   `yum install -y gcc openssl-devel popt-devel libnl-devel kernel-devel`    "
category: 技术
tags: [activeMQ, levelDB]
---
{% include JB/setup %}

#### **LevelDB是Apache activeMQ 5.9开始推荐的集群部署方式，下面采用3台来部署activeMQ 的 LevelDB 集群**   
## 1.首先安装zookeeper
    wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
    tar -zxvf zookeeper-3.4.6.tar.gz 
    mv zookeeper-3.4.6 /usr/local/
    
    vim /etc/profile
    #Set ZooKeeper Enviroment
    export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.6
    export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf
    
    mkdir /var/zookeeper
    echo 1 > /var/zookeeper/myid
    
    vim /usr/local/zookeeper-3.4.6/conf/zoo.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/var/zookeeper
    clientPort=2181
    server.1=10.22.205.111:2888:3888
    server.2=10.22.205.112:2888:3888
    server.3=10.22.205.122:2888:3888
    
    /usr/local/zookeeper-3.4.6/bin/zkServer.sh start
    /usr/local/zookeeper-3.4.6/bin/zkServer.sh status
    /usr/local/zookeeper-3.4.6/bin/zkCli.sh –server 10.22.205.111:2181

在10.22.205.112和10.22.205.122上`/var/zookeeper/myid`里的值分别是2、3   

## 2.安装activemq
    wget http://mirror.bit.edu.cn/apache/activemq/5.10.0/apache-activemq-5.10.0-bin.tar.gz
    tar -zxvf apache-activemq-5.10.0-bin.tar.gz
    mv apache-activemq-5.10.0 /usr/local/
    chmod 755 /usr/local/apache-activemq-5.10.0/bin/activemq
    
    vim /usr/local/apache-activemq-5.10.0/conf/activemq.xml
    <persistenceAdapter>
      <replicatedLevelDB
        directory="${activemq.data}/leveldb"
        replicas="3"
        bind="tcp://0.0.0.0:0"
        zkAddress="10.22.205.111:2181,10.22.205.112:2181,10.22.205.122:2181"
        hostname="10.22.205.111"
        sync="local_disk"
        zkPath="/activemq/leveldb-stores" />
    </persistenceAdapter>
在10.22.205.112和10.22.205.122上做同样配置，然后先启动zookeeper集群，再启动activemq集群。


#### **另外还可以部署一个nodejs的zookeeper监控 Node-ZK-Browser， 观察起来比较直观，nodejs的环境需要自己先搭起来。**
