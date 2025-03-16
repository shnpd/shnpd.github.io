---
title: 【Go】Channel
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang
---

点击阅读更多查看文章内容<!--more-->

## Channel

Go 语言中的 **channel** 是一种用于在 goroutine 之间进行通信的机制，支持通过channel在不同的 goroutine 之间传递数据。通过 channel，Go 提供了类似消息队列的机制，使得并发编程变得更加直观与简单。

下面我将详细介绍 Go 语言中的 channel，包括其基本概念、使用方式、特性、和一些高级用法。

### 创建channel

创建 channel 使用内置函数 `make`。有两种主要的 channel 类型：

#### 无缓冲 channel

`ch := make(chan int)`

当发送者向无缓冲的 Channel 发送数据时，若没有接收者接收数据，则发送者会被阻塞，直到有接收者接收数据后才能继续执行。反之，当接收者从无缓冲的 Channel 中接收数据时，若没有发送者发送数据，则接收者会被阻塞，直到有发送者发送数据后才能继续执行。

无缓冲 Channel 的应用场景：

1. 多个 Goroutine 之间需要进行严格的同步和协调；
2. 希望确保发送和接收操作的顺序性；
3. 希望避免数据竞争和死锁的情况

#### 有缓冲 channel

`ch := make(chan int, 3) // 创建一个大小为 3 的缓冲区`

当发送者向有缓冲的 Channel 发送数据时，若缓冲区未满，则数据将被存储在缓冲区中。反之，若缓冲区已满，则发送者将被阻塞，直到有接收者从缓冲区中取出数据后才能继续执行。当接收者从有缓冲的 Channel 中接收数据时，若缓冲区不为空，则数据将被从缓冲区中取出并返回给接收者。反之，若缓冲区为空，则接收者将被阻塞，直到有发送者向缓冲区中发送数据后才能继续执行。

有缓冲 Channel 的应用场景：

1. 多个 Goroutine 之间需要进行异步通信；
2. 希望提高数据传输的效率和吞吐量；
3. 程序中存在发送和接收的速度不匹配的情况。

示例：

```go
func main() {
	ch := make(chan int)
	go func() {
		<-ch
	}()
	ch <- 1
	fmt.Println("1")
	ch <- 2
	fmt.Println("2")
}
```

以上代码的执行结果为：

```go
1
fatal error: all goroutines are asleep - deadlock!
```

首先创建了一个channel，随后启动了一个goroutine从channel中接收**一个**数据

然后执行 ch <- 1 向channel中发送一个数据，此时主goroutine阻塞不再执行后面的代码，当子goroutine接收到channel中的数据后主goroutine继续向后执行又向channel中发送了一个数据然后阻塞，此时没有goroutine能够接收数据一直阻塞就发生了死锁。

---

我们将创建一个缓冲区为1的channel

```go
func main() {
	ch := make(chan int, 1)
	go func() {
		time.Sleep(3 * time.Second)
		<-ch
	}()
	ch <- 1
	fmt.Println("1")
	ch <- 2
	fmt.Println("2")
	ch <- 3
	fmt.Println("3")
}
```

以上代码的执行结果为：

```go
1
2
fatal error: all goroutines are asleep - deadlock!
```

首先输出1，等待3s后输出2，随后进入死锁。

因为缓冲区为1，所以最开始向channel发送了一个数据后并不会阻塞直接输出1，此时再向channel中发送数据2会阻塞，等待3s后1从channel中取出此时阻塞停止，将2发送到channel中并执行后面的代码，后面再向channel中发送3会阻塞，此时没有goroutine能从channel中取出数据，陷入死锁。

> 以下是个人的理解：
>
> 没有缓冲区时，发送一个数据后阻塞，此时channel中有发送的数据。
>
> 有缓冲区时，缓冲区满再向其中发送数据会阻塞，此时channel中没有发送的数据



### 发送和接收数据

发送数据：`ch <- 42`

接收数据：`value := <- ch`



### 关闭channel

