---
title: 【Go】gin学习
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang
---

点击阅读更多查看文章内容<!--more-->


## gin的helloworld体验

Gin 是一个用 Go 语言编写的轻量级 Web 框架，专为高性能和易用性设计

启动一个gin的server对象，当接收到get方法的/ping请求时，调用pong方法

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func pong(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{
		"message": "pong",
	})
}
func main() {
	//实例化一个gin的server对象
	r := gin.Default()
	r.GET("/ping", pong)
	r.Run(":8083") // listen and serve on 0.0.0.0:8080
}
```

![image-20250208233225159](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125914999.png)

---

## 使用New和Default初始化路由器的区别

使用 gin.new() 只创建一个路由器不附带任何中间件，使用 gin.Default() 使用默认中间件（logger and recovery ）创建路由器，logger在请求时会打印日志，recovery在程序panic时会返回500状态码

配置不同方法，不同路径的处理逻辑，在restful接口中非常有用

```go
router.GET("/someGet", getting)
router.POST("/somePost", posting)
router.PUT("/somePut", putting)
router.DELETE("/someDelete", deleting)
router.PATCH("/somePatch", patching)
router.HEAD("/someHead", head)
router.OPTIONS("/someOptions", options)
```

---

## 路由分组

将相同前缀的url提取为一个分组，简化路径表述

```go
    router := gin.Default()
    goodsGroup := router.Group("/goods")
    {
       goodsGroup.GET("/list", goodsList)
       goodsGroup.GET("/1", goodsDetail) //获取商品id为1的详细信息
       goodsGroup.POST("/add", createGoods)
    }
```

等同于

```go
    router := gin.Default()
    router.GET("/goods/list", goodsList)
    router.GET("/goods/1", goodsDetail)
    router.POST("/goods/add", createGoods)
```

---

## 获取url中的变量

将传入的对应的url的值赋值给id

```go
goodsGroup.GET("/:id", goodsDetail) 
```

通过 c.Param("id") 取出 id 的值

```go
func goodsDetail(c *gin.Context) {
    id := c.Param("id")
    c.JSON(http.StatusOK, gin.H{
       "id":     id,
    })
}
```

![image-20250208235438014](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915001.png)

星号匹配 `goodsGroup.GET("/:id/*action", goodsDetail)`

![image-20250208235811274](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915002.png)

冒号匹配 `goodsGroup.GET("/:id/:action/add", goodsDetail)`

![image-20250208235911756](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915003.png)

**约束url**

传入的url需要id和name，其中id需要为int类型，name需要为string类型，否则会返回404错误

```go
package main

import (
    "github.com/gin-gonic/gin"
    "net/http"
)

type Person struct {
    ID   int    `uri:"id" binding:"required"`
    Name string `uri:"name" binding:"required"`
}

func main() {
    router := gin.Default()
    router.GET("/:name/:id", func(c *gin.Context) {
       var person Person
       if err := c.ShouldBindUri(&person); err != nil {
          c.Status(404)
          return
       }
       c.JSON(http.StatusOK, gin.H{
          "name": person.Name,
          "id":   person.ID,
       })
    })
    router.Run(":8083")
}
```

---
## 获取get和post表单信息

获取get请求的参数：c.DefaultQuery（在没有参数时设置默认值）

```go
router.GET("/welcome", welcome)

func welcome(c *gin.Context) {
	firstName := c.DefaultQuery("firstname", "bobby")
	lastName := c.DefaultQuery("lastname", "imooc")
	c.JSON(http.StatusOK, gin.H{
		"first_name": firstName,
		"last_name":  lastName,
	})
}
```

![image-20250209102042056](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915004.png)



获取post请求的参数：c.DefaultPostForm（没有参数时指定默认值）不需要指定默认值则使用 c.PostForm

```go
router.POST("/form_post", formPost)

