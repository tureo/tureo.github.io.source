---
title: "Go基准测试windows平台执行命令"
date: 2021-10-21T15:28:18+08:00
draft: false
tags: ["go", "测试" ,"基准测试", "windows"]
categories: ["go"]
author: "tureo"
---

# 需求
在windows机器上进行go程序的基准测试

# 存在问题
linux正常的命令`go test -bench=.`在windows上执行结果异常

# 问题原因
windows的命令行解析不一样

# 解决方法
点两边添加双引号即可，执行`go test -bench="."`
