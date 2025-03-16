---
title: 【Go】反射reflect
toc: true
date: 2025-03-03 00:00:00
tags: go
categories: 
	- Golang

---

点击阅读更多查看文章内容<!--more-->

## 1 反射简介

反射（Reflection）指的是程序在运行时能够检查自身结构、动态获取类型信息以及修改变量和调用方法的能力。Go 语言通过内置的 [reflect](https://golang.org/pkg/reflect/) 包提供这一功能，使得我们能够在运行时对任意对象进行“自省”。这种能力常用于构建通用框架（如 ORM、依赖注入、序列化工具等），但同时也带来性能开销和代码复杂性的问题。

## 2 核心原理与重要组件

### 2.1 接口与反射对象

Go 中的任意值都可以看作由两部分组成：**类型信息**和**数据值**。当我们使用空接口 `interface{}` 存储一个具体变量时，实际上内部存储了该变量的类型信息和实际值。反射正是基于这一点，通过调用：

- **reflect.TypeOf(x)**：返回值的类型信息（ `reflect.Type` 接口）。
- **reflect.ValueOf(x)**：返回值的运行时表示（`reflect.Value` 结构体）。

```go
var x int = 42
fmt.Println("Type:", reflect.TypeOf(x)) // 输出：int
fmt.Println("Value:", reflect.ValueOf(x)) // 输出：42
```

### 2.2 反射中的核心类型和方法

- **reflect.Type**
  本身是一个接口，代表一个类型，通过其方法可以获取类型名称、种类（Kind）等信息。需要注意的是，自定义类型与其底层类型在 `Name()` 与 `Kind()` 上可能不同，例如自定义类型 `type MyInt int`，其 `Name()` 返回 "MyInt"，而 `Kind()` 返回底层的 `int`。

- **reflect.Value**
  本身是一个结构体，表示一个运行时值，除了可以获取内部数据外，还提供了设置值的方法（前提是该值必须“可设置”，即 settable）。例如：

  ```go
  var a int64 = 100
  v := reflect.ValueOf(&a).Elem() // 通过 Elem() 获取指针指向的值
  v.SetInt(200)                  // 将 a 的值更新为 200
  ```

这两者构成了反射操作的基础，允许我们动态查询变量的类型和数据

## 3 反射三定律

在 Go 官方博客文章 [laws-of-reflection](https://go.dev/blog/laws-of-reflection) 中，叙述了反射的 3 定律：

**第一定律：反射是从接口值到反射对象**

在一般情况下，反射是一种检查存储在**接口变量**中的类型和值的机制。

这其实从 reflect 包中的 TypeOf 和 ValueOf 函数就可以知道。在本文第二节中有讲，这 2 个函数的接收参数就是 **interface{}**。

比如 `reflect.TypeOf(6.4)`，调用 reflect.TypeOf(x) 时（这里的 x 表示 6.4），x 首先存储在一个空接口 interface{} 中，作为参数传递，reflect.TypeOf 对该接口进行类型信息解码，获取类型详细信息。

**第二定律：从反射对象可以获取接口值**

通过 `Value.Interface()` 可以将 `reflect.Value` 还原为原始接口值。

```go
	v := reflect.ValueOf(6.4)
	y := v.Interface()
	fmt.Println(reflect.TypeOf(y))
```

知道数据类型直接获取值方法，不知道数据类型用 Interface() 获取数据然后断言值

**第三定律：要修改反射对象的值，其值必须可以设置**

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic.
```

此处设置v的值会报错，Value 的 `CanSet` 方法可以获取值是否可设置，如：

```go
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet())
// settability of v: false
```

为什么有可设置性？

> 因为 reflect.ValueOf(x) 这个 x 传递的是一个原数据的副本，上面代码 `v.SetFloat(7.1)` 如果设置成功，那么更新的是副本值，原始值 x 并没有更新。这就会造成原值和新值的混乱，可设置属性就是避免这个问题。

传递的是一个副本，而不是值本身。如果希望能直接修改 x，那么必须把 x 的地址传递给函数，即指向 x 的指针：

```go
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: take the address of x.
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet())
// type of p: *float64
// settability of p: false
```

还是 `false`，为什么？

反射对象 p 不可设置，它并不是我们要设置的 p，它实际上是 *p。为了得到 p 所指向的东西，我们需要调用 Value 的 Elem 方法，通过指针进行简介寻址，然后将结果保存在一个名为 v 的反射 Value 中：

```go
v := p.Elem()
fmt.Println("settability of v:", v.CanSet())
// settability of v: true
```

然后我们可以用 `v.SetFloat()` 设置值：

```go
v.SetFloat(7.1)
fmt.Println(v.Interface())
fmt.Println(x)
// 7.1
// 7.1
```

## 4 常用反射操作

### 4.1 获取类型与数值

利用 `reflect.TypeOf(x)` 可以获取变量的类型信息，而 `reflect.ValueOf(x)` 则用于获取变量值。例如：

```go
辑var s string = "hello"
t := reflect.TypeOf(s)
v := reflect.ValueOf(s)
fmt.Printf("Type: %v, Kind: %v\n", t.Name(), t.Kind()) // 输出 "string", "string"
fmt.Println("Value:", v.String())                       // 输出 "hello"
```

### 4.2 结构体字段与标签

反射可以用来遍历结构体字段，获取字段名称、类型以及 struct tag。例如：

```go
type Student struct {
    Name  string `json:"name"`
    Score int    `json:"score"`
}

