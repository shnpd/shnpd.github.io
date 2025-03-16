---
title: 区块链学习笔记2——BTC中的数据结构
toc: true
date: 2021-12-11 11:28:22
tags: 区块链 数据结构 比特币
categories: 比特币
---

​​点击阅读更多查看文章内容<!--more-->

## 区块链学习笔记2——BTC中的数据结构
> 学习视频：[北京大学肖臻老师《区块链技术与应用》](https://www.bilibili.com/video/BV1Vt411X7JF)
笔记参考：[北京大学肖臻老师《区块链技术与应用》公开课系列笔记——目录导航页](https://blog.csdn.net/Mu_Xiaoye/article/details/104299664)

本文主要介绍四种数据结构：Hash pointers、Block chain、Merkle tree、Block

## Hash pointers（哈希指针）

> 哈希指针与普通指针类似，它除了保存地址之外还保存哈希值


## Block chain（区块链）
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/a67f05bd86c6f6651e8bdfafd55b36e0_1740931338846.png)
区块链中区块与区块之间通过哈希指针相连，这里哈希指针中的哈希值是对前一个区块的整体取哈希，包括前一个区块的哈希指针，因此区块链如果有一个区块被修改，那么他之后的所有区块都会改变，我们只需要记住最后一个区块的哈希值，就可以判断整个区块链有没有被修改。
用户一般不需要保存区块链中的全部区块，在使用的时候向其他区块要即可，通过哈希值可判断对方给的区块有没有被修改。

## Merkle tree

> Merkle Tree与二叉树类似，主要是将二叉树中的指针换成了哈希指针

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/3548bfd7b35898835792cf37ba70b77a_1740931338846.png)
在Merkle tree中，我们只需要记住根节点的哈希值，就可以检测出整棵树的内容有没有被修改。
**每个区块所包含的交易都组织成Merkle tree的形式，就是上图中的Data Blocks。**

## Block（区块）
block包括block header和block body两部分
block header：root hash（Merkle tree的根哈希值）等信息
block body：交易列表

## Merkle proof
Markle Tree可以用于提供Markle Proof。关于Markle proof，需要先了解比特币系统中节点。比特币中节点分为轻节点和全节点。全节点保存整个区块的所有内容，而轻节点仅仅保存区块的块头信息。
当需要向轻节点验证它的交易是否被写到区块中时需要用到Merkle proof。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e683ca3baccd25561f0e8a727707a446_1740931338846.png)
以上图为例，要验证黄色的交易是否被写到区块中，我们首先要请求第三层的红色的哈希值，然后根据黄色的交易求出第三层的绿色哈希值，然后通过这两个已知的哈希值求出第二层的绿色哈希值，再请求第二层的红色哈希值，以此类推最终求得根哈希值，然后与存储在自身结点中的根哈希值比较，若相同则证明交易已经写入到了区块中。

上面的验证是proof of membership 时间复杂度为O(logn)
关于proof of non-membership验证，如果交易是无序的，我们只能证明每个结点都是对的，说明没有该节点，时间复杂度为O(n)
如果交易是按hash值从小到大排序，我们对要检验的交易取hash，判断它应处的位置，然后计算他前后的两个结点的hash，若这两个结点确实相邻（merkle proof），则证明我们要找的结点不存在（若存在则会在这两个结点之间，这两个结点不会相邻），时间复杂度为O(logn)

## 注意
无环的情况下都可以使用hash pointer代替point
有环时会因为每一个hash point都与它前一个的hash值有关，所以会产生循环依赖，每个hash都定不下来。
