---
title: 【Go】make和new的区别
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang



---

点击阅读更多查看文章内容<!--more-->

# make和new的区别

[深入理解make和new，从底层到应用！](https://mp.weixin.qq.com/s?__biz=Mzk0OTMyMjkxMQ==&mid=2247485650&idx=1&sn=954b05b7d512a1619e25385234b5c332&source=41#wechat_redirect)

**new**

```go
package main

import "fmt"

//定义一个结构体，age字段为指针
type Student struct {
    age *int
}

//获取结构体对象指针
func getStudent() *Student {
    s := new(Student)
    return s
}

func main() {
    s := getStudent()
    *(s.age) = 10
    fmt.Println(s.age)
}
```

执行以上代码会发生panic

panic: runtime error: invalid memory address or nil pointer dereference
[signal 0xc0000005 code=0x1 addr=0x0 pc=0x34bffa]



new的函数声明

```go
func new(Type) *Type
```

Type是指变量的类型，`new`会根据变量类型返回一个指向该类型的指针。

`new`底层调用的是runtime.newobject申请内存空间

```go
func newobject(typ *_type) unsafe.Pointer {
    return mallocgc(typ.size, typ, true)
}
```

newobject的底层调用mallocgc在堆上按照typ.size的大小申请内存，因此`new`只会为结构体Student申请一片内存空间，不会为结构体中的指针age申请内存空间，所以 *(s.age) 的解引用操作就因为访问无效的内存空间而出现`panic`。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303123536964.png" alt="image-20241125162407521" style="zoom:67%;" />

对于结构体指针，工作中一般使用`s:=&Stuent{age:=new(int)}`的方式赋值，这样能够清晰的知道结构体中的每一个字段是什么，避免不必要的错误！

**make**

```go
package main

import "fmt"

func main() {
    nums := new([]int)
    (*nums)[0] = 1
    fmt.Println((*nums)[0])
}
```

以上程序在运行时也会出现panic，slice的底层实现为：

```go
type slice struct {
    array unsafe.Pointer    //指向用于存储切片数据的指针
    len   int
    cap   int
}
```

这就和上面的例子一样了，`new`只会为结构体slice申请内存，而不会为当中的array字段申请内存，因此用(*nums)[0]取值会发生`panic`。

如果需要对`slice`、`map`、`channel`进行内存申请，则必须使用`make`申请内存，下面看一下`make`函数声明

```go
func make(t Type, size ...IntegerType) Type
```

可以看到`make`返回的是复合类型本身，将错误代码修改如下

```
package main

import "fmt"

func main() {
    //为了让make在堆上申请内存，这里将容量写大一点
    nums := make([]int, 8192)
    nums[0] = 1
    fmt.Println(nums[0], nums[1])
}
```

分析make的实现

```go
func makeslice(et *_type, len, cap int) unsafe.Pointer {
    mem, overflow := math.MulUintptr(et.size, uintptr(cap))
    //做合法检查
    if overflow || mem > maxAlloc || len < 0 || len > cap {
        mem, overflow := math.MulUintptr(et.size, uintptr(len))
        if overflow || mem > maxAlloc || len < 0 {
            panicmakeslicelen()
        }
        panicmakeslicecap()
    }
      //做内存申请
    return mallocgc(mem, et, true)
}
```

可以看到makeslice申请内存底层调用的也是mallocgc，从这点看和`new`一样，但是细看`new`中mallocgc第一个参数（申请内存大小）用的是type.size，而`make`中的mallocgc第一个参数是mem，从MulUintptr源码中可以看出mem是`slice`的容量cap乘以type.size，因此使用makeslice可以成功的为切片申请内存。

`make`为`map`和`channel`申请内存底层分别是runtime.makemap_small，runtime.makechan，也是同样调用mallocgc，这里就不继续讨论了

更进一步举个例子，以下示例中，a的内存占用为8因为new([]int) 创建一个指向 []int 类型的指针，并返回该指针。指针的大小是 ​8 字节。 *a解指针得到的就是指针实际指向的切片包含三部分ptr：指向底层数组的指针（8 字节）。len：切片的长度（8 字节）。cap：切片的容量（8 字节）。一共24字节，b的内存占用就是24字节因为make([]int, 0)返回的就是切片本身
```go
func main() {
	a := new([]int)
	b := make([]int, 0)
	fmt.Println(unsafe.Sizeof(a))
	fmt.Println(unsafe.Sizeof(*a))
	fmt.Println(unsafe.Sizeof(b))
	//8
	//24
	//24
}
```


**make和new的区别**

相同点

- 都是Go语言中用于内存申请的关键字
- 底层都是通过mallocgc申请内存

不同点

- `make`返回点是复合结构体本身而`new`返回的是指向变量内存的指针
- `make`只能为`channel`，`slice`，`map`申请内存空间

