---
title: git
date: 2019-10-18 15:15:55
tags: 版本控制
---
git status git status git status git status git status 
<!--more-->
## 1. I was just mumbling. Skip it please.
自从开始工作，就开始用git，我记得第一个国庆的时候的任务，就是看git，还记得当时看的是廖雪峰老师的教程。
当时基本是啥都不懂，就看个热闹，把看到的指令记录一下就觉得学会了，还觉得很简单。。。
之后在工作上，用的是idea，基本不敲指令(就git clone拉一下项目)，都是图形化界面，每天也就拉拉代码，提提代码，有时候还会把别人的代码覆盖了，
也不知道是什么原因，就被告知要先pull再push。。。
后来上项目了跟着一个小姐姐，她用git bash和git gui，然后我也跟着用。
再后来到了第二个项目开始知道还有merge呀stash等等，当然知道的多了，犯错的可能性就更大了。
当时我们是多分支开发的，有一次我用master merge我的本地分支，但我的本地分支并不是最新的，导致好多提交都丢失了，经过了一上午才修复。
那之后我一般就先本地merge master再push，再在master下merge远程分支了。。。
再后来买了git的课程看，了解到它是分布式版本管理(*Distributed Version Control Systems*)，本地就可以是一个完整的仓库，没有远程也可以正常工作等等虽然是分布式的，但不是去中心化的，每个分布还是需要通过中心进行代码交换的。
但是中心宕机不会影响本地，本地还是可以继续开发，等中心修复再push就可以了。
后来小哥哥又给我讲了很多，基本上可以覆盖项目所需了。
最近开始写blog，git总不能落下，好好梳理一下记录下来吧(我觉得这将会是搭blog以来，最长的一篇文了)。

## 2. preparation
环境：macos
git版本：2.20.1
编辑器：sublime
diff工具：p4merge
tips：
   git有一个特别好的地方，就是当你打了个指令，出错了，它会告诉你出错了、为什么出错以及可能的解决指令，一般你执行一下它给你指令，就能解决问题。
   git中，任何已提交的东西几乎都可以被找恢复，而未提交的丢失后很可能再也找不到了。
   另外，当你执行一些rebase呀，merge，reset包括pull的时候，最好先保证本地工作目录是干净的，可以用<font color=#DC143C>stash</font>暂存，这样最安全，报错的可能性也会小一些。

## 3. config
安装就不说了，直接从配置开始吧。
为什么开头要写那么多遍git status呢，因为这真的是git所有指令里(well，我用到的指令里)最难打的一个了。。。
然而其实通过配置，我完全可以用一个字母替代他。。。两年了，想到这个不觉有些心酸💔
前面提到git是分布式的，每个本地都可以作为一个服务器，实现完整的版本管理。

比如在我的laptop上，我就可以在目录a建一个远程裸仓库A，在目录b分支master开发，在目录c分支dev开发，然后都push到A，从A再pull。
这个b和c就是两个local，local级别的配置优先级最高，用下面的命令，就可以看到不同本地仓库的配置。
(注：这个--local只能在git仓库中执行，如果没有仓库就会报错。)
```BASH
$ git config --list --local
```
目录b和目录c可能是由不同的登录用户操作的，所以git还有global(用户)级别的配置，他的优先级低于local。
```BASH
$ git config --list --global
```
此外还有系统级别的配置system，对所有登录用户都有效，优先级最低。
```BASH
$ git config --list --system
```
除了用命令，还可以直接查看配置文件，目录如下：

配置级别|配置文件目录|备注
--|:--:|--:
local|.git/config|在本地仓库路径下
global|~/.gitconfig|macos: /Users/ever      centos: /home/ever
system|/usr/local/etc/gitconfig|我的路径是这样的，不同的git安装目录，配置文件目录也不同，可以通过which git先看一下git安装目录

### 3.1. user
先说最重要的用户配置吧，这个是必须的，不然之后commit会报错，让你去配置，因为每一次commit都会用到这些信息，并且这些信息会写入commit中，不可更改。一般配global，如果你这个仓库需要不同的用户，就用local。
```BASH
$ git config --global user.name 'ever'
$ git config --global user.email '89******2@qq.com'
```
邮箱配你在用的，这样你的代码有啥问题，就可以发邮件告诉你了。

### 3.2. core
这里可以<font color=#FF8C00>配置编辑器</font>，我的git默认的编辑器是vim，我改成了 sublime。
```BASH
$ git config --global core.editor 'subl -w'
```
当需要你编辑信息的时候，就会自动打开<font color=#FF8C00> sublime </font>，-w表示他会等你编辑好保存关闭之后再执行指令，比如
```BASH
$ git commit
```
就会打开sublime，等你保存好关掉就提交了，如果不加-w，会打开sublime，但是直接报错回滚commit了。

core.autocrlf=input // todo
### 3.3. alias
别名，这是个好东西呀，那些又臭又长的指令，都可以简化，举个例子。
```BASH
$ git config --global alias.ca 'commit --amend'
$ git config --global alias.l "log --graph --all --pretty=format:'%C(yellow)%h%C(cyan)%d%Creset %s %C(white)- %an, %ar%Creset'" //里面有单引号所以外面用双引号
```
### 3.4. diff & merge

## 4. command
git的指令千千万，常用的也就那么几个，可以用git help看一下，每个指令的用处都可以看到，这个离线就可以看，非常方便。
你还可以git help -a 一下，所有的指令都可以看到。
这边的东西确实非常多，我竟不知道该用一个怎样的顺序来介绍。

### 4.1. git init
那就先用我这个blog项目作为实验对象，用版本管理起来吧。

