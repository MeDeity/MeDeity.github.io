---
title: MapStruct对象映射使用笔记
date: 2020-11-16
categories: 工具
tags: MapStruct
---

### MapStruct简介(是什么、有什么作用)

### MapStruct有什么优缺点

### MapStruct引入

```maven
<dependency>
  <groupId>org.mapstruct</groupId>
  <artifactId>mapstruct-jdk8</artifactId>
  <version>1.3.1.Final</version>
</dependency>
<dependency>
  <groupId>org.mapstruct</groupId>
  <artifactId>mapstruct-processor</artifactId>
  <version>1.3.1.Final</version>
</dependency>
```

笔者之前引入的是

```
<dependency>
  <groupId>org.mapstruct</groupId>
  <artifactId>mapstruct</artifactId>
  <version>1.3.1.Final</version>
</dependency>
```

然后死活报(No qualifying bean of type vailable: expected at least 1 bean ...)的错误.换成mapstruct-jdk8就通了.原因目前未知

### 工具类封装

```

```

### MapStruct常用示例

#### 常规使用

```

```

### MapStruct有哪些坑

//TODO

### 扩展(VO/DO/DTO/PO相关概念)

VO（View Object）：视图对象，用于展示层，它的作用是把某个指定页面（或组件）的所有数据封装起来

DTO（Data Transfer Object）：数据传输对象，这个概念来源于J2EE的设计模式，原来的目的是为了EJB的分布式应用提供粗粒度的数据实体，以减少分布式调用的次数，从而提高分布式调用的性能和降低网络负载，但在这里，我泛指用于展示层与服务层之间的数据传输对象。

DO（Domain Object）：领域对象，就是从现实世界中抽象出来的有形或无形的业务实体。

PO（Persistent Object）：持久化对象，它跟持久层（通常是关系型数据库）的数据结构形成一一对应的映射关系，如果持久层是关系型数据库，那么，数据表中的每个字段（或若干个）就对应PO的一个（或若干个）属性。

[官方文档](https://mapstruct.org/documentation/stable/reference/html/)
[mapstruct最佳实践](https://www.cnblogs.com/vcmq/archive/2020/03/21/12542336.html)
[MapStruct 使用简介](https://juejin.im/post/6844904094868439048)
[领域驱动设计系列文章（2）——浅析VO、DTO、DO、PO的概念、区别和用处](https://www.cnblogs.com/qixuejia/p/4390086.html)