func formPost(c *gin.Context) {
	message := c.PostForm("message")
	nick := c.DefaultPostForm("nick", "anonymous")
	c.JSON(http.StatusOK, gin.H{
		"message": message,
		"nik":     nick,
	})
}
```

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915005.png" alt="image-20250209103327847" style="zoom:80%;" />

---

## gin返回json和protobuf

**返回json：**

通过gin.H

```go
func welcome(c *gin.Context) {
    firstName := c.DefaultQuery("firstname", "bobby")
    lastName := c.DefaultQuery("lastname", "imooc")
    c.JSON(http.StatusOK, gin.H{
       "first_name": firstName,
       "last_name":  lastName,
    })
}
```

通过struct

```go
func moreJSON(c *gin.Context) {
    var msg struct {
       Name    string `json:"user"`
       Message string
       Number  int
    }
    msg.Name = "bobby"
    msg.Message = "这是一个测试json"
    msg.Number = 20

    c.JSON(http.StatusOK, msg)
}
```

**返回protobuf**

```go
func returnProto(c *gin.Context) {
    course := []string{"python", "go", "微服务"}
    user := &proto.Teacher{
       Name:   "bobby",
       Course: course,
    }
    c.ProtoBuf(http.StatusOK, user)
}
```

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915006.png" alt="image-20250209104536707" style="zoom:80%;" />

---

## 登录的表单验证

Gin使用 [go-playground/validator](https://github.com/go-playground/validator) 验证参数，[查看完整文档](https://godoc.org/github.com/go-playground/validator)。

- Must bind
  - Methods - `Bind`, `BindJSON`, `BindXML`, `BindQuery`, `BindYAML`
  - Behavior - 这些方法底层使用 `MustBindWith`，**如果存在绑定错误**，请求将被以下指令中止 `c.AbortWithError(400, err).SetType(ErrorTypeBind)`，**响应状态代码会被设置为400**，请求头`Content-Type`被设置为`text/plain; charset=utf-8`。注意，如果你试图在此之后设置响应代码，将会发出一个警告 `[GIN-debug] [WARNING] Headers were already written. Wanted to override status code 400 with 422`，如果你希望更好地控制行为，请使用`ShouldBind`相关的方法
- Should bind
  - Methods - `ShouldBind`, `ShouldBindJSON`, `ShouldBindXML`, `ShouldBindQuery`, `ShouldBindYAML`
  - Behavior - 这些方法底层使用 `ShouldBindWith`，**如果存在绑定错误，则返回错误，开发人员可以正确处理请求和错误**

当我们使用绑定方法时，Gin会根据Content-Type推断出使用哪种绑定器，如果你确定你绑定的是什么，你可以使用`MustBindWith`或者`BindingWith`。



通过struct的标签添加验证条件

- User的form key为user或json key为user，必填，最小长度为3，最大长度为10

- Password的json字段为password，必填

通过 c.ShouldBind(&loginForm) 验证

```go
package main

import (
    "github.com/gin-gonic/gin"
    "net/http"
)

type LoginForm struct {
	User     string `form:"user" json:"user" binding:"required,min=3,max=10"`
    Password string `json:"password" binding:"required"`
}

func main() {
    router := gin.Default()
    router.POST("/loginJSON", func(c *gin.Context) {
       var loginForm LoginForm
       if err := c.ShouldBind(&loginForm); err != nil {
          c.JSON(http.StatusBadRequest, gin.H{
             "error": err.Error(),
          })
          return
       }
       c.JSON(http.StatusOK, gin.H{
          "msg": "登录成功",
       })
    })
    _ = router.Run(":8083")
}
```

在form-data中添加password是无效的

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915007.png" alt="image-20250209114831770" style="zoom:80%;" />

需要在raw中添加json

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915008.png" alt="image-20250209115004410" style="zoom:80%;" />

---

## 注册的表单验证

```go
    type SignUpParam struct {
        Age        uint8  `json:"age" binding:"gte=1,lte=130"`
        Name       string `json:"name" binding:"required"`
        Email      string `json:"email" binding:"required,email"`
        Password   string `json:"password" binding:"required"`
        RePassword string `json:"re_password" binding:"required,eqfield=Password"`
    }

	router.POST("/signup", func(c *gin.Context) {
		var u SignUpParam
		if err := c.ShouldBind(&u); err != nil {
			c.JSON(http.StatusOK, gin.H{
				"msg": err.Error(),
			})
			return
		}
		// 保存入库等业务逻辑代码...

		c.JSON(http.StatusOK, "success")
	})
