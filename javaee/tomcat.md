# Tomcat
## Tomcat线程池
### 使用
线程池作为提高程序处理数据能力的一种方案，应用非常广泛。大量的服务器都或多或少的使用到了线程池技术，不管是用Java还是C++实现，线程池都有如下的特点：线程池一般有三个重要参数：

- **最大线程数**。在程序运行的任何时候，线程数总数都不会超过这个数。如果请求数量超过最大数时，则会等待其他线程结束后再处理。
- **最大共享线程数**，即**最大空闲线程数**。如果当前的空闲线程数超过该值，则多余的线程会被杀掉。
- **最小共享线程数**，即**最小空闲线程数**。如果当前的空闲数小于该值，则一次性创建这个数量的空闲线程，所以它本身也是一个创建线程的步长。 线程池有两个概念：
- **Worker线程**。工作线程主要是运行执行代码，有两种状态：空闲状态和运行状态。在空闲状态时，类似“休眠”，等待任务；处理运行状态时，表示正在运行任务（Runnable）。
- **辅助线程**。主要负责监控线程池的状态：空闲线程是否超过最大空闲线程数或者小于最小空闲线程数等。如果不满足要求，就调整之。

使用线程池，用较少的线程处理较多的访问，可以提高tomcat处理请求的能力。使用方式： 
首先。打开**/conf/server.xml**，增加：

1. 配置executor属性
```
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="500" minSpareThreads="20" maxIdleTime="60000" prestartminSpareThreads="true" maxQueueSize="100" />
```

最大并发连接数，不配置时**默认200**，一般建议设置500~800. 

2. 配置Connector

```
<Connector executor="tomcatThreadPool"  
           port="8080" protocol="HTTP/1.1"  
               connectionTimeout="20000"  
               redirectPort="8443"   
           minProcessors="5"  
           maxProcessors="75"  
           acceptCount="1000"/>  
```

重要参数说明：

- executor：表示使用该参数值对应的线程池；
- minProcessors：服务器启动时创建的处理请求的线程数；
- maxProcessors：最大可以创建的处理请求的线程数；
- acceptCount：指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理。


参考:

- [Tomcat优化之配置线程池](https://blog.csdn.net/younger_z/article/details/70308044)
- [Tomcat使用线程池配置高并发连接](https://www.cnblogs.com/chengssblog/p/6635211.html)
- [闲谈Tomcat性能优化](https://www.cnblogs.com/zhuawang/p/5213192.html)

