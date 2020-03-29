---
title: linux 指令 -- 常用指令扩展(vi、cp、tail)
date: 2020-01-07 10:26:36
tags: linux
---

linux指令万万千，用一个百度一个，百度一个忘一个。。。
<!--more-->
介绍一下平时最常用到的几个指令～～

## 1. vi 文书编辑器
在我本地，编辑器一般用sublime，图形化界面好操作，但是服务器上一般用不了。
最近项目上访问服务器都要用云桌面，Xshell又回到了我的身边。

指令|功能
--|:--:
vi + [file_name]|进入编辑器
i|进入编辑状态
esc|退出编辑状态
:q!|不保存退出
:wq|保存退出
**上面的都是废话**|**凑凑字数什么的**
<font color=#FF0000> **跳转** </font>|
G|跳到最后一行
gg|跳到第一行
$|跳到行尾
0 / ^|跳到行首
ngg|跳到第n行
<font color=#FF0000> **删除** </font>|
dd|删除光标所在行
ndd|删除包括光标所在行以下n行
d + $|删除光标所在处到行尾的所有数据
d + 0|删除光标所在处到行首的所有数据
<font color=#FF0000> **行号** </font>|
:set number/nu|显示行号
:set nonumber/nonu|不显示行号
<font color=#FF0000> **查找** </font>|
/ + [key] + 回车|查找key
n|查看下一个匹配
shift + n|查看上一个匹配
<font color=#FF0000> **more** </font>|
u|复原前一个动作
.|重复前一个动作

(加班到10点。。。再写一个指令慰劳一下辛苦的自己)
## 2. cp 复制
这个指令也是非常常用的了呀。
为了直观，我在本地建了两个文件夹作为测试案例 
~/lasting/cp-test-1 (ONE)  
~/lasting/cp-test-2 (TWO)

指令|功能
--|:--:
**基础的指令**|**复制文件**
cp ~/lasting/cp-test-1/a ~/lasting/cp-test-2(/another_name)|把ONE下的a复制到TWO下，括号里是给定复制出来的文件的名字，默认值是源文件文件名
<font color=#FF0000> **参数** </font>|
-i|覆盖已经存在的目标文件时，给出提示(我本地不加参数时默认是不给提示的)
-f|覆盖已经存在的目标文件而不给出提示
-r|若给出的源文件是一个目录文件，此时将复制该目录下所有的子目录和文件
**-r**|**复制文件夹**
cp -r ~/lasting/cp-test-1 ~/lasting/cp-test-2(/)|将ONE复制到TWO下
cp -r ~/lasting/cp-test-1/ ~/lasting/cp-test-2(/)|将ONE下的所有复制到TWO下
**相关指令**|**scp/docker cp**
scp local_file remote_username@remote_ip:remote_file
<font color=#FF0000> **主要就是加上了 远程服务器/容器名 再加上 冒号** </font>|
scp ~/lasting/cp-test-1/a  root:/usr/local|root是配置了的远程linux服务器
docker cp ~/lasting/cp-test-1/a  oracle:/usr/local|oracle是docker容器名字

## 3. tail
这个指令通常用于在启动tomcat的时候看日志
tail -f ./logs/catalina.out 

指令|功能
--|:--:
tail -f file|当某些行添加至文件时，会继续显示这些行
tail -c 10 file|显示file内容，最后10个字符
tail +10 file|显示file内容，从第十行至末尾
tail -n -10 file|显示file内容，从第十行至末尾
