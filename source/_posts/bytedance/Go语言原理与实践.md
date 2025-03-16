---
title: Go语言原理与实践
toc: true
date: 2025-02-13 00:00:00
tags: golang
categories: bytedance
---

​点击阅读更多查看文章内容<!--more-->

## 什么是Go语言

1. 高性能、高并发

   - 与C++、Java相近的性能

   - 标准库内置高并发的支持

2. 语法简单、学习曲线平缓

3. 丰富的标准库

4. 完善的工具链

   - 编译、错误检查、包管理、代码提示
   - 单元测试框架

5. 静态链接

   - 只需要拷贝编译后的可执行文件就可以部署运行，在容器中运行可以使镜像体积非常小
   - C++需要附加.so文件才能运行、Java需要附加JRE

6. 快速编译

7. 跨平台

   - Windows、Linux、MacOS、Android、IOS、路由器、树莓派

8. 垃圾回收

**字节为什么使用GO？**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125023708.png" alt="image-20250122152652613" style="zoom:50%;" />

**学习路线**

![image-20250122215518133](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125025560.png)

![image-20250122215543046](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125030640.png)

![image-20250122215556627](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125035990.png)

## Go语言进阶与依赖管理

### 并发编程

并发：多线程程序在一个核的CPU上运行，通过时间片切换交替执行多个程序

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125053534.png" alt="image-20250122220119800" style="zoom:67%;" />

并行：利用多核实现多个线程的同时运行

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125109224.png" alt="image-20250122220124552" style="zoom:67%;" />

**协程 Goroutine**

协程开销很小，是Go语言适合高并发场景的原因

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125115453.png" alt="image-20250122220355159" style="zoom:67%;" />

**CSP（Communicating Sequential Processes）**

协程间通信

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125120957.png" alt="image-20250122221117328" style="zoom:67%;" />

**Channel**

通过通信共享内存

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125135745.png" alt="image-20250122222345782" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125141332.png" alt="image-20250122222355807" style="zoom:50%;" />

**通过共享内存实现通信**

对共享内存的访问需要加锁

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125340107.png" alt="image-20250122222746635" style="zoom:50%;" />

**WaitGroup**

之前的例子是通过等待1s来等待协程执行完毕，这里可以通过WaitGroup使用计数器优雅的等待协程执行完毕

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125344841.png" alt="image-20250122222947065" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125348686.png" alt="image-20250122223024013" style="zoom:50%;" />

### 依赖管理

Go依赖管理演进

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125354556.png" alt="image-20250122223153261" style="zoom:50%;" />

**GOPATH**

所有项目共享同一个GOPATH

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125402377.png" alt="image-20250122223320351" style="zoom:50%;" />

弊端：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125406868.png" alt="image-20250122224119733" style="zoom:50%;" />

**Go Vendor**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125414545.png" alt="image-20250122224238019" style="zoom: 50%;" />

弊端：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125419355.png" alt="image-20250122224326252" style="zoom:50%;" />

**Go Module**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125423682.png" alt="image-20250122224520309" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125427825.png" alt="image-20250122224530555" style="zoom:50%;" />

**go.mod**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125435304.png" alt="image-20250122224626369" style="zoom:50%;" />

**version**

major是大版本，不同major可能是不兼容的，minor是大版本下的下版本，同一大版本下是兼容的，patch是对bug的修复 

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125439996.png" alt="image-20250122224943752" style="zoom:50%;" />

版本前缀-时间戳-哈希

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125443853.png" alt="image-20250122224952036" style="zoom:50%;" />

**indirect**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125447449.png" alt="image-20250122225007295" style="zoom:50%;" />

**incompatible**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125452074.png" alt="image-20250122225254839" style="zoom:50%;" />



如果两个依赖库需要不同版本的同一依赖库，而这些版本无法同时兼容，Go Vendor 无法解决此问题。

Go Modules 会自动选择一个最小兼容版本，避免冲突。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125457064.png" alt="image-20250122225526714" style="zoom:50%;" />

