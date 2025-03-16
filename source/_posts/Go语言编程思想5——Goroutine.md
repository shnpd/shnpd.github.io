---
title: Go语言编程思想5——Goroutine
toc: true
date: 2022-01-25 16:56:21
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

# Go语言编程思想5——Goroutine
**并发编程**



I/O操作本身有等待的过程会进行协程间的切换

**go** 建立一个goroutine，并发的执行后面的函数，主程序依然在运行。

以下函数不会输出任何内容，main函数和匿名函数同时执行，匿名函数还没来得及输出，main函数就执行完退出了，因此没有任何输出
```go
func main() {
	for i := 0; i < 10; i++ {
		go func(i int) {
			for {
				fmt.Printf("Hello from goroutine %d\n", i)
			}
		}(i)
	}
}
```


main 本身也是一个goroutine


```go
func main() {
	var a [10]int
	fmt.Println(a)
	for i := 0; i < 10; i++ {
		go func(i int) {
			for {
				a[i]++
				runtime.Gosched()//手动交出控制权
			}
		}(i)
	}
	time.Sleep(time.Millisecond)
	fmt.Println(a)
}
```

# 协程
goroutine实际上是一种协程


>协程 Coroutine
>- 轻量级“线程”（可以开1000个goroutine）
>- **非抢占式**多任务处理，由协程主动交出控制权
>- 编译器/解释器/虚拟机层面的多任务（不是OS层面的，OS只有线程没有协程）
>- 多个协程可能在一个或多个线程上运行

>**子程序是协程的一个特例**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/424fd250d64ccf494b50a04ce6f8ffb2_1740930687223.png)
普通函数main调用doWork函数，执行完成后将控制权交还给main函数，执行下一条语句
协程的main和doWork之间有一条双向的通道，数据和控制权都可以双向流通，相当于并发执行的两个线程，可以互相通信，交换控制权，main和doWork可能运行在一个线程也可能运行在多个线程，程序员只需要开两个协程即可，由调度器进行调度。

**其他语言中的协程**
- C++：Boost.Coroutine
- Java：不支持
- python：yield关键字实现协程，python3.5加入了async def对协程原生支持
- Go：原生支持 

**Go语言中的协程**

调度器负责调度协程
一个线程可能包含多个协程

- 任何函数只需加上go就能变成协程送给调度器运行
- 不需要在定义时区分是否是异步函数（python 需要加async def）
- 调度器在合适的点进行切换
   - 可能的切换点
   - I/O，select
   - channel
   - 等待锁
   - 函数调用（有时）
   - runtime.Gosched()
   - 只是参考，不能保证切换，不能保证在其他地方不切换
- 使用-race来检测数据访问冲突
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/8a66a386b7896d64e3f8b0ac259b9ecb_1740930687223.png)

