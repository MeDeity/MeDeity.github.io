---
title: NodeJs Express 使用笔记
date: 2020-04-01 16:12:25
categories: 后端
tags: NodeJs Express
---
#### 静态文件
Express 提供了内置的中间件 express.static 来设置静态文件如：图片， CSS, JavaScript 等
你可以使用 express.static 中间件来设置静态文件路径。例如，如果你将图片， CSS, JavaScript 文件放在 public 目录下，你可以这么写：
```
app.use('/public', express.static('public'));
```