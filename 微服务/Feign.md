# Feign
Feign是一个声明式的Web Service客户端，它使得编写Web Serivce客户端变得更加简单。只需要使用Feign来创建一个接口并用注解来配置它既可。它具备可插拔的注解支持，包括Feign注解和JAX-RS注解。Feign也支持可插拔的编码器和解码器。Spring Cloud为Feign增加了对Spring MVC注解的支持，还整合了Ribbon和Eureka来提供均衡负载的HTTP客户端实现。

https://www.jianshu.com/p/e6e55ee839f8

https://blog.csdn.net/u014281502/article/details/72896182


## feign GET请求的3种方式
### 单个参数
```
public User getUser(@RequestParam("userId") Long userId);
```

### 多个参数

```
public Trans getTrans(@RequestParam("userId") Long userId, @RequestParam("userId") Long transId);
```

注意，一定要在RequestParam中写参数名字， 否则会报错：

```
Caused by: java.lang.IllegalStateException: Method has too many Body parameters:
```

### 多个参数使用map

```
public Trans getTrans(@RequestParam Map<String, Object> params);
```


