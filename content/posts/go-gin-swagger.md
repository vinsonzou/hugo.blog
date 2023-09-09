---
title: "[Go] Gin使用Swagger生成接囗文档"
subtitle: ""
date: 2023-09-09T10:40:00+08:00
lastmod: 2023-09-09T10:40:00+08:00
description: ""

tags: ["Golang"]
categories: ["Golang"]
---

## 环境
- go 1.20

## 安装

```bash
go install github.com/swaggo/swag/cmd/swag
```

## 使用

在项目根目录执行

```bash
swag init
```

> 将会在项目根目录下生成 `docs` 目录和相应的 Swagger 文档文件。
>
> ```bash
> .
> ├── docs
> │   ├── docs.go
> │   ├── swagger.json
> │   └── swagger.yaml
> ├── go.mod
> ├── go.sum
> └── main.go
> ```

使用 Gin 框架集成 Swagger 的示例：

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/swaggo/files"
	"github.com/swaggo/gin-swagger"
	_ "your-module-path/docs" // 导入生成的 Swagger 文档代码
)

// @title 这里写标题
// @version 1.0
// @description 这里写描述信息
// @host localhost:8080
// @BasePath /api/v1
func main() {
	r := gin.Default()

	// 注册 Swagger 路由
	r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))

	// 添加其他路由处理器
	r.GET("/hello", helloHandler)

	r.Run(":8080")
}

// @Summary Hello API
// @Description Get hello message
// @Tags Hello
// @Produce json
// @Param project query string false "项目标识" default(test101)
// @Param lang query string false "语种" default(cn)
// @Success 200 {string} string "Hello, World!"
// @Router /hello [get]
func helloHandler(c *gin.Context) {
  project := c.DefaultQuery("project","test101")
	lang := c.DefaultQuery("lang", "cn")

	c.JSON(http.StatusOK, "Hello, World!")
}
```

在项目根目录执行，更新swagger文档

```bash
swag init
```

然后，运行示例代码，您可以通过访问 `http://localhost:8080/swagger/index.html` 来查看自动生成的 Swagger 文档界面，并在其中测试您的 API。

## 注意事项

- 在路由添加swagger的时候，需要引入项目生成的docs包；
- 访问swagger控制台报错Failed to load spec，是因为没有import引入执行`swag init`生成的docs文件夹；

## 参考

- https://github.com/swaggo/gin-swagger
- https://github.com/swaggo/swag/blob/master/README_zh-CN.md