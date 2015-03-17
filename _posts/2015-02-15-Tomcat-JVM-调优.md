---
layout: post
title: "Tomcat JVM 调优"
description: "JVM 主要参数设置：

1. Xms: 初始堆大小，通常是物理内存的50%到80%"
category: 技术
tags: [java, jvm, tomcat]
---
{% include JB/setup %}

## JVM 主要参数设置：

1. Xms: 初始堆大小，通常是物理内存的50%到80%   
2. Xmx: 最大堆大小，通常和Xms一样大   
3. Xmn：最小堆大小，通常是Xmx的一半   
4. PermSize: 永久区，物理内存的1/64   
5. MaxPermSize: 永久区最大值，物理内存的1/4   

`JAVA_OPTS="$JAVA_OPTS -server -Xms8g -Xmx8g -Xmn4g -XX:PermSize=1g -XX:MaxPermSize=2g -verbose:gc -Xloggc:${CATALINA_HOME}/logs/gc.log\`date +%Y-%m-%d-%H-%M\` -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -noclassgc"`


## server.xml 配置   

    <Connector port="8080" 
             redirectPort="8443" URIEncoding="UTF-8" 
             executor="tomcatThreadPool" protocol="org.apache.coyote.http11.Http11NioProtocol" 
             maxThreads="500" minSpareThreads="50" 
             connectionTimeout="20000" 
             enableLookups="false"
             maxPostSize="0"
             />
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="500" minSpareThreads="50" maxIdleTime="600000"/>
