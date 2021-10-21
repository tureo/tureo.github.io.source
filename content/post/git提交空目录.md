---
title: "Git提交空目录"
date: 2021-10-21T14:07:23+08:00
draft: false
tags: ["git", "shell"]
categories: ["git"]
author: "tureo"
---

[toc]

# 需求
需要提交一个项目目录结构到远程git仓库上

# 存在问题
空目录不能commit和push

# 问题原因
git默认不能提交空目录

# 解决方法
## 在每个空目录下创建.gitkeep文件
### 批量快速生成.gitkeep文件shell脚本
```shell
# 找到目录为空的文件夹并创建内容为空的.gitkeep文件
find . -type d -empty -exec touch {}/.gitkeep \;
```