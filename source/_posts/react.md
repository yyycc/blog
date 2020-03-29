---
title: react
date: 2019-12-31 16:13:43
tags: 前端
---

2019年的最后一天。
好久没写博客了，真的是从善如流，从恶如崩。
今年最大的收获应该是学习了react吧，虽然才入了个门，不记录一下，可能过完年就忘光了。。。
<!--more-->

## 1. 环境
rekit脚手架搭建的react。
node.js     v12.13.1
npm         6.12.1

webpack
redux
router
antd
axios
json-server

## 2. 目录结构
根目录：/rekit-app

### 2.1. package.json
package.json

<font color=#DC143C> **npm init** </font>就可以生成这个文件，下面就是我初始化的一个package.json。
他是一个json对象，记录了项目的一些基本信息等等
```
{
  "name": "cyy",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```
下面具体介绍一下，rekit项目的package.json文件,几个主要部分：dependencies，scripts，devDependencies等
![图1. package.json](package.png)

#### 2.1.1. dependencies
如下，是我项目的依赖，模块名+版本范围
dependencies是项目运行依赖的模块，devDependencies是项目开发依赖的模块。
也就是devDependencies并不会被打进生产的包里。
安装依赖包的时候 --save就会写入dependencies中，--save-dev就会写入devDependencies中
```
"dependencies": {
    "antd": "^3.26.0",
    "autoprefixer": "7.1.6",
    "axios": "^0.18.0",
    "babel-core": "6.26.0",
    "babel-eslint": "7.2.3",
    "babel-jest": "20.0.3",
    "babel-loader": "7.1.2",
    ...
  },
"devDependencies": {
    "babel-plugin-import": "^1.13.0"
}
```

#### 2.1.2. scripts
script中定义脚本命令
当我执行比如<font color=#DC143C> **npm run build** </font>的时候，就相当于执行了<font color=#DC143C> **node scripts/build.js** </font>
其中start可以直接写<font color=#DC143C> **npm start** </font>
```
"scripts": {
    "mock": "json-server --watch --port 6077 data/db.json",
    "start": "concurrently \"node scripts/start.js\" \"json-server --watch --port 6077 data/db.json\"",
    "build": "node scripts/build.js",
    "test": "node scripts/test.js --env=jsdom"
},
```
可以看到start后面的指令有两部分，我这边是为了在启动项目的同时，把json-server也启动，所以用到了concurrently
json-server是一个可以在前端运行，返回json数据的服务。用来提供前端测试数据还是不错的。
```
npm insatll -g concurrently

然后修改package.json中script：

"server":"react-scripts start",
"json_server":"json-server mock/db.json --port 3003",
"start": "concurrently \"npm run json_server\" \"npm run server\" ",
```
参考：[配置package.json 使一次npm run start 执行两个指令或者多个指令](https://blog.csdn.net/div_ma/article/details/80579227)

#### 2.1.3. rekit
这里配置了启动端口，可以在start.js中获取到，项目就会在你指定的端口下启动。
```
"rekit": {
    "devPort": 6075,
    "studioPort": 6076,
    "plugins": [],
    "css": "less"
  },
```

当然啦，用 rekit creat <app-name> 创建一个rekit项目，这些他都帮你配置好了，可以直接开始业务代码的编写。

### 2.2. /src
所有的文件都放在这个目录序啊

### 2.2.1. index.js
这是项目的入口文件，将资源都挂载到root节点上。
rekit中，他加载了Root组件，这个组件接收store和routeConfig，主要就是为了加载路由
```
renderApp(<Root store={store} routeConfig={routeConfig} />);
```

### 2.2.2. /common/routeConfig.js
不同feature的子路由都集中定义在这里
```
// 使用json方式定义路由、使用renderRouteConfigV3解析json，转换成react声明式语法
const routes = [{
  path: '/',
  component: App,
  childRoutes: [
    ...childRoutes,
    { path: '*', name: 'Page not found', component: PageNotFound },
  ].filter(r => r.component || (r.childRoutes && r.childRoutes.length > 0)),
}];
```

### 2.2.3. Root.js
这里定义了index.js中加载的Root组件，他调用了Root渲染函数renderRouteConfigV3解析json，转换成react声明式语法。

### 2.2.4. /images   /styles
images不说了，就是放图片，styles下主要有global.less以及index.less
global中定义一些全局的样式，而index.less中引入global.less以及每个feature中定义的style.less
```
// index is the entry for all styles.
@import './global';
@import '../features/home/style';
@import '../features/common/style';
@import '../features/examples/style';
@import '../features/blog/style';
@import '../features/log/style';
```
### 2.2.5. /common
除了上面说的routeConfig.js集中定义各个feature的路由，还有
rootReducer.js集中定义各个feature的reducer

### 2.2.6. /features
这是最重要的啦，之前说的feature就是他，features中就是项目的各个功能。
