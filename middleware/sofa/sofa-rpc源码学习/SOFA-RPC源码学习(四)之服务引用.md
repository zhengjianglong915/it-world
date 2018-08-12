# SOFA-RPC源码学习(四)之服务引用

ConsumerConfig: 实例接口名称


## 流程
1. 构建 strap。 采用可扩展方式，根据不同的协议提供不同的 strap
2. 调用 strap的 refer
3. 构建一个 cluster 
4. 对cluster 进行初始化：
  - 路由器配置
  - 负载均衡器配置
  - 过滤器设置等
  - 从服务端订阅
5. 构建成一个 ClientProxyInvoker
6. 最后将 ClientProxyInvoker 包装为 代理对象






## 参考
https://www.jianshu.com/p/47816d0f1317




