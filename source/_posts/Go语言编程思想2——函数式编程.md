---
title: Go语言编程思想2——函数式编程
toc: true
date: 2022-01-14 17:08:32
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

# Go语言编程思想2——函数式编程

- 函数是一等公民：参数，变量，返回值都可以是函数
- 高阶函数：函数的参数还可以是函数
- 函数->**闭包**


**“正统”函数式编程**
- 不可变性：不能有状态，只有常量和函数
- 函数只能有一个参数

# 闭包
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/3b054b7cd5a5f5e35f2f4f5e4f0881d9_1740930837520.png)

下例中，adder()的返回值就是一个函数体，其中v为函数体的局部变量。sum是函数所处的一个环境，是自由变量，编译器会将函数体连接到自由变量，在本例中sum是int型，实际上sum也可以是一个结构继续连接到其他变量，最终函数体会连接到全部的变量组成上图中树的形状，形成闭包。

返回值是一个闭包，不仅是func那段代码，而是这个函数以及对sum的引用，以及保存的sum变量
```go
func adder() func(int) int {
	sum := 0
	return func(v int) int {
		sum += v
		return sum
	}
}
func main() {
	a := adder()
	for i := 0; i < 10; i++ {
		fmt.Printf("0+1+...+%d=%d\n", i, a(i))
	}
}

```

“正统”函数式编程，不存在状态（没有变量sum）

```go
type iAdder func(int) (int, iAdder)

func adder2(base int) iAdder {
	return func(v int) (int, iAdder) {
		return base + v, adder2(base + v)
	}
}

func main() {
	a := adder2(0)
	for i := 0; i < 10; i++ {
		var s int
		s, a = a(i)
		fmt.Printf("0+1+...+%d=%d\n", i, s)
	}
}
```



# 例一：斐波那契数列

```go
func fibonacci() func() int {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		return a
	}
}
```

# 例二：为函数实现接口
```go

func fibonacci() intGen {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		return a
	}
}

type intGen func() int

// 函数作为接受者
func (g intGen) Read(p []byte) (n int, err error) {
	next := g()
	if next>10000{
		return 0, io.EOF
	}
	s := fmt.Sprintf("%d\n", next)

	//TODO:incorrect if p is too small!
	return strings.NewReader(s).Read(p)
}

func printFileContents(reader io.Reader){
	scanner :=bufio.NewScanner(reader)
	for scanner.Scan(){
		fmt.Println(scanner.Text())
	}
}

func main() {
	f := fibonacci()
	printFileContents(f)
}
```
# 例三：使用函数来遍历二叉树
将之前的print()换为一个函数，可以实现更多的功能

修改前：

```go
func (node *Node) Traverse() {
	if node == nil {
		return
	}
	node.Left.Traverse()
	node.print()
	node.Right.Traverse()
}
```

修改后
```go

func (node *Node) Traverse() {
	node.TraverseFunc(func(node *Node) {
		node.Print()
	})
	fmt.Println()
}

func (node *Node) TraverseFunc(f func(*Node)) {
	if node == nil {
		return
	}
	node.Left.TraverseFunc(f)
	f(node)
	node.Right.TraverseFunc(f)
}
```
将之前的打印换为一个函数，可以实现更多的功能
例如：结点计数

```go
	nodeCount:=0
	root.TraverseFunc(func(node *tree.Node) {
		nodeCount++
	})
```
# go语言闭包的应用
- 更为自然，不需要修饰如何访问自由变量
- 没有Lambda表达式，但是有匿名函数
