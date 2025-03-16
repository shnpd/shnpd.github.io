---
title: 【Golang】Golang格式化输出
toc: true
date: 2022-11-16 10:09:50
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

# fmt
Go语言用于控制文本输出常用的标准库是**fmt**

**fmt**中主要用于输出的函数有:

- Print: 输出到控制台，不接受任何格式化操作
- Println: 输出到控制台并换行
- Printf: 格式化输出，只可以打印出格式化的字符串，只可以直接输出字符串类型的变量（不可以直接输出别的类型）
- Sprintf: 格式化并返回一个字符串而不带任何输出
- Fprintf: 来格式化并输出到io.Writers而不是os.Stdout

--- 
# 格式化
通过Printf函数来测试下Go语言里面的字符串格式化:
```go
fmt.Sprintf(格式化样式, 参数列表…)
```
- 格式样式: 字符串形式，格式化符号以%开头，%s字符串格式，%d十进制的整数格式
- 参数列表: 多个参数以逗号分隔，个数必须与格式化样式中的个数一一对应，否则运行时会报错

比如:

```go
username := "boy"
fmt.Printf("welcome, %s", username)
```

## 整数格式化
|占 位 符| 描 述|
|--|--|
| %b | 整数以二进制方式显示 |
| %o | 整数以八进制方式显示 |
| %d | 整数以十进制方式显示 |
| %x | 整数以十六进制方式显示 |
| %X | 整数以十六进制、字母大写方式显示 |
| %c | 相应Unicode码点所表示的字符 |
| %U | Unicode字符，Unicode格式：123，等同于“U+007B” |

```go
func main() {
	fmt.Printf("%b \n", 123) //1111011
	fmt.Printf("%o \n", 123) //173
	fmt.Printf("%d \n", 123) //123
	fmt.Printf("%x \n", 123) //7b
	fmt.Printf("%X \n", 123) //7B   
	fmt.Printf("%c \n", 123) //{
	fmt.Printf("%U \n", 123) //U+007B 
}
```
## 浮点数格式化
|占 位 符| 描 述 |
|--|--|
| %e |科学计数法，例如 1.234560e+02 |
| %E |科学计数法，例如 1.234560E+02|
| %f |有小数点而无指数，例如 123.456  |
| %F |等价于%f |
| %g | 根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的0）输出 |
| %G | 根据情况选择 %E 或 %F 以产生更紧凑的（无末尾的0）输出  |

```go
func main() {
	fmt.Printf("%e \n", 123.456) //1.234560e+02
	fmt.Printf("%E \n", 123.456) //1.234560E+02
	fmt.Printf("%f \n", 123.456) //123.456000
	fmt.Printf("%F \n", 123.456) //123.456000
	fmt.Printf("%g \n", 123.456) //123.456
	fmt.Printf("%G \n", 123.456) //123.456 
}
```

## 布尔类型格式化
|占 位 符| 描 述 |
|--|--|
| %t | true 或 false |

```go
func main() {
	fmt.Printf("%t", true) //true
}
```
## 字符格式化
|占 位 符| 描 述 |
|--|--|
| %c | 相应Unicode码点所表示的字符 |

```go
func main() {
	fmt.Printf("%c", 0x4E2D) //中
}
```

## 字符串格式化
|占 位 符| 描 述 |
|--|--|
| %s | 直接输出字符串或者[]byte |
| %q | 双引号围绕的字符串，由Go语法安全地转义 |
| %x | 每个字节用两字符十六进制数表示（使用a-f）|
| %X | 每个字节用两字符十六进制数表示（使用A-F） |

```go
func main() {
	fmt.Printf("%s \n", "Hello world") //Hello world
	fmt.Printf("%q \n", "Hello world") //"Hello world"
	fmt.Printf("%x \n", "Hello world") //48656c6c6f20776f726c64
	fmt.Printf("%X \n", "Hello world") //48656C6C6F20776F726C64
}
```

## 指针格式化
|占 位 符| 描 述 |
|--|--|
| %p | 表示为十六进制，并加上前导的0x |
| %#p | 表示为十六进制，没有前导的0x |

```go
func main() {
	a := "Hello world"
	b := &a
	fmt.Printf("%p \n", b)  //0xc000046230
	fmt.Printf("%#p \n", b) //c000046230
}
```

## 通用占位符
|占 位 符| 描 述 |
|--|--|
| %v | 值的默认格式 |
| %+v | 类似%v，但输出结构体时会添加字段名 |
| %#v | 相应值的Go语法表示 |
| %T | 相应值的类型的Go语法表示 |
| %% | 百分号,字面上的%,非占位符含义 |

```go
func main() {
	fmt.Printf("%v \n", "Hello World")   //Hello World
	fmt.Printf("%+v \n", "Hello World")  //Hello World
	fmt.Printf("%#v \n", "Hello World")  //"Hello World"
	fmt.Printf("%T \n", "Hello World")   //string
	fmt.Printf("%%%v \n", "Hello World") //%Hello World
}
```

## 宽度表示
**浮点数精度控制**
宽度通过一个紧跟在百分号后面的十进制数指定，如果未指定宽度，则表示值时除必需之外不作填充。
精度通过（可选的）宽度后跟点号后跟的十进制数指定。如果未指定精度，会使用默认精度；如果点号后没有跟数字，表示精度为0。举例如下
```go
func main() {
	fmt.Printf("|%f|\n", 123.456)     //|123.456000|
	fmt.Printf("|%12f|\n", 123.456)   //|  123.456000|
	fmt.Printf("|%.3f|\n", 123.456)   //|123.456|
	fmt.Printf("|%12.3f|\n", 123.456) //|     123.456|
	fmt.Printf("|%12.f|\n", 123.456)  //|         123|
}
```

**字符串长度控制**

宽度设置格式：占位符中间加一个数字, 数字分正负, +: 右对齐, -: 左对齐
最小宽度：百分号后紧跟十进制数，不够部分可以选择补0
最大宽度：小数点后的十进制数，超出的部分会被截断
```go
func main() {
	fmt.Printf("|%s|\n", "123.456")    //|123.456|
	fmt.Printf("|%12s|\n", "123.456")  //|     123.456|
	fmt.Printf("|%-12s|\n", "123.456") //|123.456     |
	fmt.Printf("|%012s|\n", "123.456") //|00000123.456|
	fmt.Printf("|%.5s|\n", "123.456")  //|123.4|
}
```
