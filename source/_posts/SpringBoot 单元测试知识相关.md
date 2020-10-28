---
title: SpringBoot单元测试知识相关
---
SpringBoot提供了一个 @SpringBootTest 注解用于测试 SpringBoot 应用,通过该注解可以测试需要Spring上下文的方法.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTest {

}
```

@SpringBootTest 有以下参数可以配置

1. webEnvironment 指定Web应用环境

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
