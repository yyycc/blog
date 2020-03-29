---
title: oracle
date: 2019-10-17 09:52:27
tags: 数据库
---
oracle 那些我记不住的东西
<!--more-->
### 1. 启动
我的oracle是安装在docker容器里面的，所以。。。
``` bash
$ docker start oracle
$ ssh root@localhost -p 49160  // 49160是docker容器映射出来的端口，用ssh连接docker
```
偷偷记下来密码：admin

### 2. 连接数据库
``` bash
$ su - oracle
```
不以任何用户登录(nolog)，之后再输用户名、密码。或者直接sqlplus就行。
``` bash
$ sqlplus / nolog
```
不要密码的dba登录
``` bash
$ sqlplus / as sysdba
```
或者nolog之后
``` bash
$ conn sys as sysdba 
```
### 3. 一些sql
我喜欢用Navicat，no feeling for sql developer什么的，所以很多东西没有可视化
``` bash
$ select * from v$version;        // 查数据库版本
$ select * from dba_directories;  // 查出数据库的DATA_PUMP_DIR
$ select * from dba_users; 	   // 查出所有用户
$ select default_tablespace from dba_users where username=‘登录用户’;   // 查看该用户的默认表空间
```
``` bash
$ select a.tablespace_name,a.total,b.free from( select tablespace_name,sum(bytes)/1024/1024 || 'M'
total from dba_data_files
group by tablespace_name) a,
( select tablespace_name,sum(bytes)/1024/1024|| 'M' free from dba_free_space
group by tablespace_name) b
where a.tablespace_name=b.tablespace_name;  // 查询表空间占用情况
```
### 4. 数据的导入导出
``` bash
// 导出(DATA_PUMP_DIR就是一个逻辑地址)
$ expdp userName/pwd@ip:1521/orcl directory=DATA_PUMP_DIR dumpfile=name.DMP logfile=DATA_PUMP_DIR:name.log
// 导入
$ impdp userName/pwd remap_schema=source:target remap_tablespace=source:target directory=DATA_PUMP_DIR dumpfile=ZJZL_PROD_SIT20190823.DMP 
// linux系统导入的时候可能会碰到权限问题，看一下dump文件的读写操作权限，权限不够就chmod 777一哈
```
### 5. 创建表空间、用户
``` bash
// 创建表空间
$ CREATE TABLESPACE userName DATAFILE '/u01/app/oracle/admin/XE/dpdump/userName.dbf' SIZE 500M AUTOEXTEND ON;

// 创建用户
$ create user name identified by pwd default tablespace userName;
```
### 6. 授权
``` bash
$ grant dba to userName;
$ grant connect to userName;
$ grant alter session to userName;
$ grant create any context to userName;
$ grant create procedure to userName;
$ grant create sequence to userName;
$ grant create session to userName;
$ grant create synonym to userName;
$ grant create table to userName;
$ grant create type to userName;
$ grant create user to userName;
$ grant create view to userName;
$ grant create any table to userName;
$ grant DEBUG CONNECT SESSION to userName;
$ grant query rewrite to userName;
$ grant select any dictionary to userName;
$ grant unlimited tablespace to userName;
$ grant read,write on directory DATA_PUMP_DIR to userName;
```
### 7. 删除用户、表空间
``` bash
$ drop user username cascade;
$ drop tablespace tablespacename including contents and datafiles;
```

### 8. 表备份
表明 sys_user
``` bash
$ create table sys_user_bak/sys_user_20200114 as select * from sys_user;
```

### 9. 执行脚本文件
将多个sql语句放到test.sql文件中
``` bash
begin
    ...
    sql语句
    ...
  commit;
end;
/
```
或者以.pck结尾的包脚本 test.pck
放到某个目录下 /test
``` bash
$ su - oracle
$ cd /test
$ sqlplus
$ username
$ passworf
$ @test.sql
$ @test.pck
```
如果执行报错，可以执行show error，看具体错误。

### 10. 将sql执行结果导入excel
``` bash
$ set linesize 1000 pagesize 0 echo off termout off trimout on trimspool on feedback off heading off pause off;
$ spool test.xls;
$ select t1.id from test t1;
$ spool off;
$ exit
```
可以直接放进一个sql脚本中执行，当前目录下就会生成一个test.xls文件。



嗯，我知道你会回来看的


