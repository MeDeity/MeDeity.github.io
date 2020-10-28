---
title: Vue CLI3 项目引入px2rem适配方案笔记
---

一、引入px2rem适配方案

1. 需要用到哪些工具:
适配方案中需要用到lib-flexible及px2rem-loader

```
npm i lib-flexible
npm install px2rem-loader -d
```

2. 各个工具起到什么作用

3. 如何配置
在main.js中引入

```
import 'lib-flexible/flexible'
```

在vue.config.js文件中加入

```js
...
chainWebpack(config) {
    // set px2rem-loader
    config.module
      .rule('scss')
      .oneOf('vue')
      .resourceQuery(/\?vue/)
      .use('px2rem')
      .loader('px2rem-loader')
      // this makes it work. vue cli3 添加 px2rem-loader 样式嵌套问题 https://www.jianshu.com/p/1613b599d5be
      // 在 scss 里使用样式嵌套的时候出问题了 "px2rem-loader", it will report "undefined missing '}'" error
      .before('postcss-loader')
      .options({
        emUnit: 160
      })
},
...
```

4. 有什么特殊问题需要特殊处理

关于emUnit的设置
