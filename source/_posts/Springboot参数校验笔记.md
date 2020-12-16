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

在controller层或者service层上都可以开启校验,在对应的类或者接口上添加@Validated注解即可开启.由于controller的注解相对较多,笔者建议参数校验放置到service层上

#### 分组校验

#### 自定义校验

#### 参考

[SpringBoot 参数校验的方法](https://www.cnblogs.com/mooba/p/11276062.html)