**依赖分发**

不会直接去Github等依赖的源仓库拉取

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125501565.png" alt="image-20250122225659055" style="zoom: 50%;" />

通过Go Proxy缓存源站中的软件内容，缓存的软件版本不会改变，并且在源站删除之后依然可用

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125505108.png" alt="image-20250122225724272" style="zoom:50%;" />

**GOPROXY**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125509071.png" alt="image-20250122225848285" style="zoom:50%;" />

**工具 - go get**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125513018.png" alt="image-20250122225936094" style="zoom:50%;" />

**工具 - go mod**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125517134.png" alt="image-20250122225957337" style="zoom:50%;" />

## 测试

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125521298.png" alt="image-20250122230535255" style="zoom:50%;" />

- 回归测试：一般是QA同学手动通过终端回归些固定的主流程场景
- 集成测试：对系统功能维度做测试验证
- 单元测试：测试开发阶段，开发者对单独的函数、模块做功能验证

层级从上至下，测试成本逐渐减低，而测试覆盖率确逐步上升，所以单元测试的覆盖率定程度上决定这代码的质量。

### 单元测试

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125525595.png" alt="image-20250122230651301" style="zoom:50%;" />

规则：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125529320.png" alt="image-20250122230749558" style="zoom:50%;" />

覆盖率：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125533428.png" alt="image-20250122232017474" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125540909.png" alt="image-20250122232127066" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125545322.png" alt="image-20250122232139962" style="zoom:50%;" />

---

依赖：单元测试可能需要依赖外部的文件数据库等，单元测试要求外部依赖是幂等且稳定的

- 幂等：重复运行case的结果是相同的
- 稳定：单元测试是隔离的，能在任何时间，任何环境，运行测试。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125549700.png" alt="image-20250122232354740" style="zoom:50%;" />

要实现稳定和幂等就需要Mock函数

例子：文件处理，将第一行字符串中的11替换成00，需要依赖本地文件，如果文件修改测试就会fail

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558342.png" alt="image-20250122232646431" style="zoom:50%;" />

使用开源mock测试库monkey，对method进行mock

Monckey Patch 的作用域在 Runtime，在运行时通过Go的unsafe包，能够将内存中函数的地址替换为运行时函数的地址，将target的方法实现跳转到replacement

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558343.png" alt="image-20250122232713722" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558344.png" alt="image-20250122233212370" style="zoom:50%;" />

---

### 基准测试

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558345.png" alt="image-20250122233410720" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558346.png" alt="image-20250122233422136" style="zoom: 67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558347.png" alt="image-20250122233759927" style="zoom: 67%;" />

Benchmark开头，入参为testing.B，用b中的N值反复递增循环测试

(对一个测试用例的默认测试时间是1秒，当测试用例函数返回时还不到1秒，那么testing.B中的N值将按1、2、5、10、 20、 ....递增，并以递增后的值重新进行用例函数测试。)

Resttimer重置计时器，我们再reset之前做了init或其他的准备操作，这些操作不应该作为基准测试的范围; runparaller是多协程并发测试;执行2个基准测试，发现代码在并发情况下存在劣化，主要原因是rand为了保证全局的随机性和并发安全，持有了一把全局锁。

**使用fastrand优化**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558348.png" alt="image-20250122233948621" style="zoom:67%;" />

## 高质量编程简介及编码规范

### 编码规范

**代码格式**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558349.png" alt="image-20250123102742429" style="zoom:50%;" />

**注释**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558350.png" alt="image-20250123102814012" style="zoom:50%;" />

Open()的注释解释了函数的作用，IsTableFull()的注释实际上没有提供一些额外的信息

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558351.png" alt="image-20250123102827129" style="zoom: 67%;" />

第二个for循环的注释没有什么作用

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558352.png" alt="image-20250123103044036" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558353.png" alt="image-20250123103137624" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558354.png" alt="image-20250123103149807" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558355.png" alt="image-20250123103325507" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558356.png" alt="image-20250123103332349" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558357.png" alt="image-20250123103346998" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558358.png" alt="image-20250123103354175" style="zoom:67%;" />

