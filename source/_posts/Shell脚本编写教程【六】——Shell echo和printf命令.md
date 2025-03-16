---
title: Shell脚本编写教程【六】——Shell echo和printf命令
toc: true
date: 2023-07-07 10:07:54
tags: linux bash
categories: Shell脚本
---

​​点击阅读更多查看文章内容<!--more-->

# Shell脚本编写教程【六】——Shell echo和printf命令
---
**目录**：[https://blog.csdn.net/shn111/article/details/131590488](https://blog.csdn.net/shn111/article/details/131590488)

**参考教程**：[https://www.runoob.com/linux/linux-shell.html](https://www.runoob.com/linux/linux-shell.html)

**在线编辑器**：[https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash](https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash)

---

# Shell echo命令

>命令格式：`echo string`

**1、显示普通字符串**

双引号可以省略
```bash
echo "It is a test"
echo It is a test
# It is a test
# It is a test
```

**2、显示转义字符**
双引号也可以省略
```bash
echo \"It is a test\"
echo "\"It is a test\""
# "It is a test"
# "It is a test"
```

**3、显示变量**
read命令从标准输入中读取一行并赋值给变量

```bash
read name 
echo "$name It is a test"
```
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/3fec5647e599acf1e84dfac233aa3362_1740930230463.png)

**4、显示换行**
-e：开启转义
```bash
echo -e "OK! \n" # -e 开启转义
echo "It is a test"
# OK! 
# 
# It is a test
```

**5、显示不换行**
-e：开启转义
\c：不换行
```bash
echo -e "OK! \c" # -e 开启转义 \c 不换行
echo "It is a test"
# OK! It is a test
```
**6、显示结果重定向至文件**

```bash
echo "It is a test" > myfile
cat ./myfile
# It is a test
```

**7、原样输出字符串，不进行转义或取变量**
用单引号包裹字符串

```bash
echo '$name\"'
# $name\"
```

**8、显示命令执行结果**
注意：使用反引号`
```bash
echo `date`
# Tue 04 Jul 2023 01:30:02 PM UTC
```

---
# Shell printf命令

>printf命令的语法：
>`printf format-string [arguments]`
>format-string：格式控制字符串
>arguments：参数列表

```bash
echo "Hello, Shell"
printf "Hello, Shell\n"
# Hello, Shell
# Hello, Shell
```

实例：
```bash
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876
# 姓名     性别   体重kg
# 郭靖     男      66.12
# 杨过     男      48.65
# 郭芙     女      47.99
```
%s %c %d %f 都是格式替代符，％s 输出一个字符串，％d 整型输出，％c 输出一个字符，％f 输出实数，以小数形式输出。

%-10s 指一个宽度为 10 个字符（- 表示左对齐，没有则表示右对齐），任何字符都会被显示在 10 个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来（一个汉字占三个字符）。

%-4.2f 指格式化为小数，其中 .2 指保留2位小数。


```bash
# format-string为双引号
printf "%d %s\n" 1 "abc"

# 单引号与双引号效果一样
printf '%d %s\n' 1 "abc"

# 没有引号也可以输出
printf %s abcdef

# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def

printf "%s\n" abc def

printf "%s %s %s\n" a b c d e f g h i j

# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n"
```

以下实例的第三行输出abcdefabcdefabc
来自：
printf %s abcdef
printf %s abc def
printf "%s\n" abc def 截止到此命令输出一个abc后才会输出一个换行
```bash
# format-string为双引号
printf "%d %s\n" 1 "abc"

# 单引号与双引号效果一样
printf '%d %s\n' 1 "abc"

# 没有引号也可以输出
printf %s abcdef

# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def

printf "%s\n" abc def

printf "%s %s %s\n" a b c d e f g h i j

# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n"

# 1 abc
# 1 abc
# abcdefabcdefabc
# def
# a b c
# d e f
# g h i
# j  
#  and 0 

```

**printf的转义序列**
|序列|说明|
|--|--|
|\a|警告字符，通常为ASCII的BEL字符|
|\b|后退|
|\c|抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略|
|\f|换页（formfeed）|
|\n|换行|
|\r|回车（Carriage return）|
|\t|水平制表符|
|\v|垂直制表符|
| \\\ |一个字面上的反斜杠字符|
|\ddd|表示1到3位数八进制值的字符。仅在格式字符串中有效|
|\0ddd|表示1到3位的八进制值字符|
