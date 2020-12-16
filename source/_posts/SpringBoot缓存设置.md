---
title: 'SpringBoot缓存(Redis)设置笔记'
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
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;


@Configuration
@EnableCaching  //开启缓存支持
public class RedisCacheConfig {
    /**
     * 申明缓存管理器，会创建一个切面（aspect）并触发Spring缓存注解的切点（pointcut）
     * 根据类或者方法所使用的注解以及缓存的状态，这个切面会从缓存中获取数据，将数据添加到缓存之中或者从缓存中移除某个值
     * @return RedisCacheManager
     */
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        return RedisCacheManager.create(redisConnectionFactory);
    }

    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory factory) {
        // 创建一个模板类
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 将刚才的redis连接工厂设置到模板类中
        template.setConnectionFactory(factory);
        // 设置key的序列化器
        template.setKeySerializer(new StringRedisSerializer());
        // 设置value的序列化器
        //使用Jackson 2，将对象序列化为JSON
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        //json转对象类，不设置默认的会将json转成hashmap
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);

        return template;
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
从缓存中移除相应的数据,

参考资料:
[基于Redis的Spring cache 缓存介绍](https://www.cnblogs.com/junzi2099/p/8301796.html)
[优雅的缓存解决方案--SpringCache和Redis集成(SpringBoot)](https://juejin.cn/post/6844903807646711821)
