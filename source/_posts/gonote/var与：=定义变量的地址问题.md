---
title: 【Go】var与:=定义变量的地址问题
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang



---

点击阅读更多查看文章内容<!--more-->

## var与:=定义变量时如何分配地址空间

使用var声明变量时会为变量本身分配地址，但不会为变量指向的底层指针分配地址

使用:=初始化变量会为变量本身以及其指向的底层指针分配地址

这种区别主要体现在引用类型中（如切片、map、channel、指针等）



**在值类型中** 

int/float/bool/string/struct 使用两个方法都没有区别，都会为变量分配地址

**在引用类型中**

切片：

```go
func main() {
	var a []int
	fmt.Println(a == nil)  // true
	fmt.Printf("%p\n", &a) // 变量 a 的地址
	fmt.Printf("%p\n", a)  // 输出 0x0（因为底层数组是 nil）

	m2 := map[string]int{}
	fmt.Println(m2 == nil) // false
	m2["key"] = 10         // ✅ 正常运行
}
```

map：

```go
func main() {
	var m map[string]int
	fmt.Println(m == nil) // true
	m["key"] = 10         // 运行时报错：panic: assignment to entry in nil map

	m2 := map[string]int{}
	fmt.Println(m2 == nil) // false
	m2["key"] = 10         // ✅ 正常运行
}
```

channel：

```go
func main() {
	var ch chan int
	fmt.Println(ch == nil) // true
	ch <- 1                // ❌ panic: send on nil channel
	ch2 := make(chan int, 1)
	fmt.Println(ch2 == nil) // false
	ch2 <- 1  // ✅ 正常运行
}
```

指针：

```go
func main() {
    var p *int
    fmt.Println(p == nil) // true
    *p = 100              // ❌ panic: invalid memory address
    
    p2 := new(int)
    fmt.Println(p2 == nil) // false
    *p2 = 100  // ✅ 正常运行
    fmt.Println(*p2) // 100
}
```