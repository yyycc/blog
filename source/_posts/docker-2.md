---
title: docker-2
date: 2019-11-11 10:20:05
tags: linux
---

放在同一个日志里就太长了，所以分两个日志写吧，接[docker-1](http://localhost:4000/2019/11/08/docker/)
<!--more-->
接下来我想在里面装一个tomcat，把我的axis服务部署在上面
可以到docker hub上看tomcat的镜像，冒号后面的是他的tag(如果不写tag就默认latest，如果没有latest就会报错)
``` bash
$ docker pull tomcat:jdk8-adoptopenjdk-openj9
```
拉了茫茫久，终于拉好了，接下来就是用镜像来运行容器啦
``` bash
$ docker run -d --name tomcat -p 7080:7080 -p 8090:8090 8e6064eb6bcd
```
这个 
第一次我想映射7070，但我本地正在用7070端口，所以报错了，但是容器还是有了，所以先删了容器,然后再运行一次
``` bash
$ docker rm  <container ID/name>   // 加不加container都可以
$ docker container rm  <container ID/name>
```
删了之后，再运行，再用命令再看一下，果然有了一个有端口映射的tomcat容器
``` bash
$ docker ps -a
```
进入tomcat容器
``` bash
$ docker exec -it tomcat /bin/bash
```
失败了。。。指令没有任何效果
呜
重来

经过学习和探讨
拉了某个镜像，就要去看这个镜像发布者提供的使用方法，自己猜是猜不到的
所以就去看。

这个tomcat默认的启动端口是8080，所以我吧7080和8090映射出来有啥用呢。。。哎
``` bash
$ docker run -it -d -p8888:8080 tomcat
```
容器启动之后，在本地用 http://localhost:8888 就可以访问啦，经典的tomcat页面呀
这个命令是尝试之后最终的命令，跟它提供的不一样，主要是参数不同
加了 -d 表示容器在后台运行，不然你执行完，就会在控制台输出一堆日志， ctrl + c结束掉之后，tomcat也停掉了
去掉了 --rm 这个参数加了表示容器停止后自动删除容器，他跟 -d 互斥
再提一下 -it 也就是 -i 加上 -t，分别指用于控制台交互和支持终端登录
然后再用exec指令
``` bash
$ docker exec -it tomcat /bin/bash
```
成功地进入容器打开终端交互啦
根据它提供的路径，进入webapps把我的war放进去,然后重启一下
``` bash
$ docker restart tomcat
```
再访问一下，可以成功访问到我的服务了，棒！

