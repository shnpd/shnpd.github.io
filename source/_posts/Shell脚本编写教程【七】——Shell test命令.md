---
title: Shell脚本编写教程【七】——Shell test命令
toc: true
date: 2023-07-07 10:09:47
tags: linux bash
categories: Shell脚本
---

​​点击阅读更多查看文章内容<!--more-->

# Shell脚本编写教程【七】——Shell test命令
---
**目录**：[https://blog.csdn.net/shn111/article/details/131590488](https://blog.csdn.net/shn111/article/details/131590488)

**参考教程**：[https://www.runoob.com/linux/linux-shell.html](https://www.runoob.com/linux/linux-shell.html)

**在线编辑器**：[https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash](https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash)

---

>Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试

# 数值测试
|参数|说明|
|--|--|
|-eq|等于则为真|
|-ne|不等于则为真|
|-gt|大于则为真|
|-ge|大于等于则为真|
|-lt|小于则为真|
|-le|小于等于则为真|

实例：

```bash
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
# 两个数相等！
```

---

**代码中的`[]`执行基本的算术运算**

```bash
a=5
b=6

result=$[a+b] # 注意等号两边不能有空格
echo "result 为： $result"
# result 为： 11
```

---
# 字符串测试
|参数|说明|
|--|--|
|=|等于则为真|
|!=|不相等则为真|
|-z 字符串|字符串的长度为零则为真|
|-n 字符串|字符串的长度不为零则为真|

实例：

```bash
num1="ru1noob"
num2="runoob"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi
# 两个字符串不相等!
```
---
# 文件测试

|参数|说明|
|--|--|
|-e 文件名|如果文件存在则为真|
|-r 文件名|如果文件存在且可读则为真|
|-w 文件名|如果文件存在且可写则为真|
|-x 文件名|如果文件存在且可执行则为真|
|-s 文件名|如果文件存在且至少有一个字符则为真|
|-d 文件名|如果文件存在且为目录则为真|
|-f 文件名|如果文件存在且为普通文件则为真|
|-c 文件名|如果文件存在且为字符型特殊文件则为真|
|-b 文件名|如果文件存在且为块特殊文件则为真|

实例（在本文开头的在线编辑器中测试）：
```bash
cd /bin
if test -e ./bash
then
    echo '文件已存在!'
else
    echo '文件不存在!'
fi
# 文件已存在!
```

---
另外，shell还提供了与（-a）或（-o）非（!）三个逻辑操作符用于将测试条件连接起来，其优先级为：`!` 最高， `-a` 次之， `-o` 最低。例如：

```bash
cd /bin
if test -e ./notFile -o -e ./bash
then
    echo '至少有一个文件存在!'
else
    echo '两个文件都不存在'
fi
# 至少有一个文件存在!
```