当发送者不再向 channel 发送数据时，可以关闭 channel。关闭 channel 可以通知接收者没有更多的数据可接收。

只能由发送方关闭 channel。

```go
close(ch)
```

关闭 channel 后，接收者继续从中接收数据时，如果 channel 中的数据已经接收完，接收操作会返回零值，并且可以通过 `ok` 标志来判断 channel 是否已关闭，如果channel已关闭则ok为false。

```go
value, ok := <-ch
if !ok {
    fmt.Println("Channel is closed and empty")
}
```

### Select 语句

`select` 语句类似于 `switch`，但是它用于 channel 操作。`select` 允许一个 goroutine 等待多个 channel 操作中的任意一个完成。它常用于并发控制和多路复用。

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "Message from ch1"
    }()
    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "Message from ch2"
    }()

    // 等待两个 channel 中的一个返回
    select {
    case msg1 := <-ch1:
        fmt.Println("Received:", msg1)
    case msg2 := <-ch2:
        fmt.Println("Received:", msg2)
    case <-time.After(3 * time.Second):
        fmt.Println("Timeout!")
    }
}
```

在这个例子中，`select` 会等待 `ch1` 或 `ch2` 中的消息返回，第一个完成的分支会执行。如果在 3 秒内没有接收到消息，`time.After` 会触发超时分支。

### for range遍历channel

`for range` 用于遍历 channel 时，会一直等待，直到该 channel 关闭，并且 channel 中的数据被完全接收完为止。

具体来说，`range` 遍历 channel 会持续进行以下几个步骤：

1. 如果 channel 中有数据，`range` 会接收并处理这些数据。
2. 如果 channel 中没有数据，`range` 会阻塞，直到有数据发送到 channel。

```go
func main() {
	ch := make(chan int, 3)

	// 启动一个 goroutine 发送数据到 channel
	go func() {
		for i := 1; i <= 3; i++ {
			ch <- i
			time.Sleep(1 * time.Second) // 模拟耗时操作
		}
		time.Sleep(5 * time.Second)
		close(ch) // 发送完数据后关闭 channel
	}()

	// 使用 for range 遍历 channel
	for value := range ch {
		fmt.Println(value) // 输出 channel 中的数据
	}
	fmt.Println("Channel is closed and all data read")
}
```

执行结果：每隔1s输出1，2，3；输出3后等待5s输出 Channel is closed and all data read

```go
1
2
3
Channel is closed and all data read
```



### 实际应用

以下是一个websocket的服务处理函数，主goroutine每隔200ms向连接发送一个数据，并启动了一个goroutine不断从连接中读取数据。

此时如果连接中断，子goroutine中的读取操作会返回错误表示连接中断，子goroutine需要将连接中断的消息发送给主goroutine结束连接。

`done := make(chan struct{})`定义一个channel，子goroutine中如果报错则向done中发送一个信号，主goroutine中通过select接收，如果200ms内没有收到done的内容则会收到time.After的返回，此时不执行select不执行任何操作直接发送后面的数据，如果在200ms内收到了done的信号，则会直接return。

> channel的内容为struct{}，因为这里只用作通知消息，无需传输实际的值，因此使用空结构体不占用任何内存，（在仅传输信号的场景中一般都是用空结构体，表达的意图更清晰可以直接判断channel的用途）。

```go
func handleWebSocket(w http.ResponseWriter, r *http.Request) {
	// 升级到websocket
	u := &websocket.Upgrader{
		CheckOrigin: func(r *http.Request) bool {
			return true
		},
	}
	c, err := u.Upgrade(w, r, nil)
	if err != nil {
		fmt.Printf("upgrade error: %v\n", err)
		return
	}
	defer c.Close()

	done := make(chan struct{})
	go func() {
		for {
			m := make(map[string]interface{})
			err := c.ReadJSON(&m)
			if err != nil {
				fmt.Printf("read error: %v\n", err)
				// 如果不是正常错误则打印错误信息
				if !websocket.IsCloseError(err, websocket.CloseGoingAway, websocket.CloseNormalClosure) {
					fmt.Printf("unexpected read error: %v\n", err)
				}
				// 通知主goroutine关闭
				done <- struct{}{}
				break
			}
			fmt.Printf("recv: %v\n", m)
		}
	}()

	i := 0
	for {
		// 每200ms发送一条消息，如果在200ms内收到done信号则退出
		select {
		case <-time.After(200 * time.Millisecond):
		case <-done:
			return
		}

		i++
		err := c.WriteJSON(map[string]string{
			"message": "hello",
			"msgid":   strconv.Itoa(i),
		})
		if err != nil {
			fmt.Printf("write error: %v\n", err)
		}
	}
}
```

## 设计原理

Go 语言中最常见的、也是经常被人提及的设计模式就是：不要通过共享内存的方式进行通信，而是应该通过通信的方式共享内存。在很多主流的编程语言中，多个线程传递数据的方式一般都是共享内存，为了解决线程竞争，我们需要限制同一时间能够读写这些变量的线程数量，然而这与 Go 语言鼓励的设计并不相同。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303144417921.png" alt="shared-memory" style="zoom: 67%;" />

虽然我们在 Go 语言中也能使用共享内存加互斥锁进行通信，但是 Go 语言提供了一种不同的并发模型，即通信顺序进程（Communicating sequential processes，CSP）[1](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/#fn:1)。Goroutine 和 Channel 分别对应 CSP 中的实体和传递信息的媒介，Goroutine 之间会通过 Channel 传递数据。

<img src="https://img.draveness.me/2020-01-28-15802171487080-channel-and-goroutines.png" alt="channel-and-goroutines" style="zoom:67%;" />

上图中的两个 Goroutine，一个会向 Channel 中发送数据，另一个会从 Channel 中接收数据，它们两者能够独立运行并不存在直接关联，但是能通过 Channel 间接完成通信。

**先入先出** 

目前的 Channel 收发操作均遵循了先进先出的设计，具体规则如下：

- 先从 Channel 读取数据的 Goroutine 会先接收到数据；
- 先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利；

---

## 数据结构

Go 语言的 Channel 在运行时使用 [`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan) 结构体表示。我们在 Go 语言中创建新的 Channel 时，实际上创建的都是如下所示的结构：

