---
title: "MongoDB联表查询分组取第一条数据"
date: 2022-04-02T14:12:41+08:00
lastmod: 2022-04-02T14:12:41+08:00
draft: false
keywords: ["MongoDB","lookup","group"]
description: ""
tags: ["MongoDB","联表","分组查询"]
categories: ["MongoDB"]
author: "tureo"
---

# 说明
MongoDB多表关联获取每个分组的第一条数据。

# 示例
## 原始数据
test_party
![](/images/mongodb-lookup-group/4.jpeg)
```JSON
// 1
{
    "_id": ObjectId("623c0d285bdb127d373739d2"),
    "createAt": ISODate("2022-03-24T06:18:16.812Z"),
    "updateAt": ISODate("2022-03-24T06:18:16.812Z")
}
// 2
{
    "_id": ObjectId("6243b3f9ef71cbde1617cbfe"),
    "createAt": ISODate("2022-03-23T09:05:03.304Z"),
    "updateAt": ISODate("2022-03-23T09:05:03.304Z")
}

```
test_party_user
![](/images/mongodb-lookup-group/5.jpeg)
```JSON
// 1
{
    "_id": ObjectId("6243b6acef71cbde1617cc01"),
    "createAt": ISODate("2022-03-30T01:47:24.104Z"),
    "updateAt": ISODate("2022-03-30T01:47:24.104Z"),
    "party_id": ObjectId("6243b3f9ef71cbde1617cbfe"),
    "user_id": ObjectId("623af0d9aa982a23824ca883")
}
// 2
{
    "_id": ObjectId("6243b8b1ef71cbde1617cc29"),
    "createAt": ISODate("2022-03-30T01:56:01.166Z"),
    "updateAt": ISODate("2022-03-30T01:56:01.166Z"),
    "party_id": ObjectId("6243b3f9ef71cbde1617cbfe"),
    "user_id": ObjectId("623b08186bcfdca2a8753f1e")
}
// 3
{
    "_id": ObjectId("6243b81cef71cbde1617cc1e"),
    "createAt": ISODate("2022-03-30T01:53:32.961Z"),
    "updateAt": ISODate("2022-03-30T01:53:32.961Z"),
    "party_id": ObjectId("6243b3f9ef71cbde1617cbfe"),
    "user_id": ObjectId("623b0849ec0e4c6af559b847")
}
```
test_user_picture
![](/images/mongodb-lookup-group/6.jpeg)
```JSON
// 1
{
    "_id": ObjectId("623ade40e58d9c0eeabb8447"),
    "createAt": ISODate("2022-03-23T08:45:52.534Z"),
    "updateAt": ISODate("2022-03-23T08:45:52.534Z"),
    "user_id": ObjectId("623af0d9aa982a23824ca883"),
    "img_url": "/userImg/623af0d9aa982a23824ca883/1.jpg"
}
// 2
{
    "_id": ObjectId("623ade42e58d9c0eeabb844b"),
    "createAt": ISODate("2022-03-23T08:45:54.612Z"),
    "updateAt": ISODate("2022-03-23T08:45:54.612Z"),
    "user_id": ObjectId("623b08186bcfdca2a8753f1e"),
    "img_url": "/userImg/623b08186bcfdca2a8753f1e/A.jpg"
}
// 3
{
    "_id": ObjectId("623ade41e58d9c0eeabb8449"),
    "createAt": ISODate("2022-03-23T08:45:53.574Z"),
    "updateAt": ISODate("2022-03-23T08:45:53.574Z"),
    "user_id": ObjectId("623af0d9aa982a23824ca883"),
    "img_url": "/userImg/623af0d9aa982a23824ca883/2.jpg"
}
```

## 单表获取分组的第一条记录
根据用户分组获取用户的第一张图片。
查询语句：
```MongoDB
// select _id,fisrt_img_url from test_user_picture
// 只查看_id和fisrt_img_url字段（_id字段默认包含）
// 把test_party当做源表
db.test_user_picture.aggregate([
    // group by picture.user_id
    // 按picture.user_id分组取每组的第一条picture.img_url
    {
        $group: {
            _id: '$user_id', // key必须为_id
            fisrt_img_url: {
                // 重命名为fisrt_img_url
                $first: '$img_url' // 取第一条img_url数据
            }
        }
    }
])
```
结果：
![](/images/mongodb-lookup-group/1.jpeg)

## 多表关联获取分组的第一条记录
根据聚会里的用户分组获取用户的第一张图片。
查询语句：
```JSON
// from test_party
// 把test_party当做源表
db.test_party.aggregate([
    // join test_party_user on test_party._id=test_party_user.party_id
    // 关联test_party_user，取为别名user
    {
        $lookup: 
        {
            from: "test_party_user",
            localField: "_id",
            foreignField: "party_id",
            as: "user"
        }
    },
    // join test_user_picture on test_party_user.user_id=test_user_picture.user_id
    // 关联test_user_picture，取为别名picture
    {
        $lookup: 
        {
            from: "test_user_picture",
            localField: "user.user_id",
            foreignField: "user_id",
            as: "picture"
        }
    },
    // order by test_user_picture.createAt asc
    // 按picture.createAt升序排
    {
        $sort: {
            "picture.createAt": 1
        }
    },
    //     展开user
    //     重要，不展开的话会被内嵌到文档中，导致结果不对
    {
        $unwind: "$user"
    },
    // 展开picture
    // 重要，不展开的话会被内嵌到文档中，导致结果不对
    {
        $unwind: "$picture"
    },
    // group by picture.user_id
    // 按picture.user_id分组取每组的第一条picture.img_url
    {
        $group: {
            _id: '$picture.user_id', // key必须为_id
            fisrt_img_url: {
                // 重命名为fisrt_img_url
                $first: '$picture.img_url' // 取第一条picture.img_url数据
            }
        }
    },
    // select _id,fisrt_img_url
    // 只查看_id和fisrt_img_url字段（_id字段默认包含）
    {
        $project: {
            fisrt_img_url: 1
        },
        
    }
])
```
结果：
![](/images/mongodb-lookup-group/2.jpeg)

### 展开和不展开的区别
展开：
如果想得到类似mysql的group效果，必须要展开。
![](/images/mongodb-lookup-group/2.jpeg)
不展开：
不展开无法得到预期的效果。
![](/images/mongodb-lookup-group/3.jpeg)