---
title: docker-1
date: 2019-11-08 11:45:44
tags: linux
---
今天早上，um。。。又是无所事事的一个早上，研究了一些乱七八糟的东西，研着研着就到了docker
<!--more-->
自从装了docker之后(小哥哥装的。。。)，每天打开电脑第一件事就是
``` bash
$ docker ps -a          //查看所有的容器
$ docker ps             //查看所有运行的容器
// 后来变成了
$ docker container ls   //查看所有运行的容器

$ docker start oracle
$ docker container ls
```
启动oracle数据库，之后就不理睬docker了。
还有需要导入数据库的时候
``` bash
$ ssh root@127.0.0.1 -p 49160
$ password
```
然后执行一系列oracle命令
从来也没有仔细研究过docker，这样一想，还是蛮对不起这只可爱的小鲸鱼🐳的。
然而今天我连上docker之后，突然看到了这句话(以前一直没注意，我是瞎么)
Welcome to Ubuntu 14.04.1 LTS
哇，我家docker的操作系统是Ubuntu啊,怪不得我一直应ssh连接。。。

(补充一下[2019-11-11]：上面说docker的操作系统是Ubuntu是不对的，docker是一个容器引擎，它不是模拟一个完整的操作系统。
所以我的docker中只有一个容器：oracle，只不过这个oracle是基于Ubuntu环境的，这是由image制作者决定的，
而我能够用ssh访问容器是因为，制作者将这个功能开放出来了，不然的话，就需要用 docker exec -it oracle /bin/bash，
在容器中运行command，来访问容器了。所以其实这篇日志跟docker关系不大。。。
本来我以为docker带操作系统，我要运行tomcat还要装JDK，但其实操作系统、tomcat、JDK等等全都封装再一个镜像中，
运行后就是一个独立的容器，跟原有的oracle容器是完全隔离互不影响的。)

所以我会什么一直要用这么一长串鬼东西登录呢，还要输密码，我的ssh免密钥登录日志是白写的么
``` bash
ssh root@127.0.0.1 -p 49160
```
于是，实战开始。
首先是配置文件(/etc/ssh/sshd_config),看了一下只需要把AuthorizedKeysFile属性的注释符去掉就行了，这边看到他是这样的
```
#AuthorizedKeysFile      %h/.ssh/authorized_keys
```
百度了一下，%h就相当于～
这边我用vi想去掉注释的时候发现删除键以及上下左右键都不听话，网上说用vim，
于是我就
``` bash
$ apt-get update
$ apt-get install vim
```
经历了漫长的等待，终于装好了。。。
去掉注释之后，重启一下ssh服务
``` bash
$ /etc/init.d/ssh restart
```
之后ssh连接就断开了，直接再连也连不上，我重启了一下docker就连上了
之后就是公钥和密钥了，进入/etc/ssh目录，我看到好多对公钥和密钥，我就选了一对，把公钥放到～/.ssh目录，重命名为authorized_keys，
把私钥copy到我本地，再在本地~/.ssh目录的config里面配置一下。这边端口配49160。
ssh的端口是22，我这边docker映射出来的是49160。
``` bash
$ ssh doc
```
就直接登录啦，美滋滋～～

这边提一下docker容器和本地复制文件的命令
``` bash
$ docker cp oracle:/etc/ssh/ssh_host_rsa_key /usr/local/ssh_host_rsa_key
```
这里的oracle是我的容器的名字

























