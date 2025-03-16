---
title: Go语言基础知识3——面向对象
toc: true
date: 2022-01-12 23:51:01
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

>go语言仅支持封装，不支持继承和多态

>go语言没有class，只有struct
# 一、结构体和方法

## 结构的创建

不论地址还是结构本身，一律使用.来访问成员（在C++中指针通过->访问成员）

```go
	type treeNode struct {
	value       int
	left, right *treeNode
 	}

	func main() {
	    var root treeNode
		root = treeNode{value: 3}
		fmt.Println(root)
		
		root.left = &treeNode{}
		
		root.right = &treeNode{5, nil, nil}
		
		root.right.left = new(treeNode)
		
		nodes:=[]treeNode{
			{value: 3},
			{},
			{6,nil,&root},
		}
	}
```
Golang中没有构造函数，可以使用工厂函数，注意这里返回了局部变量的地址（在C++中这是一个明显的错误，但是在Golang中可以）
```go
    func createTreeNode(value int) *treeNode {
	    return &treeNode{value: value}
    }
    root.Left.Right = CreateTreeNode(2)
```

>**结构创建在堆上还是栈上？**
>在C++中，局部变量创建在栈上，函数一旦退出局部变量就会立刻被销毁，传出函数必须在堆上分配，堆上分配需要手动释放
>在Java中，几乎所有的东西都是分配在堆上的（都要用new），因此会有垃圾回收机制
>在Golang中，不一定，是由编译器及运行环境所决定的，以上述函数为例，编译器发现treeNode取地址返回，则会将它在堆上分配。如果treeNode不需要返回出去，则会在栈上分配。

---

## 为结构定义方法

- 在方法名前显式定义和命名方法的**接受者**

- 使用指针作为方法接受者可以改变结构内容<font color="red">（Go语言所有的参数都是值传递，不用指针的话不能修改内容）</font>

- nil指针也可以调用方法

- 使用.调用方法


```go
func (node treeNode) print() {
	fmt.Print(node.value)
}

func (node *treeNode) setValue(value int) {
	node.value = value
}

```

>中序遍历

```go
func (node *treeNode) traverse() {
	if node==nil{
		return
	}

	node.left.traverse()
	node.print()
	node.right.traverse()
}
```

>要改变内容必须使用指针接受者
>结构过大也考虑使用指针接受者
>建议：一致性，如有指针接收者，最好都是指针接收者
>值接受者 是go语言特有
>**值/指针接受者均可接收值/指针，go语言会自动转换**

---

# 二、包和封装
## 封装
- 名字一般使用CamelCase
- **首字母大写：public**
- **首字母小写：private**
- public和private是针对**包** 而言的

## 包
- 每个目录一个包（包名和目录名可以不同）
- main包包含可执行入口（包含main函数，如果包下有main函数，这个包只能是main包，否则可以取其他名字）
- 为结构定义的方法必须放在同一个包内，可以是不同的文件
- 引用其他包时要通过“包名.方法”或“包名.变量名”来使用

>如下目录所示，tree\entry目录为package main，tree目录为package tree，在main包中调用tree包的变量则使用如下语句——tree.Node{}
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/89d8504587d00f80000af5e113a13603_1740930880989.png)

---

## 扩展已有类型
**go语言没有继承
如何扩充系统类型或别人的类型？**
- 定义别名：最简单
- 使用组合：最常用
- 使用内嵌：省下很多代码，但是比较难看懂

### 组合方式，为原类型添加后序遍历

```go

type myTreeNode struct {
	node *tree.Node
}

func (myNode *myTreeNode) postOrder() {
	if myNode==nil||myNode.node==nil{
		return
	}
	left:=myTreeNode{myNode.node.Left}
	left.postOrder()
	right:=myTreeNode{myNode.node.Right}
	right.postOrder()
	myNode.node.Print()
}
```
### 别名方式，将[]int封装为队列

```go

type Queue []int

func (q *Queue) Push(v int) {
	fmt.Println(&q)
	*q = append(*q, v)
}

func (q *Queue) Pop() int {
	head := (*q)[0]
	*q = (*q)[1:]
	return head
}

func (q *Queue) IsEmpty() bool {
	return len(*q)==0
}

```

---

### 使用内嵌方式来扩展已有类型

**组合方式中的myTreeNode中的node删掉，tree.Node的所有成员变量和方法都可以给myTreeNode使用**

首先可以通过myTreeNode.Node调用，更进一步可以直接通过myTreeNode.直接调用

```go
type myTreeNode struct {
	*tree.Node //Embedding
}
```
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/9aa47985a54bb0c2283582e2176cfe80_1740930880989.png)



**方法重载**

如果myTreeNode中定义有与tree.Node相同名称的方法则可以使用myTreeNode.调用myTreeNode中的方法，或者使用myTreeNode.Node.调用Node中的方法
```go
func (myNode *myTreeNode)Traverse()  {
	fmt.Println("this method is shadowed.")
}
func main() {
	root := myTreeNode{&tree.Node{Value: 3}}
	root.Traverse() //重载函数
	root.Node.Traverse() //重载前的函数
}
```

---
 
 与其他语言不同，以下代码把子类赋值给基类是错误的，这只是组合的一个语法糖，对编译器而言，以下两种类型并没有联系
```go
var baseRoot *tree.Node
baseRoot := &root
```
 如果想把子类赋值给基类，Go语言是通过接口实现的，而不是继承

