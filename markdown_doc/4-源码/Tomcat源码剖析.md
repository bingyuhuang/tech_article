---
title: Tomcat源码剖析 
tags: tomcat
renderNumberedHeading: true
grammar_cjkRuby: true
---


## Tomcat源码剖析

tomcat是应用级服务器软件

### 架构设计
架构属于设计层次，源码是对设计的实现。tomcat属于套娃式架构设计
**tomcat功能(需求)**

![tomcat能干啥 =600x300](./images/1589241622841.png)
Tomcat两个功能：
- *HTTP服务器*：Socket通信(TCP/IP)。接受请求，解析HTTP报文，传递参数到Servlet服务器
- *Servlet容器*：有多个Servlet(自带Servlet + 自定义Servlet)，Servlet处理具体业务逻辑

**Tomcat架构(架构是为了完成功能需求所做的设计)**
什么是tomcat架构：为了实现上述功能，Tomcat设计封装了很多组件(Java类)，组件间的关系构成了架构。

![tomcat架构图 =700x400](./images/1589244073852.png)

**套娃式架构和配置文件对应**

**套娃时架构设计优点**
### 源码剖析经验
原则、方法、技巧

### tomcat实例构建脉络
tomcat启动过程源码分析

### servlet请求处理链路
servlet请求如何被tomcat处理