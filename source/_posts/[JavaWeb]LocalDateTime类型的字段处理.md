---
title: LocalDateTime 类型的字段处理
---
##### 前言

以下处理方案借助jackson依赖库

##### 返回值格式化
未格式化的 LocalDateTime 类型的字段 在返回给前端时,显示如:
```
"handleTime": "2020-06-03T11:10:50",
```
这并不是我们需要看到的数据格式,一般情况下我们希望时间格式是这样的："yyyy-MM-dd HH:mm:ss"，


##### 返回值格式化问题解决方案
```java
import com.fasterxml.jackson.annotation.JsonFormat;

@JsonFormat( pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime handleTime;
```


##### 前端传递LocalDateTime报错问题
LocalDateTime 类型的字段 前端在请求时发送类似 "yyyy-MM-dd HH:mm:ss"这样的字符串时,会提示类似错误:
```
"Failed to convert property value of type 'java.lang.String' to required type 'java.time.LocalDateTime' fr property 'handleTime'; nested exception is org.springframework.core.convert.ConversionFailedException: Failed to convert from type [java.lang.String] to type [@com.fasterxml.jackson.annotation.JsonFormat java.time.LocalDateTime] for value '2020-06-03 11:10:50'; nested exception is java.lang.IllegalArgumentException: Parse attempt failed for value [2020-06-03 11:10:50]",
```
#### 解决方案

1.在application.yml配置文件中加入jackson对时间格式化的配置
```yml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
```
在实体类的LocalDateTime属性上加入注解
```java
import org.springframework.format.annotation.DateTimeFormat;

@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime handleTime;
```