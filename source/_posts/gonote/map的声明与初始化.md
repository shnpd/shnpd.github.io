---
title: 【Go】map的声明与初始化问题
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang



---

点击阅读更多查看文章内容<!--more-->

## map的声明与初始化

### 问题描述：

在使用以下代码生成map并赋值时会报panic: assignment to entry in nil map

```go
func main() {
	var a map[int]int
	a[0] = 1
}
```

### 出现原因：

**var只是声明没有初始化**

在 Go 中，使用 `var` 声明一个变量时，**变量本身会被分配内存地址并初始化为零值**

> 例如 `var a int` 这里 a 的值为 0

但对于复合数据类型（如 `map`、`slice` 和 `channel`），它们的零值是 `nil`。这意味着使用 `var` 声明的 `map` 变量虽然有地址，但它指向的底层数据结构尚未被分配内存。

因此当使用**`var a map[int]int`**：声明了一个 `map` 类型的变量，其初始值为 `nil`，此时没有底层数据结构支持存储键值对。试图对 `nil` 的 `map` 进行写操作时，会导致 `panic`。

### 解决方法

1. 使用 make 初始化 map：`var a = make(map[int]int)`
2. 使用 `a:=map[int]int{}`,它在声明的同时对 `map` 进行了初始化。其效果等同于`a := make(map[int]int)`