---
title: Go语言基础知识1——基础语法
toc: true
date: 2022-01-11 16:14:05
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

# 一、变量
## 变量定义
**1.使用var关键字定义**

>变量名在前，变量类型在后
```go
	var a,b,c bool

	var s1,s2 string="hello","world"

	//编译器可以自动决定类型
	var a, b, c, s = 3, 4, true, "def"
 
	//使用var()集中定义变量
	var (
	    aa = 3
 	    ss ="kkk"
 	    bb=true
    )
```

<font color="red">可以放在函数内，也可放在包内，Go语言没有全局变量</font>


**2.使用:=定义变量**

```go
	a, b, c, s := 3, 4, true, "def"
```
<font color="red">只能在函数内使用，在包内(函数外)不能使用 </font>

---

## 变量类型
- bool，string

- (u)int，(u)int8，(u)int16，(u)int32，(u)int64，uintptr
Go语言中没有long long类型，用int64可以代替；u代表无符号数

- byte，**rune**
byte占8位；rune是字符型相当于char类型，占32位

- float32，float64，**complex64，complex128**
complex表示复数，complex64，实部虚部都是float32；complex128，实部虚部都是float64



**强制类型转换**
<font color="red">类型转换是强制的，不存在隐式转换<font> 
```go
    var a,b int=3,4
    var c int = math.Sqrt(a*a+b*b) ×
    var c int =int(math.Sqrt(float64(a*a+b*b))) √
```

---
# 二、常量
## 常量定义
>使用**const**关键字
用法与var类似，也可使用const()集中定义常量，也可定义在包内

const不定义类型时，可作为各种类型使用，如下： 
**这里a，b不用强制转float**
```go
    const a, b = 3, 4
	var c int
	c = int(math.Sqrt(a*a + b*b))
```

## 枚举类型

>Go语言没有枚举关键字，通常通过const块来定义
```go
	const (
		cpp    = 0
		java   = 1
		python = 2
		golang = 3
	)
```

可以通过iota实现自增值
```go
	const (
		cpp        = iota //0
		python            //1
		golang            //2
		javascript        //3
	)
	
	const (
		b  = 1 << (10 * iota) //1
		kb                    //1024
		mb                    //1048576
		gb                    //1073741824
		tb                    //1099511627776
		pb                    //1125899906842624
	)
```

# 三、条件语句

**if**
if的条件不需要括号
if的条件里可以赋值；赋值的变量作用域就在这个if语句里（赋值语句后加分号，再加判断语句）
```go
    if contents, err := ioutil.ReadFile(filename); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%s\n", contents)
	}
```

**switch**

switch后可以没有表达式
case会自动break，除非使用fallthrough
case可以加多个条件
```go
func grade(score int) string {
	g := ""
	switch {
	case score < 0 || score > 100:
		panic(fmt.Sprintf("Wrong score: %d", score))
	case score < 60:
		g = "D"
	case score < 80:
		g = "C"
	case score < 90:
		g = "B"
	case score <= 100:
		g = "A"
	}
	return g
}
```
# 四、循环语句
**for**

for的条件里不需要括号
for的条件里可以省略初始条件，结束条件，递增表达式

整数转二进制，**省略初始条件**
```go
func convertToBin(n int) string {
	s := ""
	for ; n != 0; n /= 2 {
		t := n % 2
		s = strconv.Itoa(t) + s
	}
	return s
}
```

**省略初始条件和递增条件**，相当于**while**
```go
func main() {
	i := 1
	for i <= 100 {
		fmt.Println(i)
		i++
	}
}
```

**什么都不写**，相当于死循环
```go
func main() {
	for {
		fmt.Println("abc")
	}
}
```


# 五、函数
>**与变量定义类似，函数名在前，返回值类型在后**

**函数可以返回多个值**
函数返回多个值时可以**起名字**，比较适用于非常简单的函数，复杂函数体分不清楚返回值何时赋值。
多个返回值通常用在返回Error，即一个正确的返回值和一个错误
```go
func div(a, b int) (q, r int) {
	q = a / b
	r = a % b
	//return a/b,a%b
	return
}
```
有多个返回值的函数，只取其中一个返回值时，可以把不需要的返回值使用下划线填充

```go
q, _ := div(a, b)
```


**函数可以作为参数**
```go
func apply(op func(int, int) int, a, b int) int {
	p := reflect.ValueOf(op).Pointer()
	opName:=runtime.FuncForPC(p).Name()
	fmt.Printf("Calling function %s with args "+
		"(%d,%d)\n", opName, a, b)
	return op(a, b)
}
func pow(a,b int) int {
	return int(math.Pow(float64(a),float64(b)))
}
func main() {
	fmt.Println(apply(pow,3,4))
}
//输出：
//Calling function main.pow with args (3,4)
//81
//main是包名，pow是函数名
```
**匿名函数**
```go
func main() {
	fmt.Println(apply(
		func (a,b int)int{
			return int(math.Pow(float64(a),float64(b)))
		},3,4))
}
//输出:
//Calling function main.main.func1 with args (3,4)
//81
//第一个main是包名，第二个main是主函数名，func1匿名函数名
```

**Go语言没有默认参数、可选参数、函数重载、操作符重载等**
 
**可变参数列表**

传多少个参数都可以
```go
func sum(numbers ...int) int {
	s := 0
	for i := range numbers {
		s += numbers[i]
	}
	return s
}
func main() {
	fmt.Println(sum(1, 2, 3, 4, 5))
}
//15
```

# 六、指针
> \*int代表指针，与C++中的int \*相反

```go
func main() {
	var a int = 2
	var pa *int = &a
	*pa = 3
	fmt.Println(a)
}
//3
```

**Go语言中指针不能运算**

---

**参数传递**

>Go语言只有**值传递**一种方式

- **值传递**
拷贝一份变量传入到函数中
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/cd67257e5137ed2554d7969464259dee_1740930916135.png)


- **指针传递**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/c705bd57c7c0520e38f0ee91bed0350b_1740930916135.png) 
- **Object传递**，具体传递方式根据封装类型选择
cache中不包含data，而是有一个指向data的指针，将cache拷贝一份传递到函数中，cache中包含有指向data的指针
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/3f707e322b89214f12be1d067c94febb_1740930916135.png)

 
**swap函数的两种实现**
```go
func swap(a, b *int) {
	*a, *b = *b, *a
}

func swap(a, b int) (int, int) {
	return b, a
}
```




