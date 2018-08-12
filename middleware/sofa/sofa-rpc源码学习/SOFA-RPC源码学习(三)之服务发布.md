# SOFA-RPC 源码学习(三)之服务发布


## 点滴记录

- 默认配置文件存放在 rpc-config-default.json 目录，启动后会保存到线程安全的 map结构 CFG中. 可以被用户定义的配置文件 覆盖


1. 通过 extension 获取 配置



## 流程
1. 配置 serviceConfig 和 providerConfig
2. 执行 providerConfig.export 进行服务发布
3. 首先看是否 providerBootstrap ，如果没有则创建
   - 采用扩展机制获取，目前有几种 BoltProviderBootstrap、RestProviderBootstrap、Http2ClearTextProviderBootstrap 和DubboProviderBootstrap。 这部分是根据协议来执行，提高可扩展性
   - 但是为什么 export 是放在 配置中，符合对象设计的思路么？ 配置发布？好像也对
4. 执行 DubboProviderBootstrap 的export 
5. 因为一个 provider 可以发布到多个服务端口上，这些端口可以配置不同的协议，首先获取 比遍历 ServerConfig 列表
6. 构建一个 ProviderProxyInvoker 代理对象， ProviderProxyInvoker 中对 ProviderInvoker 进行包装，增加了很多Filter。 
7. 构建服务
7. 将invoker 注册到服务上
8. 启动服务，比如注册响应处理器等
8. 注册到注册中心去


## 参考
https://www.jianshu.com/p/6a9b7e630fa1


