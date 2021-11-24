---
title: "Windows批处理脚本bat连接到远程Linux服务器自动部署应用"
date: 2021-11-24T13:02:55+08:00
lastmod: 2021-11-24T13:02:55+08:00
draft: false
keywords: [Windows,自动部署,bat,Shell]
description: ""
tags: [Windows,自动部署,bat,Shell]
categories: [Windows,自动部署,bat,Shell]
author: "tureo"
---

# 需求背景
有时候需要把本地Windows修改的文件自动部署到远程Linux服务器，可使用Windows批处理脚本bat来实现连接远程服务器并自动执行Linux服务器上的脚本和命令实现。

# 脚本示例
以下使用一个简单的从Windows本地桌面复制一个静态页面压缩包dist.zip到远程Linux服务器的/appdata/myweb目录并解压为例。
## Windows本地bat脚本示例
复制和执行命令脚本文件，deploy-myweb-dist.bat
```bat
:: 关闭命令执行屏幕回显
:: 需要查看详情可以去掉下面这句，会有很多执行过程和结果输出到cmd窗口
@echo off

:: 定义相关变量，目录
:: 远程服务器目录
set REMOTEPATH=/appdata/myweb
:: 源文件，用户桌面的zip压缩文件
set ZIPFILE=%USERPROFILE%\Desktop\dist.zip
:: 需要执行的远程脚本命令
set SHFILE=deploy-dist.sh

:: 开始复制，输出提示信息到屏幕
echo %date% %time% scp to remote start

:: 复制本地zip压缩文件到远程服务器指定目录
:: 指定远程服务器ssh登录端口36000，一般为22
scp.exe -P 36000 %ZIPFILE% root@41.12.76.120:%REMOTEPATH%

echo %date% %time% scp to remote finished

:: 开始部署，输出提示信息到屏幕
echo %date% %time% remote deploy start

:: 登录到远程服务器执行命令
:: 指定远程服务器ssh登录端口36000，一般为22
ssh.exe -p 36000 root@41.12.76.120 "cd %REMOTEPATH%;sh %SHFILE% 1>/dev/null 2>&1"

echo %date% %time% remote deploy finished

::等待按键输入，没有下面这句cmd窗口会一闪而过
pause
```
ssh和scp命令需要输入账号和密码，可以配置成公钥访问，这样就可以免去每次输入密码的步骤。

## Linux服务器Shell脚本示例
备份解压部署脚本文件，/appdata/myweb/deploy-dist.sh 
```shell
#!/bin/bash

set -eux

# 定义相关变量，目录
PROJECTPATH=/appdata/myweb
TARGETDIR=${PROJECTPATH}/dist
BACKUPDIR=${PROJECTPATH}/dist-backup
ZIPFILE=${PROJECTPATH}/dist.zip

# 创建目录
mkdir -p ${PROJECTPATH} ${TARGETDIR} ${BACKUPDIR}

cd ${PROJECTPATH}
echo "`date` deploying" >> deploy-dist.log

#备份
cp -r ${TARGETDIR}/* ${BACKUPDIR}/

# 解压
unzip -o -d ${PROJECTPATH} ${ZIPFILE}

echo "`date` deployed" >> deploy-dist.log

```