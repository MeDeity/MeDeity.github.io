---
title: Vue项目的部署 笔记
---
### 本地预览

dist 目录需要启动一个 HTTP 服务器来访问 (除非你已经将 publicPath 配置为了一个相对的值)，所以以 file:// 协议直接打开 dist/index.html 是不会工作的。在本地预览生产环境构建最简单的方式就是使用一个 Node.js 静态文件服务器，例如 serve：

```shell
npm install -g serve
# -s 参数的意思是将其架设在 Single-Page Application 模式下
serve -s dist
```

[Vue项目的部署](https://cli.vuejs.org/zh/guide/deployment.html#docker-nginx)
