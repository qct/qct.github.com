---
layout: post
title: "Tomcat Loadbalance 搭建"
description: "先说下系统的架构，两台nginx服务器做HA，反向代理到后面两台tomcat做loadbalance。   "
category: 技术
tags: [linux, tomcat, java, loadbalance]
---
{% include JB/setup %}

先说下系统的架构，两台nginx服务器做HA，反向代理到后面两台tomcat做loadbalance。   

### *1. 配置文件 /etc/nginx/nginx.conf：   *


        user  nginx;
        worker_processes  8;
        
        error_log  /var/log/nginx/error.log warn;
        pid        /var/run/nginx.pid;
        
        
        events {
        	worker_connections  1024;
        }
        
        
        http {
        	include       /etc/nginx/mime.types;
        	default_type  application/octet-stream;
        
        	log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        					  '$status $body_bytes_sent "$http_referer" '
        					  '"$http_user_agent" "$remote_addr" "$http_x_forwarded_for"';
        
        	access_log  /var/log/nginx/access.log  main;

		sendfile        on;
		#tcp_nopush     on;

		keepalive_timeout  65;

		#gzip  on;

		upstream tomcatserver {
			#根据ip计算将请求分配各那个后端tomcat,许多人误认为可以解决session问题,其实并不能.
			#同一机器在多网情况下,路由切换,ip可能不同
			server  10.22.205.121:8080;
			server  10.22.205.122:8080;

			#根据IP做分配策略
			ip_hash;

			#down 表示单前的server暂时不参与负载
			#weight  默认为1.weight越大，负载的权重就越大。
			#max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误
			#fail_timeout:max_fails 次失败后，暂停的时间。
			#backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

			#nginx 的 upstream目前支持 4 种方式的分配
			#1)、轮询（默认） 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
			#2)、weight 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
			#2)、ip_hash 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
			#3)、fair（第三方）按后端服务器的响应时间来分配请求，响应时间短的优先分配。
			#4)、url_hash（第三方）
		}
		upstream pythonserver {
			server  10.22.205.121:5001;
			server  10.22.205.122:5001;
			ip_hash;
		}

		include /etc/nginx/conf.d/*.conf;
        }
		   
### *2. 配置文件 /etc/nginx/conf.d/virtual.conf：   *

        #
        # A virtual host using mix of IP-, name-, and port-based configuration
        #
        
        #server {
        #    listen       8000;
        #    listen       somename:8080;
        #    server_name  somename  alias  another.alias;
        
        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}
        
        
        server {
        	listen       80;
        	server_name xxx.com;
        
        	#error_page 405 =200 @405;
		#location @405 {
		#    proxy_pass http://10.22.205.102:8080;
		#}

		#rewrite "^/xxx-portal$" http://www.xxx.com/ redirect;
		if ( $request_filename ~ /xxx-portal ) {
		   rewrite ^ http://www.xxx.com permanent;
		}
		location /{
			root   /home/qct/project/xxx-portal;
			index  index.html index.htm;
		}

		location ~* /xxx-(platform|ws|storage-ws|mobile-ws)|pic_mount|msp-web{
		#location /{
					#proxy_pass http://10.22.205.102:8080;
			#proxy_connect_timeout 60s;
			#proxy_read_timeout 5400s;
			#proxy_send_timeout 5400s;
			proxy_pass                  http://tomcatserver;
			proxy_redirect              off;
			proxy_set_header            Host $host;
			proxy_set_header            X-Real-IP $remote_addr;
			proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
			client_max_body_size        10m;    #允许客户端请求的最大单文件字节数
			client_body_buffer_size     128k;   #缓冲区代理缓冲用户端请求的最大字节数，
			proxy_connect_timeout       90;     #nginx跟后端服务器连接超时时间(代理连接超时)
			proxy_send_timeout          5400s;  #后端服务器数据回传时间(代理发送超时)
			proxy_read_timeout          5400s;  #连接成功后，后端服务器响应时间(代理接收超时)
			proxy_buffer_size           4k;     #设置代理服务器（nginx）保存用户头信息的缓冲区大小
			proxy_buffers               4 32k;  #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
			proxy_busy_buffers_size     64k;    #高负荷下缓冲大小（proxy_buffers*2）
			proxy_temp_file_write_size  64k;    #设定缓存文件夹大小，大于这个值，将从upstream服务器传
		}
        }
