---
title: Paragon NTFS For Mac
date: 2020-03-17 18:25:15
tags: Mac
---
MacBook连移动硬盘发现文件复制不进去，拖也拖不进去。。。
<!--more-->

## 1. 原因
学术方面的解释是：因为移动硬盘或 U 盘是使用 Windows 系统下的 NTFS 分区格式，
而 Mac 系统原生是不支持这种格式的，也就是为什么不能向硬盘里拷贝资料的原因。

## 2. 解决
百度到一个方法，链接在参考中，成功的复制了文件。
这边简述一下：
我的设备默认在/Volumes/ever
查看硬盘挂在的节点Device Node: 
``` bash
diskutil info /Volumes/YOUR_NTFS_DISK_NAME
```
查出来是
``` bash
Device Node:    /dev/disk2s1
```

然后将硬盘弹出，但是不要拔掉移动硬盘连接
``` bash
hdiutil eject /Volumes/ever/
```

构建一个目录
``` bash
sudo mkdir /Volumes/ever
```

将NTFS硬盘 挂载 mount 到mac
``` bash
sudo mount_ntfs -o rw,nobrowse /dev/disk2s1 /Volumes/ever/
```

这时候就可以拷贝文件啦，用cp指令就可以了
最后将NTFS硬盘从 mac 上卸载 umount
``` bash
sudo umount /Volumes/ever/
```

发现了另一个一劳永逸的方法
安装Paragon NTFS For Mac。
目前最优秀的NTFS分区驱动软件，安装后相当于装了一个支持读写NTFS格式的驱动程序，以后就可以自由的读写移动硬盘和U盘啦。




## Z. 参考
[1.Mac上挂载移动硬盘出现"Read-only file system"问题](https://www.xiaorongmao.com/blog/49)
[1.macbook为什么无法向移动硬盘复制文件？因为你缺少这个软件！](http://www.360doc.com/content/17/1009/22/912420_693616011.shtml)
