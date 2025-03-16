---
title: 【Go】gin源码分析
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang

---

点击阅读更多查看文章内容<!--more-->

转载：https://zhuanlan.zhihu.com/p/611116090

## Gin的优势

- 支持中间件操作（ handlersChain 机制 ）
- 更方便的使用（ [gin.Context](https://zhida.zhihu.com/search?content_id=223934997&content_type=Article&match_order=1&q=gin.Context&zhida_source=entity) ）
- 更强大的路由解析能力（ [radix tree](https://zhida.zhihu.com/search?content_id=223934997&content_type=Article&match_order=1&q=radix+tree&zhida_source=entity) 路由树 ）

---

## Gin与HTTP的关系

Gin 是在 Golang HTTP 标准库 net/http 基础之上的再封装，两者的交互边界如下图：

在 net/http 的既定框架下，gin 所做的是提供了一个 [gin.Engine](https://zhida.zhihu.com/search?content_id=223934997&content_type=Article&match_order=1&q=gin.Engine&zhida_source=entity) 对象作为 Handler 注入net/http其中，从而实现路由注册/匹配、请求处理链路的优化.

<img src="https://pic4.zhimg.com/v2-306593a0eeef2b37f7da021272aae7ad_1440w.jpg" alt="img" style="zoom:50%;" />

使用示例：

```go
package main

import (
    "fmt"
    "github.com/gin-gonic/gin"
    "net/http"
)

func myMiddleWare(c *gin.Context) {
    fmt.Println("middleware starting...")
}
func main() {
    // 创建一个 gin Engine，本质上是一个 http Handler
    mux := gin.Default()
    // 注册中间件
    mux.Use(myMiddleWare)
    // 注册一个 path 为 /ping 的处理函数
    mux.POST("/ping", func(c *gin.Context) {
       c.JSON(http.StatusOK, "pone")
    })
    // 运行 http 服务
    if err := mux.Run(":8080"); err != nil {
       panic(err)
    }
}
```

## 核心数据结构

gin.Engine：Engine 为 Gin 中构建的 HTTP Handler，其实现了 net/http 包下 Handler interface 的抽象方法： Handler.ServeHTTP

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

因此可以作为 Handler 注入到 net/http 的 Server 当中.

```go
type Engine struct {
   // 路由组
    RouterGroup
    // ...
    // context 对象池
    pool             sync.Pool
    // 方法路由树
    trees            methodTrees
    // ...
}
```

Engine包含的核心内容包括：

- 路由组 RouterGroup：第（2）部分展开
- Context 对象池 pool：基于 sync.Pool 实现，作为复用 gin.Context 实例的缓冲池. gin.Context 的内容于本文第 5 章详解
- 路由树数组 trees：共有 9 棵路由树，对应于 9 种 http 方法. 路由树基于压缩前缀树实现，于本文第 4 章详解.

<img src="https://picx.zhimg.com/v2-9f452cc2bcd08ab6444750d5b0e750dd_1440w.jpg" alt="img" style="zoom: 50%;" />

### RouteGroup

```go
type RouterGroup struct {
    Handlers HandlersChain
    basePath string
    engine *Engine
    root bool
}
```

RouterGroup 是路由组的概念，其中的配置将被从属于该路由组的所有路由复用：

- Handlers：路由组共同的 handler 处理函数链. 组下的节点将拼接 RouterGroup 的公用 handlers 和自己的 handlers，组成最终使用的 handlers 链
- basePath：路由组的基础路径. 组下的节点将拼接 RouterGroup 的 basePath 和自己的 path，组成最终使用的 absolutePath
- engine：指向路由组从属的 Engine
- root：标识路由组是否位于 Engine 的根节点. 当用户基于 RouterGroup.Group 方法创建子路由组后，该标识为 false

HandlersChain

```go
type HandlersChain []HandlerFunc


type HandlerFunc func(*Context)
```

HandlersChain 是由多个路由处理函数 HandlerFunc 构成的处理函数链. 在使用的时候，会按照索引的先后顺序依次调用 HandlerFunc.

---

## handler注册流程

下面以创建 gin.Engine 、注册 middleware 和注册 handler 作为主线，进行源码走读和原理解析：

```go
func main() {
    // 创建一个 gin Engine，本质上是一个 http Handler
    mux := gin.Default()
    // 注册中间件
    mux.Use(myMiddleWare)
    // 注册一个 path 为 /ping 的处理函数
    mux.POST("/ping", func(c *gin.Context) {
        c.JSON(http.StatusOK, "pone")
    })
    // ...
}
```

**初始化 Engine**

方法调用：gin.Default -> gin.New

- 创建一个 gin.Engine 实例
- 创建 Enging 的首个 RouterGroup，对应的处理函数链 Handlers 为 nil，基础路径 basePath 为 "/"，root 标识为 true
- 构造了 9 棵方法路由树，对应于 9 种 http 方法
- 创建了 gin.Context 的对象池

```go
func Default() *Engine {
    engine := New()
    // ...
    return engine
}

func New() *Engine {
    // ...
    // 创建 gin Engine 实例
    engine := &Engine{
        // 路由组实例
        RouterGroup: RouterGroup{
            Handlers: nil,
            basePath: "/",
            root:     true,
        },
        // ...
        // 9 棵路由压缩前缀树，对应 9 种 http 方法
        trees:                  make(methodTrees, 0, 9),
        // ...
    }
    engine.RouterGroup.engine = engine     
    // gin.Context 对象池   
    engine.pool.New = func() any {
        return engine.allocateContext(engine.maxParams)
    }
    return engine
}
```

**注册 middleware**

通过 Engine.Use 方法可以实现中间件的注册，会将注册的 middlewares 添加到 RouterGroup.Handlers 中. 后续 RouterGroup 下新注册的 handler 都会在前缀中拼上这部分 group 公共的 handlers

```go
func (engine *Engine) Use(middleware ...HandlerFunc) IRoutes {
    engine.RouterGroup.Use(middleware...)
    // ...
    return engine
}
func (group *RouterGroup) Use(middleware ...HandlerFunc) IRoutes {
    group.Handlers = append(group.Handlers, middleware...)
    return group.returnObj()
}
```

**注册 handler**

<img src="https://pic4.zhimg.com/v2-10825ae29725230f61a5b9b1f89dcc59_1440w.jpg" alt="img" style="zoom: 50%;" />

以 http post 为例，注册 handler 方法调用顺序为 RouterGroup.POST-> RouterGroup.handle，接下来会完成三个步骤：

- 拼接出待注册方法的完整路径 absolutePath
- 拼接出代注册方法的完整处理函数链 handlers
- 以 absolutePath 和 handlers 组成 kv 对添加到路由树中

```go
func (group *RouterGroup) POST(relativePath string, handlers ...HandlerFunc) IRoutes {
    return group.handle(http.MethodPost, relativePath, handlers)
}

func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
    absolutePath := group.calculateAbsolutePath(relativePath)
    handlers = group.combineHandlers(handlers)
    group.engine.addRoute(httpMethod, absolutePath, handlers)
    return group.returnObj()
}
```

---

## 服务端启动流程

**流程入口**：

下面通过 Gin 框架运行 http 服务为主线，进行源码走读：

```go
func main() {
    // 创建一个 gin Engine，本质上是一个 http Handler
    mux := gin.Default()
    
    // 一键启动 http 服务
    if err := mux.Run(); err != nil{
        panic(err)
    }
}
```

**启动服务**：

一键启动 Engine.Run 方法后，底层会将 gin.Engine 本身作为 net/http 包下 Handler interface 的实现类，并调用 http.ListenAndServe 方法启动服务.（ListenerAndServe 方法本身会基于主动轮询 + IO 多路复用的方式运行）

```go
func (engine *Engine) Run(addr ...string) (err error) {
    // ...
    err = http.ListenAndServe(address, engine.Handler())
    return
}
```

**处理请求**

在服务端接收到 http 请求时，会通过 Handler.ServeHTTP 方法进行处理. 而此处的 Handler 正是 gin.Engine，其处理请求的核心步骤如下：

- 对于每笔 http 请求，会为其分配一个 gin.Context，在 handlers 链路中持续向下传递
- 调用 Engine.handleHTTPRequest 方法，从路由树中获取 handlers 链，然后遍历调用
- 处理完 http 请求后，会将 gin.Context 进行回收. 整个回收复用的流程基于对象池管理

```go
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    // 从对象池中获取一个 context
    c := engine.pool.Get().(*Context)
    
    // 重置/初始化 context
    c.writermem.reset(w)
    c.Request = req
    c.reset()
    
    // 处理 http 请求
    engine.handleHTTPRequest(c)


    // 把 context 放回对象池
    engine.pool.Put(c)
}
```

<img src="https://pic1.zhimg.com/v2-9514b00586e9c7337c03c2e761549032_1440w.jpg" alt="img" style="zoom:50%;" />

Engine.handleHTTPRequest 方法核心步骤分为三步：

- 根据 http method 取得对应的 methodTree
- 根据 path 从 methodTree 中找到对应的 handlers 链
- 将 handlers 链注入到 gin.Context 中，通过 Context.Next 方法按照顺序遍历调用 handler

---

## gin路由树

<img src="https://pic2.zhimg.com/v2-b1f2d4d481cbc48f767128b7cba86f17_1440w.jpg" alt="img" style="zoom:50%;" />

路由树是使用压缩前缀树实现的

（1）前缀树

前缀树又称 trie 树，是一种基于字符串公共前缀构建索引的树状结构，核心点包括：

- 除根节点之外，每个节点对应一个字符
- 从根节点到某一节点，路径上经过的字符串联起来，即为该节点对应的字符串
- 尽可能复用公共前缀，如无必要不分配新的节点

（2）压缩前缀树

压缩前缀树又称基数树或 radix 树，是对前缀树的改良版本，优化点主要在于空间的节省，核心策略体现在：

倘若某个子节点是其父节点的唯一孩子，则与父节点进行合并

在 gin 框架中，是用压缩前缀树

（3）为什么使用压缩前缀树

与压缩前缀树相对的就是使用 hashmap，以 path 为 key，handlers 为 value 进行映射关联，这里选择了前者的原因在于：

- path 匹配时不是完全精确匹配，比如末尾 ‘/’ 符号的增减（/ping与/ping/应该是同一个路径）、全匹配符号 '*' 的处理等，map 无法胜任（模糊匹配部分的代码于本文中并未体现，大家可以深入源码中加以佐证）
- 路由的数量相对有限，对应数量级下 map 的性能优势体现不明显，在小数据量的前提下，map 性能甚至要弱于前缀树
- path 串通常存在基于分组分类的公共前缀，适合使用前缀树进行管理，可以节省存储空间

（4）补偿策略

在 Gin 路由树中还使用一种补偿策略，在组装路由树时，会将注册路由句柄数量更多的 child node 摆放在 children 数组更靠前的位置.

这是因为某个链路注册的 handlers 句柄数量越多，一次匹配操作所需要话费的时间就越长，被匹配命中的概率就越大，因此应该被优先处理.

---

## Gin.Context

<img src="https://pic2.zhimg.com/v2-9c5a2c7a07f1062cdb00490f8b99eb09_1440w.jpg" alt="img" style="zoom:50%;" />

**gin.Context 的定位是对应于一次 http 请求，贯穿于整条 handlersChain 调用链路的上下文，其中包含了如下核心字段：**

- Request/Writer：http 请求和响应的 reader、writer 入口
- handlers：本次 http 请求对应的处理函数链
- index：当前的处理进度，即处理链路处于函数链的索引位置
- engine：Engine 的指针
- mu：用于保护 map 的读写互斥锁
- Keys：缓存 handlers 链上共享数据的 map

```go
type Context struct {
    // ...
    // http 请求参数
    Request   *http.Request
    // http 响应 writer
    Writer    ResponseWriter
    // ...
    // 处理函数链
    handlers HandlersChain
    // 当前处于处理函数链的索引
    index    int8
    engine       *Engine
    // ...
    // 读写锁，保证并发安全
    mu sync.RWMutex
    // key value 对存储 map
    Keys map[string]any
    // ..
}
```

### 复用

gin.Context 作为处理 http 请求的通用数据结构，不可避免地会被频繁创建和销毁. 为了缓解 GC 压力，gin 中采用对象池 sync.Pool 进行 Context 的缓存复用，处理流程如下：

- http 请求到达时，从 pool 中获取 Context，倘若池子已空，通过 pool.New 方法构造新的 Context 补上空缺
- http 请求处理完成后，将 Context 放回 pool 中，用以后续复用

sync.Pool 并不是真正意义上的缓存，将其称为回收站或许更加合适，放入其中的数据在逻辑意义上都是已经被删除的，但在物理意义上数据是仍然存在的，这些数据可以存活两轮 GC 的时间，在此期间倘若有被获取的需求，则可以被重新复用.

```go
type Engine struct {
    // context 对象池
    pool             sync.Pool
}
func New() *Engine {
    // ...
    engine.pool.New = func() any {
        return engine.allocateContext(engine.maxParams)
    }
    return engine
}
func (engine *Engine) allocateContext(maxParams uint16) *Context {
    v := make(Params, 0, maxParams)
   // ...
    return &Context{engine: engine, params: &v, skippedNodes: &skippedNodes}
}
```

### 分配与回收

<img src="https://picx.zhimg.com/v2-5bafe9bee6e19cbdb0fd908a2ceeac35_1440w.jpg" alt="img" style="zoom:50%;" />

gin.Context 分配与回收的时机是在 gin.Engine 处理 http 请求的前后，位于 Engine.ServeHTTP 方法当中：

- 从池中获取 Context
- 重置 Context 的内容，使其成为一个空白的上下文
- 调用 Engine.handleHTTPRequest 方法处理 http 请求
- 请求处理完成后，将 Context 放回池中

```go
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    // 从对象池中获取一个 context
    c := engine.pool.Get().(*Context)
    // 重置/初始化 context
    c.writermem.reset(w)
    c.Request = req
    c.reset()
    // 处理 http 请求
    engine.handleHTTPRequest(c)
    
    // 把 context 放回对象池
    engine.pool.Put(c)
}
```

### 使用时机

（1）handlesChain 入口

在 Engine.handleHTTPRequest 方法处理请求时，会通过 path 从 methodTree 中获取到对应的 handlers 链，然后将 handlers 注入到 Context.handlers 中，然后启动 **Context.Next** 方法开启 handlers 链的遍历调用流程.

```go
func (engine *Engine) handleHTTPRequest(c *Context) {
    // ...
    t := engine.trees
    for i, tl := 0, len(t); i < tl; i++ {
        if t[i].method != httpMethod {
            continue
        }
        root := t[i].root        
        value := root.getValue(rPath, c.params, c.skippedNodes, unescape)
        // ...
        if value.handlers != nil {
            c.handlers = value.handlers
            c.fullPath = value.fullPath
            c.Next()
            c.writermem.WriteHeaderNow()
            return
        }
        // ...
    }
    // ...
}
```

**handlerschain遍历调用**

<img src="https://pic4.zhimg.com/v2-e513ca240a12587e1928a750ae555cad_1440w.jpg" alt="img" style="zoom:50%;" />

推进 handlers 链调用进度的方法正是 Context.Next. 可以看到其中以 Context.index 为索引，通过 for 循环依次调用 handlers 链中的 handler.（调用next会顺序执行其后的每个handler）

```go
func (c *Context) Next() {
    c.index++
    for c.index < int8(len(c.handlers)) {
        c.handlers[c.index](c)
        c.index++
    }
}
```

由于 **Context 本身会暴露于调用链路**中，因此用户可以在某个 handler 中通过手动调用 Context.Next 的方式来打断当前 handler 的执行流程，提前进入下一个 handler 的处理中，此时本质上是一个方法压栈调用的行为，因此在后置位 handlers 链全部处理完成后，最终会回到压栈前的位置，执行当前 handler 剩余部分的代码逻辑.

结合下面的代码示例来说，用户可以在某个 handler 中，于调用 **Context.Next** 方法的前后分别声明前处理逻辑和后处理逻辑，这里的“前”和“后”相对的是后置位的所有 handler 而言.

```go
func myHandleFunc(c *gin.Context){
    // 前处理
    preHandle()  
    c.Next()
    // 后处理
    postHandle()
}
```

此外，用户可以在某个 handler 中通过调用 **Context.Abort** 方法实现 handlers 链路的提前熔断.

其实现原理是将 Context.index 设置为一个过载值 63，导致 Next 流程直接终止. 这是因为 handlers 链的长度必须小于 63，否则在注册时就会直接 panic. 因此在 Context.Next 方法中，一旦 index 被设为 63，则必然大于整条 handlers 链的长度，for 循环便会提前终止.

```go
const abortIndex int8 = 63


func (c *Context) Abort() {
    c.index = abortIndex
}
```

---

## 总结

- gin 将 Engine 作为 http.Handler 的实现类进行注入，从而融入 Golang net/http 标准库的框架之内
- gin 中基于 handler 链的方式实现中间件和处理函数的协调使用
- gin 中基于压缩前缀树的方式作为路由树的数据结构，对应于 9 种 http 方法共有 9 棵树
- gin 中基于 gin.Context 作为一次 http 请求贯穿整条 handler chain 的核心数据结构
- gin.Context 是一种会被频繁创建销毁的资源对象，因此使用对象池 sync.Pool 进行缓存复用