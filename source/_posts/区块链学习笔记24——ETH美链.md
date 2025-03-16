---
title: 区块链学习笔记24——ETH美链
toc: true
date: 2022-01-24 13:15:28
tags: 
categories: 以太坊
---

​​点击阅读更多查看文章内容<!--more-->

# 区块链学习笔记24——ETH美链
> 学习视频：[北京大学肖臻老师《区块链技术与应用》](https://www.bilibili.com/video/BV1Vt411X7JF)
笔记参考：[北京大学肖臻老师《区块链技术与应用》公开课系列笔记——目录导航页](https://blog.csdn.net/Mu_Xiaoye/article/details/104299664)

>ICO：Initial Coin Offering  首次代币发行
>IPO：Initial Public Offering 首次公开募股

# 背景介绍
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/fc8f072bc7fc8591546ac79a57f32c2d_1740930710198.png)


这些发行的代币没有自己的区块链，而是以智能合约的形式运行在以太坊的EVM平台上，发行代币的智能合约对应的是以太坊状态树中的一个节点，这个节点有他自己的账户余额，就相当于这个智能合约一共有多少个以太币，就是这个发行代币的智能合约他的总资产是多少个以太币，然后在合约里每个账户有多少代币是作为存储树中的变量，存储在智能合约的账户里，代币的发行，转账，销毁都是通过调用智能合约中的函数来实现的，每个代币都可以制定自己的转换规则，比如1个以太币=100个代币，比如外部账户给这个智能合约转一个以太币，智能合约就会给你在合约里的代币账户发送100个代币，每个账户的余额都是维护在发行代币的智能合约的存储树里面。
美链的代币叫BEC，比如我有很多BEC，给十个不同的账户发送代币，调用这个batchTransfer 函数，每个人发送100个代币，那么这个batchTransfer函数先从我的账户上扣掉1000个代币，然后给十个账户分别增加100个代币。

# batchTransfer函数
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/5bb353882a60bc82f4646bc07b27327d_1740930710198.png)
# 攻击细节
一堆数字是函数调用的参数，函数有两个参数，分别对应那串数字的前两行，第一个参数是地址，第一行给出的实际上是第一个参数出现的具体位置，这里是16进制的，40也就是64，也就是说第一个参数出现在第64个字节的位置，每一行是32个字节，所以实际上是从第2号开始出现的。第二行是这个value的值，这是个很大的数，前面是8，后面都是0，第三行是这个数组的具体内容，数组的长度，是2，接下来两行是两个接收的地址。

参数的特征：第二行amount是8，再乘以2，算出来的amount恰好溢出为0，add(value)的时候还是加原来特别大的那一串数目。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b887e8267c60986ca87ad23986b7f822_1740930710198.png)
红框中是接收地址接收的代币，每个地址都接收到了很大一部分代币， 

被攻击之后暂停提币的功能，防止黑客携款逃走。两天之后回滚交易，事件影响没有The DAO影响深远。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/60d622d7c275281f37302e5feb1869f9_1740930710198.png)
攻击发生后这个代币的价格暴跌
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/2db2d132bf857e003c0ecf59fcca594f_1740930710198.png)
代币上市的交易所在发生攻击后暂停提币的功能
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/1c53e5a6536fcc97a0d947772880fb25_1740930718336.png)
# 反思
在进行数学运算的时候一定要考虑溢出的可能性。solidity有一个safeMath库，里面提供的操作运算都会自动检测有没有出现溢出。

C语言里，两个数相乘会有一定的精度损失，再除以一个数，不一定会得到和另外一个数一模一样的数。但是在solidity里面是不存在的，因为两个数都是256位的整数，整数先进行乘法，再进行除法。

batchTransfer的加法和减法都用的safeMath库，只有乘法不小心没有使用，结果酿成了悲剧。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/9eb43b84bcebd949dfee71c62da3affd_1740930718336.png)


