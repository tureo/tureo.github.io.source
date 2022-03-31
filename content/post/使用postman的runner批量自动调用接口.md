---
title: "使用postman的runner批量自动调用接口"
date: 2022-03-31T18:07:51+08:00
lastmod: 2022-03-31T18:07:51+08:00
draft: false
keywords: ["postman","测试","接口"]
description: ""
tags: ["postman","测试","接口"]
categories: ["postman","测试","接口"]
author: "tureo"
---

# 说明
有时候需要批量调用接口进行测试或者造数据，可以使用postman的runner进行。

# 示例
以下用`http post + json`的方式演示批量调用注册接口插入用户数据。
1. 左边选择`collection`，再选择需要批量调用的请求目录，点击右边的`run`。
![](/images/postman-runner/1.jpeg)

2. 中间勾选需要批量调用的`request`，可以多选。右边可以看到一些参数配置，可以设置每次请求的间隔时间等。
![](/images/postman-runner/2.jpeg)
请求的body形式为json，json的值需要设置为变量形式`{{变量名}}`，如下：
    ```JSON
    {
        "username": "{{username}}",
        "password": "{{password}}",
        "gender": {{gender}},
        "nickname": "{{nickname}}"
    }
    ```
    ![](/images/postman-runner/8.jpeg)


3. 添加json数据文件
json文件需要为一个数组，字段名对应body json中的变量名
users.json：
    ```JSON
    [
        {
            "username": "user1@abc.com",
            "password": "awsd456",
            "gender": 2,
            "nickname": "nick1"
        },
        {
            "username": "user2@abc.com",
            "password": "abc321",
            "gender": 1,
            "nickname": "nick2"
        }
    ]
    ```
    ![](/images/postman-runner/3.jpeg)

4. 看到选择的文件名，自动设置了迭代次数和文件类型。
![](/images/postman-runner/4.jpeg)

5. 可以预览json文件数据，一行代表一次请求数据。
![](/images/postman-runner/5.jpeg)

6. 运行所有任务。
![](/images/postman-runner/6.jpeg)

7. 查看运行结果。
![](/images/postman-runner/7.jpeg)

# 附录
## 如何快速批量生成json测试数据文件
打开[网址https://json-generator.com/](https://json-generator.com/)，
1. 左边是预设模版，右边是生成的结果数据区域，可点击`generate`生成，可点击`help`查看模板和函数说明
![](/images/postman-runner/9.jpeg)
![](/images/postman-runner/10.jpeg)

2. 示例
设置模板，`{{}}`中为函数或变量，`{{repeat(3)}}`表示生成三条数据，
    ```JSON
    [
    '{{repeat(3)}}',
        {
            "username": "user{{index()}}@abc.com",
            "password": "awsd456",
            "gender": '{{random(1,2)}}',
            "nickname": "nick{{index()}}"
        }
    ]
    ```
    生成的数据文件
    ```JSON
    [
        {
            "username": "user0@abc.com",
            "password": "awsd456",
            "gender": 2,
            "nickname": "nick0"
        },
        {
            "username": "user1@abc.com",
            "password": "awsd456",
            "gender": 1,
            "nickname": "nick1"
        },
        {
            "username": "user2@abc.com",
            "password": "awsd456",
            "gender": 2,
            "nickname": "nick2"
        }
    ]
    ```
    ![](/images/postman-runner/11.jpeg)