```

![image-20250209115348355](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915009.png)

---

## 表单验证错误翻译成中文

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/binding"
	"github.com/go-playground/locales/en"
	"github.com/go-playground/locales/zh"
	ut "github.com/go-playground/universal-translator"
	"github.com/go-playground/validator/v10"
	enTranslations "github.com/go-playground/validator/v10/translations/en"
	zhTranslations "github.com/go-playground/validator/v10/translations/zh"
)

// 定义一个全局翻译器T
var trans ut.Translator

// InitTrans 初始化翻译器
func InitTrans(locale string) (err error) {
	// 修改gin框架中的Validator引擎属性，实现自定制
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {

		zhT := zh.New() // 中文翻译器
		enT := en.New() // 英文翻译器

		// 第一个参数是备用（fallback）的语言环境
		// 后面的参数是应该支持的语言环境（支持多个）
		// uni := ut.New(zhT, zhT) 也是可以的
		uni := ut.New(enT, zhT, enT)

		// locale 通常取决于 http 请求头的 'Accept-Language'
		var ok bool
		// 也可以使用 uni.FindTranslator(...) 传入多个locale进行查找
		trans, ok = uni.GetTranslator(locale)
		if !ok {
			return fmt.Errorf("uni.GetTranslator(%s) failed", locale)
		}

		// 注册翻译器
		switch locale {
		case "en":
			err = enTranslations.RegisterDefaultTranslations(v, trans)
		case "zh":
			err = zhTranslations.RegisterDefaultTranslations(v, trans)
		default:
			err = enTranslations.RegisterDefaultTranslations(v, trans)
		}
		return
	}
	return
}

type SignUpParam struct {
	Age        uint8  `json:"age" binding:"gte=1,lte=130"`
	Name       string `json:"name" binding:"required"`
	Email      string `json:"email" binding:"required,email"`
	Password   string `json:"password" binding:"required"`
	RePassword string `json:"re_password" binding:"required,eqfield=Password"`
}

func main() {
	if err := InitTrans("zh"); err != nil {
		fmt.Printf("init trans failed, err:%v\n", err)
		return
	}

	r := gin.Default()

	r.POST("/signup", func(c *gin.Context) {
		var u SignUpParam
		if err := c.ShouldBind(&u); err != nil {
			// 获取validator.ValidationErrors类型的errors
			errs, ok := err.(validator.ValidationErrors)
			if !ok {
				// 非validator.ValidationErrors类型错误直接返回
				c.JSON(http.StatusOK, gin.H{
					"msg": err.Error(),
				})
				return
			}
			// validator.ValidationErrors类型错误则进行翻译
			c.JSON(http.StatusOK, gin.H{
				"msg": errs.Translate(trans),
			})
			return
		}
		// 保存入库等具体业务逻辑代码...

		c.JSON(http.StatusOK, "success")
	})

	_ = r.Run(":8999")
}
```

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915010.png" alt="image-20250209124359503" style="zoom:80%;" />

---

## 表单中文翻译的json格式化细节

错误中的字段仍然是go语言中定义的结构体字段名称，并且带有SignUpParam前缀



将字段名称修改为实际的json字段：在初始化翻译器获取到validator引擎后，注册tag方法，获取tag中json的值并根据逗号分隔获取第一个即可

```go
	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
		v.RegisterTagNameFunc(func(fld reflect.StructField) string {
			name := strings.SplitN(fld.Tag.Get("json"), ",", 2)[0]
			if name == "-" {
				return ""
			}
			return name
		})
```

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915011.png" alt="image-20250209125013383" style="zoom:80%;" />

去除前缀：去除map key中的前缀，获取.的位置，只保留.之后的内容

```go
func removeTopStruct(fields map[string]string) map[string]string {
    rsp := map[string]string{}
    for field, err := range fields {
       rsp[field[strings.Index(field, ".")+1:]] = err
    }
    return rsp
}
```

对返回值应用以上方法处理：

```go
c.JSON(http.StatusOK, gin.H{
    "msg": removeTopStruct(errs.Translate(trans)),
})
```

![image-20250209125944773](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915012.png)

---

## 自定义gin中间件

使用中间件

```go
	router := gin.New()
	// 使用logger和recovery中间件 全局所有
	router.Use(gin.Logger(),gin.Recovery())
```

中间件函数签名

```go
// HandlerFunc defines the handler used by gin middleware as return value.
type HandlerFunc func(*Context)
```

为/goods开头的url添加自定义中间件

```go
router := gin.Default()
authrized:=router.Group("/goods")
authrized.Use(func(context *gin.Context) {
    ...
})
```

自定义中间件的使用：

```go
func MyLogger() gin.HandlerFunc {
    return func(c *gin.Context) {
       t := time.Now()
       c.Set("example", "123456")
       //让原本改执行的逻辑继续执行
       c.Next()

       end := time.Since(t)
       fmt.Printf("耗时:%V\n", end)
       status := c.Writer.Status()
       fmt.Println("状态", status)
    }
}

func main() {
	router := gin.Default()
	router.Use(MyLogger())
	router.GET("/ping", func(c *gin.Context) {
		e, _ := c.Get("example")
		c.JSON(http.StatusOK, gin.H{
			"message": e,
		})
	})
	router.Run(":8083")
}
```

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915013.png" alt="image-20250209173008628" style="zoom:80%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915014.png" alt="image-20250209173023871" style="zoom: 67%;" />

