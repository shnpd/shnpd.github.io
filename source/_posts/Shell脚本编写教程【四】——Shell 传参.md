---
title: Shell脚本编写教程【四】——Shell 传参
toc: true
date: 2023-07-07 10:00:01
tags: linux bash
categories: Shell脚本
---

​​点击阅读更多查看文章内容<!--more-->

# Shell脚本编写教程【四】——Shell 传参
---
**目录**：[https://blog.csdn.net/shn111/article/details/131590488](https://blog.csdn.net/shn111/article/details/131590488)

**参考教程**：[https://www.runoob.com/linux/linux-shell.html](https://www.runoob.com/linux/linux-shell.html)

**在线编辑器**：[https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash](https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash)

---

>在执行Shell脚本时，可以向脚本传递参数，脚本内获取参数的格式为：$n。n代表一个数字，1为执行脚本的第一个参数，2为执行脚本的第二个参数，以此类推......

**实例**
以下实例我们向脚本传递三个参数，并分别输出，其中 $0 为执行的文件名（包含文件路径）
```bash
echo "Shell 传递参数实例！"
echo "执行的文件名：$0"
echo "第一个参数为：$1"
echo "第二个参数为：$2"
echo "第三个参数为：$3"
```
执行结果：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e4ceb732b86cf65a66c815e678e0faee_1740930237039.png)

---
# 特殊参数说明
|参数处理|说明|
|--|--|
|$#|传递到脚本的参数个数|
|$*|用一个字符串显示所有参数|
|$$|脚本运行的当前进程ID号|
|$!|后台运行的最后一个进程的ID号|
|$@|与$@类似，区别在下文提及|
|$-|显示Shell使用的当前选项，与set命令功能相同|
|$?|显示最后命令的退出状态，0表示没有错误，其他任何值表明有错误|

实例：
```bash
#!/bin/bash
echo "Shell 传递参数实例！"
echo "$#"
echo "$*"
echo "$$"
echo "$!"
echo "$@"
echo "$-"
echo "$?"
```
执行结果：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/008aca5e46538eb8f09a5d0805b461de_1740930237039.png)


>$* 与 $@ 区别：
>相同点：都是引用所有参数
>不同点：只有在双引号中体现出来，假设脚本运行时传递了三个参数1、2、3，则"*"等价于"1 2 3"(传递了一个参数)，而"@"等价于"1" "2" "3"(传递了三个参数)

实例：

```bash
echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```
执行结果：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/50cd00ad8a6fa3870d59390456eabbfa_1740930237039.png)

