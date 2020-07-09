## SpringBoot整合Apollo

官网地址：

```
https://github.com/ctripcorp/apollo/wiki/Java%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97
```

> 关键注解

- @ApolloConfig注解：将Apollo服务端的中的配置注入这个类中。
- @ApolloConfigChangeListener注解：监听配置中心配置的更新事件，若该事件发生，则调用refreshLoggingLevels方法，处理该事件。
- ConfigChangeEvent参数：可以获取被修改配置项的key集合，以及被修改配置项的新值、旧值和修改类型等信息。

### 1、依赖

```
<!-- https://mvnrepository.com/artifact/com.ctrip.framework.apollo/apollo-client -->
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>1.6.0</version>
</dependency>
<!-- 为了编码方便，并非apollo 必须的依赖 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.10</version>
</dependency>
```

### 2、YML

* application.yml

```
info: #定义各种额外的详情给服务端显示
  app:
    name: "@project.name@" #从pom.xml中获取
    description: "@project.description@"
    version: "@project.version@"
    spring-boot-version: "@project.parent.version@"

apollo:
  bootstrap:
  	# 在应用启动阶段，向Spring容器注入被托管的application.properties文件的配置信息。
    enabled: true
    # will inject 'application' and 'TEST1.apollo' namespaces in bootstrap phase
    namespaces: application,hst.mysql,hst.redis,hst.ops,hst.eureka,hst.redisson,hst.rocketmq
    
logging:
  level:
    cn.com.hopson: debug # debug日志 方便本地/开发/测试环境的日志定位    

```

* 在resources/META-INF/下创建app.properties

```
app.id=hopsonone-park-reduce
```



> 配置信息详情

```
# apollo 配置应用的 appid
app.id=springboot-apollo-demo1

# apollo meta-server地址，一般同config-server地址
apollo.meta=http://192.168.0.153:8080

# 启用apollo配置开关
# 在应用启动阶段，向Spring容器注入被托管的application.properties文件的配置信息。
apollo.bootstrap.enabled=true
# 将Apollo配置加载提到初始化日志系统之前。
apollo.bootstrap.eagerLoad.enabled=true

# apollo 使用配置的命名空间，多个以逗号分隔
apollo.bootstrap.namespaces = application,hst.mysql,hst.redis,hst.ops,hst.eureka,hst.redisson,hst.rocketmq

logging.level.cn.com.hopson：调整log 级别，为了后面演示在配置中心动态配置日志级别。

```

### 3、配置类

```
package cn.com.hopson.hopsonone.reduce.config;


import com.ctrip.framework.apollo.spring.annotation.EnableApolloConfig;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableApolloConfig
public class AppConfig {}
```

