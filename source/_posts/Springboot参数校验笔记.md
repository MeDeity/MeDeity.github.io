---
title: 'Springboot参数校验笔记'
date: 2020-11-18
categories: 
  - 后端-Java
tags:
  - 技巧
  - TODO
---
#### 简介(为了解决什么痛点)

#### 环境准备

```maven
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 开启校验

一般在controller层可以开启校验(Service层好像不能生效?),在对应的类或者接口上添加@Validated注解即可开启.
Controller 类上需要添加注解  `@Validated` 不然不会生效

一、以下是单个参数的校验使用示例

```java
@RestController
@Validated
public class ThirdPartController {
   /**
     * 人脸核身功能
     * @param image1Url 基准图片地址
     * @param image2Url 待校验的图片地址
     * @param threshold 阀值
     * @return          人脸核身结果
     */
    @GetMapping("/compareImages")
    public ResponseEntity<Response> compareImages(@NotNull(message = "人像对比基准图片不能为空") String image1Url, @NotNull(message = "人像对比校验图片不能为空") String image2Url, Double threshold){
        return thirdPartService.compareImages(image1Url,image2Url,threshold);
    }
}
```

#### 分组校验

#### 自定义校验

#### 校验模式

#### 参考

[SpringBoot 参数校验的方法](https://www.cnblogs.com/mooba/p/11276062.html)
[Spring Boot 参数校验](https://www.cnblogs.com/mrchenzheng/p/12165757.html)
