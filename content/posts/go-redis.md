---
title: "[Go] redis库使用"
subtitle: ""
date: 2022-10-18T17:11:03+08:00
lastmod: 2022-10-18T17:11:03+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 环境

- redis库：github.com/go-redis/redis/v8

## 需求

使用redis存储Unity Meta guid信息，用于提交svn时检查guid是否存在，如存在则禁止提交svn。

redis键设计

- key/value
  - key为guid，path为Unity资源文件路径（主要用于查询guid是否存在）
- hash
  - key为path的md5值，value有guid、path，图片textureType（主要用于删除，因svn hook在delete时无法读取文件内容，所以使用的path）

## 实现

- `global.go`

```go
package global

import (
	"github.com/go-redis/redis/v8"
)

var (
	Redis *redis.Client
)
```

- `main.go`片段

```go
// Redis连接
func initRedis() *redis.Client {
	rdb := redis.NewClient(&redis.Options{
		Addr:     "127.0.0.1:6379",
		Password: "",
		DB:       0,
	})

	result := rdb.Ping(context.Background())
	log.Println("redis ping:", result.Val())
	if result.Val() != "PONG" {
		return nil
	}
	return rdb
}

func main() {
	  global.Redis = initRedis()
}
```

- Redis调用示例

```go
// Gin中使用Redis Get
func assetMapGetByGuid(c *gin.Context) {
	guid := c.Query("guid")

	result, err := global.Redis.Get(c, guid).Result()
	if err == redis.Nil {
		c.JSON(200, gin.H{"code": 1003, "msg": "guid not exists"})
		return
	} else if err != nil {
		c.JSON(200, gin.H{"code": 1001, "msg": err.Error()})
		return
	}

	c.JSON(200, gin.H{"code": 1002, "msg": "guid exists", "path": result})
}

// Redis Set示例
global.Redis.Set(c, guid, assetPath, 0)

// Redis HSet示例
data := make(map[string]interface{}, 2)
data["guid"] = guid
data["textureType"] = 5

global.Redis.HSet(c, pathKey, data)

// Redis HGet示例
global.Redis.HGet(c, pathKey, "guid").Result()

// Redis Del示例
global.Redis.Del(c, guid)
```

现在就用到Redis常用功能，更多用法移步[Github](https://github.com/go-redis/redis)
