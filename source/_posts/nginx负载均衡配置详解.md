---
title: nginx负载均衡配置详解
date: 2017-03-28 10:55:21
categories: 服务器
tags: nginx
description: nginx负载均衡反向代理配置详解
---

nginx负载均衡反向代理配置详解：
<!--more-->
```
worker_processes  2;

events {   
    worker_connections  1024;	
}


http {
    include       mime.types;
    default_type  application/octet-stream;

	# upstream 配置一组后端服务器，
	# 请求转发到upstream后，nginx按策略将请求指派出某一服务器
	# 即配置用于负载均衡的服务器群信息
	upstream backends {
		#均衡策略
		#none 轮询（权重由weight决定）
		#ip_hash
		#fair
		#url_hash
		
		server 192.168.1.62:8080;
		server 192.168.1.63;
		
		# weight:权重，值越高负载越大；
		# server 192.168.1.64 weight=5;
		
		# backup：备份机，只有非备份机都挂掉了才启用；
		server 192.168.1.64 backup;
		
		# down: 停机标志，不会被访问
		server 192.168.1.65 down;
		
		# max_fails:达到指定次数认为服务器挂掉；
		# fail_timeout:挂掉之后过多久再去测试是否已恢复
		server 192.168.1.66 max_fails=2 fail_timeout=60s;
	}

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }
		
		# 反向代理设置，将所有/proxy_test/路径下请求发给本机上的tomcat
		location /proxy_test/ {
			proxy_pass http://localhost:8080;
		}
		
		# 负载均衡设置，将所有jsp请求发送到upstream backends指定的服务器群上
		location ~ \.jsp$ {
			proxy_pass http://backends;
			
			# 真实的客户端IP
			proxy_set_header   X-Real-IP        $remote_addr; 
			# 请求头中Host信息
			proxy_set_header   Host             $host; 
			# 代理路由信息，此处取IP有安全隐患
			proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
			# 真实的用户访问协议
			proxy_set_header   X-Forwarded-Proto $scheme;
			# 默认值default，
			# 后端response 302时 tomcat header中location的host是http://192.168.1.62:8080
			# 因为tomcat收到的请求是nginx发过去的, nginx发起的请求url host是http://192.168.1.62:8080
			# 设置为default后，nginx自动把响应头中location host部分替换成当前用户请求的host部分
			# 网上很多教程将此值设置成 off，禁用了替换，
			# 这样用户浏览器收到302后跳到http://192.168.1.62:8080，直接将后端服务器暴露给浏览器
			# 所以除非特殊需要，不要设置这种画蛇添足的配置
			proxy_redirect default;
		}
		
		# 一个url重写的例子，浏览器请求 /page.go时，url被重写成/test/page.jsp
		location ~ \.go$ {
			rewrite ^(.*)\.go$ /test/$1\.jsp last;
		}

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```
