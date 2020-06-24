## 1、依赖

​	POM

```
        <!-- spring boot redis 缓存引入 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!-- lettuce pool 缓存连接池 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
```

## 2、约定

​	application.properties

```
redis.sentinel.nodes=luliang888.top:26379,luliang888.top:26380,luliang888.top:26381
redis.sentinel.master.name=mymaster
redis.auth=123456
redis.database=0
redis.pool.maxTotal=10
redis.pool.maxIdle=5
redis.pool.minIdle=3
redis.pool.testOnBorrow=true
```

## 3、配置

```java
package com.example.redis.config;

import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisSentinelConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceClientConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettucePoolingClientConfiguration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.util.Arrays;
import java.util.HashSet;
import java.util.Set;

@Configuration
public class RedisConfig {

    @Bean("redisSentinelConfiguration")
    public RedisSentinelConfiguration getRedisSentinelConfiguration(
            @Value("${redis.sentinel.nodes}") String nodes,
            @Value("${redis.sentinel.master.name}") String masterName,
            @Value("${redis.auth}") String password,
            @Value("${redis.database}") int database
    ) {
        Set<String> redisNodes = new HashSet<>();
        redisNodes.addAll(Arrays.asList(nodes.split(",")));
        RedisSentinelConfiguration configuration = new RedisSentinelConfiguration(masterName, redisNodes);
        configuration.setDatabase(database);
        configuration.setPassword(password);
        return configuration;
    }

    @Bean("objectPoolConfig")
    public GenericObjectPoolConfig getObjectPoolConfig(
            @Value("${redis.pool.maxTotal}") int maxTotal,
            @Value("${redis.pool.maxIdle}") int maxIdle,
            @Value("${redis.pool.minIdle}") int minIdle,
            @Value("${redis.pool.testOnBorrow}") boolean testOnBorrow
    ) {
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
        poolConfig.setMaxTotal(maxTotal);
        poolConfig.setMaxIdle(maxIdle);
        poolConfig.setMinIdle(minIdle);
        poolConfig.setTestOnBorrow(testOnBorrow);
        return poolConfig;
    }

    @Bean("lettuceClientConfiguration")
    public LettuceClientConfiguration getLettuceClientConfiguration(
            @Autowired GenericObjectPoolConfig poolConfig
    ) { // 创建Lettuce组件的连接池客户端配置对象
        return LettucePoolingClientConfiguration.builder().poolConfig(poolConfig).build();
    }

    @Bean("redisConnectionFactory")
    public RedisConnectionFactory getConnectionFactory(
            @Autowired RedisSentinelConfiguration redisSentinelConfiguration,
            @Autowired LettuceClientConfiguration lettuceClientConfiguration
    ) {
        LettuceConnectionFactory connectionFactory = new LettuceConnectionFactory(redisSentinelConfiguration, lettuceClientConfiguration);
        return connectionFactory;
    }

    @Bean("redisTemplate")
    public RedisTemplate getRedisTempalate(
            @Autowired RedisConnectionFactory connectionFactory
    ) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer()); // 数据的key通过字符串存储
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer(Object.class)); // 保存的value为对象
        redisTemplate.setHashKeySerializer(new StringRedisSerializer()); // 数据的key通过字符串存储
        redisTemplate.setHashValueSerializer(new Jackson2JsonRedisSerializer(Object.class)); // 保存的value为对象
        return redisTemplate;
    }

    @Bean("stringRedisTemplate")
    public StringRedisTemplate getStringRedisTemplate(
            @Autowired RedisConnectionFactory connectionFactory
    ) {
        StringRedisTemplate redisTemplate = new StringRedisTemplate();
        redisTemplate.setConnectionFactory(connectionFactory);
        return redisTemplate;
    }
}
```

## 4、测试

```jav
@Test
void testRedisTemplate() {
    redisTemplate.opsForValue().set("name", "学习");
    System.out.println(redisTemplate.opsForValue().get("name"));
}
```

