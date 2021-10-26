---
title: "内网通过安装squid代理访问公网"
date: 2021-10-26T17:41:07+08:00
lastmod: 2021-10-26T17:41:07+08:00
draft: false
keywords: []
description: ""
tags: [squid,docker,代理,容器]
categories: [squid,docker]
author: "tureo"
---

由于内网机器不能直接访问公网，需要一台机器搭建代理服务器，其他机器通过代理服务器访问公网，这里基于docker安装squid代理实现

创建相关目录：
```shell
mkdir -p /var/spool/squid # 创建缓存目录
mkdir -p /var/log/squid  # 创建日志目录
mkdir -p /etc/squid  # 创建配置目录
```

编辑`vi /etc/squid/squid.conf`配置文件，加入如下内容

![](/images/squid-docker/1.png)

如果不添加`hosts_file /etc/hosts`，squid将不会使用hosts文件的映射，导致curl出现找不到主机名的错误：

![](/images/squid-docker/2.png)

编辑hosts文件`vi /etc/hosts`，输入主机对应域名映射

启动容器
```shell
docker run --name squid -d --restart=always \
  --publish 3128:3128 \
  --volume /etc/squid/squid.conf:/etc/squid/squid.conf \
  --volume /var/spool/squid:/var/spool/squid \
  --volume /var/log/squid:/var/log/squid \
  --volume /etc/hosts:/etc/hosts \
  sameersbn/squid
```

查看容器运行情况
`docker ps`
![](/images/squid-docker/3.png)

