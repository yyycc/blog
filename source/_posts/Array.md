---
title: Array
date: 2020-03-02 18:33:49
tags:
---
js的文档也看了好多遍了，怎么每次碰到数组的操作，还是要bd。。。
<!--more-->

## 1. 初始化
两种初始化的方式，第一种是使用构造函数
``` bash
var arr = new Array();
```
括号里面可以给参数，但是不同的参数，会导致它的行为不一致。
所以，建议使用另一种方式，也就是数组字面量
``` bash
var arr = [1, 2, 3];
```

## 2. 实例方法
数组的示例方法真的有好多，这里我把它分成两种：
会改变原数组的 & 不会改变原数组的
本来想一一列举的，但是好像没啥意义
就画个表格吧～～

方法|描述|参数1|参数2|返回|
--|:--:|--:|--:|--:
**不改变原数组**|
join|<font color=#FF0000> **转字符串** </font>|分隔符|--|字符串
concat()<sup>[1]</sup>|<font color=#FF0000> **合并** </font>|多个各种类型值|--|合并后的新数组|
slice()<sup>[2]</sup>|<font color=#FF0000> **提取** </font>|start(从0开始)|end(不包含,没有就返回到最后一个成员)|start，end之间的数据组成的新数组|
map()|<font color=#FF0000> **遍历** </font>|function (elem, index, arr) {}|绑定参数函数内部的this变量|func处理后的新数组|
filter()|<font color=#FF0000> **过滤** </font>|function (elem, index, arr) {}|绑定参数函数内部的this变量|结果为true的数据组成的新数组|
reduce()|<font color=#FF0000> **累计** </font>|function (a, b, index, arr) {}|起始值|起始值&数组成员累计值
some()|<font color=#FF0000> **一真** </font>|function (elem, index, arr) {}|起始值|一个true就返回true
every()|<font color=#FF0000> **全真** </font>|function (elem, index, arr) {}|起始值|全是true才返回true
**改变原数组**|
push()<sup>[3.1]</sup>|<font color=#FF0000> **添加** </font>|多个各种类型值|--|添加元素后数组的长度
pop()<sup>[4.1]</sup>|<font color=#FF0000> **删除** </font>|--|--|删除的元素
shift()<sup>[4.2]</sup>|<font color=#FF0000> **删除** </font>|--|--|删除的元素
unshift()<sup>[3.2]</sup>|<font color=#FF0000> **添加** </font>|多个各种类型值|--|添加元素后数组的长度
reverse()|<font color=#FF0000> **倒置** </font>|--|--|倒置后的数组
splice()|<font color=#FF0000> **删除&添加** </font>|start(从0开始)|删除个数(后面的参数都是添加的元素)|删除的元素组成的数组
sort()<sup>[5]</sup>|<font color=#FF0000> **排序** </font>|function (a, b) {}|--|排序后的数组
forEach<sup>[6]</sup>|<font color=#FF0000> **遍历** </font>|function (elem, index, arr) {}|绑定参数函数内部的this变量
indexOf()|<font color=#FF0000> **位置** </font>|元素|--|元素第一次出现的位置/-1

[1]. 如果数组成员包括对象，concat方法返回当前数组的一个浅拷贝。新数组拷贝的是对象的引用。
[2]. 将类数组转化为真正的数组:  Array.prototype.slice.call(document.querySelectorAll("div"));
[3.1]. push: 在最后一个位置添加。
[3.2]. unshift: 在第一个位置添加。
[4.1]. pop: 删除最后一个元素。
[4.2]. shift: 删除第一个元素。
[5].   默认按照字典顺序排序，所以如果要按数字大小，则要写function，并且最好返回数值
[6].   forEach方法只能改变数组里的对象，而不能改变数组中的基本数据类型值！！！

最后再提一下for...in和for...of
for...in会遍历所有可枚举属性，所以适合对象，不适合数组
for...in遍历值是键名
for...of只遍历数组元素
for...of遍历值是键值

## z. 参考
[1. Array 对象](https://wangdoc.com/javascript/stdlib/array.html)
