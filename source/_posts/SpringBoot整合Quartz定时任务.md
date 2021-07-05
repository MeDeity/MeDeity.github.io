---
title: 'SpringBoot整合Quartz定时任务'
date: 2021-03-01
categories: 
  - 后端
tags:
  - 技巧
---
一、依赖引入

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

二、Quartz相关的属性配置

```yml
# spring的datasource等配置未贴出
spring:
  quartz:
      # 将任务等保存化到数据库
      job-store-type: jdbc
      # 程序结束时会等待quartz相关的内容结束
      wait-for-jobs-to-complete-on-shutdown: true
      # QuartzScheduler启动时更新己存在的Job,这样就不用每次修改targetObject后删除qrtz_job_details表对应记录
      overwrite-existing-jobs: true
      properties:
        org:
          quartz:
            # scheduler相关
            scheduler:
              # scheduler的实例名
              instanceName: scheduler
              instanceId: AUTO
            # 持久化相关
            jobStore:
              class: org.quartz.impl.jdbcjobstore.JobStoreTX
              driverDelegateClass: org.quartz.impl.jdbcjobstore.StdJDBCDelegate
              # 表示数据库中相关表是QRTZ_开头的
              tablePrefix: QRTZ_
              useProperties: false
            # 线程池相关
            threadPool:
              class: org.quartz.simpl.SimpleThreadPool
              # 线程数
              threadCount: 10
              # 线程优先级
              threadPriority: 5
              threadsInheritContextClassLoaderOfInitializingThread: true
```

三、定时任务及触发器配置

```java
import org.quartz.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 新增定时器，需要手动清空qrtz开头的表两次?
 */
@Configuration
public class QuartzConfig {

    @Bean(value = "demo-quartz-job-detail")
    public JobDetail demoQuartzJobDetail(){
        JobDetail jobDetail = JobBuilder.newJob(DemoQuartzJob.class)
                .withIdentity("demoQuartzJob", "demoQuartzJobGroup")
                .storeDurably()
                .build();
        return jobDetail;
    }

    @Bean
    public Trigger demoQuartzJobTrigger(@Qualifier(value = "demo-quartz-job-detail") JobDetail jobDetail) {
        Trigger trigger = TriggerBuilder.newTrigger()
                .forJob(jobDetail)
                .withIdentity("demoQuartzJobTrigger", "demoQuartzJobTriggerGroup")
                .startNow()
                .withSchedule(CronScheduleBuilder.cronSchedule("* * * * * ?"))
                .build();
        return trigger;
    }
}
```

四、测试用的定时任务

```java
import lombok.extern.slf4j.Slf4j;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.stereotype.Component;


@Slf4j
@Component
public class DemoQuartzJob extends QuartzJobBean {
    /**
     * Execute the actual job. The job data map will already have been
     * applied as bean property values by execute. The contract is
     * exactly the same as for the standard Quartz execute method.
     *
     * @param context 上下文
     * @see #execute
     */
    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        log.info("测试用的定时任务执行");
    }
}
```

[Quartz数据库建表sql](https://github.com/quartz-scheduler/quartz/tree/9f9e400733f51f7cb658e3319fc2c140ab8af938/quartz-core/src/main/resources/org/quartz/impl/jdbcjobstore)
