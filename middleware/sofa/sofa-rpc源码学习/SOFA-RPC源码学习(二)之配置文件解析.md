# SOFA-RPC 源码学习之配置文件解析
sofa-rpc 的配置文件解析 和 dubbo的配置文件解析思路一样，利用 spring 配置解析功能，实现了自定义的 配置标签。 这样的好处可以极大的减少开发成本，同时又可以和spring完美的结合。

spring自定义标签 的使用可以参考：[于Spring开发——自定义标签及其解析](https://www.cnblogs.com/lizo/p/6683606.html)
源码实现参考： [Spring源码阅读之-自定义配置的解析](https://blog.csdn.net/weiythi/article/details/75899547)

如果你了解 spring 自定义标签使用，第一反应就是 sofa-rpc 的 META-INF/spring.handler 文件中配置有 sofarpc 标签的解析。 然而在 sofa-rpc 的源码中找不到该文件，原因是 sofaboot 里将 sofa相关的 标签都统一做了处理。

因此在sofaboot 的源码中我们可以找到 spring.handlers 文件， 其配置如下：

```
http\://sofastack.io/schema/sofaboot=com.alipay.sofa.infra.config.spring.namespace.handler.SofaBootNamespaceHandler
```

## 解析器的注册过程
sofaboot 的自定义解析代码 比 dubbo优雅，具有很强的可扩展性，同时不破坏开发封闭的设计思路。如果要新增一个 sofa 标签，不需要在 SofaBootNamespaceHandler 中修改代码，注册解析标签类。 只要继承 sofa 的 SofaBootTagNameSupport 接口即可。 可以看具体代码：

首先看看 SofaBootTagNameSupport 接口定义，该接口主要是用于获取 标签名字：

```
public interface SofaBootTagNameSupport {
    /**
     * 获取支持的 TagName
     *
     * @return 支持的 TagName
     */
    String supportTagName();
}
```

那  如何做到动态解析配置文件呢：

```
 @Override
    public void init() {
        // 首先获取所有实现 SofaBootTagNameSupport 的类
        ServiceLoader<SofaBootTagNameSupport> serviceLoaderSofaBoot = ServiceLoader
            .load(SofaBootTagNameSupport.class);
            
        // 通过for 循环方式进行注册
        for (SofaBootTagNameSupport tagNameSupport : serviceLoaderSofaBoot) {
            this.registerTagParser(tagNameSupport);
        }
    }

    private void registerTagParser(SofaBootTagNameSupport tagNameSupport) {
        if (!(tagNameSupport instanceof BeanDefinitionParser)) {
            logger.error(tagNameSupport.getClass() + " tag name supported ["
                         + tagNameSupport.supportTagName() + "] parser are not instance of "
                         + BeanDefinitionParser.class);
            return;
        }
        
        // 注册使用的的标签名字，这便是接口的一个用处。
        String tagName = tagNameSupport.supportTagName();
        // 将解析器进行注册
        registerBeanDefinitionParser(tagName, (BeanDefinitionParser) tagNameSupport);
    }
```

## 标签解析器
目前 sofaboot 提供了两个标签解析器，这两个都是 提供给 sofarpc使用的：

- ServiceDefinitionParser: 服务发布标签的解析，即 <sofa:service>
- ReferenceDefinitionParser: 服务引用标签的解析，即 <sofa:reference>

具体标签内容的解析这里不进行阐述，感兴趣的可以看看源码，看看每个标签怎么解析。这里重点是分析 标签 中的 服务是如何 被发布和引用的过程。

### 发布
ServiceDefinitionParser 是负责将 service 标签解析，注册。 那最后如何被发布出去呢？
sofa-rpc 和 dubbo 的做法一致，都是通过 实现FactoryBean, 通过懒加载的方式完成 服务的发布。  


怎么和 serviceConfig 关联呢？  zz

