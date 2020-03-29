---
title: mysql安装
date: 2019-10-22 16:00:30
tags: 数据库
---
好久都没用mysql了，赶紧肥来看看
<!--more-->

## centos 7的服务器下安装mysql
下载mysql源安装包
``` bash
$ wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
```
安装mysql源
``` bash
$ yum localinstall mysql57-community-release-el7-8.noarch.rpm 
```
安装mysql
``` bash
$ yum install mysql-community-server
```
启动MySQL服务并设置开机启动
``` bash
$ systemctl start mysqld
$ systemctl enable mysqld
$ systemctl daemon-reload
```
到此已经安装好啦可以通过 mysql -u -p 访问，输入密码就行啦
``` bash
$ mysql -u root -p
```
当然刚安装好，我门不知道初始密码是啥，可以通过下面的命令查看
``` bash
$ grep 'temporary password' /var/log/mysqld.log
```
登录后先改root密码，初始化的密码要求规则比较复杂，可以先更改一下
``` bash
$ set global validate_password_policy=0;
```
修改密码
``` bash
$ ALTER USER 'root'@'localhost' IDENTIFIED BY '050511';
or
$ set password for 'root'@'localhost'=password('ever050511'); 
```
@'localhost'的用户是只支持本地访问的，我们添加一个支持远程访问的用户
``` bash
$ GRANT ALL PRIVILEGES ON *.* TO 'ever'@'%' IDENTIFIED BY 'ever050511' WITH GRANT OPTION;
```
创建一个数据库试一哈
``` bash
$ CREATE DATABASE IF NOT EXISTS last DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```
远程连接
``` bash
$ mysql -u ever -h **.**.***.* -p
```
如果访问不了，那就是端口没开放，mysql默认端口是3306
``` bash
$ firewall-cmd --query-port=3306/tcp                //  查看端口是否开放
$ firewall-cmd --add-port=3306/tcp --permanent      //  开放端口
$ firewall-cmd --reload                             //  重载入添加的端口
$ firewall-cmd --permanent --remove-port=3306/tcp   //  移除指定端口
```