**命名规范**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558359.png" alt="image-20250123103515372" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558360.png" alt="image-20250123103530943" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558361.png" alt="image-20250123103556044" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558362.png" alt="image-20250123103741298" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558363.png" alt="image-20250123103833562" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558364.png" alt="image-20250123103848758" style="zoom:67%;" />

**控制流程**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558365.png" alt="image-20250123103931638" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558366.png" alt="image-20250123103954166" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558367.png" alt="image-20250123104115753" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558368.png" alt="image-20250123104145030" style="zoom:67%;" />

**错误和异常处理**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558369.png" alt="image-20250123104227159" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558370.png" alt="image-20250123104317872" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558371.png" alt="image-20250123104409602" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558372.png" alt="image-20250123104423988" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558373.png" alt="image-20250123104523677" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558374.png" alt="image-20250123104619633" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558375.png" alt="image-20250123104641510" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558376.png" alt="image-20250123104648252" style="zoom:67%;" />

### 性能优化

**性能优化建议**

benchmark

函数命名必须以Benchmark开头，后面跟的单词必须以大写字母开头

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558378.png" alt="image-20250123110855049" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558379.png" alt="image-20250123111354113" style="zoom:67%;" />

**Slice**

预分配内存

data := make([]int, 0, size)：长度为0，容量为size

data := make([]int, size)：长度，容量都为size

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558380.png" alt="image-20250123112206318" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558381.png" alt="image-20250123112420595" style="zoom: 50%;" />

大内存未释放

```go
func main() {
	data := make([]int, 100)
	data2 := data[:2]
	fmt.Println(cap(data2))
}
// 100
```

```go
func main() {
	data := make([]int, 100)
	// 必须先为data2分配内存才能copy，否则会panic
	data2 := make([]int, 2)
	copy(data2, data[:2])
	fmt.Println(cap(data2))
	// 2
}
```

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558382.png" alt="image-20250123112634329" style="zoom: 50%;" />

**Map**

预分配内存

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558383.png" alt="image-20250123113331272" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558384.png" alt="image-20250123113338257" style="zoom:67%;" />

**字符串处理**

使用strings.Builder

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558385.png" alt="image-20250123113443683" style="zoom: 50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558386.png" alt="image-20250123113914497" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558387.png" alt="image-20250123113935731" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558388.png" alt="image-20250123113954450" style="zoom: 50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558389.png" alt="image-20250123114807705" style="zoom:50%;" />

**空结构体**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558390.png" alt="image-20250123114952316" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558391.png" alt="image-20250123115001008" style="zoom:50%;" />

**atomic包**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558392.png" alt="image-20250123115028623" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558393.png" alt="image-20250123115040568" style="zoom: 67%;" />

**小结**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558394.png" alt="image-20250123115049414" style="zoom: 67%;" />

### 性能优化分析工具

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558395.png" alt="image-20250123125459514" style="zoom: 67%;" />

**性能分析工具pprof**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558396.png" alt="image-20250123125547173" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558397.png" alt="image-20250123125609275" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558398.png" alt="image-20250123231636479" style="zoom:67%;" />

启动main函数，访问 http://localhost:6060/debug/pprof/ 在浏览器查看指标

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558399.png" alt="image-20250123232255802" style="zoom:67%;" />

采集10s的CPU数据 ` go tool pprof "http://localhost:6060/debug/pprof/profile?seconds=10"`

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558400.png" alt="image-20250123232752702" style="zoom:67%;" />

输入top查看占用资源最多的函数

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558401.png" alt="image-20250123232836032" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558402.png" alt="image-20250123232854933" style="zoom:67%;" />

Eat函数占用资源最多，使用`list Eat`查看函数

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558403.png" alt="image-20250123233112882" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558404.png" alt="image-20250123233204932" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125558405.png" alt="image-20250123234000475" style="zoom:67%;" />
