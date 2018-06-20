# kafka总结
## 特性
- 高性能： 
    - 日志的append
    - Zero-copy: 消费者发消息采用了 transferFor和transferFrom
    - 生成者采用按批量的方式发送
    - 依赖操作系统随机读写、操作系统的缓存特性
- 高可用
    - 多副本
- 可扩展性
    -  可以灵活添加生成者和消费者
    -  可以通过添加broker分散集群压力  

