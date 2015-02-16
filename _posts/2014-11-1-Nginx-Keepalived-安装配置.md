---
layout: post
title: "Nginx Keepalived 安装配置"
description: "先说下系统的架构，两台nginx服务器做HA，反向代理到后面两台tomcat做loadbalance。   "
category: 技术
tags: [linux, tomcat, java, loadbalance]
---
{% include JB/setup %}

## 安装keepalived

·yum install -y gcc openssl-devel popt-devel libnl-devel kernel-devel·      

    wget http://www.keepalived.org/software/keepalived-1.2.2.tar.gz
    tar xzf keepalived-1.2.2.tar.gz
    cd /tmp/keepalived-1.2.2
    ./configure --with-kernel-dir=/usr/src/kernels/2.6.32-431.29.2.el6.x86_64
    make && make install
    cp /usr/local/etc/rc.d/init.d/keepalived /etc/init.d/
    cp /usr/local/etc/sysconfig/keepalived /etc/sysconfig/
    chmod +x /etc/init.d/keepalived
    chkconfig --add keepalived
    chkconfig keepalived on
    mkdir /etc/keepalived 
    ln -s /usr/local/sbin/keepalived /usr/sbin/

## keepalived 配置,master_IP: 10.22.205.111, slave_IP:10.22.205.112, VIP:10.22.205.110

**master:/etc/keepalived/keepalived.conf**    

    global_defs {
       notification_email {
         277620233@qq.com
       }
       notification_email_from keepalived@loongrender.com
       smtp_server 10.22.205.102
       smtp_connect_timeout 30
       router_id LVS_DEVEL
    }
    vrrp_script chk_http_port {
                    script "/opt/nginx_pid.sh"
                    interval 2
                    weight 2
    }
    vrrp_instance VI_1 {
        state MASTER        ############ 辅机为 BACKUP
        interface eth0
        virtual_router_id 51
        mcast_src_ip 10.22.205.111
        priority 102                  ########### 权值要比 back 高
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 1111
        }
    track_script {
            chk_http_port ### 执行监控的服务
            }
        virtual_ipaddress {
           10.22.205.110
        }
    }
		   
**slave:/etc/keepalived/keepalived.conf**    

    global_defs {
       notification_email {
         277620233@qq.com
       }
       notification_email_from keepalived@loongrender.com
       smtp_server 10.22.205.102
       smtp_connect_timeout 30
       router_id LVS_DEVEL
    }
    vrrp_script chk_http_port {
                    script "/opt/nginx_pid.sh"
                    interval 2
                    weight 2
    }
    vrrp_instance VI_1 {
        state BACKUP        ############ 辅机为 BACKUP
        interface eth0
        virtual_router_id 51
        mcast_src_ip 10.22.205.112
        priority 101                   ##########权值 要比 master 低。。
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 1111
        }
    track_script {
            chk_http_port ### 执行监控的服务
            }
        virtual_ipaddress {
           10.22.205.110
        }
    }


**监控脚本,主从都要**   

    vi /opt/nginx_pid.sh
    #!/bin/bash
    A=`ps -C nginx --no-header |wc -l`
    if [ $A -eq 0 ];then
       /usr/sbin/nginx
       sleep 3
       if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
              killall keepalived
       fi
    fi