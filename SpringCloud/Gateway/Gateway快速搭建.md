## Gateway

>  geteway三大核心

* 路由
* 断言
* 过滤

> 依赖

        <!--springcloud gateway-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
## cloud-gateway-gateway9527快速搭建

* POM

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud</artifactId>
        <groupId>com.example</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-gateway-gateway9527</artifactId>

    <dependencies>
        <!--springcloud gateway-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--eureka client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--引入自己定义的api通用包，可以使用Payment支付Entity-->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!--devtools-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>
</project>
```

* YML

```
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh           #路由的ID，没有固定的规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001  #匹配后提供服务的路由地址
          uri: lb://CLOUD-PAYMENT-SERVICE #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**    #断言，路径相匹配进行路由

        - id: payment_routh2         #路由的ID，没有固定规则但要求统一，建议配合服务名
          #uri: http://localhost:8001  #匹配后提供服务的路由地址
          uri: lb://CLOUD-PAYMENT-SERVICE #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**     #断言，路径相匹配的进行路由

eureka:
  client:
    register-with-eureka: true    # 是否将自己注册进eurekaServer，默认true
    fetchRegistry: true         #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    service-url:
      defaultZone: http://localhost:7001/eureka   #单机版
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
  instance:
    hostname: cloud-gateway-service
```

* 主启动

```
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableDiscoveryClient
@EnableEurekaClient
public class GatewayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run(GatewayMain9527.class, args);
    }
}
```

## 路由

Gateway网关路由有两种配置方法：

* 在配置文件yml中配置

```
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh           #路由的ID，没有固定的规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001  #匹配后提供服务的路由地址
          uri: lb://CLOUD-PAYMENT-SERVICE #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**    #断言，路径相匹配进行路由

        - id: payment_routh2         #路由的ID，没有固定规则但要求统一，建议配合服务名
          #uri: http://localhost:8001  #匹配后提供服务的路由地址
          uri: lb://CLOUD-PAYMENT-SERVICE #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**     #断言，路径相匹配的进行路由

eureka:
  client:
    register-with-eureka: true    # 是否将自己注册进eurekaServer，默认true
    fetchRegistry: true         #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    service-url:
      defaultZone: http://localhost:7001/eureka   #单机版
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
  instance:
    hostname: cloud-gateway-service
```

* 代码中注入RouteLocator的Bean

```
package com.example.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator sustomRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_route_atguigu", r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
        return routes.build();
    }
}
```

## 断言

Spring Cloud Gateway将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。

Spring Cloud Gateway包括许多内置Route Predicate工厂。所以这些Predicate都与HTTP请求的不同属性匹配。多个Route Predicate工厂可以进行组合。

Spring Cloud Gateway创建Route对象时，使用RoutePredicateFactory创建 Predicate对象，Predicate对象可以赋值给Route。Spring Cloud Gateway 包含许多内置的Route Predicate Factories。

所有这些谓词都匹配HTTP请求的不同属性。多种谓词工厂可以进行组合，并通过逻辑and。



>  说白了，Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理

## 过滤

* 两个主要接口

  imolements GlobalFilter,Ordered

* 能干嘛

  全局日志记录、统一网关鉴权等

* 案例代码

```
package com.example.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Date;

@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("********come in MyLogGatewayFilter：" + new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname == null) {
            log.info("***** 用户名为null，非法用户！！！");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

* 测试

  正确：http://localhost:9527/payment/lb?uname=z3

  错误：http://localhost:9527/payment/lb