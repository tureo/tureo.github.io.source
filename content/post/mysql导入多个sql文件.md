---
title: "Mysql导入多个sql文件"
date: 2021-10-21T16:36:46+08:00
draft: false
tags: ["mysql", "windows" ,"linux", "navicat"]
categories: ["mysql"]
author: "tureo"
---

# 需求
导入一个数据库的多个sql文件

# 存在问题
需要一个一个的执行导入sql文件语句

# 问题原因
mysql没有批量导入多个sql文件的命令

# 解决方法
## windows
1.合并多个sql文件为一个文件
powershell执行命令`get-content -Encoding utf8 *.sql > all.sql`
[参考](https://docs.microsoft.com/zh-cn/powershell/module/Microsoft.PowerShell.Management/Get-Content?view=powershell-7.1&viewFallbackFrom=powershell-3.0)

网上说的执行`type *.sql > all.sql`命令，会存在sql文件中文乱码问题，设置`chcp 65001`也无效，导致navicat等导入存在如下问题
```
[SQL] Query all start
[ERR] 1065 - Query was empty
[ERR] 
[SQL] Finished successfully
```

## linux
1.合并多个sql文件为一个文件
```shell
cat *.sql >> all.sql
```
2.单个文件写入多条sql语句
新建一个all.sql，`vim all.sql`，在里面写入需要执行的sql
```
source 1.sql
source 2.sql
source 53.sql
source 54.sql
```
## 登录mysql执行导入命令
`source all.sql`
mysql> source all.sql