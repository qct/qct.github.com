---
layout: post
title: "IDCLOUD持续化集成部署(jenkens docker)"
description: "文档包含以下内容：Jenkins 的 docker 镜像准备 ,配置Jenkins ,创建持续集成任务 ,启动脚本编写 ,持续集成 ,后话"
category: 技术
tags: [docker, jenkins, 持续集成]
---
{% include JB/setup %}


文档包含以下内容：

 * Jenkins 的 docker 镜像准备
 * 配置Jenkins
 * 创建持续集成任务
 * 启动脚本编写
 * 持续集成
 * 后话

### 1. Jenkins 的 docker 镜像准备

本文假设你已经熟悉docker的使用。

从ubuntu 14.04版本制作Jenkins 的 docker 镜像，安装好docker之后，使用 `docker pull jenkinsci/jenkins` 先拉取jenkins 的 docker 镜像到本地。

修改Jenkins启动端口：
```
touch /root/docker_jenkins/Dockerfile
echo -e "FROM jenkinsci/jenkins \nENV JENKINS_OPTS --httpPort=8765 \nEXPOSE 8765" > /root/docker_jenkins/Dockerfile
```
然后build成一个镜像：`cd /root/docker_jenkins/ && docker build -t myjenkins .`

准备一个Jenkins的数据目录，用来备份数据：
```
mkdir /jenkins_data_test
chmod 777 /jenkins_data_test
```

然后用刚刚build的镜像启动容器：
`docker run -itd --name myjenkins -p 8765:8765 -v /jenkins_data_test:/var/jenkins_home myjenkins`

之后就可以用 http://ip: port 方式访问Jenkins了，例如：http://192.168.9.112:8765/

### 2. 配置Jenkins

Jenkins的配置都在 系统管理-->系统配置 里面。

**jdk安装：**

JDK安装可以在 系统管理-->系统配置 里面让Jenkins去oracle官网下载，也可以自己在Jenkins容器里安装好后在这里配置JAVA_HOME，这里我们自己选择后者，因为自己去下需要提供oracle的账号信息并且下载时间比较长。自己下载好jdk之后放到容器里(可以通过映射出去的目录)，这里使用 jdk-8u74-linux-x64.tar.gz，比如在Jenkins容器里解压的位置是：/usr/lib/jvm/jdk1.8.0_74/ ，为了让容器中默认java是这个版本（后面的脚本中要用到），使用alternatives安装java：

```
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_74/bin/java 300
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_74/bin/javac 300
sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/jdk1.8.0_74/bin/jar 300
sudo update-alternatives --install /usr/bin/javah javah /usr/lib/jvm/jdk1.8.0_74/bin/javah 300
sudo update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/jdk1.8.0_74/bin/javap 300
```
然后配置默认java版本：
`update-alternatives --config java` 选择刚才安装的版本就好。

然后在 系统管理-->系统配置 中配置刚刚安装的JAVA_HOME。

**maven安装：**

maven是作为插件的方式被Jenkins默认安装过的，当然可以自己下载安装包解压，同时在Jenkins 系统配置里配置MAVEN_HOME。这里就不安装了，直接使用默认的。当然，默认的maven配置文件我们也可以修改，位置在：/jenkins_data_test/maven-settings.xml

**git安装：**

同样，git也是作为插件被Jenkins支持的，在 系统管理-->管理插件 中，点击可选插件，通过过滤条件 Git plugin 查询到之后，勾选前面的复选框点击直接安装，Jenkins就可以帮我们装好git插件了。

### 3. 创建持续集成任务

Jenkins的全局设置搞定之后，就可以创建一个持续集成任务了，Item名称随便填，选择构建一个maven项目，在任务配置中配置任务：

* 为了磁盘不被撑爆，勾选 “丢弃旧的构建”，在 保持构建的最大个数中 填5，这个值可以自己根据实际情况确定。
* 在 源码管理 中选择git，在 Repository URL 填上 项目地址，如：http://192.168.9.235:8080/xcloud/idcloud.git ，在 Credentials 中添加你的用户名密码。保证Jenkins可以取到源码。
* 在 Branches to build 中可以选择你想要在哪个分支上build，这里填 "\*/dev"
* 在 构建触发器 中选择 Build periodically，在日程表中可以写 cron 表达式，Jenkins会按你写的 cron表达式定时触发构建任务，这里填 "H 1 * * * " ，表示每天凌晨1点触发构建。同时勾选 Poll SCM ，在 Poll SCM 中也可以写cron表达式，Jenkins会按表达式去检查是否有commits，如果有，触发构建任务。这里填 "H/10 * * * *"，表示每10分钟检查一次。
* 在build里面，可以配置 root pom，这里就填"pom.xml"，意思是项目根目录下的pom.xml。在Goals and options可以填maven的goal和参数，这里填：`-Dmaven.test.skip=true clean package install -P mysql assembly:assembly`

