---
title: 【Go】常见问题
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang



---

点击阅读更多查看文章内容<!--more-->

## 协程与线程的区别

协程：由 Go 语言 runtime 管理的轻量级并发执行单元，是用户态的线程，可以通过用户程序创建、删除。协程切换时不需要切换内核态。

线程：由操作系统内核管理的并发执行单元。

区别：

1. 调度：线程是操作系统的概念，而协程是程序级的概念。线程由操作系统内核调度。而协程由Go runtime 负责调度，基于 G-M-P 模型。
2. 性能：线程上下文切换涉及用户态和内核态切换，开销较大。而协程切换时不需要操作系统的介入，上下文切换完全在用户态完成，切换开销较小。
3. 并发模型：协程基于 CSP 模型（Communicating Sequential Processes），通过 Channel 通信。通信更安全，避免数据竞争。线程基于共享内存，需要锁（如互斥锁）同步。容易引发数据竞争，需要开发者手动管理锁。
4. 内存占用：协程初始栈大小 2KB，可动态扩容（最大 1GB）。线程默认栈大小 1MB（不同操作系统可能不同）

---

## new和make的区别

var声明值类型的变量时，系统会默认为他分配内存空间，并赋该类型的零值
如果是指针类型或者引用类型的变量，系统不会为它分配内存，默认是nil。

1.make 仅用来分配及初始化类型为 slice、map、chan 的数据。
2.new 可分配任意类型的数据，根据传入的类型申请一块内存，返回指向这块内存的指针，即类型 *Type。
3.make 返回引用，即 Type，new 分配的空间被清零， make 分配空间后，会进行初始。
4.make函数返回的是slice、map、chan类型本身
5.new函数返回一个指向该类型内存地址的指针

---

## Golang的slice的实现原理

slice不是线程安全的
切片是基于数组实现的，底层是数组，可以理解为对底层数组的抽象

```golang
type slice struct{
	array unsafe.Pointer
	len int
	cap int
}
```

slice占24个字节
array：指向底层数组的指针，占用8个字节
len: 切片的长度，占用8个字节
cap：切片的容量，cap总是大于等于len，占用8个字节

初始化slice调用的是runtime.makeslice，makeslice函数的工作主要就是计算slice所需内存大小，然后调用mallocgc进行内存的分配

所需内存的大小=切片中元素大小*切片的容量

---

## array和slice的区别

1. 长度不同
   - 数组初始化必须指定长度，并且长度就是固定的 
   - 切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大

2. 变量类型不同
   - 数组是值类型：将一个数组赋值给另一个数组时，传递的是一份深拷贝，函数传参操作都会复制整个数组数据，会占用额外的内存，函数内对数组元素值的修改，不会修改原 数组内容。
   - 切片是引用类型：切片是引用类型，将一个切片赋值给另一个切片时，传递的是一份浅拷贝，函数传参操作不会拷贝整个切片，只会复制len和cap，底层共用同一个数组，不会占用额外的内存，函数内对数组元素值的修改，会修改原数组内容。

3. 底层实现
   - 数组：数组是一个连续的内存块，存储固定数量的元素。
   - 切牌：是一个结构体，包含指向底层数组的指针、长度和容量。

4. 性能/使用场景
   - 数组更高效但灵活性差，适合固定大小的集合；存储在连续的内存中，访问速度快。
   - 切片灵活但可能有性能开销，适合动态大小的集合；切片的底层数组可能需要在运行时动态分配和扩容，涉及额外的内存管理开销。

---

## 原子操作

Go atomic包是最轻量级的锁（也称无锁结构），可以在不形成临界区和创建互斥量的情况下完成并发安全的值替换操作，不过这个包只支持int32/Int64/uint32/uint64/uintptr这几种数据类型的一些基础操作（增减、交换、载入、存储等）

原子操作仅会由一个独立的CPU指令代表和完成。原子操作是无锁的，常常直接通过CPU指令直接实现。事实上，其它同步技术的实现常常依赖于原子操作。

