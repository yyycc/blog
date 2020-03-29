---
title: 如何让img居中显示
date: 2020-03-18 18:52:46
tags: css
---
今天画blog的时候遇见了一个小问题，记录一下。。。 
<!--more-->
## 1. 问题
<img>标签如何在父<div>中居中显示

## 2. 解决办法一

``` bash 
<div style="text-align: center;">
 <p styl="text-align:center"><img/></p>
</div>
```
这就轻松解决了呀。。。还要啥plan B呀。

## 3. 解决办法二
利用p标签
``` bash 
<div>
 <p styl="text-align:center"><img/></p>
</div>
```
这种方法真的很有意思

## 4. 解决办法三
flex布局
``` bash 
<div style="display: flex;justify-content: center">
 <img/>
</div>
```
或者
``` bash 
<div style="display: flex;flex-direction: column;align-items: center">
 <img/>
</div>
```

