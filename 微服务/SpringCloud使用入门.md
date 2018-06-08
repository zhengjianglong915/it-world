# SpringCloud使用入门
# 配置
#### @EnableAutoConfiguration
#### @EnableAspectJAutoProxy


# 错误
> If you want an embedded database please put a supported one on the classpath. If you have database settings to be loaded from a particular profile you may need to active it 

大概的意思是说springboot自动帮我们注入DataSource了，而我们在yml文件里面还并没有配置数据库的连接信息，所以就报错了。

参考：https://www.cnblogs.com/skyblue-li/p/8672778.html



