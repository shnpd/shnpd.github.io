---
title: 区块链学习笔记19——ETH难度调整
toc: true
date: 2022-01-20 11:22:26
tags: 区块链 以太坊 数字货币
categories: 以太坊
---

​​点击阅读更多查看文章内容<!--more-->

# 区块链学习笔记19——ETH难度调整
> 学习视频：[北京大学肖臻老师《区块链技术与应用》](https://www.bilibili.com/video/BV1Vt411X7JF)
笔记参考：[北京大学肖臻老师《区块链技术与应用》公开课系列笔记——目录导航页](https://blog.csdn.net/Mu_Xiaoye/article/details/104299664)

前面学过，比特币是每隔2016个区块调整一次挖矿难度，目的是维持出块时间在10分钟左右；**以太坊是每个区块都有可能调整挖矿难度**，调整的方法比较复杂而且改过好几个版本。网络上存在诸多不一致，这里遵循以代码为准的原则，从以太坊代码中查看以太坊难度调整算法。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/dbe7bdc46b1a392a24d5d2d9de6af4c1_1740930778079.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/01e3c22cf35808f1f5ba5320f87da74a_1740930778079.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e2a89e6acbd37f59bf23723ba701e7f5_1740930778079.png)

# 难度炸弹
以太坊在设计之初就计划要逐步从POW（工作量证明）转向POS（权益证明），而权益证明不需要挖矿，这就带来一个问题——已经在挖矿设备上投入大量资金的矿工会不会联合起来抵制这个转换，从POW转向POS需要通过硬分叉来实现，这样就可能导致社区分裂，以太坊可能分裂成两条平行的链。
在以太坊早期时，区块号较小，难度炸弹计算所得值较小，难度调整级别基本上通过难度调整中的自适应难度调整部分决定，而随着越来越多区块被挖出，难度炸弹的威力开始显露出来，这也就使得挖矿变得越来越难，从而迫使矿工愿意转入POS。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/1adb9af6a4b1831584ee1fe9fd12bc4b_1740930778079.png)
**难度炸弹的调整**
因为开发者低估了POS的设计难度，导致其迟迟没有设计出来，但是难度炸弹的威力已经开始显现出来，系统的出块时间已经开始逐渐变长，但矿工还需要继续挖矿，因此我们可以看到，上图中第二项的fake block number，该数对当前区块编号减去了三百万，也就是相当于将区块编号回退了三百万个。从而降低了出块的难度。当然，为了保持公平，也将出块奖励从5个以太币减少到了3个以太币。
下图显示了难度调整对难度炸弹难度影响的结果：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/93b82ff2627198a9251780b96c2abd76_1740930786883.png)
# 以太坊发展的四个阶段
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/1cecc962ba1c3001bbf96db5e81bdbd8_1740930786883.png)
# 具体的代码实现
**难度计算公式**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/8d1bf4e265a88f08e21950f83ccd4780_1740930786883.png)
**基础部分的计算**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/75b058b49fa9f5189edf27d5b0aae65e_1740930786883.png)

**难度炸弹的计算**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/bed554cc90cb1b59cb2807a3d5c00ee8_1740930786883.png)
# 以太坊实际统计数据（截止2018年）
**挖矿难度变化曲线**
断崖式下跌是难度炸弹的调整
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/bbb9bc26245c2a6be27d58990b86b641_1740930795765.png)
**出块时间**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/918cf6e4e6e0779815c3edd3f8d13c71_1740930795765.png)
**实际区块例子**
最长合法链对于以太坊来说也叫作最难合法链
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ceb089ccabc841b1b167c7b62b101ecf_1740930795765.png)


