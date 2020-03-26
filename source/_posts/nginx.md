---
title: nginx
date: 2020-03-26 19:58:10
tags: 软件
---
# Nginx配置反向代理的一些坑
* 目标：做一个一机多tomcat模拟集群，使用nginx做反向代理到本机不同的端口上
* 反向代理的网址配置：abc11.com
## 具体步骤
1. 将多个tomcat程序拷贝到文件夹并配置环境变量，配置tomcat集群有以下几个要点：
   1. 修改server.xml文件，把端口区分开来（如：tomcat1是8080，tomcat2就不能是8080了）
   2. 将不同的tomcat环境变量区分开来
      * linux 在/etc/profile，windows在高级系统设置中
      * 添加不同名字的环境变量
        * windows：将CATALINA_HOME 和 CATALINA_BASE 改成对应的环境变量
        * linux: linux中直接在catalina.sh中将CATALINA_BASE，CATALINA_HOME赋值成你的环境变量
2. 使用nginx反向代理
   1. nginx.conf中配置
		```
		worker_processes  1;
		events {
			worker_connections  1024;
		}


		http {
			include       mime.types;
			default_type  application/octet-stream;
			sendfile        on;
			keepalive_timeout  65;

			server {
				listen       80;
				server_name  localhost;
				location / {
					root   html;
					index  index.html index.htm;
				}
			}
			# 我们将要反向代理的配置文件放在vhost/下
			# 这样做的好处就是目录结构清晰，nginx配置文件和反向代理配置文件分离
			include vhost/*.conf; 
		}
		```
	2. vhost中的配置
		```
		upstream www.imooc11.com {
			server 127.0.0.1:8080;
			server 127.0.0.1:9080;
		}

		server {
			listen 80;
			autoindex on;
			server_name www.imooc11.com imooc11.com;
			#access_log /usr/local/nginx/logs/access.log combined;
			index index.html index.htm index.jsp index.php;
			if ( $query_string ~* ".*[\;'\<\>].*" ){
					return 404;
			}

			location / {
				proxy_pass http://www.imooc11.com/;
			}

			location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|ico)$ {
				proxy_pass http://www.imooc11.com;
				expires 30d;
			}

			location ~ .*\.(js|css)?$ {
				proxy_pass http://www.imooc11.com;
				expires 7d;
			}
		}
		```
		注意这里的坑是最大的，文件目录千万不可以写错。  
		如果写错了，日志不会提醒你，而且反向代理不会成功，我搞了几个小时才在linux服务器下发现问题。  
		然后把windows下的nginx改成功了。  
		下面是我之前的错误配置文件：
		```
		upstream www.imooc11.com {
			server 127.0.0.1:8080;
			server 127.0.0.1:9080;
		}

		server {
			listen 80;
			autoindex on;
			server_name www.imooc11.com imooc11.com;

			# 这里错误，windows下没有这样的目录
			access_log /usr/local/nginx/logs/access.log combined;
			index index.html index.htm index.jsp index.php;
			if ( $query_string ~* ".*[\;'\<\>].*" ){
				return 404;
			}

			location = / {
				# 这里错误，没有这样的目录
				root /product/dist/view;
				index index.html;
			}

			location ~ .*\.html$ {
				# 这里错误
				root /product/front/mmall_fe/dist/view;
				index index.html;
			}

			location / {
				proxy_pass http://www.imooc11.com/;
			}

			location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|ico)$ {
				proxy_pass http://www.imooc11.com;
				expires 30d;
			}

			location ~ .*\.(js|css)?$ {
				proxy_pass http://www.imooc11.com;
				expires 7d;
			}
		}
		```
		所以还是要把别人的配置文件想清楚再用，没日志记录是最糟糕的。  
		希望以后不要再碰到这种。。。。。
	3. 修改host文件
		* windows: Windows/System32/driver/etc/hosts中添加
		* linux: /etc/hosts中添加
			```
			127.0.0.1 www.imooc11.com
			```

文章到这里，反向代理配置就成功了。  
如果需要添加新的tomcat服务器，在vhost下继续添加相应的配置即可。  
还有如服务器权重配置参数 wight ...等都在upstream那块设置。  
nginx不愧是妈妈桑。		