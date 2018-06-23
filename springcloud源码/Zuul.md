# Zuul 源码

![](../images/15296585957815.jpg)



整个请求的过程是这样的：

- 首先将请求给zuulservlet处理
- zuulservlet中有一个zuulRunner对象，该对象中初始化了RequestContext：作为存储整个请求的一些数据，并被所有的zuulfilter共享。
- zuulRunner中还有 FilterProcessor，FilterProcessor作为执行所有的zuulfilter的管理器。FilterProcessor从filterloader 中获取zuulfilter，
- 而zuulfilter是被filterFileManager所加载，并支持groovy热加载，采用了轮询的方式热加载。
- 有了这些filter之后，zuulservelet首先执行的Pre类型的过滤器，再执行route类型的过滤器，最后执行的是post 类型的过滤器
- 如果在执行这些过滤器有错误的时候则会执行error类型的过滤器。执行完这些过滤器，最终将请求的结果返回给客户端。
 
 

- FilterProcessor.processZuulFilter
- ZuulServlet---主要
- ZuulRunner.postRoute