stu := Student{"小王子", 90}
t := reflect.TypeOf(stu)
for i := 0; i < t.NumField(); i++ {
    field := t.Field(i)
    fmt.Printf("字段名：%s, 类型：%v, Tag：%v\n", field.Name, field.Type, field.Tag.Get("json"))
}
```

这种方式常用于 ORM 框架中自动解析结构体字段

### 4.3 动态方法调用

通过反射，我们可以动态地调用对象的方法。首先通过 `MethodByName` 获取方法对应的反射对象，再用 `Call` 方法调用它：

```go
type Person struct {
    Name string
}
func (p Person) SayHello() {
    fmt.Println("Hello, my name is", p.Name)
}
p := Person{"Alice"}
v := reflect.ValueOf(p)
method := v.MethodByName("SayHello")
method.Call(nil) // 输出 "Hello, my name is Alice"
```

这种动态调用常用于插件化架构或 RPC 框架中

### 4.4 设置变量值

如需修改变量值，必须传入变量地址并使用 `Elem()` 获取可设置的值：

```go
var num int64 = 100
v := reflect.ValueOf(&num).Elem()
if v.CanSet() {
    v.SetInt(200)
}
fmt.Println(num) // 输出 200
```

### 4.5 比较slice、map等是否相等

在比较数组是否相等时可以直接使用 == ，因为数组的大小和元素类型是固定的，`==` 比较的是数组的每个元素是否相等。

但是，切片（`slice`）不支持使用 `==` 直接比较，因为切片是动态的，底层数组可能共享或者切片的容量和长度不同，所以 Go 语言并不允许直接比较切片（同理map也无法比较）。但是可以使用反射 `reflect.DeepEqual` 获取运行时的值来进行比较。

```go
	var a []int = []int{1, 2, 3}
	var b []int = []int{1, 2, 3}
	if reflect.DeepEqual(a, b) {
		fmt.Println(true)
	} else {
		fmt.Println(false)
	}
```

## 5 反射的优缺点与使用场景

### 优点

- **灵活性与动态性**
  允许在运行时对未知类型进行操作，便于编写通用库或框架，如 ORM、依赖注入和序列化工具​
- **解耦设计**
  通过反射可以减少硬编码，使代码更加模块化和可配置。

### 缺点

- **性能开销**
  反射操作比直接调用要慢一个甚至两个数量级，不适用于性能敏感的场景。
- **代码可读性和维护性下降**
  反射代码通常较难理解和调试，容易引入运行时错误。

### 适用场景

- **框架开发**：如 web 框架、ORM、序列化工具等需要处理多种类型时。

- **依赖注入和插件架构**：运行时动态加载和调用模块功能。

- **通用工具库**：如 JSON 编解码、日志系统等需要处理结构体标签和字段信息的场景

  