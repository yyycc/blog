---
title: ssh免密钥登录
date: 2019-10-21 09:14:07
tags: linux
---
大清早，先偷会儿懒，把昨天弄好的ssh免密钥登录记录一哈，免得过两天忘记了。
<!--more-->
### 先做一哈背景介绍
我是centos 7 的服务器，在vultr上搭建的，一开始搭的时候我就选择了之前生成的SSH Keys。
后来想实现免密登录，发现本地私钥找不到了。。。
所以就得新生成一个，把之前的SSH key换掉。
具体肿么实现呢：

### Step 1 配置文件
进入服务器
``` bash
$ vi /etc/ssh/sshd_config
```

找到以下内容并且把注释符 # 删掉：
``` 
RSAAuthentication yes 
PubkeyAuthentication yes 
AuthorizedKeysFile .ssh/authorized_keys 
```
然后重启sshd服务：

``` bash
$ /sbin/service sshd restart 
```

### Step 2 生成ssh key
因为我原来就有ssh，所以～目录下有.ssh文件夹，里面有authorized_keys文件，内容是以ssh-rsa打头的公钥，将其备份一下(其实不需要了可以删掉)。
生成新的ssh key：
``` bash
$ ssh-keygen -t rsa 
```

之后的提示都直接回车，默认会在～/.ssh目录下生成id_rsa私钥、id_rsa.pub公钥两个文件,known_hosts文件会记录ssh密钥登陆的主机列表。

### Step 3 将公钥导入认证文件
之前，我们将 AuthorizedKeysFile .ssh/authorized_keys 这句话启用，就是将.ssh目录下的authorized_keys作为认证文件，
现在我们换了新的ssh key，就把新生成的id_rsa.pub公钥复制到.ssh目录下，并改名为authorized_keys。

### Step 4 本地配置
将密钥从服务器复制到本地

``` bash
$ scp root@ever:~/.ssh/id_rsa /usr/local/aliyun/SSH-ever-vultr-private
```

在本地～目录下新建.ssh文件夹，进入该文件夹，新建config文件，写入这段配置：

```
Host ever // 自定义服务名
HostName ip地址
Port 22
User root  //登录用户
IdentityFile    /usr/local/aliyun/SSH-ever-vultr-private   //本地密钥
```
然后 ssh ever 就可以免密登录我的服务器啦！

ps 如果不免密登录，也可以在config里面配置除了IdentityFile以外的属性，登录的时候就可以不用写ip和登录用户啦。

pps 百度的时候看到了用ssh-copy-id也可以实现免密登录，打算再搭一个服务器试一下，先去工作啦～～

