# 自动生成mybatis
需要涉及到对  mybatis-generator-core的改造

### 1. 获取源码
直接下载是下载不到到，可以首先新建一个 maven 工程,添加下面的依赖,使用 maven 的 Download Sources ,获得 mybatis-generator-core 的源码。

```
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.6</version>
</dependency>
```