当我们想要对某个变量并发安全的修改，除了使用官方提供的 mutex，还可以使用sync/atomic包的原子操作，它能够保证对变量的读取或修改期间不被其他的协程所影响。atomic 包提供的原子操作能够确保任一时刻只有一个goroutine对变量进行操作，善用atomic能够避免程序中出现大量的锁操作。

```go
func add(addr *int64， delta int64) {
atomic.AddInt64(addr, delta)//加操作
fmt.Println("add opts: ",*addr)
}
```

原子操作和锁的区别
1.原子操作由底层硬件支持，而锁是基于原子操作+信号量完成的。若实现相同的功能，前者通常会更有效率
2.原子操作是单个指令的互斥操作；互斥锁/读写锁是一种数据结构，可以完成临界区（多个指令）的互斥操作，扩大原子操作的范围
3.原子操作是无锁操作，属于乐观锁；说起锁的时候，一般属于悲观锁
4.原子操作存在于各个指令/语言层级，比如*机器指令层级的原子操作"，““汇编指令层级的原子操作”，“Go语言层级的原子操作”等。
5.锁也存在于各个指令/语言层级中，比如“机器指令层级的锁”，“汇编指令层级的锁“Go语言层级的锁“等

---

## Channel死锁场景

1.非缓存channel只写不读
2.非缓存channel读在写后面
3.缓存channel写入超过缓冲区数量
4.空读
5.多个协程相互等待

---

## Go的Struct能不能比较？

1.相同struct类型的可以比较
2.不同struct类型的不可以比较,编译都不过，类型不匹配

---

## 内存泄漏

内存泄露（Memory Leak）是指程序在运行过程中分配的内存无法被垃圾回收器（GC）回收，导致内存占用持续增加，最终可能耗尽系统内存。Go 的垃圾回收机制虽然强大，但在某些场景下仍可能发生内存泄露。以下是内存泄露的常见原因、检测方法和预防措施：

**全局变量或长生命周期对象的引用**

全局变量或长生命周期对象（如缓存、单例）持有对某些对象的引用，导致这些对象无法被回收。

```go
var cache = make(map[string]*BigObject)

func addToCache(key string, obj *BigObject) {
    cache[key] = obj
}
```

如果 `cache` 中的对象不再使用但没有删除，会导致内存泄露。

**未关闭的资源**

未关闭的文件、网络连接、数据库连接等资源会占用内存，导致泄露。

```go
func readFile() {
    file, _ := os.Open("data.txt")
    // 未调用 file.Close()
}
```

**Goroutine 泄露**

Goroutine 未正确退出，导致其引用的资源无法被回收。

```go
func leakyFunc() {
    go func() {
        for {
            time.Sleep(time.Second)
        }
    }()
}
```

**Channel 阻塞**

Goroutine 因 Channel 阻塞而无法退出，导致泄露。

```go
func leakyChan() {
    ch := make(chan int)
    go func() {
        ch <- 1
    }()
    // 未读取 ch，导致 Goroutine 阻塞
}
```

**循环引用**

```go
type Node struct {
    next *Node
}

func createCycle() {
    a := &Node{}
    b := &Node{}
    a.next = b
    b.next = a // 循环引用
}
```

---

## go 打印时 %v %+v %#v 的区别？

%v 只输出所有的值；
%+v 先输出字段名字，再输出该字段的值；
%#v 先输出结构体名字值，再输出结构体（字段名字+字段的值）；

---

## 什么是 rune 类型？
Go语言的字符有以下两种：

1.uint8 类型，或者叫 byte 型，代表了 ASCII 码的一个字符。
2.rune 类型，代表一个 UTF-8 字符，当需要处理中文、日文或者其他复合字符时，则需要用到 rune 类型。rune 类型等价于 int32 类型。

单个字符为int32，字符串中取字符为uint8

