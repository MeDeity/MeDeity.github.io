---
title: Git 常用命令记录
---

##### 撤销commit但未push的操作
```
# 获取上一条commit 的id
git reflog 
# --hard 表示同时撤销本地代码的修改
git reset [--hard] lastCommitId
```