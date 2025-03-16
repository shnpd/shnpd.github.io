---
title: Go语言编程思想6——Channel
toc: true
date: 2022-01-26 14:56:22
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

# Go语言编程思想6——Channel
>**Channel：goroutine和goroutine之间双向的通道**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/a6be9387163121ce4e75a4df657703f3_1740930687223.png)
# 一、基本语法
- 创建int类型的channel
```c := make(chan int)```
- 发送数据
```c <- 1```
- 接受数据
```n := <-c```
```go
func chanDemo() {
	//创建int类型的channel
     c := make(chan int)
	//接收数据
	go func() {
		for {
			n := <-c
			fmt.Println(n)
		}
	}()
	//发送数据
	c <- 1
	c <- 2
}
```

---

channel发数据必须有goroutine接受，否则会发生死锁

```go
func chanDemo() {
	cha := make(chan int)
	cha <- 1
}
/*fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:                             
main.chanDemo(...)                                   
        D:/goworkstation/src/learngo/test.go:19      
main.main()                                          
        D:/goworkstation/src/learngo/test.go:22 +0x31*/

```

---

## Channel是一等公民
Channel可以作为参数或返回值

**chan<- int 只能向channel发数据
<-chan int 只能从channel收数据**
```go
func createWorker(id int) chan<- int { //发数据
	c := make(chan int)
	go func(){
		for{
			fmt.Printf("Worked %d received %c\n", id, <-c)
		}
	}()
	return c
}
func chanDemo() {
	//创建10个channel分发给10个worker
	var channels [10]chan<- int
	for i := 0; i < 10; i++ {
		channels[i] = createWorker(i)
	}
	//给10个channel分发数据
	for i := 0; i < 10; i++ {
		channels[i] <- 'a' + i
	}

	for i := 0; i < 10; i++ {
		channels[i] <- 'A' + i
	}

	time.Sleep(time.Millisecond)
}
```

---

## Channel缓冲区
发数据必须有人收，否则会deadlock
缓冲区大小设为3，只有在发送三个以上数据时才会出现deadlock
以下代码不会出现死锁
```go
func bufferedChannel() {
	c := make(chan int, 3)
	c <- 'a'
	c <- 'b'
	c <- 'c'
	time.Sleep(time.Millisecond)
}
```

---

## Channel的close

>发送方close：
>close\(c\) ，结束后若channel为空接收方依旧会接收到数据——(channel具体类型的零值)

>接收方判断channel是否还有值的两种方法：
>n, ok := <-c ，判断是否还有值，如果没有值了ok为false
>for n := range c {}，若没有值会自己跳出

```go
func worker(id int, c chan int) {
	//读完channel内的数据就退出的两种方法
	for n := range c {
		fmt.Printf("Worker %d received %c\n", id, n)
	}
	/*
	for {
		n, ok := <-c //如果没有值了ok为false
		if !ok {
			break
		}
		fmt.Printf("Worker %d received %c\n", id, n)
	}
	*/
}

func channelClose() {
	c := make(chan int)
	go worker(0, c)
	c <- 'a'
	c <- 'b'
	c <- 'c'
	c <- 'd'
	//发送方可以close
	//接收方有两种判断方法 ok，range
	close(c) //结束后依旧会接收到数据——(channel具体类型的零值)
	time.Sleep(time.Millisecond)
}
```

>Go语言并发执行理论基础：Communication Sequential Process (CSP)

>不要用共享内存实现通信，而要用通信实现共享内存

---

## Channel阻塞
发送者角度：对于同一个通道，发送操作（协程或者函数中的），在接收者准备好之前是阻塞的。如果chan中的数据无人接收，就无法再给通道传入其他数据。因为新的输入无法在通道非空的情况下传入。所以发送操作会等待 chan 再次变为可用状态：就是通道值被接收时（可以传入变量）。
接收者角度：对于同一个通道，接收操作是阻塞的（协程或函数中的），直到发送者可用：如果通道中没有数据，接收者就阻塞了。
通过一个简单的例子来说明：

```go
func f1(in chan int) {
	fmt.Println(<-in)
}

func main() {
	out := make(chan int)
	out <- 2
	go f1(out)
}
```
运行结果：fatal error: all goroutines are asleep - deadlock!

这是由于out <- 2之前不存在对out的接收，所以，对于out <- 2来说，永远是阻塞的，即一直会等下去。
　　
将out <- 2与go f1(out)互换
```go
func f1(in chan int) {
	fmt.Println(<-in)
}

func main() {
	out := make(chan int)
	go f1(out)
	out <- 2
}
```
运行结果：2

