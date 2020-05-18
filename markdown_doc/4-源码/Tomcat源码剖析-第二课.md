---
title: Tomcat源码剖析-第二课 
tags: tomcat
renderNumberedHeading: true
grammar_cjkRuby: true
---
[toc]
## Tomcat源码剖析-Tomcat实例构建脉络追踪
### Tomcat启动过程源码剖析
- 启动过程，如何将架构中的组件实例化?（创建->销毁）
- 对组件如何进行统一生命周期？(抽象出LifeCycle接口)

#### 基础分析
LifeCycle生命周期接口方法
![LifeCycle生命周期接口方法 =350x250](./images/1589685029690.png)

LifeCycle生命周期接口继承体系
![继承体系 =350x350](./images/1589685159991.png)

Tomcat启动入口分析
> startup.sh -> catalina.sh start -> java xxxx.jar org.apache.catalina.startup.Bootstrap(main) start(参数)

Tomacat启动流程分析
>catalinaDaemon = catalina对象
>daemon = bootstrap对象
![启动流程图 =600x250](./images/1589687750093.png)

### init初始化阶段
![初始化过程 =800x400](./images/1589702060337.png)
