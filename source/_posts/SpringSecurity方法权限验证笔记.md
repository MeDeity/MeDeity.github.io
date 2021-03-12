---
title: 'SpringSecurity方法权限验证笔记'
date: 2020-03-31
categories: 
  - 后端
tags:
  - 技巧
---

一、配置支持
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
  ...
}
```
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true) 注解
- prePostEnabled 方法执行前进行权限检查
- Secured  开启Secured,可以看作是prePostEnabled注解的封装版本(@PreAuthorize("hasRole('admin')")),并且支持多个角色
- PostAuthorize 比较少用.