```go
type hchan struct {
	// chan 里元素数量
	qcount   uint
	// chan 底层循环数组的长度
	dataqsiz uint
	// 指向底层循环数组的指针
	// 只针对有缓冲的 channel
	buf      unsafe.Pointer
	// chan 中元素大小
	elemsize uint16
	// chan 是否被关闭的标志
	closed   uint32
	// chan 中元素类型
	elemtype *_type // element type
	// 已发送元素在循环数组中的索引
	sendx    uint   // send index
	// 已接收元素在循环数组中的索引
	recvx    uint   // receive index
	// 等待接收的 goroutine 队列
	recvq    waitq  // list of recv waiters
	// 等待发送的 goroutine 队列
	sendq    waitq  // list of send waiters

	// 保护 hchan 中所有字段
	lock mutex
}
```

- `buf` 指向底层循环数组，只有缓冲型的 channel 才有。
- `sendx`，`recvx` 均指向底层循环数组，表示当前可以发送和接收的元素位置索引值（相对于底层数组）。
- `sendq`，`recvq` 分别表示被阻塞的 goroutine，这些 goroutine 由于尝试读取 channel 或向 channel 发送数据而被阻塞。
- `waitq` 是 `sudog` 的一个双向链表，而 `sudog` 实际上是对 goroutine 的一个封装
- `lock` 用来保证每个读 channel 或写 channel 的操作都是原子的。

例如，创建一个容量为 6 的，元素为 int 型的 channel 数据结构如下 ：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303144420556.png" alt="chan data structure" style="zoom: 33%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303144417995.png" alt="在这里插入图片描述" style="zoom:50%;" />