其实，之前在搭建的时候按照教程已经用了git，还上传了git hub，但之后push主题的时候一直报错。。。好像不支持我改配置。。。就很难受
但是，我这个要放到服务器上去的，每次都scp的话太麻烦了，所以还是得传到git hub上，到时候就可以用jenkins部署启动啦。
到需要git管理的目录下，我这边是 */Users/ever/lasting/ever-blog/public*，初始化git仓库。
```BASH
$ git init
```
你会看到多了一个.git目录,之前提到的config配置文件就在这个目录里面，还有很多文件，文件夹，具体都是些啥，我们先不看。
init之后，你就可以在这个目录下执行git的众多指令啦。

### 4.2. git add
当然init只是初始化，所有的文件还没有被git管理起来。用<font color=#00FFFF> **git add** </font>就可以实现对文件的跟踪啦。
```BASH
$ git add [file_name]  将file_name添加到暂存区
$ git add -u           将已被git管理过的又修改了的文件添加到暂存区  
$ git add .            将当前目录下的所有文件添加到暂存区
$ git add -A           将所有文件添加到暂存区
```
### 4.3. git commit
再用<font color=#00FFFF> **git commit** </font>将文件提交到本地仓库
```BASH
$ git commit    // 会自动打开你配置好的编辑器
$ git commit -m 'writing your commit message here'
```
这边提一句git commit -am，省略了git add步骤，可以直接提交，但是如果是从未被git跟踪过的文件，是不行的，只有git add过的才行。

### 4.4. 关于四个去和4种状态
上边提到了暂存区、本地仓库，就涉及到了git的四个区：
工作区(workspace)、暂存区(Index/Stage)、本地仓库(Repository)、远程仓库(Remote)

![图1. git四大区](git四大区.jpg)

git中的文件也有多种状态：
未跟踪(Untracked)、已入库未修改(Unmodified)、已修改(Modified)、暂存(Staged)、已提交(Committed)

![图2. git文件四种状态](git文件四种状态.png)

![图3. 状态转变](状态转变.jpeg)

实际操作一下，就一目了然啦～
这边还没有远程仓库，就先到本地仓库为止。

目前工作目录有文件index，未跟踪状态进行如下操作

序号|状态(区)|指令/操作|状态(区)
--|:--:|--:|--:
1|Untracked(工作目录)|git add index|Staged(暂存区)
2|Staged(暂存区)|git commit|Committed(本地仓库)
3|Committed(本地仓库)|edit index|Modified(暂存区)
4|Modified(暂存区)|git add index|Staged(暂存区)
5|Staged(暂存区)|<font color=#00FFFF> **git reset HEAD** </font> index|Modified(暂存区)
6|Modified(暂存区)|<font color=#00FFFF> **git checkout - -** </font> index|Unmodified(暂存区)
7|Staged(暂存区)|<font color=#00FFFF> **git rm - -cached** </font> index|Untracked(工作目录)

这边乍一看蛮混乱的，举几个实际例子吧。
Example 1
有两个文件file1、file2，我 git add -A 全部放进暂存区了，但是后来我发现这两个文件我必须分开提交，所以我得先把一个unstage，就可以用到第7条或者第5条也行。
如果这个文件已经commit过了，那么5和7的效果是不一样的，5是把修改的部分从暂存区挪到工作区，而7是直接从暂存区删除整个文件，相当于rm，只是文件内容会保留在工作区，所以7主要是为了rm，而5是为了unstage。
当你提交了7的操作，那个本地仓库当前版本中就是没有这个文件的。可以用cat-file看，这个我们后面说。
Example 2
文件file1我提交了一次，之后又做了修改，后来这些修改我都不想要了，就可以用到6，恢复到和本地仓库一样的状态；
我又修改了file1，add了之后，又修改了，这时候执行6，就会回到add之后的状态。
所以6就是用于删掉工作目录的修改，这部分未被跟踪的修改是无法恢复的，所以使用时要确认自己确实不要这部分修改了，不然还是建议先commit。

*(Ps：为什么关于几个状态要说这么多呢，其实我最开始在idea中用到git，从来都没有用过git add，也不知道还有暂存区，但是随着项目做得多了，代码写得多了，你会发现，规范很重要，commit message很重要，它能告诉你，告诉别人，这个commit到底做了什么，有时候看到一个commit里面8万个文件，message只有寥寥几个字，真的很气。通过add、commit你可以记录自己开发的历程，不同的方案，很重要。)*

上述的各种状态都可以通过<font color=#00FFFF> **git status** </font>看到，当然配置了alias之后，我可以直接用 git s 啦。
多打打git status，git不仅会告诉你文件当前状态，还会告诉你你接下去可以执行的操作，非常人性化。

上面提到了 git rm --cached ，如果你完全不想要这个文件了，可以省略 --cached ，直接<font color=#00FFFF> **git rm** </font>[file name]。
如果你想重命名某个文件，你可以rm再创建，或者<font color=#00FFFF> **git mv** </font>[old name] [new name]。

### 4.5. git clone
好像有点跑偏，赶紧拉回来，继续说指令
用<font color=#00FFFF> **git clone** </font>可以把远程项目拉到本地，比如下面是我的项目在github上的url，用git clone就可以拉到本地。
```BASH
$ git clone https://github.com/yyycc/ever.git
```

### 4.6. git push

git remote
git fetch
git pull
git push
git merge
git mergetool
git tag
git checkout
git branch
git log
git diff
git cat-file
git stash
git commit --amend
git reset -i HEAD~

如何
```BASH

```
```BASH

```
```BASH

```

## Z. 参考
[1.【Git】(1)---工作区、暂存区、版本库、远程仓库](https://www.cnblogs.com/qdhxhz/p/9757390.html)
[2. Git官网](https://git-scm.com/)

