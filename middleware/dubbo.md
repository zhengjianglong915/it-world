# Dubbo知识地图
## 一.入门
 - [官网--使用手册](https://dubbo.gitbooks.io/dubbo-user-book/)
 - [官网--源码设计](http://dubbo.apache.org/books/dubbo-dev-book/)

## 二.源码学习
### 1. [大程熙--dubbo源码系列](http://cxis.me/categories/dubbo/)
这个dubbo源码系列，对代码各个细节注释的非常清楚，代码跟读的时候建议参考参考。

 - 1.1 [Dubbo中SPI扩展机制详解](http://cxis.me/2017/02/18/Dubbo中SPI扩展机制详解/) | 对dubbo spi的实现分析的比较全面
 - 1.2 [Dubbo中暴露服务的过程解析](http://cxis.me/2017/02/19/Dubbo中暴露服务的过程解析/)
 - 1.3 [Dubbo中编码和解码的解析](http://cxis.me/2017/03/19/Dubbo中编码和解码的解析/)
 - 1.4 [Dubbo中消费者初始化的过程解析](http://cxis.me/2017/03/21/Dubbo中消费者初始化的过程解析/)
 - 1.5 [Dubbo中服务消费者和服务提供者之间的请求和响应过程](http://cxis.me/2017/03/21/Dubbo中服务消费者和服务提供者之间的请求和响应过程/)
 - 1.6 [Dubbo中集群Cluster，负载均衡，容错，路由解析](http://cxis.me/2017/03/26/Dubbo中集群Cluster，负载均衡，容错，路由解析/)
 - 1.7 [Dubbo中订阅和通知解析](http://cxis.me/2017/04/02/Dubbo中订阅和通知解析/)
 - 1.8 [Dubbo中Directory解析](http://cxis.me/2017/04/02/Dubbo中Directory解析/)
 

### 2. [肥朝--dubbo源码解析系列](https://www.jianshu.com/nb/6137390) 
比较适合源码跟读，将阅读过程一步步截图出来了，主要是还提供了一些代码跟读的思路。

 - Dubbo的SPI
    - 1 [dubbo源码解析-spi(一)](https://www.jianshu.com/p/99f568df0f05) | SPI基础
    - 2 [dubbo源码解析-spi(二)](https://www.jianshu.com/p/e7446cdc7161) | dubbo spi 如何提高性能部分，主要根据缓存机制。
    - 3 [dubbo源码解析-spi(三)](https://www.jianshu.com/p/718acbda838c) | 结合spring IOC原理，讲解dubbo spi增强功能的实现
    - 4 [dubbo源码解析-spi(四)](https://www.jianshu.com/p/dba4447e5dc5) | 结合spring AOP，讲解dubbo spi的aop设计
    - 5 [dubbo源码解析-spi(五)](https://www.jianshu.com/p/27dc92362de4) | 模仿dubbo spi 自己手动写一个spi

### 3. RPC的三大核心元素
- dubbo 线程模型
    - [Dubbo源代码实现六：线程池模型与提供者](https://blog.csdn.net/manzhizhen/article/details/73436619)  
    - [Dubbo源代码分析九：优雅停机](https://blog.csdn.net/manzhizhen/article/details/78756370)
    - [dubbo请求处理线程模型实现分析](https://cloud.tencent.com/developer/article/1109467) | 腾讯云 | 分析了线程模型的装饰者设计模式
    - [第十章 dubbo线程模型](https://www.cnblogs.com/java-zhao/p/7822766.html)
    - [dubbo底层RPC通讯Netty线程模型源码分析--视频](https://www.bilibili.com/video/av20698482/)
    
- Dubbo编码协议
    - [Dubbo编码解码](https://www.jianshu.com/p/608195a1767a) 
    - [在Dubbo中使用高效的Java序列化（Kryo和FST）](https://dangdangdotcom.github.io/dubbox/serialization.html)  

- 通讯协议
    - [精通Dubbo——Dubbo支持的协议的详解](https://blog.csdn.net/fuyuwei2015/article/details/72848310)
    

### 其他
- [dubbo源码解析系列--博客园](http://www.cnblogs.com/java-zhao/category/1090034.html)


 
## 三.思考(面试题)
- 你是否了解spi,讲一讲什么是spi,为什么要使用spi?
- 对类加载机制了解吗,说一下什么是双亲委托模式,他有什么弊端,这个弊端有没有什么我们熟悉的案例,解决这个弊端的原理又是怎么样的?
- 既然你对spi有一定了解,那么dubbo的spi和jdk的spi有区别吗?有的话,究竟有什么区别?
   - JDK的spi要用for循环,然后if判断才能获取到指定的spi对象,dubbo用指定的key就可以获取 
   - JDK的spi不支持默认值,dubbo增加了默认值的设计  
   - 拓展点增加了缓存,提高了性能
   - 增加了Ioc和AOP功能
- dubbo中spi也增加了IoC,那你先讲讲Spring的IoC, 然后再讲讲dubbo里面又是怎么做的
- dubbo中spi也增加了AOP,那你讲讲这用到了什么设计模式,dubbo又是如何做的.
   




 

