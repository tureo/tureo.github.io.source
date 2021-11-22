---
title: "Mysql Insert Ignore Into把数据插入数据库"
date: 2021-11-22T21:52:45+08:00
lastmod: 2021-11-22T21:52:45+08:00
draft: false
keywords: [mysql,go,gin,gorm,toml,insert ignore]
description: ""
tags: [mysql,go,gin,gorm,toml]
categories: [mysql,go,gin,gorm,toml]
author: "tureo"
---

# 问题背景
最近在管控平台中的一个项目需要维护一个用户名列表，但管控平台不提供获取所有用户名的接口，这个用户名只能从管控平台前端页面调用后端接口的请求头中取，所以需要写一个api路由中间件，每当用户请求后端接口时：
**判断用户名是否存在本地数据库的user用户表中，如果存在，则忽略，不存在则插入**。
user表建表语句大致如下：
```SQL
CREATE TABLE `user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` varchar(255) NOT NULL DEFAULT '' COMMENT '用户名',
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uidx_name` (`username`) USING BTREE COMMENT '用户名需要唯一'
) ENGINE=InnoDB CHARSET=utf8 COMMENT='用户记录表';
```
# mysql直接插入的解决方法
使用mysql的insert ignore into语句
`insert ignore into user(username) values("xyz");`
不会影响原有的数据值，`updated_at`的值也不会变。
[参考：insert-ignore](https://www.yiibai.com/mysql/insert-ignore.html)
[扩展：Insert into，Replace Into，Insert ignore](https://www.cnblogs.com/stevin-john/p/4768904.html)

<details>
  <summary>其他想法</summary>
最开始想的是：先select再insert

1.先根据用户名查询数据库：
`select id,username from user where username="xyz";`
2.如果查询不到用户就插入：
`insert into user(username) values("xyz");`
这样就需要两个步骤，后来搜索找到上面只需要一个步骤的解决方法。
TODO：（用redis实现？）
</details>

--- 
# 使用mysql+go+gin+gorm的解决方法
由于项目框架主要基于mysql+go+gin+gorm搭建，所以可以实现如下写法：

路由文件，myweb/internal/router/router.go
```Go
package router
import (
	"myweb/internal/middleware"
	"github.com/gin-gonic/gin"
)
func InitRouter() *gin.Engine {
	r := gin.New()
	api := r.Group("/api")
	// 使用中间件，将用户名插入用户表中
	api.Use(middleware.InsertUser()) // 重点
	return r
}
```

中间件文件，myweb/internal/middleware/user.go
```Go
package middleware
import (
	// "myweb/internal/pkg/logger"
	"myweb/internal/service/user_service"
	"github.com/gin-gonic/gin"
)
// 将用户名插入用户表中，如果用户已经存在，则忽略
func InsertUser() gin.HandlerFunc {
	return func(c *gin.Context) {
        // 从请求头获取用户名
		username := c.GetHeader("X-User-Name")
		err := user_service.InsertIgnore(username) // 重点
		if err != nil {
			//logger.Error("user middleware InsertUser: failed to InsertIgnore user ", username, err.Error())
		}
		c.Next()
	}
}
```

service文件，myweb/internal/service/user_service/user.go
```Go
package user_service
import (
	"myweb/internal/model"
	// "myweb/internal/pkg/logger"
)
// 插入用户，如果用户已经存在，则忽略
func InsertIgnore(username string) (err error) {
	user := new(model.User)
	err = user.InsertIgnore(username)
	if err != nil {
		// logger.Error(err)
		return err
	}
	return err
}
```

model文件，myweb/internal/model/user.go
```Go 
package model
import (
	// "myweb/internal/pkg/logger"
	"myweb/internal/pkg/mysql"
	"github.com/pkg/errors"
)
// user表结构体
type User struct {
	ID         int
	Username   string
	CreatedAt  time.Time `gorm:"time"`
	UpdatedAt  time.Time `gorm:"time"`
}
//  插入用户，如果已经存在则忽略
func (t *User) InsertIgnore(username string) (err error) {
    // gorm v1还不支持ignore
    // gorm v2才支持用db.Clauses(clause.Insert{Modifier: "IGNORE"}).Create(&user)
    // 这里用原始sql执行
	_, err = mysql.GetDB().DB().Exec("INSERT IGNORE INTO user(username) VALUES(?);", username) // 重点
	if err != nil {
		// logger.Error("InsertIgnore user err: ", err)
		return errors.Wrap(err, "InsertIgnore user err")
	}
	return
}
```

gorm连接mysql数据库文件，myweb/internal/pkg/mysql/mysql.go
```Go
package mysql
import (
	"myweb/configs"
	"fmt"
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)
var db *gorm.DB
// InitMysql Setup initializes the database instance
func InitMysql() error {
	var err error
	db, err = gorm.Open("mysql", fmt.Sprintf("%s:%s@tcp(%s)/%s?charset=utf8&parseTime=True&loc=Local",
		configs.Get().Mysql.User,
		configs.Get().Mysql.Password,
		configs.Get().Mysql.Host,
		configs.Get().Mysql.Name))
	if err != nil {
		return err
	}
	//db.LogMode(true)
	gorm.DefaultTableNameHandler = func(db *gorm.DB, defaultTableName string) string {
		return configs.Get().Mysql.TablePrefix + defaultTableName
	}
	db.SingularTable(true)
	db.DB().SetMaxIdleConns(configs.Get().Mysql.MaxIdle)
	db.DB().SetMaxOpenConns(configs.Get().Mysql.MaxActive)
	return nil
}
// GetDB get database client
func GetDB() *gorm.DB {
	return db
}
```

读取配置文件，configs/configs.go
```Go
package configs
import (
	"myweb/internal/pkg/env"
	"time"

	"github.com/fsnotify/fsnotify"
	"github.com/spf13/viper"
)
var config = new(Config)
type Config struct {
	Server struct {
		HttpPort     int           `toml:"HttpPort"`
		ReadTimeout  time.Duration `toml:"ReadTimeout"`
		WriteTimeout time.Duration `toml:"WriteTimeout"`
		Version      string        `toml:"Version"`
		TmpFilePath  string        `toml:"TmpFilePath"`
	} `toml:"server"`

	Mysql struct {
		Host        string `toml:"Host"`
		User        string `toml:"User"`
		Password    string `toml:"Password"`
		Name        string `toml:"Name"`
		TablePrefix string `toml:"TablePrefix"`
		MaxIdle     int    `toml:"MaxIdle"`
		MaxActive   int    `toml:"MaxActive"`
	} `toml:"mysql"`

	Redis struct {
		Host        string        `toml:"Host"`
		Password    string        `toml:"Password"`
		RedisPrefix string        `toml:"RedisPrefix"`
		MaxIdle     int           `toml:"MaxIdle"`
		MaxActive   int           `toml:"MaxActive"`
		RedisDB     int           `toml:"RedisDB"`
		IdleTimeout time.Duration `toml:"IdleTimeout"`
	} `toml:"redis"`

	Language struct {
		Local string `toml:"Local"`
	} `toml:"language"`

	Log struct {
		Level         int    `toml:"Level"`
		LogFormatJson bool   `toml:"LogFormatJson"`
		TimeKey       string `toml:"TimeKey"`
		LevelKey      string `toml:"LevelKey"`
		NameKey       string `toml:"NameKey"`
		CallerKey     string `toml:"CallerKey"`
		MessageKey    string `toml:"MessageKey"`
		StackTrackKey string `toml:"StackTrackKey"`
		MaxSize       int    `toml:"MaxSize"`
		MaxBackups    int    `toml:"MaxBackups"`
		MaxAge        int    `toml:"MaxAge"`
	} `toml:"log"`
}

