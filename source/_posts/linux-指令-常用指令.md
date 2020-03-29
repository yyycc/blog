---
title: linux 指令 -- 常用指令
date: 2020-01-13 15:27:19
tags: linux
---
记录一下linux中常用的指令
<!--more-->

## 1. netstat
查看端口占用情况
<font color=#FF0000> netstat -anp|grep 4000 </font>

## 2. find
在 当前目录下，查找所有内容包含‘abc’的文件
<font color=#FF0000> find . -type f | xargs grep -l 'abc' </font>

在当前目录下，查找所有文件名以my开头的文件
<font color=#FF0000> find . -name "my*" </font>

在当前文件夹下所有文件中(*)找包含'指定的内容'的文件输出文件名
<font color=#FF0000> grep -rnRi 指定的内容 * | awk -F":" '{print $1}' </font>

## 3. env
看环境变量

## 4. whereis
搜索程序名

## 5. which
搜索系统命令

## 6. lsnrctl
oracle中设置监听器指令 +[status]/[start]/[stop]

## 7. tar
打成tar包
<font color=#FF0000> tar -cvf cyy.tar cyy </font>
解压tar包
<font color=#FF0000> tar -xvf cyy.tar  </font>

如果要操作.tar.gz的压缩包，就多加一个z指令

## 8. >
将指令的输出放进文件
git config --list > config
当前目录就会出现config文件，里面是git配置内容

## 9. echo
输出cyy
echo cyy
输出黄底 红色的 ever
<font color=#FF0000> echo -e "\033[43;31mever\033[0m"  </font>


## z. 参考
[1. Linux xargs 命令](https://www.runoob.com/linux/linux-comm-xargs.html)