---

## 通过abort终止中间件后续逻辑的执行

添加验证token的中间件，如果token不符则终止后续逻辑的执行

需要使用c.Abort()，不能通过return结束，具体原因在于中间件的执行逻辑，gin会维护一个要执行的函数的队列，并通过index指明当前要执行的函数，执行c.Next()时会将index++执行下一个函数，而执行return只会退出TokenRequired，后面的函数仍在队列中并且index没有改变，所以中间件结束后，index仍会++，只是不再由c.Next()驱动而是由gin驱动继续向后执行。

调用c.Abort()时会将index指向一个大数 `math.MaxInt8 / 2`，此时后面没有待执行的函数就终止了后续逻辑的执行

```go
func TokenRequired() gin.HandlerFunc {
    return func(c *gin.Context) {
       var token string
       for k, v := range c.Request.Header {
          if k == "X-Token" {
             token = v[0]
          }
       }
       if token != "bobby" {
          c.JSON(http.StatusUnauthorized, gin.H{
             "msg": "未登录",
          })
          c.Abort()
       }
       c.Next()
    }
}
```

---

## gin返回html

模板：非前后端分离的系统中，后端接收到请求后，会将数据填充到模板html中，再将html返回给前端显示

官方地址：https://golang.org/pkg/html/template/
翻译： [[译\]Golang template 小抄](https://colobu.com/2019/11/05/Golang-Templates-Cheatsheet/)

使用以下代码直接运行会报错，找不到文件，这是因为goland在执行代码时会将生成的exe文件放在临时目录下，相对于该临时目录的路径找不到文件，解决方法要么使用绝对路径，要么在代码目录下使用 go build 生成exe文件执行

```go
func main() {
    router := gin.Default()

    //为什么我们通过goland运行main.go的时候并没有生成main.exe文件
    dir, _ := filepath.Abs(filepath.Dir(os.Args[0]))
    fmt.Println(dir)
    router.LoadHTMLFiles("templates/index.tmpl")

    //如果没有在模板中使用define定义 那么我们就可以使用默认的文件名来找
    router.GET("/index", func(c *gin.Context) {
       c.HTML(http.StatusOK, "index.tmpl", gin.H{
          "title": "慕课网",
       })
    })

    router.Run(":8083")
}
```

模板文件：模板文件中使用.title取出title字段的值

```html
.tmpl
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>
        {{ .title }}
    </h1>
</body>
</html>
```

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915015.png" alt="image-20250209182958066" style="zoom:80%;" />

---

## 加载多个html文件

加载两个文件

```go
router.LoadHTMLFiles("templates/index.tmpl","templates/goods.html")
```

加载templates目录下所有目录的所有文件（只会加载二级目录的文件，templates目录下的文件不会加载）

```go
router.LoadHTMLGlob("templates/**/*")
```

在定义html时如果多个目录下的文件重名，则使用define定义一个名称

```html
{{define "goods/list.html"}}
<!DOCTYPE html>
<html lang="en">
<link rel="stylesheet" href="/static/css/style.css">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>商品列表页</h1>
</body>
</html>
{{end}}
```

在返回时，使用定义的名称返回

```go
	router.GET("/goods/list", func(c *gin.Context) {
		c.HTML(http.StatusOK, "goods/list.html", gin.H{
			"title": "慕课网",
		})
	})
```

---

## static静态文件的处理

静态文件：图片、CSS

html页面要使用css文件

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915016.png" alt="image-20250209220249276" style="zoom:80%;" />

直接访问该页面会报以下错误：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915017.png" alt="image-20250209220410825" style="zoom:67%;" />

此时需要通过router.Static()加载静态文件，添加如下语句以/mystatic开头的url都去当前目录下的static目录下查找

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125915018.png" alt="image-20250209220310437" style="zoom:80%;" />

---

## gin的优雅退出

优雅退出，当我们关闭程序的时候应该做的后续处理

```go
func main() {
    router := gin.Default()
    router.GET("/", func(c *gin.Context) {
       c.JSON(http.StatusOK, gin.H{
          "msg": "pong",
       })
    })
    go func() {
       router.Run(":8080")
    }()
    
    //优雅退出
    quit := make(chan os.Signal)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    //接收到关闭信号
    <-quit

    fmt.Println("关闭server中......")
    fmt.Println("注销服务......")
}
```
