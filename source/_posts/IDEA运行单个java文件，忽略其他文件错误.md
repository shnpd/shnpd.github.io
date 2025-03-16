---
title: IDEA运行单个java文件，忽略其他文件错误
toc: true
date: 2023-12-26 17:59:34
tags: java intellij-idea ide
categories: Java
---

​​点击阅读更多查看文章内容<!--more-->

# IDEA运行单个java文件，忽略其他文件错误
使用idea想单独运行一段测试程序，但是直接run的话会报其他程序的错误，使用以下方法可以实现单独运行一段测试程序。
1. File | Settings | Build, Execution, Deployment | Compiler | Java Compiler，将compiler修改为Eclipse（注意这里有一个Project bytecode version后面可能会报错）
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/2451c6d300777a6533c14b748bd51c32_1740930192419.png)
设置完成之后如果直接编译会报错：java: java.lang.IllegalArgumentException: source level should be in '1.1'...'1.8','9'...'18' (or '5.0'..'18.0'): 21
这里是因为默认的版本21不支持，按照错误信息，我们将Project bytecode version修改为18即可。
2. 修改Run/Debug Configurations，添加Add before launch task，去掉默认的Build，添加Build，no error check
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/7e0b59a97a1c7a9365d3d2f84045fba0_1740930200442.png)
3. 直接run测试代码即可
