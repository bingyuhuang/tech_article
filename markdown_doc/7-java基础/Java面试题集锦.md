---
title: Java面试题集锦
tags: java面试
renderNumberedHeading: true
grammar_cjkRuby: true
---
[toc]

# 0.内存
## 什么是内存屏障?
博客：[一文解决内存屏障](https://monkeysayhi.github.io/2017/12/28/%E4%B8%80%E6%96%87%E8%A7%A3%E5%86%B3%E5%86%85%E5%AD%98%E5%B1%8F%E9%9A%9C/)

```
    1.高速缓存：解决CPU和内存之间速度匹配的问题
    2.指令重排序：编译器、处理器、缓存的指令重排序
    3.可见性：缓存重排序导致并发环境下数据可见性问题
    4.读写屏障：storestore/storeload/loadstore/loadload
```
## java探针如何使用?
博客：[java探针](https://www.cnblogs.com/aspirant/p/8796974.html)
感觉使用了该技术的工具: [arthas](https://alibaba.github.io/arthas/watch.html)
## G1垃圾回收器有何特点，与CMS有啥区别？
博客: [G1入门](https://www.cnblogs.com/aspirant/p/8663872.html)

- 范围不同：CMS老年代，G1新生代和老年代
- STW时间不同:CMS以最小停顿时间为目标，G1可预测垃圾回收停顿时间
- 垃圾碎片：CMS使用标记-清除算法，容易产生内存碎片，G1使用标记-整理，对内存空间整合，降低内存空间碎片
- 垃圾回收过程不一样：CMS(初始标记-并发标记-重新标记-并发清除)，G1(初始标记-并发标记-最终标记-筛选回收)

## 什么时候栈会溢出？
博客：

[1.十中JVM溢出情况](https://segmentfault.com/a/1190000017226359)

[2.java内存溢出和栈溢出](https://blog.csdn.net/hu1991die/article/details/43052281)

    栈溢出有两种，一种是stackoverflow，另一种是outofmemory

    前者一般是因为方法递归没终止条件：
        * 线程请求分配的栈容量超过java虚拟机栈允许的最大容量
        * 某个线程所需的栈内存超过了JVM的限制，物理内存足够可用
    后者一般是方法中线程启动过多：
        * java虚拟机栈可以动态拓展，并且扩展的动作已经尝试过，但是目前无法申请到足够的内存去完成拓展，或者在建立新线程的时候没有足够的内存去创建对应的虚拟机栈
        * 物理内存已没有足够的可用内存分配给JVM的栈使用
    
    堆溢出：java heap space
        堆中主要存储的是对象，不断new对象导致堆溢出
## AQS与CAS的区别？
博客：

[1.从 synchronized 到 CAS 和 AQS - 彻底弄懂 Java 各种并发锁](https://juejin.im/post/5c37377351882525ec200f9e)

[2.不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)

[3.AQS与CAS详解](http://www.aremi.cn/wordpress/2019/03/28/aqs%E4%B8%8Ecas%E8%AF%A6%E8%A7%A3/)

    AQS(队列同步器)：用于构建锁或其他同步组件的基础框架
        * 使用int成员变量表示同步状态
        * 通过内置FIFO的双向队列完成获取锁线程的排列工作。
        * 两种资源共享方式：独占和共享
    CAS(比较并替换)：解决多线程并行情况下使用锁造成性能损耗的一种机制。
        * 三个操作数，内存位置(V)、预期原值(A)、新值(B)
        * j.u.c完全建立在CAS之上，借助CAS实现了区别于synchronized同步锁的乐观锁。
        * CAS通过调用JNI的代码实现,缺点：ABA问题、循环时间长开销大、只能保证一个共享变量的原子操作
        
# JVM与性能优化
## 描述一下 JVM 加载 Class 文件的原理机制?
博客：

[1.JVM 加载 class 文件的原理机制（类的生命周期、类加载器）](https://blog.csdn.net/Jerome_s/article/details/52080261)
[2.JVM加载class文件的原理机制](https://www.iteye.com/blog/samuschen-1119539)
    
    java中所有的类，需要类加载器装载到JVM中才能运行。类加载器本身也是一个类，使用动态加载方式，使用到某个类才会去加载，以便节省开销。
        * 类加载器的原理和步骤
            1.装载(查找和导入class文件)，隐式装载(new)、显示装载(class.forname()
            2.连接：检查(检查载入class文件数据的正确性)、准备(为类的静态变量分配存储空间)、解析(可选，将符号引用准换成直接引用)
            3.初始化：初始化静态变量和静态代码块
        * 加载器类型
            1.Bootstrap Loader(负责加载系统类)
            2.ExtClassLoader(负责加载扩展类)
            3.AppClassLoader(负责加载应用类)
        * 父类委托模型机制：当类加载器需要加载类的时候，先请示其Parent(即上一层加载器)在其搜索路径载入，如果找不到，才在自己的搜索路径搜索该类
## 什么是类加载器？

[1.深入探讨 Java 类加载器](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/index.html)
[2.深入分析java类加载器原理](https://juejin.im/post/5c866e00f265da2dd1689f8b)
    
    类加载器用来将java类，加载到JVM中。读取Java字节码文件，转换成java.lang.Class类的一个实例。
    
    
## 类加载器有哪些？
## 什么是tomcat类加载机制？
## 类加载器双亲委派模型机制？
## Java 内存分配？
## Java 堆的结构是什么样子的？
## 简述各个版本内存区域的变化？
## 说说各个区域的作用？
## Java 中会存在内存泄漏吗，简述一下？
## Java 类加载过程？
## 什么是GC? 为什么要有 GC？
## 简述一下Java 垃圾回收机制？
## 如何判断一个对象是否存活？
## 垃圾回收的优点和原理，并考虑 2 种回收机制？基本原理是什么？
## 深拷贝和浅拷贝？
## 什么是分布式垃圾回收（DGC）？它是如何工作的？
## 在 Java 中，对象什么时候可以被垃圾回收？
## 简述Minor GC 和 Major GC？
## Java 中垃圾收集的方法有哪些？
## 讲讲你理解的性能评价及测试指标？
## 常用的性能优化方式有哪些？
## 说说分布式缓存和一致性哈希？
## 什么是GC调优？
# 2.Redis
## redis数据结构有哪些？
## Redis缓存穿透，缓存雪崩？
## 如何使用Redis来实现分布式锁？
## Redis的并发竞争问题如何解决？
## Redis持久化的几种方式，优缺点是什么，怎么实现的？
## Redis的缓存失效策略？
## Redis集群，高可用，原理？
## Redis缓存分片？
## Redis的数据淘汰策略？
## redis队列应用场景？
## 分布式使用场景（储存session）？
# 3.网络编程
## TCP建立连接和断开连接的过程？
## HTTP协议的交互流程• HTTP和HTTPS的差异，SSL的交互流程？
## TCP的滑动窗口协议有什么用？
## HTTP协议都有哪些方法？
## Socket交互的基本流程？
## 讲讲tcp协议（建连过程，慢启动，滑动窗口，七层模型）？
## webservice协议（wsdl/soap格式，与restt办议的区别）？
## 说说Netty线程模型，什么是零拷贝？
## TCP三次握手、四次挥手？
## DNS解析过程？
## TCP如何保证数据的可靠传输的？
# 4.设计模式与重构
## 说说几个常见的设计模式（23种设计模式）？
## 设计一个工厂的包的时候会遵循哪些原则?
## 列举一个使用了 Visitor/ Decorator模式的开源项目/库?
## 如何实现一个单例?
## 代理模式(动态代理)？
## 单例模式(懒汉模式,恶汉模式,并发初始化如何解决, volatile与lock的使用)？
## JDK源码里面都有些什么让你印象深刻的设计模式使用,举例看看?
# 5.分布式
## 什么是CAP定理？
## 说说CAP理论和BASE理论？
## 什么是最终一致性？最终一致性实现方式？
## 什么是一致性Hash？
## 讲讲分布式事务？
## 如何实现分布式锁？
## 如何实现分布式 Session?
## 如何保证消息的一致性?
## 负载均衡的理解？
## 正向代理和反向代理？
## CDN实现原理？
## 怎么提升系统的QPS和吞吐？
## Dubbo的底层实现原理和机制？
## 描述一个服务从发布到被消费的详细过程？
## 分布式系统怎么做服务治理？
## 消息中间件如何解决消息丢失问题？
## Dubbo的服务请求失败怎么处理？
## 对分布式事务的理解？
## 如何实现负载均衡,有哪些算法可以实现?
## Zookeeper的用途,选举的原理是什么?
## 讲讲数据的垂直拆分水平拆分？
## zookeeper原理和适用场景？
## zookeeper watch机制？
## redis/zk节点宕机如何处理？
## 分布式集群下如何做到唯一序列号？
## 用过哪些MQ,怎么用的,和其他mq比较有什么优缺点,MQ的连接是线程安全的吗？
## MQ系统的数据如何保证不丢失？
## 列举出能想到的数据库分库分表策略？

## 其他
[1.java高级要求](https://cloud.tencent.com/developer/article/1505503)

[2.面经](https://mp.weixin.qq.com/s/3AJSfb5cNyh5MCKHkcO7OQ)

[3.2019年Java经典面试题汇总](https://mp.weixin.qq.com/s/ysrxRYG8N-f_kwg78qTsbw)