func Setup() {
	viper.SetConfigName(env.Active().Value() + "_configs")
	viper.SetConfigType("toml")
	viper.AddConfigPath("./configs")

	if err := viper.ReadInConfig(); err != nil {
		panic(err)
	}

	if err := viper.Unmarshal(config); err != nil {
		panic(err)
	}

	viper.WatchConfig()
	viper.OnConfigChange(func(e fsnotify.Event) {
		if err := viper.Unmarshal(config); err != nil {
			panic(err)
		}
	})
}

func Get() Config {
	return *config
}

```

配置文件，configs/dev_configs.toml
```TOML
[server]
    HttpPort = 8000
    ReadTimeout = 60
    WriteTimeout = 60
    Version = "1.0.0"
    TmpFilePath = "/tmp"

[language]
    local = "zh-cn"

[mysql]
    Host = "127.0.0.1:3306"
    Name = "myweb"
    Password = "123456"
    User = "root"
    TablePrefix = ""

[redis]
    Host = "127.0.0.1:6379"
    Password = ""
    MaxIdle = 30
    MaxActive = 30
    IdleTimeout = 200
    RedisDB = 0
    RedisPrefix = ""

[log]
    Level = 0
    LogFormatJson = true
    TimeKey = "time"
    LevelKey = "level"
    NameKey = "name"
    CallerKey = "caller"
    MessageKey = "message"
```