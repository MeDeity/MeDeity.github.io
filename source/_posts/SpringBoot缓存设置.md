---
title: 'SpringBoot缓存(Redis)设置(自定义KEY的过期时间)笔记'
date: 2020-12-02
categories: 
  - 后端-SpringBoot
tags:
  - 技巧
---

引入依赖:

```xml
<dependencies>
  <!--redis缓存-->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  <!--cache支持-->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-cache</artifactId>
  </dependency>
</dependencies>
```

application.yml

```yml
spring:
  redis:
    # Redis数据库索引（默认为0）
    database: 0
    # Redis服务器地址
    host: localhost
    # Redis服务器连接端口
    port: 6379
    # 连接超时时间（毫秒）
    timeout: 0
    # Redis服务器连接密码（默认为空）
    password:
    # 连接池最大连接数（使用负值表示没有限制）
    pool:
      max-active: 8
      # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-wait: -1
      # 连接池中的最大空闲连接
      max-idle: 8
      # 连接池中的最小空闲连接
      min-idle: 0
```

默认配置情况下只支持 RedisTemplate<String, String>,我们通过添加配置进行扩展

```java
package com.liyisoft.filemanagement.v1.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import org.springframework.cache.annotation.EnableCaching;

import static org.springframework.data.redis.cache.RedisCacheConfiguration.defaultCacheConfig;


@Configuration
@EnableCaching  //开启缓存
public class RedisCacheConfig {
    /**
     * 申明缓存管理器，会创建一个切面（aspect）并触发Spring缓存注解的切点（pointcut）
     * 根据类或者方法所使用的注解以及缓存的状态，这个切面会从缓存中获取数据，将数据添加到缓存之中或者从缓存中移除某个值
     * @return RedisCacheManager
     */
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        RedisCacheConfiguration cacheConfiguration =
                defaultCacheConfig()
                        .disableCachingNullValues()
                        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
        return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(cacheConfiguration).build();
    }

    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory) {
        // 创建一个模板类
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // 将刚才的redis连接工厂设置到模板类中
        redisTemplate.setConnectionFactory(factory);
        //使用Jackson 2，将对象序列化为JSON
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        // 设置key的序列化器
        redisTemplate.setKeySerializer(genericJackson2JsonRedisSerializer);
        // 设置value的序列化器
        redisTemplate.setValueSerializer(genericJackson2JsonRedisSerializer);

        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(genericJackson2JsonRedisSerializer);

        return redisTemplate;
    }
}

```

```java
@RestController
@CacheConfig(cacheNames = {"sysUsers"})
public class SysUserController{
    
    @RequestMapping(value = "/getAllUsers")
    @Cacheable(key = "targetClass+methodName+#p0+#p1")
    public Response getAllUsers(@RequestParam int page, @RequestParam int limit){
        return sysUserService.getAllUsers(page,limit);
    }
}
```

注解说明:
@CacheConfig(cacheNames = {"sysUsers"})
该注解在类上全局声明缓存的命名空间(缓存名称).如果@Cacheable也指定了命名空间,则该命名空间会被@Cacheable的value所覆盖
@Cacheable(value="users"),则方法的缓存的名称变为users,而不再是sysUsers

@Cacheable(value={"user,sysUsers"},key = "targetClass+methodName+#p0+#p1")
标记在方法上,value指定了缓存的命名空间,可以指定多个,key参数指定了缓存的ID,可以为空,如果为空,则按照所有参数进行组合生成对应的系统Key

@CacheEvict(value={"user"})
从缓存中移除相应的数据

#### 自定义KEY键值的过期时间

redis-cache-timeout.yml

```yml
# 该配置文件主要用于Redis缓存KEY 自定义超时时间
redis-keys:
  list:
    # 格式key&time 单位:分钟
    - aliyunoss&30
    - stsToken&30
```

>由于@ConfigurationProperties 注解取消 locations 属性。因此读取其他配置文件内配置项，需@ConfigurationProperties与@PropertySource配合使用,但是
@PropertySource只支持properties 文件不支持yml文件,因此解析器需要重写
YamlPropertySourceFactory.java 解析yml文件

