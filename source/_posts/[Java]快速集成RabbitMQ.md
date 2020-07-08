pom.xml
```
<!--RabbitMQ依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

配置信息
```java
package com.liyisoft.licenseprocess.mq;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 同步任务
 */
@Configuration
public class SyncDataMqConfig {
    /**
     * 同步任务的队列名称
     */
    static final String QUEUE_SYNC = "queue.sync";

    /**
     * 同步任务的交换机名称
     */
    public static final String EXCHANGE_QUEUE_SYNC = "exchange_queue_sync";

    /**
     * 同步任务 成功交换机
     */
    private static final String EXCHANGE_TOPIC_SYNC_SUCCESS = "exchange_topic_sync_success";

    /**
     * 同步任务 失败交换机
     */
    private static final String EXCHANGE_TOPIC_SYNC_FAIL = "exchange_topic_sync_fail";


    /**
     * 交换机与队列绑定的RoutingKey
     */
    public static final String ROUTING_SYNC = "*.sync";


    /**
     * 同步任务队列
     *
     * @return
     */
    @Bean(QUEUE_SYNC)
    public Queue queueSync() {
        //true 是否持久
        return new Queue(QUEUE_SYNC, true);
    }

    /**
     * 声明同步任务交换机
     *
     * @return
     */
    @Bean(EXCHANGE_QUEUE_SYNC)
    DirectExchange exchangeQueueSync() {
        return new DirectExchange(EXCHANGE_QUEUE_SYNC);
    }

    /**
     *
     * 声明同步任务成功交换机
     *
     * @return
     */
    @Bean(EXCHANGE_TOPIC_SYNC_SUCCESS)
    DirectExchange exchangeTopicSyncSuccess() {
        return new DirectExchange(EXCHANGE_TOPIC_SYNC_SUCCESS);
    }

    /**
     * 声明同步任务失败交换机
     *
     * @return
     */
    @Bean(EXCHANGE_TOPIC_SYNC_FAIL)
    DirectExchange exchangeTopicSyncFail() {
        return new DirectExchange(EXCHANGE_TOPIC_SYNC_FAIL);
    }

    /**
     * 同步任务队列和同步任务交换机绑定
     * @param queue 同步任务队列
     * @param exchange 同步任务交换机
     * @return
     */
    @Bean
    Binding syncBindingQueue(@Qualifier(QUEUE_SYNC) Queue queue, @Qualifier(EXCHANGE_QUEUE_SYNC) Exchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(ROUTING_SYNC).noargs();
    }
    
}

```

消费者
```
package com.liyisoft.licenseprocess.mq;


import com.alibaba.fastjson.JSON;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.liyisoft.licenseprocess.constant.HandleStatus;
import com.liyisoft.licenseprocess.entity.GatherCollectObjectInfo;
import com.liyisoft.licenseprocess.entity.LicenseMessage;
import com.liyisoft.licenseprocess.service.IGatherCollectObjectInfoService;
import com.liyisoft.licenseprocess.util.spider.TaskGetInfoDetail;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * 同步数据任务
 * @author fwenhua
 */
@SuppressWarnings("Duplicates")
@Component
@RabbitListener(queues = SyncDataMqConfig.QUEUE_SYNC)
@Slf4j
public class SyncConsumer {

    @Autowired
    private TaskGetInfoDetail taskGetInfoDetail;

    /**
     * 消息处理方法
     */
    @RabbitHandler
    public void process(GatherCollectObjectInfo gatherCollectObjectInfo) {
        try {
            log.info("接收到推送任务:{}", JSON.toJSONString(gatherCollectObjectInfo));
            taskGetInfoDetail.spiderAutoInput(gatherCollectObjectInfo.getCitizenIdcardNum(),gatherCollectObjectInfo.getAvatarImgProcessedAddr());
        }catch (Exception e){
            log.info("接收到推送任务异常",e);
        }
    }

    /**
     * 消息处理方法
     */
    @RabbitHandler
    public void process(LicenseMessage licenseMessage) {
        try {
            log.info("第三方推送,并爬入:{}", JSON.toJSONString(licenseMessage));
            //do something
        }catch (Exception e){
            log.info("第三方推送,并爬入异常",e);
        }
    }

}

```

controller
```java

@RestController
@RequestMapping("/test")
public class Controller {

    /**
     * mq消息发送处理类
     */
    @Autowired
    private RabbitTemplate rabbitTemplate;


    /**
     * 上传单条信息
     * @return 返回上传结果
     */
    @GetMapping("/uploadSingle")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "citizenIdcardNum", value = "身份证号")
    })
    @ApiOperation(value = "上传任务", notes = "上传任务",httpMethod = "GET")
    public ResponseEntity<?> uploadSingle(@Valid GatherCollectObjectInfo info) throws Exception {
        //下发到MQ
        rabbitTemplate.convertAndSend(SyncDataMqConfig.EXCHANGE_QUEUE_SYNC, SyncDataMqConfig.ROUTING_SYNC, info);
        return ResponseEntity.ok(Response.buildSuccess(gatherCollectObjectInfo,1));
    }

}

```