```go
func main() {
    var a = 'c'
    fmt.Println(reflect.TypeOf(a)) // int32
    var b = "abc"
    fmt.Println(reflect.TypeOf(b[0])) // uint8
}
```

在使用range遍历字符串时直接取得的字符为int32，下标取得的字符为uint8（range取得的v是t[i]的副本）

```go
	t := "avc"
	for i, v := range t {
		fmt.Println(reflect.TypeOf(t[i])) //uint8
		fmt.Println(reflect.TypeOf(v)) //int32
	}
```

---

## 空 struct{} 占用空间么？用途是什么？
空结构体 struct{} 实例不占据任何的内存空间。

用途：

1.将 map 作为集合(Set)使用时，可以将值类型定义为空结构体，仅作为占位符使用即可。

2.不发送数据的信道(channel)
使用 channel 不需要发送任何的数据，只用来通知子协程(goroutine)执行任务，或只用来控制协程并发度。

```go
func worker(done chan struct{}) {
    fmt.Println("Working...")
    time.Sleep(time.Second)
    fmt.Println("Done")
    done <- struct{}{} // 发送完成信号
}

func main() {
    done := make(chan struct{})
    go worker(done)
    <-done // 等待任务完成
    fmt.Println("Worker finished")
}
```

3.结构体只包含方法，不包含任何的字段

---

## golang值接收者和指针接收者的区别

golang函数与方法的区别是，方法有一个接收者。

如果方法的接收者是指针类型，无论调用者是对象还是对象指针，修改的都是对象本身，会影响调用者

如果方法的接收者是值类型，无论调用者是对象还是对象指针，修改的都是对象的副本，不影响调用者

通常我们使用指针类型作为方法的接收者的理由：

1. 使用指针类型能够修改调用者的值
2. 使用指针类型可以避免在每次调用方法时复制该值，在值的类型为大型结构体时，这样做更加高效

---

## 引用传递和值传递
什么是引用传递?
将实参的地址传递给形参，函数内对形参值内容的修改，将会影响实参的值内容。**Go语言是没有引用传递的**，向函数传参时都会传递变量的副本，不过有的变量是引用类型它的值本身是就是一个指针因此即使传递副本他们内部的指针还是一样的。
Go的值类型(int、struct等）、引用类型（指针、slice、map、 channel)

---

## defer关键字

1. **延迟执行**
   `defer` 语句会将函数调用压入一个栈中，在当前函数返回之前按照 ​**后进先出（LIFO）​**​ 的顺序执行。

2. **参数预计算**
   `defer` 语句中的函数参数会在 `defer` 语句执行时立即计算，而不是在延迟调用时计算。

   ```go
   func main() {
       i := 1
       defer fmt.Println("Deferred call:", i)
       i++
       fmt.Println("Main function:", i)
   }
   // Main function: 2
   // Deferred call: 1
   ```

3. **作用域**
   `defer` 语句的作用域是当前函数，函数返回时才会执行 `defer` 语句。

4. 如果函数中有多个 `defer` 语句，它们会按照 ​**后进先出（LIFO）​**​ 的顺序执行。

5. panic后的defer不会被执行，panic之前的defer会被执行（遇到panic，如果没有捕获错误，函数会立刻终止）

6. panic没有被recover时，抛出的panic到当前goroutine最上层函数时，最上层程序直接异常终止

7. defer、return、返回值三者的执行逻辑应该是：return最先执行，return负责将结果写入返回值中；接着defer开始执行一些收尾工作；最后函数携带当前返回值退出

   ```go
   func testd() int {
       i := 1
       defer func() {
          i++
       }()
       return i
   }
   
   func main() {
       fmt.Println(testd())
   }
   // 1
   ```

---

## select

`select` 是一种用于处理多 Channel 操作的机制，类似于 `switch` 语句，但专门用于 Channel 的读写操作。`select` 的主要作用是 **监听多个 Channel 的操作**，并在其中一个 Channel 就绪时执行对应的分支。以下是 `select` 的详细使用方法和底层原理。

