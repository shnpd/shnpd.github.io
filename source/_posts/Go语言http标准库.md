---
title: Go语言http标准库
toc: true
date: 2022-01-30 13:29:11
tags: golang http 开发语言
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

# Go语言http标准库
- 使用http客户端发送请求
- 使用http.Client控制请求头部
- 使用httputil简化工作

**示例代码**

```go
package main

import (
	"fmt"
	"net/http"
	"net/http/httputil"
)

func main() {
	request, err := http.NewRequest(
		http.MethodGet,
		"http://www.imooc.com",
		nil,
	)
	client := http.Client{
		CheckRedirect: func(
			req *http.Request,
			via []*http.Request) error {
			fmt.Println("Redirect:", req)
			return nil
		},
	}
	request.Header.Add("User-Agent", "Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_1 like Mac OS X) AppleWebKit/603.1.30 (KHTML, like Gecko) Version/10.0 Mobile/14E304 Safari/602.1")
	resp, err := client.Do(request)
	if err != nil {
		panic(err)
	}
	defer resp.Body.Close()
	s, err := httputil.DumpResponse(resp, true)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%s\n", s)
}

```

# http服务器的性能分析
- import _ "net/http/pprof"     
前面加下划线表示虽然没有用到但要load其中一些帮助程序进来
- 访问/debug/pprof/
- 使用go tool pprof分析性能
点开net/http/pprof的帮助文档，根据文档来写


**查看服务器有关信息**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e7052f2276065576ccb91d33cd2aa182_1740930679947.png)
**查看内存占用**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/7848e57d84aa57adc447b1e013ad2361_1740930687223.png)
# JSON数据格式

- 结构体的tag用来处理字段名
- json的marshal与Unmarshal，用来转换json格式
- 注意不能直接将属性名改为小写，go语言中小写为private不会显示

**示例代码**

```go
package main

import (
	"encoding/json"
	"fmt"
)

type OrderItem struct {
	ID    string  `json:"id"`
	Name  string  `json:"name"`
	Price float64 `json:"price"`
}

//不能直接将属性名改为小写，go语言中小写为private不会显示
//可以通过打标签的方式更改json中的属性名
//omitempty当属性为空时不显示，如果不加omitempty会显示一个空字段
type Order struct {
	ID         string      `json:"id"`
	Items      []OrderItem `json:"items"`
	TotalPrice float64     `json:"total_price"`
}

func main() {
	unmarshal()
}
func marshal() {
	o := Order{
		ID:         "1234",
		TotalPrice: 20,
		Items: []OrderItem{
			{
				ID:    "item_1",
				Name:  "learn go",
				Price: 15,
			},
			{
				ID:    "item_2",
				Name:  "interview",
				Price: 10,
			},
		},
	}
	b, err := json.Marshal(o)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%s\n", b)
}
func unmarshal() {
	s := `{"id":"1234","items":[{"id":"item_1","name":"learn go","price":15},{"id":"item_2","name":"interview","price":10}],"total_price":20}`
	var o Order
	err := json.Unmarshal([]byte(s), &o)
	if err != nil {
		panic(err)
	}
	fmt.Printf("%+v\n", o)
}

```

# gin框架
- middleware的使用
通过middleware注册一些函数，所有的请求都会经过middleware
- context的使用
关于请求的所有信息，可以自己添加一些key-value

```go
package main

import (
	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
	"math/rand"
	"time"
)

func main() {
	r := gin.Default()
	logger, err := zap.NewProduction()
	if err != nil {
		panic(err)
	}

	//middleware
	const keyRequestId = "requestId"
	r.Use(func(c *gin.Context) {
		s := time.Now()
		c.Next()
		// path,status,log latency
		logger.Info("incoming request",
			zap.String("path", c.Request.URL.Path),
			zap.Int("status", c.Writer.Status()),
			zap.Duration("elapsed", time.Now().Sub(s)))
	}, func(c *gin.Context) {
		//可以在middleware中往context里面塞东西，在具体的handler中拿东西
		c.Set(keyRequestId, rand.Int())
		c.Next()
	})



	//handler 处理对应的路径
	r.GET("/ping", func(c *gin.Context) {
		h := gin.H{
			"message": "pong",
		}
		if rid, exists := c.Get(keyRequestId); exists {
			h[keyRequestId] = rid
		}

		c.JSON(200, h)
	})
	r.GET("/hello", func(c *gin.Context) {
		c.String(200, "hello")
	})
	r.Run() // listen and serve on 0.0.0.0:8080 (for windows "localhost:8080")
}

```

