---
title: Shell脚本编写教程【二】——Shell 数组
toc: true
date: 2023-07-07 09:56:33
tags: linux bash
categories: Shell脚本
---

​点击阅读更多查看文章内容<!--more-->

# Shell脚本编写教程【二】——Shell 数组
---
**目录**：[https://blog.csdn.net/shn111/article/details/131590488](https://blog.csdn.net/shn111/article/details/131590488)

**参考教程**：[https://www.runoob.com/linux/linux-shell.html](https://www.runoob.com/linux/linux-shell.html)

**在线编辑器**：[https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash](https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash)

---

>bash支持一维数组（不支持多维数组），并且没有限定数组的大小
>类似于 C 语言，数组元素的下标由 0 开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于 0。
---

# 创建数组
>Shell 数组用括号来表示，元素用"空格"符号分割开，语法格式如下：
array_name=(value1 value2 ... valuen)

```bash
my_array=(A B "C" D)
```
也可以使用数字下标来定义数组

```bash
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
```
---

# 读取数组
>读取数组元素值的一般格式是：
>`${array_name[index]}`

```bash
my_array=(A B "C" D)

echo "第一个元素为: ${my_array[0]}"
echo "第二个元素为: ${my_array[1]}"
echo "第三个元素为: ${my_array[2]}"
echo "第四个元素为: ${my_array[3]}"
```
---

# 关联数组（字典）
>关联数组可以使用任意的字符串或者整数作为下标来访问数组元素

关联数组使用`declare`命令来声明，语法格式如下：
`declare -A array_name`

-A：用于声明一个关联数组

关联数组的键是唯一的

实例：

```bash
declare -A site=(["google"]="www.google.com" ["runoob"]="www.runoob.com" ["taobao"]="www.taobao.com")
```
也可以先声明一个关联数组（自己试的不声明也可以（发现不声明的话还是有问题，在后面取数组所有元素时，如果没有声明那么只会取到最后一个元素）），再设置键和值：

```bash
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"
```

访问关联数组元素时使用指定的键，格式如下：
`array_name["index"]`

```bash
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"

echo ${site["runoob"]}
# www.runoob.com
```
---

# 获取数组中的所有元素
使用`@`或`*`可以获取数组中的所有元素

实例：

```bash
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"
# 数组的元素为: A B C D
# 数组的元素为: A B C D
```

关联数组：
```bash
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"

echo "数组的元素为: ${site[*]}"
echo "数组的元素为: ${site[@]}"
# 数组的元素为: www.google.com www.taobao.com www.runoob.com
# 数组的元素为: www.google.com www.taobao.com www.runoob.com
```
在数组前面加一个感叹号`!`可以获取数组的所有键

```bash
declare -A site
site["google"]="www.google.com"
site["runoob"]="www.runoob.com"
site["taobao"]="www.taobao.com"

echo "数组的键为: ${!site[*]}"
echo "数组的键为: ${!site[@]}"
# 数组的键为: google taobao runoob
# 数组的键为: google taobao runoob
```
---
# 获取数组的长度
>获取数组长度的方法与获取字符串长度的方法相同，在数组名前加`#`获取

#my_array[*]：获取数组元素个数
#my_array[@]：获取数组元素个数
#my_array[i]：获取数组中第i个元素的长度

```bash
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"
echo "数组第0个元素的长度为: ${#my_array[0]}"

# 数组元素个数为: 4
# 数组元素个数为: 4
# 数组第0个元素的长度为: 1
```
