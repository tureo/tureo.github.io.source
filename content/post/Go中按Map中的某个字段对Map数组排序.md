---
title: "Go中按Map中的某个字段对Map数组排序"
date: 2022-03-31T17:09:14+08:00
lastmod: 2022-03-31T17:09:14+08:00
draft: false
keywords: ["go","map","slice","sort"]
description: ""
tags: ["go","map","slice","sort"]
categories: ["go","map","slice","sort"]
author: "tureo"
---

# 说明
有时候需要按Map中的某个字段对Map数组排序，使用标准包的sort.Sort即可以自定义排序规则，需要实现相关的Len、Swap、Less接口。

# 示例代码
```Go
package main

// 定义数据类型
type users []map[string]interface{}

// 实现该数据类型排序的三个接口
func (s users) Len() int {
	return len(s)
}
func (s users) Swap(i, j int) {
	s[i], s[j] = s[j], s[i]
}
func (s users) Less(i, j int) bool {
	return s[i]["number"].(int) < s[j]["number"].(int)
}

// 测试map数组排序
func MapSliceSort() {
	resultArr := []map[string]interface{}{
		{
			"number":   2,       // 名次
			"nickname": "user1", // 昵称
			"id":       "x001",  // id
		},
		{
			"number":   1,       // 名次
			"nickname": "user2", // 昵称
			"id":       "x002",  // id
		},
		{
			"number":   3,       // 名次
			"nickname": "user3", // 昵称
			"id":       "x003",  // id
		},
	}
	fmt.Println("unordered:", resultArr)
	// 排序
	sort.Sort(users(resultArr))
	fmt.Println("ordered:", resultArr)
}

func main() {
	MapSliceSort()
}
```