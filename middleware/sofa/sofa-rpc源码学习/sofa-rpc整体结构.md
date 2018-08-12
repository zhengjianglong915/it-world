# SOFA-RPC源码学习(一)--框架之总体设计
本文主要从以下几个方面去思考：

- RPC 框架是什么
- SOFA-RPC 和 Dubbo的对比
- SOFA-RPC 和 gRPC 的对比
- SOFA-RPC 框架的


## SOFA-RPC 和 Dubbo 的区别
1. 增加了限流
2. 数据透传 等能力
3. 形成一个微服务生态体系，提供了除了RPC以外的其他功能，如 trace、熔断等
4. dubbo 参数通过 url 方式传递，sofa-rpc 通过 config 方式传递和设置。 这两者的优缺点各是什么？
5. 
6. 

## 参考
- [剖析 | SOFARPC 框架之总体设计与扩展机制](https://mp.weixin.qq.com/s/ZKUmmFT0NWEAvba2MJiJfA) | 大神碧远

