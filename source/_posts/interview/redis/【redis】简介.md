---
title: 【redis】简介
toc: true
date: 2025-03-03 00:00:00
tags: 
  - redis
categories: 
	- 知识点整理
	- redis
---



点击阅读更多查看文章内容<!--more-->

## 什么是redis

Redis诞生于2009年全称是Remote Dictionary Server，远程词典服务器，是一个基于内存的键值型NoSQL数据库。

特征：

- 键值型，value支持多种不同数据结构，功能丰富
- 单线程，redis处理命令请求是单线程的避免上下文切换
- 低延迟，速度快（基于内存、IO多路复用、良好的编码）
- 支持数据持久化（redis数据会落到磁盘持久化）
- 支持主从集群、分片集群
- 支持多语言客户端



## 安装redis

[Install Redis on Windows | Docs](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-windows/)

windows下需要安装wsl，通过wsl进入linux子系统

通过redis-cli进入redis命令行客户端进行交互

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618055.png" alt="image-20250214191352253" style="zoom:67%;" />

## redis常见命令

### 数据结构介绍

key一般是String类型，不过value类型多样，前五种为基本类型，后三种为特殊类型

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618056.png" alt="image-20250214191735105" style="zoom:50%;" />

命令帮助文档：[Commands | Docs](https://redis.io/docs/latest/commands/)

通过命令行中 `help @数据类型`，也可以查看对应类型的文档

### 通用命令

- keys：查看符合模板的所有key，pattern是redis的通配符，不建议在生产环境设备中使用，匹配时间较长单线程会阻塞

![image-20250214192257206](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618057.png)

- del：删除一个指定的key
- exists：判断key是否存在
- expire：给一个key设置有效期，有效期到期时该key会被自动删除，节省内存空间（短期验证码）
- ttl：查看一个key的剩余有效期

### String类型

根据字符串的格式不同，又可以分为3类：

- string：普通字符串
- int：整数类型，可以做自增、自减操作
- float：浮点类型，可以做自增、自减操作

不管哪种格式，底层都是字节数组形式存储，只不过编码方式不同，字符串类型的最大空间不超过512m

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618058.png" alt="image-20250214193133049" style="zoom: 33%;" />

常见命令：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618059.png" alt="image-20250214193159019" style="zoom: 33%;" />

### key的层级格式

Redis没有类似MySQL中的Table的概念，我们该如何区分不同类型的key呢?

例如，需要存储用户、商品信息到redis,有一个用户id是1,有一个商品id恰好也是1

Redis的key允许有多个单词形成**层级结构**，多个单词之间用 ‘:’ 隔开，例如：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618060.png" alt="image-20250214194202898" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618062.png" alt="image-20250214194217918" style="zoom: 33%;" />

如果value是一个对象，则可以将对象序列化为json字符串后存储：  

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618063.png" alt="image-20250214194257951" style="zoom:33%;" />

### Hash类型

其value是一个无序字典，类似于哈希表

存储对象时使用JSON序列化修改不方便

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618064.png" alt="image-20250214194730112" style="zoom: 33%;" />

hash可以将对象中的每个字段独立存储，针对单个字段做CRUD

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618065.png" alt="image-20250214194809233" style="zoom:33%;" />

常见命令：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618066.png" alt="image-20250214194853155" style="zoom:33%;" />

### List类型

list可以看做是一个双向链表的结构，既可以支持正向检索也可以支持反向检索

- 有序
- 元素可以重复
- 插入和删除快（直接修改节点指向）
- 查询速度一般（遍历）

常用来存储一个**有序**数据：朋友圈点赞列表、评论列表等

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618067.png" alt="image-20250214200237901" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618068.png" alt="image-20250214200858462" style="zoom:33%;" />

### Set类型

集合

- 无序
- 元素不可重复
- 查找快
- 支持交集、并集、差集等功能

常见命令：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618069.png" alt="image-20250214201622112" style="zoom:33%;" />

### SortedSet类型

SortedSet是一个可排序的set集合，SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表加hash表

- 可排序
- 元素不重复
- 查询速度快

因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618070.png" alt="image-20250214202545320" style="zoom:33%;" />

---

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

## 短信登陆

基于session

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618071.png" alt="image-20250215131829814" style="zoom: 80%;" />

集群的session共享问题：多台服务器并不共享session存储空间，当请求切换到不同的服务器时会导致数据丢失的问题。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618072.png" alt="image-20250215133747768" style="zoom: 33%;" />

session的替代方案应该满足：

- 数据共享
- 内存存储
- key-value数据结构

**基于Redis实现共享session登录**

 <img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618073.png" alt="image-20250215134531249" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618074.png" alt="image-20250215134451373" style="zoom:33%;" />

## 商户查询缓存

什么是缓存？

缓存就是数据交换的缓冲区（Cache），是存贮数据的临时地方，一般**读写性能较高**。它可以位于内存中，也可以位于 CPU、磁盘等位置。缓存的主要目的是减少访问 **慢速存储**（如磁盘或数据库）的时间，从而提高系统性能。

> CPU内部具有一个缓存，通过缓存读写数据比内存或磁盘快得多。

web应用开发中缓存的使用场景：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618075.png" alt="image-20250215140503531" style="zoom: 25%;" />

作用：

- 降低后端负载
- 提高读写效率，降低响应时间

成本：

- 数据一致性成本
- 代码维护成本（解决一致性问题）
- 运维成本（缓存穿透、缓存雪崩、缓存击穿）



### 添加Redis缓存

在查询商户时添加缓存

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618076.png" alt="image-20250215143003012" style="zoom: 50%;" />

### 缓存更新策略

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618077.png" alt="image-20250215143806462" style="zoom:33%;" />

   

主动更新策略：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618078.png" alt="image-20250215143939272" style="zoom:33%;" />

 

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618079.png" alt="image-20250215144351275" style="zoom:33%;" />

线程安全问题： **线程安全问题** 是指在 **多线程环境** 中，多个线程同时访问共享资源时，如果没有采取适当的同步机制，可能导致程序的行为变得不确定，甚至出现错误的情况。

- 先删除缓存，再操作数据库：线程1删除缓存后，在更新数据库之前，线程2查询缓存未命中再查询数据库将旧的数据写入缓存。线程1更新数据库的操作是比较慢的，线程2查询的操作相对比较快，因此这种缓存与数据库不一致的现象发生的概率是比较高的。
- 先操作数据库，再操作缓存：线程1查询缓存未命中，查询数据库的数据，再将数据写入缓存之前，线程2更新了数据库并删除缓存了，此时线程1将更新前的数据写入缓存。线程1写缓存的速度是非常快的，在这期间内线程2要完成数据库更新以及缓存删除的操作，这种现象发生的概率是比较低的。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618080.png" alt="image-20250215145836960" style="zoom: 33%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618081.png" alt="image-20250215150003534" style="zoom:33%;" />

---

### 缓存穿透

 缓存穿透：客户端请求的数据在缓存和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。

解决方案：

- 缓存空对象

  - 实现简单，维护方便

  - 额外的内存消耗；可能造成短期的不一致

    <img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618082.png" alt="image-20250215152154261" style="zoom:33%;" />

- 布隆过滤
  - 内存占用少，没有多余key
  - 实现复杂，存在误判可能

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618083.png" alt="image-20250215152207354" style="zoom:33%;" />

使用缓存空对象解决缓存穿透问题：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618084.png" alt="image-20250215152442652" style="zoom: 33%;" />

 

除了以上两种被动解决方案（在发生缓存穿透之后的解决方案），还可以主动解决（在未发生缓存传统之前避免出现缓存传统）：

- 增强id的复杂度，避免被猜测id规律，做好数据的基础格式校验
  - 在请求到达缓存层之前，先进行基本参数检查，避免无意义的查询。
- 加强用户权限校验
- 做好热点参数的限流

---

### 缓存雪崩

缓存雪崩：在同一时段内大量的缓存key同时失效或者redis服务宕机，导致大量请求到达数据库，带来巨大压力

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618085.png" alt="image-20250215154246689" style="zoom: 33%;" />

解决方案：

- 给不同的Key的TTL添加随机值
- 利用Redis集群提高服务的可用性
- 给缓存业务添加降级限流策略
- 给业务添加多级缓存

---

### 缓存击穿

缓存击穿：缓存击穿问题也叫**热点Key**问题，就是一个**高并发访问**并且**缓存重建业务较复杂**的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618086.png" alt="image-20250215160117291" style="zoom: 33%;" />

  解决方案：

- 互斥锁

  - 效率较低：只有一个线程在重建数据，其它的所有线程等在等待

  <img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618087.png" alt="image-20250215160254198" style="zoom: 50%;" />

- 逻辑过期：一个线程重建，其它线程返回旧数据。不为key设置TTL，而是添加一个expire字段，如果数据过期则获取锁开启新线程执行数据重建，其它的线程都直接返回旧数据

  <img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618089.png" alt="image-20250215160635217" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618090.png" alt="image-20250215160704109" style="zoom:33%;" />

基于互斥锁解决缓存击穿问题：（互斥锁可以用redis的setnx设置一个key，只有第一个设置才会生效，后续的设置因为已经存在不会生效）

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618091.png" alt="image-20250215161056488" style="zoom:33%;" />

基于逻辑过期解决缓存击穿问题：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123618092.png" alt="image-20250215162612189" style="zoom:33%;" />



> 缓存雪崩：大量key到期；缓存击穿：少量热点key到期；这两种问题的后果都是大量请求打到数据库