至此，构建任务配置完毕，可以点立即构建来一次构建了，在构建的时候可以点Console Output查看日志，如果构建过程中出错，根据错误提示解决错误。

### 4. 启动脚本编写

构建好的文件在：/jenkins_data_test/jobs/idcloud/workspace/target

* 重启dubbo服务，restart-all-service.sh 内容如下：
```
#!/bin/bash

kill -9 $(ps aux |grep '[t]est-idcloud-core-service'|awk '{print $2}')

SERVICE_HOME=/root/idcloud-service
JENKINS_TARGET=/jekins_data/jobs/idcloud/workspace/target
alias cp='cp'

rm -rf $SERVICE_HOME/*

cp $JENKINS_TARGET/*.gz $SERVICE_HOME
for file in `find $SERVICE_HOME -name "*.gz"`
  do
  dir=${file//.tar.gz/}
  mkdir -p $dir
  tar -zxvf $file -C $dir
done

for file in `find $SERVICE_HOME -name "*.sh"`
  do
  chmod 777 $file
  echo $file
  cd ${file%bin/start-service.sh}
  $file
  #if [[ $file == *"user"* ]] ; then
  #  $file
  #fi
done
```

* 重启tomcat， sh_restart_qomcat.sh 内容如下：
```
#!/bin/bash
tomcat_home=/usr/local/apache-tomcat-8.0.32
if [ "$1" = "restart" ] ; then
  today=`date +"%Y-%m-%d"`
  file=${tomcat_home}/logs/catalina-$today.out
  ps -ef | grep  '[t]omcat' | grep -v cronolog
  kill -9 $(ps -ef | grep  '[t]omcat' | grep -v cronolog |awk '{print $2}' | xargs)


  rm -rf /tomcat_data/idcloud-rest*
  cp /jekins_data/jobs/idcloud/workspace/target/test-idcloud-rest-war/*.war /tomcat_data/idcloud-rest.war

  rm -rf /tomcat_data/idcloud-biz-portal*
  cp /jekins_data/jobs/idcloud/workspace/target/test-idcloud-biz-portal/*.war /tomcat_data/idcloud-biz-portal.war


  rm -rf /usr/local/apache-tomcat-8.0.32/work/Catalina/localhost/*
  cat /dev/null > $file
  sleep 2
  sh -x /usr/local/apache-tomcat-8.0.32/bin/startup.sh
else
  cat /dev/null > $file
fi
```
用法：sh_restart_qomcat.sh restart

* 重启user-portal，user-portal是部署在nginx中，nginx不需要重启，只要把build好的文件拷过来覆盖就行，restart-user-portal.sh 内容如下：
```
#!/bin/bash
alias cp='cp'
cp -R /jekins_data/jobs/idcloud/workspace/idcloud-portal/user-portal /usr/share/nginx/html/
```

### 5. 持续集成

按照之前的配置，Jenkins每10分钟检查一次commit，如果有新commit就触发构建任务，或者每天晚上1点触发构建任务，为了自动化，我们再写crontab让系统每天定时自动发布Jenkins的构建：

`crontab -e` 然后用你习惯的编辑器写入一下内容：
```
0 2 * * * /root/restart-all-service.sh
1 2 * * * /root/sh_restart_qomcat.sh restart
2 2 * * * /root/restart-user-portal.sh
```

表示每天凌晨2:00，2:01， 2:02，自动执行这三个脚本，这样就实现了自动发布。

### 6. 后话

这个持续和集成环境搭建配置过程中还可以更智能，现在是Jenkins 的docker容器配合host主机上的tomcat、nginx、mysql、rabbitmq等完成，后面这些组件都可以做成docker镜像。

现在是Jenkins配合crontab分两步完成构建和发布任务，后面可以合为一部，如使用docker in docker方式，Jenkins构建好之后，使用新构建的代码重新build tomcat镜像，然后推送镜像到registry，再调用脚本用新的镜像重启tomcat等容器完成自动发布。
