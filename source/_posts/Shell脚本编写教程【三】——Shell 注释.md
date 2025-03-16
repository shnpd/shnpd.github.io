---
title: Shell脚本编写教程【三】——Shell 注释
toc: true
date: 2023-07-07 10:00:50
tags: linux bash
categories: Shell脚本
---

​​点击阅读更多查看文章内容<!--more-->

# Shell脚本编写教程【三】——Shell 注释
---
**目录**：[https://blog.csdn.net/shn111/article/details/131590488](https://blog.csdn.net/shn111/article/details/131590488)

**参考教程**：[https://www.runoob.com/linux/linux-shell.html](https://www.runoob.com/linux/linux-shell.html)

**在线编辑器**：[https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash](https://www.runoob.com/try/runcode.php?filename=helloworld&type=bash)

---
以 # 开头的行就是注释，会被解释器忽略。

通过每一行加一个 # 号设置多行注释

>如果在开发过程中，遇到大段的代码需要临时注释起来，过一会儿又取消注释，怎么办呢？
每一行加个#符号太费力了，可以把这一段要注释的代码用一对花括号括起来，定义成一个函数，没有地方调用这个函数，这块代码就不会执行，达到了和注释一样的效果。

---
# 多行注释
多行注释还可以使用以下格式

```bash
:<<EOF
注释内容...
注释内容...
注释内容...
EOF
```

EOF可以使用任意符号代替

```bash
:<<'
注释内容...
注释内容...
注释内容...
'

:<<!
注释内容...
注释内容...
注释内容...
!
```