```go
select {
case <-ch1:
    // ch1 可读时执行
case ch2 <- value:
    // ch2 可写时执行
default:
    // 没有任何 Channel 就绪时执行
}
```

**使用场景**

- **多 Channel 监听**：同时监听多个 Channel 的读写操作。
- **超时控制**：结合 `time.After` 实现超时机制。
- **非阻塞操作**：使用 `default` 分支实现非阻塞的 Channel 操作。



**`selectgo` 的执行流程**

`runtime.selectgo` 的执行流程如下：

1. 初始化：

   - 遍历所有 `case` 分支，检查每个 Channel 的状态（是否可读或可写）。
   - 将可操作的 Channel 加入一个随机顺序的列表中。

2. 随机选择：

   - 从可操作的 Channel 中随机选择一个执行。
   - 如果没有任何 Channel 就绪，则执行 `default` 分支（如果存在）。

3. 执行分支：

   - 执行选中的 `case` 分支，完成对应的 Channel 操作。

4. 返回结果：

   - 返回选中的 `case` 分支的索引，以及是否成功执行

   

**`select` 阻塞的实现细节**

- 如果 `select` 的所有 `case` 分支都阻塞，并且没有 `default` 分支，`select` 会 阻塞，而不是自旋。
- `select` 的阻塞行为是通过挂起当前 Goroutine 实现的，不会浪费 CPU 资源。
- `select` 的底层实现依赖于 Go 的运行时机制，包括 Goroutine 的挂起和唤醒。

1. Goroutine 的挂起

当 `select` 阻塞时，当前 Goroutine 会被挂起，并加入到所有相关 Channel 的等待队列中。具体步骤如下：

1. 检查 Channel 状态：遍历所有 `case` 分支，检查每个 Channel 是否可读或可写。
2. 加入等待队列：如果没有任何 Channel 就绪，将当前 Goroutine 加入到所有相关 Channel 的等待队列中。
3. 挂起 Goroutine：将 Goroutine 的状态设置为 `Gwaiting`，并释放 CPU 资源。

2. Goroutine 的唤醒

当任何一个 Channel 就绪时，Go 的运行时机制会唤醒等待的 Goroutine，并执行对应的 `case` 分支。具体步骤如下：

1. Channel 就绪：某个 Channel 接收到数据或可以发送数据。
2. 唤醒 Goroutine：从 Channel 的等待队列中取出 Goroutine，并将其状态设置为 `Grunnable`。
3. 执行 `case` 分支：`runtime.selectgo` 函数返回对应的 `case` 分支索引，执行对应的代码。

---

## 服务发现是怎么做的？
主要有两种服务发现机制：客户端发现和服务端发现

客户端发现：是指客户端应用程序主动扫描、获取和管理可用的服务实例列表。**客户端通过向服务注册中心发送请求来获取服务实例列表，然后根据负载均衡算法选择其中一台实例进行服务调用。**客户端负责维护可用实例列表，包括实例的添加、删除和更新等操作。**客户端发现需要在客户端应用程序中集成相应的服务发现客户端**，例如Netflix的Eureka客户端。

服务端发现：是指服务注册中心负责主动维护和管理可用的服务实例列表。服务实例在启动时向服务注册中心进行注册，注册中心负责记录和维护服务实例的信息，包括实例的网络地址、健康状态、负载情况等。**当客户端需要调用服务时，向服务注册中心发送请求，注册中心根据负载均衡算法选择合适的实例返回给客户端**。服务端发现通常需要使用专门的服务注册中心，例如Consul、ZooKeeper等。

---

## HTTP和RPC对比
RPC（Remote Produce Call）：远程过程调用，
HTTP：网络传输协议

相同点：

1. 都是基于TCP协议的应用层协议
2. 都可以实现远程调用，服务调用服务

不同点：