out <- 2前存在对通道的读操作，所以out <- 2 是合法的。就像前文说的，发送操作在接收者准备好之前是阻塞的。

---

# 二、使用Channel等待goroutine的结束
**一个死锁的例子**
>channel的发送和接收都是阻塞式的
>**发送操作在接受者准备好之前是阻塞的**，这里doWork中打印完小写字母之后给done发送了一个true，此时等待接受这个true的接受程序在后面，并没有准备好所以造成阻塞，而此时又向channel中发送打印大写字母，故造成死锁。

```go
type worker struct {
	in   chan int
	done chan bool
}
func doWork(id int, c chan int, done chan bool) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
		done <- true
	}
}
func createWorker(id int) worker {
	w := worker{
		in:   make(chan int),
		done: make(chan bool),
	}
	go doWork(id, w.in, w.done)
	return w
}
//所有channel的发送的都是阻塞式的
func chanDemo() {
	var workers [10]worker
	for i := 0; i < 10; i++ {
		workers[i] = createWorker(i)
	}
	for i, worker := range workers{
		worker.in <- 'a' + i
	}
	for i, worker := range workers {
		worker.in <- 'A' + i
	}
	for _, worker := range workers {
		<-worker.done
		<-worker.done
	}
}
```
**解决方法：**
再开一个goroutine并行

```go
func doWork(id int, c chan int, done chan bool) {
	for {
		fmt.Printf("Worker %d received %c\n", id, <-c)
		go func() { done <- true }()
	}
}
```

---


## WaitGroup的使用

等待多人完成任务

**方法**
Add(delta int)：添加任务个数
Wait()：等待任务完成
Done()：任务完成
```go

type worker struct {
	in   chan int
	done func()
}

func doWork(id int, w worker) {
	for n := range w.in {
		fmt.Printf("Worker %d received %c\n", id, n)
		//函数式编程，只调用done方法，具体执行什么函数由外面的createWorker来控制
		w.done()
	}
}

func createWorker(id int, wg *sync.WaitGroup) worker {
	w := worker{
		in: make(chan int),
		done: func() {
			wg.Done()
		},
	}
	go doWork(id, w)
	return w
}

//所有channel的发送的都是阻塞式的
func chanDemo() {
	var wg sync.WaitGroup

	var workers [10]worker
	for i := 0; i < 10; i++ {
		workers[i] = createWorker(i, &wg)
	}

	wg.Add(20)
	for i, worker := range workers {
		worker.in <- 'a' + i
	}

	for i, worker := range workers {
		worker.in <- 'A' + i
	}
	wg.Wait()
}
```
# 三、Select
>select 是 Go 中的一个控制结构，类似于用于通信的 switch 语句。每个 case 必须是一个通信操作，要么是发送要么是接收。
>select 随机执行一个可运行的 case。如果没有 case 可运行，它将阻塞，直到有 case 可运行。一个默认的子句应该总是可运行的。
>- 每个 case 都必须是一个通信
>- 所有 channel 表达式都会被求值
>- 所有被发送的表达式都会被求值
>- 如果任意某个通信可以进行，它就执行，其他被忽略。
如果有多个 case 都可以运行，Select 会随机公平地选出一个执行。其他不会执行。
否则：
如果有 default 子句，则执行该语句。
如果没有 default 子句，select 将阻塞，直到某个通信可以运行；Go 不会重新对 channel 或值进行求值。


在Select中可以使用Nil Channel，当数据还没准备好的时候可以把Channel置为nil，这样case就不会执行


select+default可以实现类似于非阻塞式的获取，如下示例输出结果为“no value”
```go
func main() {
	c1 := make(chan int)
	select {
	case n := <-c1:
		fmt.Println(n)
	default:
		fmt.Println("no value")
	}
}
```

**示例**
```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}
func worker(id int, c chan int) {
	for n := range c {
		time.Sleep(time.Second)
		fmt.Printf("Worker %d received %d\n", id, n)
	}
}
func createWorker(id int) chan<- int {
	c := make(chan int)
	go worker(id, c)
	return c
}
func main() {
	var c1, c2 chan int = generator(), generator()
	worker := createWorker(0)
	var values []int
	tm := time.After(10 * time.Second)
	tick := time.Tick(time.Second)
	for {
		var activeWorker chan<- int
		var activeValue int

		//values中存有数据，对activeWorker初始化
		if len(values) > 0 {
			activeWorker = worker
			activeValue = values[0]
		}
		select {
		case n := <-c1:
			values = append(values, n)
		case n := <-c2:
			values = append(values, n)

		//activeWorker没有值的时候为nil，此时阻塞不会执行
		case activeWorker <- activeValue:
			values = values[1:]

		
		//相邻两个请求之间超过800ms即select阻塞时间超过800ms，则输出timeout
		case <-time.After(800 * time.Millisecond):
			fmt.Println("timeout")

		//每隔一秒输出长度
		case <-tick:
			fmt.Println("queue len =", len(values))

		//总时间10s后退出
		case <-tm:
			fmt.Println("bye")
			return
		}
	}
}

```

