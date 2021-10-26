---
title: "错误启动jenkins.war包会删除当前目录下所有文件"
date: 2021-10-26T17:15:16+08:00
lastmod: 2021-10-26T17:15:16+08:00
draft: false
keywords: [jenkins,错误]
description: ""
tags: [jenkins]
categories: [jenkins]
author: "tureo"
---

# 正确启动
正确的命令：`java -jar jenkins.war --httpPort=8080`

正常启动：
<br>
![](/images/jenkins-deleted-file/1.png)

文件还在:
<br>
![](/images/jenkins-deleted-file/2.png)

# 错误启动
错误的命令：`java -jar jenkins.war --httpPort=8080 .`

**（错误的命令后面多了个空格和.）**

错误的命令启动jenkins.war包会删除当前目录下（包括子目录）本地文件，测试如下：

1. 目录结构如下：
<br>
![](/images/jenkins-deleted-file/3.png)
<br>

2. 两个仓库先拉取代码
![](/images/jenkins-deleted-file/4.png)
<br>
![](/images/jenkins-deleted-file/5.png)

3. 执行错误的命令行
<br>
![](/images/jenkins-deleted-file/6.png)

    输出如下：
    可以看到日志中提示会去删除部分文件，有权限的已经被删除了，没权限的就会报错
    ![](/images/jenkins-deleted-file/7.png)

    ![](/images/jenkins-deleted-file/8.png)

# 参考历史命令
![](/images/jenkins-deleted-file/9.png)

![](/images/jenkins-deleted-file/10.png)

