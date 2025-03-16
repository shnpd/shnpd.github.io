---
title: 【Go】defer和os.Exit()
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang



---

点击阅读更多查看文章内容<!--more-->

# defer和os.Exit

问题：在使用os.Exit退出后，defer func() {...}()没有执行 

![image-20241205204106562](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/202412052041751.png)

原因：os.Exit函数描述如上，不会执行deferred functions