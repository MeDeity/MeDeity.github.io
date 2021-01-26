---
title: '运维技巧'
date: 2020-03-31
categories: 
  - 运维
tags:
  - 技巧
---

#### 查看服务器上端口是否开放

在ubuntu下:

```shell
#其中参数a->all p->programs[显示PID/Program name] -t->tcp -n->numeric[don't resolve names]
>netstat -aptn
```