```java
import java.io.FileNotFoundException;
import java.io.IOException;
import java.util.Properties;
 
import org.springframework.beans.factory.config.YamlPropertiesFactoryBean;
import org.springframework.core.env.PropertiesPropertySource;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.support.EncodedResource;
import org.springframework.core.io.support.PropertySourceFactory;
import org.springframework.lang.Nullable;
 
public class YamlPropertySourceFactory implements PropertySourceFactory {
 
    @Override
    public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource) throws IOException {
        Properties propertiesFromYaml = loadYamlIntoProperties(resource);
        String sourceName = name != null ? name : resource.getResource().getFilename();
        return new PropertiesPropertySource(sourceName, propertiesFromYaml);
    }
 
    private Properties loadYamlIntoProperties(EncodedResource resource) throws FileNotFoundException {
        try {
            YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
            factory.setResources(resource.getResource());
            factory.afterPropertiesSet();
            return factory.getObject();
        } catch (IllegalStateException e) {
            // for ignoreResourceNotFound
            Throwable cause = e.getCause();
            if (cause instanceof FileNotFoundException)
                throw (FileNotFoundException) e.getCause();
            throw e;
        }
    }
}
```

改造RedisCacheConfig.java配置文件

```java
import com.alibaba.fastjson.JSON;
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.ObjectUtils;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.interceptor.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import javax.annotation.Resource;

import java.time.Duration;
import java.util.HashSet;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

import static org.springframework.data.redis.cache.RedisCacheConfiguration.defaultCacheConfig;

/**
 * 缓存配置
 */
@Configuration
@Slf4j
public class RedisConfig extends CachingConfigurerSupport {

    @Resource
    private RedisConnectionFactory factory;
    @Resource
    private RedisCacheTimeoutKey redisCacheTimeoutKey;

    /**
     * 自定义生成redis-key
     *
     * @return
     */
    @Override
    @Bean
    public KeyGenerator keyGenerator() {
        return (o, method, objects) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(o.getClass().getName()).append(".");
            sb.append(method.getName()).append(".");
            for (Object obj : objects) {
                sb.append(obj.toString());
            }
            System.out.println("keyGenerator=" + sb.toString());
            return sb.toString();
        };
    }

    @Bean
    @Override
    public CacheResolver cacheResolver() {
        return new SimpleCacheResolver(cacheManager());
    }

    @Bean
    @Override
    public CacheErrorHandler errorHandler() {
        // 用于捕获从Cache中进行CRUD时的异常的回调处理器。
        return new SimpleCacheErrorHandler();
    }

    private RedisCacheConfiguration getRedisCacheConfigurationWithTtl(Integer seconds) {
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();
        redisCacheConfiguration = redisCacheConfiguration.serializeValuesWith(
                RedisSerializationContext
                        .SerializationPair
                        .fromSerializer(jackson2JsonRedisSerializer)
        ).entryTtl(Duration.ofSeconds(seconds));
        return redisCacheConfiguration;
    }

    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory) {
        // 创建一个模板类
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // 将刚才的redis连接工厂设置到模板类中
        redisTemplate.setConnectionFactory(factory);
        //使用Jackson 2，将对象序列化为JSON
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        // 设置key的序列化器
        redisTemplate.setKeySerializer(genericJackson2JsonRedisSerializer);
        // 设置value的序列化器
        redisTemplate.setValueSerializer(genericJackson2JsonRedisSerializer);

        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(genericJackson2JsonRedisSerializer);

        return redisTemplate;
    }
    
    /**
     * 申明缓存管理器，会创建一个切面（aspect）并触发Spring缓存注解的切点（pointcut）
     * 根据类或者方法所使用的注解以及缓存的状态，这个切面会从缓存中获取数据，将数据添加到缓存之中或者从缓存中移除某个值
     * @return RedisCacheManager
     */
    @Bean
    @Override
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        RedisCacheConfiguration cacheConfiguration =
                defaultCacheConfig()
                        .disableCachingNullValues()
                        .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
        Set<String> cacheNames =new HashSet<>();
        ConcurrentHashMap<String,RedisCacheConfiguration> cacheConfig = new ConcurrentHashMap<>();
        if (ObjectUtils.allNotNull(redisCacheTimeoutKey,redisCacheTimeoutKey.getList())) {
            for (String keyAndValue : redisCacheTimeoutKey.getList()) {
                String[] array = keyAndValue.split("&");
                if (array.length != 2) {
                    continue;
                }
                String key = array[0];
                int value = Integer.parseInt(array[1]);
                log.info("缓存设置:键值:{},有效期:{}分钟", key, value);
                cacheNames.add(key);
                cacheConfig.put(key, cacheConfiguration.entryTtl(Duration.ofMinutes(value)));
            }
        }
        return RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(cacheConfiguration)
                .initialCacheNames(cacheNames)
                .withInitialCacheConfigurations(cacheConfig)
                .build();

    }
}

```

参考资料:
[基于Redis的Spring cache 缓存介绍](https://www.cnblogs.com/junzi2099/p/8301796.html)
[优雅的缓存解决方案--SpringCache和Redis集成(SpringBoot)](https://juejin.cn/post/6844903807646711821)
[@PropertySource 注解实现读取 yml 文件](https://juejin.cn/post/6844903768308334606)
