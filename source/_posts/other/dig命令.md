---
title: dig命令
date: 2024-05-18 16:19:59
toc: true
categories: 杂项
tags: 
    - DNS
    - 命令
---
dig是一个网络管理命令行工具，用于查询域名系统（DNS）。dig是域名服务器软件套件BIND的组成部分。
<!-- more -->

## dig命令

dig是一个网络管理命令行工具，用于查询域名系统（DNS）。dig是域名服务器软件套件BIND的组成部分。

## windows安装dig命令

[windows下dig安装教程](https://cloud.tencent.com/developer/article/1569087)

注意：现在对windows最高支持到9.16.30版本，9.18.4版本已经没有windows下载选项了

## 使用

查询域名信息：`dig baidu.com`

### 常用参数

- @：指定进行域名解析的域名服务器
- +trace：输出迭代查询的过程
- 查询NS记录：`dig baidu.com NS`（查询某个记录直接加上记录名）

### 输出内容

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240418153250.png" width="80%">

- dig 命令的版本和输入的参数。
- 服务返回的一些技术详情，比较重要的是 status。如果 status 的值为 NOERROR 则说明本次查询成功结束。
- "QUESTION SECTION"：显示我们要查询的域名以及要查询的记录，默认查询A记录。
- "ANSWER SECTION"：是查询到的结果。
- "AUTHORITY SECTION"：权威部分，查询域名的权威服务器，当无法给出答案时会给出下一步查询的服务器，详解在后。
- "ADDITIONAL SECTION"：附加信息，通常是AUTHORITY SECTION中域名对应的IP，也称作GLUE RECORD
- 本次查询的一些统计信息，比如用了多长时间，查询了哪个 DNS 服务器，在什么时间进行的查询等等。

### +trace递归解析示例

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240418155542.png" width="90%">

首先从本地DNS服务器172.19.2.1查询了根域名服务器的信息，然后从l.root-servers.net查询到了定义域名服务器的信息，然后从a.gtld-servers.net查询到了baidu.com的权威服务器，然后从ns4.baidu.com查到了baidu.com的A记录

### AUTHORITY SECTION

[参考文章](https://serverfault.com/questions/1088257/why-does-dig-not-show-the-authority-section-and-how-to-make-it-show-the-authorit)

AUTHORITY SECTION取决于查询使用的名称服务器。如果您没有使用@标志指定任何一个，它将使用本地递归的方式为您提供最终答案。这个答案可能是由递归名称服务器在得到答案之前查询许多不同的权威名称服务器计算出来的，因此在这个场景中可能没有权威名称服务器。

示例：

使用DNS服务器9.9.9.9查询serverfault.com结果中没有AUTHORITY SECTION，因为递归解析器不具有权威性，所以只返回了ANSWER
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240418160845.png" width="70%">

我们使用根权威服务器查询serverfault.com，因为根服务器不知道serverfault.com记录所以结果中没有ANSWER，但是根服务器知道需要去com顶级域名服务器查询该域名，所以在AUTHORITY中给出了顶级域名com的权威服务器地址
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240418161215.png" width="70%">

继续使用com服务器查询该域名，同样无法直接获得该域名的A记录，ANSWER为空，但是在AUTHORITY中给出了serverfault.com的NS记录，即该域名的权威服务器地址
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240418161519.png" width="70%">

使用该域名的权威服务器查询到了该域名的A记录，此时没有AUTHORITY，因为你根本不需要它，这是一种优化。
<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/20240418161711.png" width="80%">
