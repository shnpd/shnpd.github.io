---
title: Shell脚本编写教程【十】——Shell 输入输出重定向
toc: true
date: 2023-09-16 15:24:14
tags: linux bash
categories: Shell脚本
---

​​点击阅读更多查看文章内容<!--more-->

# Shell脚本编写教程【十】——Shell 输入/输出重定向
---
**目录**：[https://blog.csdn.net/shn111/article/details/131590488](https://blog.csdn.net/shn111/article/details/131590488)

**参考教程**：[https://www.runoob.com/linux/linux-shell.html](https://www.runoob.com/linux/linux-shell.html)

**在线编辑器**：[https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash](https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash)

---

>大多数 UNIX 系统命令从你的终端接受输入并将所产生的输出发送回​​到您的终端。一个命令通常从一个叫标准输入的地方读取输入，默认情况下，这恰好是你的终端。同样，一个命令通常将其输出写入到标准输出，默认情况下，这也是你的终端

重定向命令列表如下：
|命令|说明|
|--|--|
|command > file|将输出重定向到 file|
|command < file|将输入重定向到 file|
|command >> file|将输出以追加的方式重定向到 file|
|n > file|将文件描述符为 n 的文件重定向到 file|
|n >> file|将文件描述符为 n 的文件以追加的方式重定向到 file|
|n >& m|将输出文件 m 和 n 合并|
|n <& m|将输入文件 m 和 n 合并|
|<< tag|将开始标记 tag 和结束标记 tag 之间的内容作为输入|

>需要注意的是文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）

---
# 输出重定向
>重定向一般通过在命令间插入特定的符号来实现。

```bash
command1 > file1
```

`>`：重定向内容覆盖原文件
`>>`：重定向内容追加到原文件

实例（执行下面的 who 命令，它将命令的完整的输出重定向在用户文件中(users)）

```bash
who > users
```

---
# 输入重定向
和输出重定向一样，Unix 命令也可以从文件获取输入，语法为：

```bash
command1 < file1
```
这样，本来需要从键盘获取输入的命令会转移到文件读取内容

实例（统计 users 文件的行数）

```bash
wc -l users
# 2 users
```
也可以将输入重定向到 users 文件

```bash
wc -l < users
#  2
```
注意：上面两个例子的结果不同：第一个例子，会输出文件名；第二个不会，因为它仅仅知道从标准输入读取内容。


```bash
command1 < infile > outfile
```
同时替换输入和输出，执行command1，从文件infile读取内容，然后将输出写入到outfile中
