### 如何访问

[博客地址连接](https://medeity.github.io/)

#### 特别注意
需要更新时请clone 该仓库,并切换到hexo分支
```
git clone https://github.com/MeDeity/MeDeity.github.io.git
git checkout hexo
```

由于hexo是基于nodejs,首次使用时强烈建议执行以下命令
```
# hexo脚手架
npm install -g hexo-cli
# 安装必要的依赖
npm install
```

需要更新BLOG时,可以直接在source目录下新增md文档.或者执行以下命令
```
# 效果为在source目录下新建一个md文档
hexo new post "new blog name"
# 生成静态文件
hexo generate/hexo g
# 开启本地服务 http://localhost:4000 可访问博客
hexo server/hexo s
```

发布更新
```
# 发布到github.io主页上
hexo deploy
```
关于搭建教程可以参考以下文章:

扩展阅读:
[Hexo and GitHub Pages 博客搭建](https://juejin.im/post/5c08beb65188257c3045d61e)



