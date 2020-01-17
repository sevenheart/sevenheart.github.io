---
title:  SpringMVC概览
date: 2018-12-29 22:31:06
categories:
 - Spring
tags:
 - SpringMVC
---

### SpringMVC概览

##### 一、请求流程：

![](../../../sources/img/springmvc执行流程.png)

##### 二、请求处理过程概述及内部执行流程

![](../../../sources/img/springmvc处理流程.png)

![](../../../sources/img/springmvc内部执行流程.png)

##### 三、源码流程概览

![](../../../sources/img/DisparchServlet初始化.png)

![](../../../sources/img/springmvc中各组件的做法.png)

**解析：**

servlet的配置文件在WEB-INFO指定位置就是为了tomcat可以找到并且创建servlet。

dispatcherServlet类下有一个配置文件里边配置了所有的处理器、适配器视图解析器等。该文件在DispatcherServlet的静态代码块中随着类加载而被解析加载。

**http服务请求：**
   1.浏览器发起请求到http服务器。
    2.http服务器接收请求，解析请求，
    3.http服务器转发请求到servlet容器。
**servlet容器的工作流程：**
    1.servlet容器拿到请求后根据请求根据servlet
      的映射和url定位要执行的servlet，判断是否初始化然后初始化，
      调用service中的业务类方法，再返回servletresponse对象给http服务器，
      服务器解析成静态页面后返回封装成http的静态资源交给浏览器。。

**Tomcat架构：**
       1.处理socket请求。（连接器）
        2.加载和管理servlet，以及处理具体的request请求。（容器）
连接器：将请求转换为servletrequest对象发送给容器。容器处理成serveltresponse后发送给连接器。
容器：定位相应的servlet，然后执行完后返回一个serveltresponse给连接器。