time.After():是本次监听动作的超时时间， 意思就说，只有在本次select 操作中会有效， 再次select 又会重新开始计时
# 四、传统的同步机制
- WaitGroup
- Mutex
- Cond

传统同步机制（共享内存实现通信）较少使用，一般使用channel进行通信（通信实现共享内存）

**示例**
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type atomicInt struct {
	value int
	lock  sync.Mutex
}

func (a *atomicInt) increment() {
	fmt.Println("safe increment")
	func() {
		a.lock.Lock()
		defer a.lock.Unlock()
		a.value++
	}()
}

func (a *atomicInt) get() int {
	a.lock.Lock()
	defer a.lock.Unlock()
	return a.value
}

func main() {
	var a atomicInt
	a.increment()
	go func() {
		a.increment()
	}()
	time.Sleep(time.Millisecond)
	fmt.Println(a.get())
}

```
# 五、并发编程模式
- 生成器，可以将其抽象为服务/任务
- 同时等待多个服务：两种方法
    - select：知道channel的具体数量
    - 给每个channel开一个goroutine：不知道channel的具体数量（注意循环变量的坑，全局只有一份全局变量，在执行的时候只会向最后一个channel发送数据，可以通过拷贝一份变量或者通过函数传参来解决）

**示例**

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func msgGen(name string) chan string {
	c := make(chan string)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(2000)) * time.Millisecond)
			c <- fmt.Sprintf("service %s message %d", name, i)
			i++
		}
	}()
	return c
}

//不知道有多少个channel的时候用这种fanIn
func fanIn(chs ...chan string) chan string {
	c := make(chan string)
	for _, ch := range chs {
		//如果直接用ch的话，ch只有一个值，只会取最后一个channel的值送给c
		//所以要通过一个参数拷贝ch进行传递
		go func(in chan string) {
			for {
				c <- <-in
			}
		}(ch)
	}
	/*
	变量chCopy在全局有两份，通过chCopy拷贝ch
	for _, ch := range chs {
		chCopy := ch
		go func() {
			for {
				c <- <-chCopy
			}
		}()
	}
	*/
	return c
}

//明确channel个数时用select
func fanInBySelect(c1, c2 chan string) chan string {
	c := make(chan string)
	go func() {
		for {
			select {
			case m := <-c1:
				c <- m
			case m := <-c2:
				c <- m
			}
		}
	}()
	return c
}

func main() {
	m1 := msgGen("service1")
	m2 := msgGen("service2")
	m3 := msgGen("service3")
	m := fanIn(m1, m2, m3)
	for {
		fmt.Println(<-m)
	}
}

```
# 六、任务的控制
- 非阻塞等待
- 超时机制
- 任务中断/退出
- 优雅退出

**示例**
```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func msgGen(name string, done chan struct{}) chan string {
	c := make(chan string)
	go func() {
		i := 0
		for {
			select {
			case <-time.After(time.Duration(rand.Intn(5000)) * time.Millisecond):
				c <- fmt.Sprintf("service %s message %d", name, i)
			case <-done:
				fmt.Println("cleaning up")
				time.Sleep(2 * time.Second)
				fmt.Println("cleanup done")
				done<- struct{}{}//双向通信，对方收到done才退出
				return
			}
		}
	}()
	return c
}

//非阻塞等待
//等到返回true，没等到返回false
func nonBlockingWait(c chan string) (string, bool) {
	select {
	case m := <-c:
		return m, true
	default: //一旦阻塞就进入default
		return "", false
	}
}

//超时机制
func timeoutWait(c chan string, timeout time.Duration) (string, bool) {
	select {
	case m := <-c:
		return m, true
	case <-time.After(timeout):
		return "", false
	}
}
func main() {
	done := make(chan struct{})
	m1 := msgGen("service1", done)
	for i := 0; i < 5; i++ {
		if m, ok := timeoutWait(m1, time.Second); ok {
			fmt.Println(m)
		} else {
			fmt.Println("timeout")
		}
	}
	done <- struct{}{}
	<-done
}
```

