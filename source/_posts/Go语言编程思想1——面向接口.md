---
title: Go语言编程思想1——面向接口
toc: true
date: 2022-01-13 23:34:34
tags: golang 开发语言 后端
categories: Golang
---

​点击阅读更多查看文章内容<!--more-->

# Go语言编程思想1——面向接口

**示例**

以下示例中包含有两个Retriever分别是实际使用的infra.Retriever以及测试用的testing.Retriever，再两者互换时，传统的方法需要修改多处变量，较为繁琐。

分析本例可以发现Retriever只是一个具有Get方法的变量，即getRetriever只需要返回一个可以调用Get方法的变量即可

这里的retriever即为一个具有Get方法的接口，GetRetriever返回接口，这样在修改变量时只需要修改getRetriever中的返回值即可，而不需每个变量都做修改。
```go
func getRetriever() retriever {
	return testing.Retriever{}
}
type retriever interface {
	Get(string) string
}
func main() {
	var r retriever = getRetriever()
	fmt.Println(r.Get("https://www.imooc.com"))
}
```

# 一、接口的概念


## duck typing
 
>传统的鸭子指的是真正的有生命的鸭子，必须属于脊索动物门，脊椎动物亚门...
>duck typing则认为下图中的大黄鸭也是鸭子，只要像鸭子就认为是鸭子
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/d477e435af8d3bab62c5037d7f5be942_1740930859166.png%20=100x)

描述事物的外部行为而非内部结构

严格说go属于结构化类型系统，类似duck typing

---


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/0816f18e294e6f4e686a3ea0193d9ac1_1740930866296.png)

在download前写一个注释说明必须实现了get方法才能作为函数的参数，如果参数没有实现get方法则会报错

python中的duck typing非常灵活，不管retriever是什么，只要实现了get方法，就可以传入download使用

---
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/479460c592f54f6abfb71c228519f3a3_1740930866296.png)

灵活性与python类似

---

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/29a763abe90e965d5b0b92eb96071af3_1740930866296.png)

java中没有duck typing，必须实现Retriever接口，只有get方法也无法调用
（与前面说的鸭子必须是指有生命的类似）

缺点：无法实现多个接口 

---

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/071fda669ff0fc54ba5c04e8f3c01ffc_1740930866296.png)
同时需要两个接口、具有灵活性、具有类型检查（无需注释说明）

---

# 二、接口定义
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/d1e0d4868240c689a9603ed045a36f4f_1740930866296.png)

## 定义接口
**接口由使用者定义**

download是使用者，使用者要get，因此我们要在Retriever中定义get方法

接口中定义方法不需要加func关键字，它里面本身就全是函数
```go
type Retriever interface {
	Get(url string)string 
}

func download(r Retriever)string  {
	return r.Get("www.imooc.com")
}
```

## 实现接口
接口的实现是隐式的
 
只要实现了Retriever接口中的方法就认为实现了Retriever接口
```go
type Retriever struct {
	Contents string
}

func (r Retriever) Get(url string) string {
	return r.Contents
}
```

---
# 三、接口的值类型

接口变量中包含有实现者的类型和实现者的值![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/7cdc0ec28e81790e0523716fc535f29f_1740930874000.png)
- 接口变量自带指针
- 接口变量同样采用值传递，几乎不需要使用接口的指针，因为它可以包含一个指针
- 指针接受者只能用指针方式使用，值接受者两者都可以

**查看接口变量**
- 表示任何类型：interface{}
Queue可以传入任何类型
```go
//将类型设为interface后，队列可以存放任何类型
type Queue []interface{}

func (q *Queue) Push(v interface{}) {
	fmt.Println(&q)
	*q = append(*q, v)
}

func (q *Queue) Pop() interface{} {
	head := (*q)[0]
	*q = (*q)[1:]
	return head
}
```
如要将Push()和Pop()的值都限定为int
Push的输入限定为int
Pop的输出为int，返回时通过.()将interface{}转换为int

```go
//将类型设为interface后，队列可以存放任何类型
type Queue []interface{}

func (q *Queue) Push(int) {
	fmt.Println(&q)
	*q = append(*q, v)
}

func (q *Queue) Pop() int {
	head := (*q)[0]
	*q = (*q)[1:]
	return head.(int)
}
```

- Type Assertion

```go
	r = &real.Retriever{
		UserAgent: "Mozilla/5.0",
		TimeOut:   time.Minute,
	}

// Type assertion，这里r是&real.Retriever，如下语句ok值为false
//通过.(具体的名字)来取得interface中的类型
	if mockRetriever, ok := r.(mock.Retriever); ok {
		fmt.Println(mockRetriever.Contents) 
	} else {
		fmt.Println("not a mock retriever")
	}

```

- Type Switch

```go
func inspect(r Retriever) {
	fmt.Printf("%T %v\n", r, r)
	switch v := r.(type) {
	case mock.Retriever:
		fmt.Println("Contents:", v.Contents)
	case *real.Retriever:
		fmt.Println("UserAgent:", v.UserAgent)
	}
}
```
# 四、接口的组合

```go

type Retriever interface {
	Get(url string) string
}

type Poster interface {
	Post(url string,form map[string]string)string
}

type RetrieverPoster interface {
	Retriever
	Poster
}

func session(s RetrieverPoster) string {
	s.Post(url,map[string]string{
		"contents":"another faked imooc.com",
	})
	return s.Get(url)
}

```

实现接口只需要把所有方法都定义即可
```go
type Retriever struct {
	Contents string
}

func (r *Retriever) Post(url string, form map[string]string) string {
	r.Contents=form["contents"]
	return "ok"
}

func (r *Retriever) Get(url string) string {
	return r.Contents
}
```
# 五、常用系统接口
- Stringer
**任何类型只要定义了String()方法，进行Print输出时，就可以得到定制输出。**
```go
type Stringer interface {
    String() string
}
```
```go
func (r *Retriever) String() string {
	return fmt.Sprintf(
		"Retriever:{Contents=%s}",r.Contents)
}
```

```go
func main() {
	mk := &mock.Retriever{Contents: "Hello World"}
	fmt.Println(mk)//Retriever:{Contents=Hello World}
}
```

- Reader

- Writer
