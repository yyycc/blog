---
title: font-awesome
date: 2020-03-19 19:14:34
tags: css
---
一个非常好用的icon库。。。 
<!--more-->
其实很久之前就用到过，只是当时不知道这个是font-awesome。
我记得当时就是在样式中写了
``` bash
content: "\f2b9"
```
一个小图标就出现了
真的很神奇
## 1. 问题：只显示方框
最近又一次用到，但是图标都显示不出来，只显示一个小方块。
于是，我就去研究了一下，这个font-awesome。

## 2. 解决
怎么解决呢，很简单，去他的官网(https://fontawesome.dashgame.com/)下载。
从下载下来的包找到font-awesome-4.7.0/css/font-awesome.min.css，在你的页面上引入就好啦

``` bash
<head>
  <link rel="stylesheet" type="text/css" href="～/font-awesome-4.7.0/css/font-awesome.min.css">
  <style>
    .fa:before{
      content: "\f2b9";
    }
  </style>
</head>
...
<li>
  <i class="fa"></i>
</li>
```
还有更简单的写法，你可以在官网上找到你想要的图标，边上有它的名字name，你只要class给fa fa-name就可以啦。(前面的fa 一定要写哦)
或者，如果你不知道名字，只知道content值，也可以
``` bash
<li>
  <i>&#xf2b9;</i>
</li>
```

## 3. react中使用
最后再讲一下react框架下怎么使用
首先
``` bash
npm install --save font-awesome
```
然后在你要用的地方，或者是公共的less文件中引入
``` bash
@import '～/node_modules/font-awesome/css/font-awesome.min.css';
```
页面上使用的写法还是跟上面说的一样一样的。