1. RPC主要用于在不同的进程或计算机之间进行函数调用和数据交换。HTTP主要用于数据传输和通信。
2. RPC协议通常采用二进制协议和高效的序列化方式，而HTTP通常采用文本协议和基于ASCII码的编码方式，数据传输效率较低
3. RPC通常需要使用专门的IDL文件来定义服务和消息类型，生成服务端和客户端的代码。而HTTP没有这个限制，可以使用套接字进行通信

---

## gRPC和RPC对比

gRPC是一种高性能，通用的远程过程调用（RPC）框架，采用基于**HTTP/2**的二进制传输协议实现可以实现双向流、头部压缩和多路复用等，使用**Protocol Buffers**作为默认的序列化协议

区别：

1. 通信协议不同：gRPC是基于HTTP/2协议进行数据传输，而传统的RPC框架通常使用TCP和UDP等传输层协议
2. 序列化方式不同：gRPC使用Protocol Buffers作为默认的序列化协议，而传统的RPC框架使用JSON、XML等格式。
3. 支持多种语言：gRPC支持多种编程语言，包括C++、Java、Python、Go、Ruby等，而传统的RPC框架通常只支持少数几种语言
4. 高性能：由于采用了HTTP/2协议和Protocol Buffers序列化协议，gRPC具有更高的性能和效率
5. 自动生成代码：gRPC可以根据服务定义文件自动生成客户端和服务端的代码，大大简化了开发过程。
6. 安全性：gRPC提供了TLS加密和认证等安全机制，保障通信的安全性。

---

## Sync.Pool的使用

在 Go 中，频繁地创建和销毁对象会导致以下问题：

- **内存分配开销**：每次创建对象都需要分配内存，可能增加 GC（垃圾回收）的压力。
- **GC 性能下降**：大量临时对象会增加 GC 的工作量，导致程序性能下降。

`sync.Pool` 通过缓存和复用对象，可以减少内存分配和 GC 的开销，从而提高程序性能

**`sync.Pool` 的核心特性**

- **对象复用**：`sync.Pool` 缓存对象，供后续复用。
- **线程安全**：`sync.Pool` 是并发安全的，多个 goroutine 可以安全地从中获取和放回对象。
- **自动清理**：`sync.Pool` 中的对象可能会被 GC 清理，因此不能依赖它来长期保存对象。

```go
func main() {
    pool := &sync.Pool{
       New: func() interface{} {
          fmt.Println("Creating new object")
          return make([]byte, 1024) // 创建一个 1KB 的字节切片
       },
    }

    // 从池中获取对象
    obj := pool.Get().([]byte)
    fmt.Println("Got object from pool", obj)

    // 使用对象
    obj[0] = 1

    // 将对象放回池中
    pool.Put(obj)
    fmt.Println("Put object back to pool", obj)

    // 再次从池中获取对象（可能复用之前放回的对象）
    obj2 := pool.Get().([]byte)
    fmt.Println("Got object from pool again", obj2)
}
```

---

## Gin框架

1.支持中间件操作（handlersChain机制） 

2.更方便的使用（gin.Context）

3.更强大的路由解析能力（radix tree路由树）

---

## 协程池

协程已经很轻量了，为什么还要有协程池？

1. 限制协程的数量，不让协程无限制地增长
2. 减少GC和协程创建的开销

---

## golang中指针的作用

1. 传递大对象
2. 修改函数外部变量
3. 动态分配内存
4. 函数返回指针

---

## for range

在 Go 语言中，使用 `for i, list := range lists` 遍历切片时，变量 `list` 是切片元素的**副本**，lists[i]取得的才是切片中实际的元素。

在使用range遍历字符串时直接取得的字符为int32，下标取得的字符为uint8

```go
	t := "avc"
	for i, v := range t {
		fmt.Println(reflect.TypeOf(t[i])) //uint8
		fmt.Println(reflect.TypeOf(v)) //int32
	}
```
