---
title: spring 事务配置
date: 2020-01-07 09:19:16
tags: spring
---

service中存在众多业务逻辑，往往一个方法中就涉及多个增删改，当一个ddl报错，我们肯定希望之前执行的回滚，之后的不再执行，
这样才能保证事务的一致性。不然我只记录账上少了100，没有记录这个100的去处，账就崩了。
<!--more-->

## 1. @Transactional
最开始要求在每个service类上都加上这么一个注解
那么这个service中的所有方法，报错就会回滚。
```
@Transactional(rollbackFor = Exception.class)
```
但是这个就存在问题，因为一个service下面的众多方法不是都需要事务支持的，有很多查询方法完全不需要事务，
而将@Transactional注释在类上，就相当于将每个方法都纳入事务管理。会影响性能。

所以最好不要再接口上加@Transactional注解，而是在每个具体的方法上加注解。

查询的方法上可以这么写(没试过。。。)
```
@Transactional(readOnly=true) //配置事务 查询使用 只读
```
### 1.2. 注意事项
使用@Transactional注解的，只能是public，@Transactional注解的方法都是被外部其他类调用才有效，故只能是public。
在 protected、private 或者 package-visible 的方法上使用 @Transactional 注解，它也不会报错，但事务无效。


 
## 2. Propagation
事务属性|解释
--|:--:
REQUIRED|支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 
SUPPORTS|支持当前事务，如果当前没有事务，就以非事务方式执行。 
MANDATORY|支持当前事务，如果当前没有事务，就抛出异常。 
REQUIRES_NEW|新建事务，如果当前存在事务，把当前事务挂起。 
NOT_SUPPORTED|以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 
NEVER|以非事务方式执行，如果当前存在事务，则抛出异常。 
NESTED|支持当前事务，如果当前事务存在，则执行一个嵌套事务，如果当前没有事务，就新建一个事务。 

### 2.1. REQUIRES_NEW
举个例子：
A类a方法中调用B类b方法。
a方法不能回滚，因为无论是否报错我都要记录日志，所以a没有开启事务，但是b需要事务支持，
所以在b方法上加上以下注解，表示新建事务
```
@Transactional(rollbackFor = Exception.class, propagation = Propagation.REQUIRED) // or REQUIRES_NEW
```
(写着写着就感觉不对劲了。。。我当初为啥要用REQUIRES_NEW来着，REQUIRED不好么？。。。)
这个事务还是多尝试，找到适合对应业务需求的。

## Z. 参考
[1. Spring中propagation的7种事务配置](https://blog.csdn.net/sayoko06/article/details/79164858)
[2. @Transactional事务几点注意](https://www.cnblogs.com/happyday56/p/8906443.html)

