---
title: Shell脚本编写教程【一】——Shell 变量
toc: true
date: 2023-07-07 09:56:38
tags: linux bash
categories: Shell脚本
---

​​点击阅读更多查看文章内容<!--more-->

# Shell脚本编写教程【一】——Shell 变量
---
**目录**：[https://blog.csdn.net/shn111/article/details/131590488](https://blog.csdn.net/shn111/article/details/131590488)

**参考教程**：[https://www.runoob.com/linux/linux-shell.html](https://www.runoob.com/linux/linux-shell.html)

**在线编辑器**：[https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash](https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash)

---
# Shell变量
## 变量定义

```bash
name="shn"
```
**注意：变量名和等号之间不能有空格**
- 变量名只能使用英文字母，数字和下划线，首个字符不能以数字开头
- 不能使用bash里的关键字（可用help命令查看保留关键字）

除了显式的直接赋值，还可以用语句给变量赋值
```bash
name=$(ls)
name=`ls`
```
---
## 变量使用
>使用一个定义过的变量，只需要在变量名前加美元符号$即可

```bash
name="shn"
echo $name	
echo ${name}
```
变量名外面的花括号在这里是可选的，加花括号是为了区分变量的边界，推荐给所有使用的变量都加上花括号

```bash
echo "I am ${name}hahaha"
```
上面这种情况如果不加花括号写成`echo "I am $namehahaha"`就无法区分变量name

已定义的变量可以被重新定义，重新定义时不需要加$符号，只有在使用变量的时候才需要

```bash
your_name="tom"
echo $your_name
your_name="alibaba"
echo $your_name
```
---
## 只读变量
>使用`readonly`命令可以将变量定义为只读变量

只读变量的值不能被改变，执行以下脚本会报错

```bash
your_name="tom"
readonly your_name
your_name="alibaba"
# script.sh: line 3: your_name: readonly variable
```
---
## 删除变量
> 使用`unset`命令可以删除变量

变量被删除后不能再次使用。unset 命令不能删除只读变量。

执行以下命令不会有任何输出
```bash
your_name="tom"
unset your_name
echo ${your_name}
```
---
# Shell字符串
字符串表示可以用单引号，双引号，也可以不用引号
## 单引号

```bash
name='bob'
str='this is a string'
echo ${str}
# this is a string
str2='hello ${name}'
echo ${str2}
# hello ${name}
str3='this is a \'string'
echo ${str3}
# unexpected EOF while looking for matching `''
str4='this is ''a string'
echo ${str4}
# this is a string
```
- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的
- 单引号字符串中不能出现单独的一个单引号（即使转义也不行），但可以成对出现，作为字符串拼接使用
- 要包含单引号可以将其包含在双引号字符串内
---
## 双引号

```bash
name="bob"
str="this is a string"
echo ${str}
# this is a string
str2="hello ${name}"
echo ${str2}
# hello bob
str3="this is a \"string"
echo ${str3}
# this is a "string
str4="this is ""a string"
echo ${str4}
# this is a string
str5="this is a\tstring"
echo -e ${str5}
# this is a		string
```
- 双引号里可以有变量
- 双引号里可以出现转义字符
- echo -e 解释\t \a \b \n等转义字符

---
## 拼接字符串
```bash
your_name="bob"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1
# hello, bob ! hello, bob !

# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3
# hello, bob ! hello, ${your_name} !
```
---

## 获取字符串长度
>使用#获取

变量为字符串时，`${#string}` 等价于 `${#string[0]}`

```bash
string="abcd"
echo ${#string}
# 4
string="abcd"
echo ${#string[0]}   
# 4
```
---
## 提取子字符串
>string:n:m
>提取字符串string中从第n个字符开始的m个字符

注意：第一个字符的索引为0
```bash
string="helloworld"
str2=${string:2:4}
echo ${str2}
# llow
```
---
## 查找子字符串
查找字符i或o的位置（哪个字母先出现就计算哪个）
```bash
string="runoob is a great site"
echo `expr index "$string" io`
# 4
```
