---
title: Go语言基础知识2——内建容器
toc: true
date: 2022-01-11 16:12:56
tags: golang 容器 开发语言
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->


# 一、数组

**数组定义**

var arr [5]int：变量个数在前，变量类型在后
```go
	var arr1 [5]int
	arr2 := [3]int{1, 3, 5}          //需赋初值
	arr3 := [...]int{2, 4, 6, 8, 10} //数量由初始值决定
	var grid [4][5]int               //二维数组，4行5列
```

---

**range关键字**

```go
	//通过range关键字可以同时获得索引及其对应元素
	//for _,v:= range arr3 通过_省略变量
	//如果只要i，可写成for i:=range arr3
	for i,v :=range arr3{
		fmt.Println(i,v)
	}
```

---

**数组是值传递**

[10]int 和 [20]int 是不同类型

调用func f(arr [10]int)会**拷贝数组** (其他大部分语言传数组都是引用传递)，

通过**指针**可以修改数组的值
```go
//通过指针可以修改数组的值
func printArray(arr *[5]int){
	for i,v :=range arr{
		fmt.Println(i,v)
	}
	arr[0]=100 //不需要加*
}

//调用函数的时候要用&arr1，不像C语言数组名就是首地址
printArray(&arr1)
```

<font color="red">**Go语言中一般不直接使用数组**</font>

---


# 二、Slice（切片）

Slice区间**前闭后开**
 
arr[2:6]、arr[:6]、arr[2:]、arr[:]都是切片
```go
func main() {
	arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7}
	fmt.Println("arr[2:6] =", arr[2:6]) //2,3,4,5
	fmt.Println("arr[:6] =", arr[:6])   //0,1,2,3,4,5
	fmt.Println("arr[2:] =", arr[2:])   //2,3,4,5,6,7
	fmt.Println("arr[:] =", arr[:])     //0,1,2,3,4,5,6,7
}

```
 
Slice本身没有数据，是对底层array的一个view
```go
arr := [...]int{0,1,2,3,4,5,6,7}
s := arr[2:6]
s[0] = 10
// array变为[0,1,10,3,4,5,6,7]
```

**Reslice**
对slice再求slice
```go
	arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7}
	s1 := arr[2:5]
	fmt.Println(s1) //2,3,4
	s1 = s1[1:3]
	fmt.Println(s1) //3,4
```
## Slice的扩展
在以下示例中，s1为[2,3,4,5]，如果直接取s1[4]会产生越界错误

使用s2再取s1的切片s1[3:5]，则**不会报错**
```go
	arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7}
	s1 := arr[2:6] //2,3,4,5
	s2 := s1[3:5]  //5,6
```
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/49198e6c97126c54d0674aa81bf1ab08_1740930916135.png)

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/3b5e2aed8a37590f768d3a02c72ef7e1_1740930923045.png)
<font color="red">**slice可以向后扩展，不可以向前扩展
s[i]不能超过len(s)，向后扩展可以超过len(s)不可以超过cap(s)**</font>

```go
func main() {
	arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7}
	s1 := arr[2:6]                //2,3,4,5
	fmt.Println(len(s1), cap(s1)) //4,6
	s2 := s1[3:5]                 //5,6
	fmt.Println(len(s2), cap(s2)) //2,3
}
```

---

## Slice的操作

**向Slice添加元素**

```go
func main() {
	arr := [...]int{0, 1, 2, 3, 4, 5, 6, 7}
	s1 := arr[2:6]       //2,3,4,5
	s2 := s1[3:5]        //5,6
	s3 := append(s2, 10) //5,6,10
	s4 := append(s3, 11) //5,6,10,11
	s5 := append(s4, 12) //5,6,10,11,12
	//arr数组为[0,1,2,3,4,5,6,10]
}
```

切片在添加元素时如果超越cap，那么就不再是对原数组的view，系统会重新分配更大的底层数组
由于值传递的关系，在append时slice的len和cap都有可能改变，必须接受append的返回值 s=append(s,val)

---

**创建Slice**
```go
	var s []int //Zero value for slice is nil; len = 0; cap = 0

	s1 := []int{2, 4, 6, 8, 10}

	s2 := make([]int, 16)
	s3 := make([]int, 10, 32)//type,len,cap
```

--- 

**copy**
```go
	//只copy数值
	arr := []int{0, 1, 2, 3, 4, 5, 6, 7}
	s1 := arr[2:6]        //[2 3 4 5]
	s2 := make([]int, 10) //[0 0 0 0 0 0 0 0 0 0]
	copy(s2, s1)
	fmt.Println(s1, s2) //[2 3 4 5] [2 3 4 5 0 0 0 0 0 0]
```

---

**delete**

删除元素没有内建函数

```go
	//删除下标为3的元素
	s2=append(s2[:3],s2[4:]...)//使用...将Slice解开逐个append 
	//删除头
	s2=s2[1:]
	//删除尾
	s2=s2[:len(s2) - 1]
```

---

# 三、Map
map定义：map[Key]Value{}，方括号中是key的类型，后面跟value的类型，大括号中加初始化的值

复合map：map[K1]map[K2]V，外层map的key为K1，value为map[K2]V

