---
title: 【Go】go-redis使用
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang



---

点击阅读更多查看文章内容<!--more-->

## go-redis

[redis/go-redis: Redis Go client](https://github.com/redis/go-redis)

文档：[Golang Redis客户端](https://redis.uptrace.dev/zh/)

安装：`go get github.com/redis/go-redis/v9`

建立连接：

```go
rdb := redis.NewClient(&redis.Options{
    Addr:     "localhost:6379",
    Password: "", // 没有密码，默认值
    DB:       0,  // 默认DB 0
})
```

Set/Get：

```go
ctx := context.Background()
err := rdb.Set(ctx, "key", "value", 10*time.Second).Err()
if err != nil {
    panic(err)
}
val, err := rdb.Get(ctx, "key").Result()
if err != nil {
    panic(err)
}
```