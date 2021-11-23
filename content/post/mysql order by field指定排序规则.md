---
title: "Mysql Order by Field指定排序规则"
date: 2021-11-23T12:49:25+08:00
lastmod: 2021-11-23T12:49:25+08:00
draft: false
keywords: []
description: ""
tags: [mysql,排序]
categories: [mysql]
author: "tureo"
---

# 需求背景
有时候业务要求我们从数据库查询数据需要根据某些字段排序，但不是按简单的升序或降序排列，而是有如下的特殊要求：
1. 先按审核状态自定义升序排序，再按id降序排序；
2. 审核状态排序要求：待审核（2）的排在最前面，审核驳回（3）的排中间，审核通过（1）的排在最后面；

mysql大致表结构如下：
```SQL
-- 审批表
CREATE TABLE `t_approval` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `status` int(11) NOT NULL DEFAULT '2' COMMENT '审批状态：1审批通过，2待审批，3审批驳回',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```
表测试数据：
|id|status|
|---|---|
|1|1|
|2|3|
|3|2|
|4|2|

# 解决方法
## 使用Mysql Order by Field指定排序规则
```SQL
-- 返回结果将按status的值先排序（status=2的排前面，status=3的排中间，status=1的排最后），再按id降序排序
SELECT id,status FROM t_approval WHERE status in (1,2,3) ORDER BY FIELD(status,2,3,1) ASC,id DESC;
```
查询结果如下：
|id|status|
|---|---|
|4|2|
|2|2|
|3|3|
|1|1|