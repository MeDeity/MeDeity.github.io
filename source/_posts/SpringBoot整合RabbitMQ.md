---
title: 'SpringBoot整合RabbitMQ'
date: 2020-03-31
categories: 
  - 后端
tags:
  - Java
---
引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

rabbitmq相关配置属性

```yml
  rabbitmq:
    username: guest
    password: guest #请自行修改用户名及密码
    addresses: 127.0.0.1:5672
    cache:
      connection:
        # 缓存连接模式,默认一个连接,多个channel
        mode: channel
```

Direct直连模式交换机类型
DemoMqConfig MQ配置信息(主要创建消息队列、交换机实体并指定他们的名称以及绑定KEY(ROUTING_KEY))

```java
import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


/**
 * Direct直连模式交换机类型
 * create by fengwenhua at 2021-2-26 16:20:32
 */
@Configuration
public class DemoMqConfig {
    /**
     * 指定消息队列的名称
     */
    public final static String DEMO_QUEUE_NAME = "queue-demo";
    /**
     * 指定交换机的名称
     */
    public final static String DEMO_EXCHANGE_NAME = "exchange-demo";

    /**
     * 交换机可以绑定多个消息队列;消息发布者通过指定交换机及ROUTING_KEY,
     * MQ才知道往哪个消息队列中推送数据
     */
    public final static String DEMO_ROUTING_KEY = "routing-key-demo";

    /**
     * 创建一个消息队列
     * @return 返回消息队列实体类
     */
    @Bean(value = DEMO_QUEUE_NAME)
    public Queue queue(){
        //true if we are declaring a durable queue (the queue will survive a server restart)
        return new Queue(DEMO_QUEUE_NAME,true);
    }

    /**
     * 创建一个直连交换机
     * @return 返回一个直连交换机实体类
     */
    @Bean(value = DEMO_EXCHANGE_NAME)
    public DirectExchange exchange(){
        return new DirectExchange(DEMO_EXCHANGE_NAME);
    }

    @Bean
    public Binding exchangeBindQueue(@Qualifier(value = DEMO_QUEUE_NAME) Queue queue,@Qualifier(value = DEMO_EXCHANGE_NAME) DirectExchange exchange){
        //直接使用消息队列的名称 作为 routingKey
        return BindingBuilder.bind(queue).to(exchange).with(DEMO_ROUTING_KEY);
    }

}

```

DemoConsumer队列消息 消费者通过RabbitListener指定处理的是哪个消息队列的消息

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * 消费者[@RabbitListener(queues = DemoMqConfig.DEMO_QUEUE_NAME)]
 */
@Component
@RabbitListener(queues = DemoMqConfig.DEMO_QUEUE_NAME)
@Slf4j
public class DemoConsumer {

    /**
     * 消费者处理方法
     * {@link RabbitHandler} 该注解将方法标记为用{@link RabbitListener}注释的类中的Rabbit消息侦听器的目标
     * @param message 消息可以是任何类型的Object数据;但是需要序列化
     */
    @RabbitHandler
    public void process(String message){
        log.info("消费者监听到发布者消息:{}",message);
    }
}

```

消息发布者

```java
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest(classes = DemoApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class RabbitMqTest {
    /**
     * mq消息发送处理类
     */
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testRabbitMq(){
        rabbitTemplate.convertAndSend(DemoMqConfig.DEMO_EXCHANGE_NAME,DemoMqConfig.DEMO_ROUTING_KEY,"可以是任何类型的Object数据(但需要序列化)");
    }
}

```

最后执行测试方法

```
INFO 11556 --- x.mq.DemoConsumer     : 消费者监听到发布者消息:可以是任何类型的Object数据(但需要序列化)
```

成功接收到消息。说明Direct直连模式交换机类型成功工作
