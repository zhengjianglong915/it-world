# SpringBoot
## 数据源自动配置原理
https://blog.csdn.net/hengyunabc/article/details/78762097



## 数据源配置问题
默认需要配置数据源，但是有的服务

```
2018-06-01 15:55:52,433 ERROR [][main] org.springframework.boot.SpringApplication - Application startup failed
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'dataSource': Unsatisfied dependency expressed through field 'basicProperties'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'spring.datasource-org.springframework.boot.autoconfigure.jdbc.DataSourceProperties': Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.boot.autoconfigure.jdbc.DataSourceProperties]: Constructor threw exception; nested exception is java.lang.NoClassDefFoundError: org/springframework/jdbc/datasource/embedded/EmbeddedDatabaseType

```

原因是pom.xml 中存在数据源配置：

```
 <dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid-spring-boot-starter</artifactId>
  <version>1.1.6</version>
</dependency>
```




