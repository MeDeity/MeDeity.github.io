---
title: 使用TypeScript开发NodeJs后端项目
---
##### 简介
暂无
##### 前期准备
开发环境准备
```json
# 添加typescript支持,这里推荐全局安装
npm install -g typescript (全局安装)/ npm install typescript -d(局部安装)
# 自动监视node项目的改动,并自动重启环境(ctrl+s 也可以触发重启),非常适用于开发调试阶段
npm install -g nodemon /npm install nodemon -d 
# 线上部署监控解决方案 (相似功能 forever) -一般用于生产环境
npm install -g pm2 /npm install pm2
# 线下监控解决方案 -一般用于开发调试阶段
npm install -g supervisor /npm install supervisor -d 
# node不能直接运行ts代码，需要使用tsc将ts代码编译成js代码,解决方案: ts-node则包装了node，它可以直接的运行ts代码
npm install ts-node -d
# 可以说是typescript版的nodemon了,可以使用在开发阶段
npm install ts-node-dev -d

```
##### 项目初始化
1. 这里我们创建一个 node-typescript-demo 项目作为演示,执行
```
npm init -y
```
命令生成package.json文件


##### 项目结构
|-- project-name          # 项目名称
    |-- assets            # 存放项目的图片、视频等资源文件
    |-- bin               # CLI命令入口，require('../dist/cli')，注意文件头加上#!/usr/bin/env node
    |-- dist              # 项目使用ts开发，dist为编译后文件目录，注意package.json中main字段要指向dist目录
    |-- docs              # 存放项目相关文档
    |-- scripts           # 对应package.json中scripts字段需要执行的脚本文件
    |-- src               # 源码目录，注意此目录只放ts文件，其他文件如json、模板等文件放templates目录
        |-- sub etc       # 众多子目录
        |-- cli.ts        # cli入口文件
        |-- index.ts      # api入口文件
    |-- templates         # 存放json、模板等文件
    |-- tests             # 测试文件目录
    |-- typings           # 存放ts声明文件，主要用于补充第三方包没有ts声明的情况
    |-- .eslintignore     # eslint忽略规则配置
    |-- .eslintrc.js      # eslint规则配置
    |-- .gitignore        # git忽略规则
    |-- package.json      # 
    |-- README.md         # 项目说明
    |-- tsconfig.json     # typescript配置，请勿修改
> 该项目结构出处[Node.js项目TypeScript改造指南](https://juejin.im/post/5de4867f51882573135415dd)

##### 开发过程中的调试

##### 应用Docker部署


[Node.js项目TypeScript改造指南](https://juejin.im/post/5de4867f51882573135415dd)