**创建**
```go
	m := map[string]string{
		"name":    "ccmouse",
		"course":  "golang",
		"site":    "imooc",
		"quality": "notbad",
	}

	m2 := make(map[string]int) //m2 == empty map

	var m3 map[string]int //m3 == nil
```

---

**遍历**

使用range遍历
使用len获得元素个数
```go
	//遍历，Map使用哈希表元素无序，每次输出的顺序都不同
	//如需顺序，需要手动对key排序，全取出来加到slice中
	for k, v := range m {
		fmt.Println(k, v)
	}
```

---

**取元素，判断是否存在**
```go
	//取元素,判断元素是否存在，若存在ok为true，否则为false
	//key不存在时获得元素类型的初始值
	courseName, ok := m["course"]
	fmt.Println(courseName, ok) //golang true
	causeName, ok := m["cause"]
	fmt.Println(causeName, ok) // false,键不存在输出为空
```

---

**删除元素**
```go
	delete(m,"name")
```
---

**map中的key**
map使用哈希表，key必须可以比较相等
除了slice，map，function的内建类型都可以作为key
struct类型不包含上述字段，也可作为key

---

# 四、字符串
## 字符串的实现
**range []byte(s)：使用[]byte获得字节(底层），字符串采用utf-8编码，英文一字节，中文三字节，直接输出s[idx]获得的是字符串对应位置的字节**

**range s：获得的是字符的Unicode编码**

字符串的不同输出形式
|字符串|Y|e|s|我|爱|慕|课|网|!|
|--|--|--|--|--|--|--|--|--|--|
|UTF-8（range []byte(s)）|59|65|73|E6 88 91|E7 88 B1|E6 85 95|E8 AF BE|E7 BD 91|21|
|Unicode（range s）|0,59|1,65|2,73|3,6211|6,7231|9,6155|12,8BFE|15,7F51|18,21|
|range []rune(s)|0,59|1,65|2,73|3,6211|4,7231|5,6155|6,8BFE|7,7F51|8,21|

Ps，后两行为：下标,字符；
即range s中每个中文字符占三个长度，因为字符串底层是由字节存储的，每个中文字符占三个字节，所以占三个长度
[]rune(s)中每个中文字符占一个长度，一个rune类型占四个字节，所以一个rune可以存储一个字符，
```go
	s := "Yes我爱慕课网!"
	for i, b := range []byte(s) {
		fmt.Printf("(%d %X)", i, b)
	}
	fmt.Println()
	//结果
	//(0 59)(1 65)(2 73)(3 E6)(4 88)(5 91)(6 E7)(7 88)(8 B1)(9 E6)(10 85)(11 95)(12 E8)(13 AF)(14 BE)(15 E7)(16 BD)(17 91)(18 21)
	for i, ch := range s { //ch is a rune
		fmt.Printf("(%d %X)", i, ch)
	}
	fmt.Println()
	//结果 utf8转为Unicode
	//(0 59)(1 65)(2 73)(3 6211)(6 7231)(9 6155)(12 8BFE)(15 7F51)(18 21)
```

---

len：返回字节长度
utf8.RuneCountInString：返回字符个数
```go
	fmt.Println(len(s))//19
	fmt.Println(utf8.RuneCountInString(s))//9
```

---

**使用utf8.DecodeRune解码**

func DecodeRune(p []byte) (r rune, size int) {}：
输入为UTF-8编码的字节，输出为rune以及字符的所占的字节数，使用%c获得字符，汉字每三个字节进行解码

```go
	bytes:=[]byte(s)
	for len(bytes)>0{
		ch,size:=utf8.DecodeRune(bytes)
		fmt.Printf("%c ",ch)
		bytes=bytes[size:]
	}
	//结果
	//Y e s 我 爱 慕 课 网 ! 
```

---

<font color="red">**顶层使用：将s转为[]rune下标从0递增**</font>
[]rune转换并不是对内存的重新理解，而是会将字节进行解码，将转换后的字符存放在一个新开的rune slice里面
```go
	for i,ch:=range []rune(s){  //ch is a rune
		fmt.Printf("(%d %c)",i,ch)
	}
	//结果
	//(0 Y)(1 e)(2 s)(3 我)(4 爱)(5 慕)(6 课)(7 网)(8 !)
```

---

## 字符串的有关操作
>字符串的有关操作都在**strings**包中
```go
	//字符串分隔、拼接
	fmt.Println(strings.Fields("a    b c d"))//[a b c d] 空格分割
	fmt.Println(strings.Split("abcbde","b"))//[a c de]
	fmt.Println(strings.Join([]string{"aa","bb","cc"},"A"))//aaAbbAcc

	//子串查找
	fmt.Println(strings.Index("abcec","c"))//2
	fmt.Println(strings.Contains("abcec","c"))//true

	//大小写转换
	fmt.Println(strings.ToLower("AAA"))//aaa
	fmt.Println(strings.ToUpper("aaAb"))//AAAB

	//切割首尾字符
	fmt.Println(strings.Trim("abcdaaa","abc"))//d
	fmt.Println(strings.TrimLeft("abcdaaa","abc"))//daaa
	fmt.Println(strings.TrimRight("abcdaaa","abc"))//abcd
```
