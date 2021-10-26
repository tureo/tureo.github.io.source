---
title: "Redis不能远程访问"
date: 2021-10-26T16:35:14+08:00
lastmod: 2021-10-26T16:35:14+08:00
draft: false
keywords: [保护模式,远程访问,密码]
description: ""
tags: [redis]
categories: [redis]
author: "tureo"
---

# 问题
远程访问redis执行命令报错如下:
```
100.128.100.100:6379> flushall
(error) DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.
100.128.100.100:6379>
```

# 原因
redis默认开启了保护模式

# 两个解决办法
## 1.关闭保护模式
### 1）.临时关闭
登录redis服务器执行`redis/src/redis-cli`，连接成功后执行命令`CONFIG SET protected-mode no`关闭保护模式

### 2）.永久关闭
登录redis服务器修改配置文件,设置`protected-mode no`

## 2.设置密码访问
登录redis服务器修改配置文件设置登录密码`requirepass Dev@321^`

# 注意事项
允许远程访问需要修改redis配置
`vi /etc/redis.conf`
注释掉绑定127.0.0.1，允许远程登录：
`# bind 127.0.0.1 -::1`

修改配置文件后都需要重启redis
`systemctl restart redis`

