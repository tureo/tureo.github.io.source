---
title: "Go和swag自定义类型参数"
date: 2021-10-21T15:46:45+08:00
draft: true
tags: ["go", "swagger","swaggo","goswagger"]
categories: ["go","swagger"]
author: "tureo"
---

# 需求
swagger文档注释引用到自定义类型或外部的go结构体

# 存在问题
执行`swag init`后出现错误：ParseComment error in file create.go :cannot find type definition: multipart.FileHeader


# 问题原因
swag无法识别非swagger内置类型或者找不到包外部的go结构体

# 解决方法
1.生成swagger文档时设置解析外部依赖和深度，可根据需要调整parseDepth的值，值越大解析越慢
`swag init --parseDependency --parseInternal --parseDepth 2`
2.设置为其他swagger内置类型`swaggertype:"string"`
3.暂时忽略该字段`swaggerignore:"true"`

以上三种方法推荐第一种或第二种方法

