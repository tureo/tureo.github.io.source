---
title: "Mysql导入多个sql文件"
date: 2021-10-21T16:36:46+08:00
draft: false
tags: ["mysql", "windows" ,"linux", "navicat"]
categories: ["mysql"]
author: "tureo"
---

# 需求背景
因mysql数据库表初始化或迁移需要导入一个数据库的多个sql文件，而mysql没有批量导入多个sql文件的命令，一个一个的执行导入sql文件语句很繁琐而且容易错漏，因此需要一个可以快速导入多个sql文件的方法。

# 解决方法
## windows
1. 合并多个sql文件内容到一个文件
`cmd`执行命令`type *.sql > all.sql`或者`powershell`执行命令`get-content -Encoding utf8 *.sql > all.sql`
[type命令参考](https://www.maixj.net/ict/windows-type-19106)
[Get-Content命令参考](https://docs.microsoft.com/zh-cn/powershell/module/Microsoft.PowerShell.Management/Get-Content?view=powershell-7.1&viewFallbackFrom=powershell-3.0)
2. 导入sql
    登陆mysql执行命令
    ```
    mysql> use my_test_db
    mysql> source D:/sql/all.sql
    ```
    **注意路径，分割符为/，末尾没有分号**    

## linux
1. 合并多个sql文件内容到一个文件
- cat加重定向实现合并
    ```shell
    cat *.sql > all.sql
    ```
- 单个文件写入多条sql语句
    新建一个all.sql，`vi all.sql`，在里面写入需要执行的sql
    ```shell
    source /app/sql/1.sql
    source /app/sql/2.sql
    source /app/sql/53.sql
    source /app/sql/54.sql
    ```
2. 导入sql
    登录mysql执行导入命令
    ```shell
    mysql> use my_test_db
    mysql> source /app/sql/all.sql
    ```

## 中文乱码和导入出错

可能会存在sql文件中文乱码（设置`chcp 65001`也无效）或导入sql出现如下错误情况：
- navicat导入报错：
    ```
    [SQL] Query all start
    [ERR] 1065 - Query was empty
    [ERR] 
    [SQL] Finished successfully
    ```
    此问题可通过拷贝all.sql文件全部sql语句到navicat查询框中执行进行解决。
<br/>

- 命令行导入报错：
    ```
    ERROR:
    ASCII '\0' appeared in the statement, but this is not allowed unless option --binary-mode is enabled and mysql is run in non-interactive mode. Set --binary-mode to 1 if ASCII '\0' is expected. Query: 'ÿþC'.
    ```
    此问题可网上搜索解决办法，如：复制all.sql文件全部sql语句到a.sql中，然后再导入a.sql（**未验证**）