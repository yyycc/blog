---
title: nginx
date: 2019-10-17 11:32:50
tags: 服务
---
那些年我用nginx做过的事
<!--more-->
## 一、搭建nginx
搭建nginx的方法很多，都很easy。。。

### 1. mac
我的机子是mac的，我本地装了homebrew，所以我本地的nginx使用homebrew安装的
``` bash
$ brew install nginx
```
安装好就可以启动啦(~~status~~)
``` bash
$ brew services start|restart|stop|status nginx
```
用 http://localhost:8080 就可以访问
nginx -t 检查配置文件语法，他会返回给你配置文件的目录结构的
``` bash
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```

### 2. centos
centos 7的服务器上搭建nginx
``` bash
$ yum install -y nginx
```

### 3. RedHat 6(离线安装nginx)
本来也想用yum的，奈何这个服务器不通外网。。。我q
所以我就把安装包下载到我本地，然后sftp到服务器上
nginx的安装包不止nginx，他还有很多依赖模块，包括pcre、zlib、openssl 
```
nginx-1.14.2.tar.gz    
openssl-1.1.1a.tar.gz  
pcre-8.42.tar.gz       
zlib-1.2.11.tar.gz
```
这些都下下来，解压，进入文件目录
``` bash
$ ./configure  //有的是需要带参数，这里不详了
$ make
$ install
```
/usr/local/nginx/sbin/nginx -t 检查配置文件语法
/usr/local/nginx/sbin/nginx    启动nginx
这个跟用yum安装是一样的
可以配置成系统服务启动 
``` bash
service nginx start
```

## 二、nginx部署前端项目
安装完了，就来说使用吧，nginx搭建非常的容易，用他来实现各种代理，主要就是写配置文件nginx.conf
我的blog就是部署在nginx上的，之前搭的vue项目，也部署在nginx上，这样我们就不需要在控制台运行项目啦。
配置文件主要是这么一个结构
这个配置问题，一定记得加分号，每次都是因为忘记;然后就报错。。。
``` list
#user  nobody;
worker_processes  1;

# error_log  logs/error.log;
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    server {
        listen       8092;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  cyy.html ;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            include        fastcgi_params;
        }
    }
    include servers/*;
}
``` 
listen是监听的端口，配多个server就可以监听多个端口，server_name是服务名，可以随意取。
我本地的nginx是监听了8091端口，部署了blog，8092端口会显示cyy.html。配置分别如下
``` bash
location / {
            alias html/public/;
            index index.html;
        }
``` 
``` bash
location / {
            root   html;
            index  cyy.html ;
        }
``` 
alias和root后面都是目录，我blog的index.html，就在nginx(/usr/local/Cellar/nginx/1.15.8/html)html/public目录下,
cyy.html是我自己新建的就在html下，这个文件夹里面一开始就会生成50x.html、index.html，刚安装好访问的时候就是访问这个index.html。

之前还在服务器上用nginx部署过vue服务
## 三、转发
之前项目上有一个任务，是调接口的时候要做一个转发，因为生产服务器在内网，访问不了外网，要调用的服务在外网。
需要从生产先发请求到中间区域，从中间区域接受请求转发到外网服务。
``` bash
location / {
            if ($request_uri ~* "/cyy") {
                proxy_pass http://ip:8070;
                break;
            }
        }
``` 
根据请求路径内容匹配，如果匹配成功，就转发到外网ip，请求路径和参数都不变。
