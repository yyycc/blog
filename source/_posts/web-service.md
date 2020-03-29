---
title: web service -- spring + cxf 发布web service(1)
date: 2019-11-13 16:45:49
tags: web service
---

今天又差点被web service搞崩溃，起因是在跟客户联调的时候报错：
<!--more-->
Couldn't create soap message. Expecting Envelop in namespace http://www.w3.org/2003/05/soap-envelope, 
but got http://schemas.xmlsoap.org/soap/envelope/.
(这还是他们提供的，说什么一定要一模一样，我真是不想吐槽。。。)
经过一番百度，终于搞明白了是soap协议不一致导致的，前者是soap1.1，后者是soap1.2。
我的服务使用Axis搭建的，我各种百度怎么改协议都失败了，这个需要之后继续研究。
一气之下关了所有窗口，自己重写一个web service。
鉴于之前用Endpoint.publish(*address, new MyFirstWebService()*)方法发布过web service，一直想用cxf框架搭一个，这次就用cxf吧。

## 1. 准备
IDE工具：idea

## 2. 搭建

先新建一个 maven project，创建出来后，会有一个pom.xml，这是maven项目的配置文件：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>core</groupId>
    <artifactId>ws</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>My Cxf Web Service</name>
    <url>http://maven.apache.org</url>

    <dependencies>
        <dependency>
            。。。
        </dependency>
    </dependencies>
    <build>
        <finalName>ws</finalName>
    </build>

</project>
```
这边packaging配了war，就可以直接用maven的package指令打war包，name是项目的名字，version是版本号，finalName是war包包名
可以看打包时候的部分输出
```
[INFO] ------------------------------< core:ws >-------------------------------
[INFO] Building My Cxf Web Service 1.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
```
在pom中引入cxf的jar包：
```
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-api</artifactId>
        <version>2.5.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-rt-frontend-jaxws</artifactId>
        <version>2.5.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-rt-bindings-soap</artifactId>
        <version>2.5.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-rt-transports-http</artifactId>
        <version>2.5.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.cxf</groupId>
        <artifactId>cxf-rt-ws-security</artifactId>
        <version>2.5.0</version>
    </dependency>
```
接下去就可以写相关接口和实现类啦，下面是一个简单的接口，用@WebService和@WebMethod注解(必要的)，还有别的一些注解
```
@WebService(serviceName = "cyy")
public interface Show {
    @WebMethod(operationName = "calc")
    Number showNumber(Double a);
}
```
注解|作用
--|:--:
@WebService|标志这是一个webservice服务
@WebService(serviceName = "cyy")|产生的服务的名称
@WebService(name = "ss", targetNamespace = "http://core/")|修改命名空间
@WebMethod(operationName = "calc")|修改方法名称
@WebParam(name = "num")|修改参数名称
@WebResult(name = "returnNum")|修改返回名称
@WebMethod(exclude = true)|将制定的public方法排除，用户将不能访问

这些后续测试访问的时候再提

接下来就是配置web.xml
WEB-INF/web.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
    http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
    <!-- 配置 Spring 配置文件的名称和位置 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/cxf-beans.xml</param-value>
    </context-param>
    <!-- 启动 IOC 容器的 ServletContextListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- 配置字符集 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    <servlet>
        <servlet-name>CXFServlet</servlet-name>
        <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>CXFServlet</servlet-name>
        <!--==这个设置很重要，那么我们的webservice的地址就是http://localhost:8080/yourProgramName/webservice/Greeting=== -->
        <url-pattern>/webservice/*</url-pattern>
    </servlet-mapping>
</web-app>
```
然后是spring配置
WEB-INF/spring/cxf-beans.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:jaxws="http://cxf.apache.org/jaxws" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-3.0.xsd
http://cxf.apache.org/jaxws
http://cxf.apache.org/schemas/jaxws.xsd">
    <!--=============== 实现类的bean，需要spring注入 ============================-->
    <bean id="IcxfWBImpl" class="cxfWB.IcxfWBImpl"/>
    <jaxws:endpoint id="invoice"  implementor="#IcxfWBImpl"   address="/invoice" />

    <bean id="ShowImpl" class="com.ShowImpl"/>
    <jaxws:endpoint id="one" implementor="#ShowImpl" address="/one"/>
</beans>
```
配置好了，就可以部署再tomcat上跑起来啦，http://localhost:7070/ws/webservice/one?wsdl可以访问。

