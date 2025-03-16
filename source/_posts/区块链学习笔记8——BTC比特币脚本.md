---
title: 区块链学习笔记8——BTC比特币脚本
toc: true
date: 2022-01-12 15:56:00
tags: 区块链 学习 笔记
categories: 比特币
---

​​点击阅读更多查看文章内容<!--more-->

## 区块链学习笔记8——BTC比特币脚本
> 学习视频：[北京大学肖臻老师《区块链技术与应用》](https://www.bilibili.com/video/BV1Vt411X7JF)
笔记参考：[北京大学肖臻老师《区块链技术与应用》公开课系列笔记——目录导航页](https://blog.csdn.net/Mu_Xiaoye/article/details/104299664)

## 交易实例
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20bec33b5bb5d3f6d58281f165cd00b4_1740930880989.png)
比特币系统中使用的脚本语言非常简单，唯一可以访问的内存空间只有栈，所以也被称为“基于栈的语言”
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/641d735f630ae90329c8cec689fa7add_1740930880989.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/2d7a05fbab3e74a1f693e55350d1b51f_1740930880989.png)
如果存在 一个交易有多个输入，那么每个输入都要说明币的来源并给出签名（BTC中一个交易可能需要多个签名）
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/01a590b05ef29b7e3bb9699457095d03_1740930887862.png)
## 输入输出脚本的执行
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/523fef0bdd58232da5474eb2f88ca408_1740930887862.png)
如图所示，为脚本执行流程。在早期，直接将两个脚本按照如图顺序(input script在前，output script在后) 拼接后执行，后来考虑到安全性问题，两个脚本改为分别执行：先执行input script，若无出错，再执行output script。
如果脚本可以顺利执行，最终栈顶结果为非零值即true，则验证通过，交易合法；如果执行过程中出现任何错误，则交易非法。
如果一个交易有多个输入脚本，则每个输入脚本都要和对应的输出脚本匹配执行，全部验证通过才能说明该交易合法。
## 输入输出脚本的几种形式
### P2PK（Pay to Public Key）
最简单的方式
输出脚本直接给出收款人的公钥，CHECKSIG是检查签名的操作
输入脚本直接给出签名，这个签名是用私钥对整个输入脚本所在交易的签名
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/8e672cbc63293afbb2ca9cb0fe3cd10c_1740930887862.png)
**实际执行情况**：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/15dce9d1c973c27a6413849d822c437c_1740930887862.png)
实例：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/0c60e302458c5a4b4d7df546e5a2d618_1740930887862.png)

### P2PKH（Pay to public key hash）
输出脚本没有直接给出收款人的公钥，而是给出公钥的哈希，是最常用的形式
 ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/82e1af2084a2611be4da077099eddc0a_1740930894957.png)
**实际执行情况**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/1503fe4787495f9750f23efb2580f51b_1740930894957.png)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ca8d625793eb4bb9aafb0e45a9d62326_1740930894957.png)
>说明：
1.图中第5步，两个公钥哈希是不同的。上面一个是输出脚本提供的收款人的哈希，下面一个是要花钱时候输入脚本要给出的公钥通过HASH160操作得到的。
2…图中第6步，该操作的目的是为了防止冒名顶替(公钥)。假设比较正确，则两个元素消失（不往栈中压入TRUE或FALSE）。

实例：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/bc6ec3b6f90789aef5efb19d3a9ce214_1740930894957.png)
### P2SH（Pay to script hash）
输出脚本给出的不是收款人公钥的哈希，而是收款人提供的一个脚本的哈希。该脚本称为redeemScript,即赎回脚本。等未来花钱的时候，输入脚本要给出redeemScript的具体内容以及可以使之正确运行需要的签名。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ca9fa91224c5c62b291d395a8d351ac5_1740930894957.png)
**验证过程**：
1.验证序列化的redeemScript是否与output script中哈希值匹配。
2.反序列化并执行redeemScript，验证iutput script中给出签名是否正确。（将赎回脚本内容当作操作指令执行一遍）
**实例：用P2SH实现P2PK**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/50f7e60d457d694cb1ae81a9ec2af9f6_1740930901574.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/545c76380c97f1d2616a1fa7693a4ef7.png)![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/c1293a73439b1471820d600a090aec61_1740930901574.png)
第一阶段执行拼接后的输入和输出脚本。
第二阶段执行反序列化后的赎回脚本（反序列化操作并未展现，因为其是每个节点需要自己执行的）
> 针对以上例子使用赎回脚本可能会有些复杂，但实际上，该功能的一个主要的应用场景是多重签名。
> 即一个输出需要多个签名才能取出钱来，例如一个公司账户，需要五个合伙人中的任意三个的签名才能取走钱，这样便为私钥泄露和丢失提供了一定程度的保护。

## 多重签名
这个功能是通过CHECKMULTISIG来实现的，输出脚本给定N个公钥，同时指定一个阈值M，输出脚本只要给定这个N个公钥中M个合法签名即可通过验证，给出的M个签名顺序要和N个公钥中相对顺序一致


输出脚本最前面有一个红色的X，是因为比特币中CHECKMULTISIG的实现存在一个bug，执行时会从堆栈上多弹出一个元素。这个bug现在已经无法修改(去中心化系统中软件升级代价极大，需要硬分叉修改)。所以，实际中采用的方案是往栈中多压入一个无用元素。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/84acd26af6745680de55946bca03ed6c_1740930901574.png)
**执行实例**：
如图为一个N=3，M=2的多重签名脚本执行过程。其中前三行为输入脚本内容，后续为输出脚本内容。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ad7f9931bed3d48eaf7c14dc5cd8b34.png)![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/666be9efe9391ef4e1ee74f93c583c4b_1740930901574.png)
>这个过程当中并没有用到P2SH，只是用原生的CHECKMULTISIG实现的
早期的实际应用中，多重签名就是这样写的。但是，在应用中体现出了一些问题。例如，在网上购物时候，某个电商使用多重签名，要求5个合伙人中任意3个人才能将钱取出。这就要求用户在生成，转账交易时候，要给出五个合伙人的转账公钥以及N和M的值。而对于用户来说，需要购物网站公布出来才能知道这些信息。不同电商对于数量要求不一致，会为用户转账交易带来不便之处(因为这些复杂性全暴露给了用户)。
为了解决这一问题，就需要用到**P2SH**

### P2SH实现多重签名
>使用P2SH本质上是将复杂度从输出脚本转移到赎回脚本，输出脚本只需要给出赎回脚本的哈希值即可，该赎回脚本在输入脚本提供，即收款人提供。
>这样做，类似之前提到的电商，收款人只需要公布赎回脚本哈希值即可，用户只要在输出脚本中包含该哈希值，用户无需知道收款人的相关规则(对用户更加友好)。
>如果电商改变了多重签名的规则，只要改变输入脚本和赎回脚本的内容，然后把新的哈希值公布出去就行了
>![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f23b7da2add4ac525a64204086dce036_1740930901574.png)


**执行过程**

>第一阶段验证（输入输出脚本）：
>![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3bda4aaecab75bcfa0c70ad5f4b12ba2.png)![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/915c404991ba4eb39ee42cb586b8ac03_1740930908650.png)
>第二阶段验证（赎回脚本）：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/914a11046e2015d8e52dbaf586a4a432_1740930908650.png)


**使用P2SH做多重签名的实例**
>![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ac72e015b2932125f75fa4f286acf98e_1740930908650.png)
>输入脚本push的最后一个数据就是序列化的赎回脚本，反序列化得到的就是三个里面取两个的多重签名脚本
>现在的多重签名，大多都采用P2SH的形式

## 一个特殊的脚本
>以RETURN开始，后面可以跟任何内容。
RETURN操作，无条件返回错误，所以该脚本永远不可能通过验证。执行到RETURN，后续操作不会再执行。
该脚本是销毁比特币的一种方法。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/4da6fe39c934c2a91f1c16218249df64_1740930908650.png)

>Q：为什么要销毁比特币？？？现在比特币价值极高，销毁是不是很可惜？
1.部分小币种(AltCoin)要求销毁部分比特币才能得到该种小币种。例如，销毁一个BTC可以得到1000个小币。即，使用这种方法证明付出了一定代价，才能得到小币种。
2.往区块链中写入内容。我们经常说，区块链是不可篡改的账本，有人便利用该特性往其中添加想要永久保存的内容。例如：股票预测情况的哈希、知识产权保护——取知识产权的哈希值放在return的后面，反正return之后的都不会执行，放什么都没关系，而且放置的是一个哈希值不会占太大的空间，也没有泄露知识产权的具体内容，将来如果产生了纠纷，就把这个具体内容公布出去，证明在某个时间点已经知道某个知识了

>有没有觉得第二个应用场景有些熟悉？实际上，之前谈到BTC发行的唯一方法，便是通过铸币交易凭空产生（数据结构篇中）。在铸币交易中，有一个CoinBase域，其中便可以写入任何内容。那么为什么不使用这种方法呢，而且这种方法不需要销毁BTC，可以直接写入。
>因为这种方法只有获得记账权的节点才可以写入内容。而上面的方法，可以保证任何一个BTC系统中节点乃至于单纯的用户，都可以向区块链上写入想写入的内容。**【发布交易不需要有记账权，发布区块需要有记账权】**
任何用户都可以使用这种方法，通过销毁很小一部分比特币，换取向区块链中写入数据的机会。


实际上，很多交易根本没有销毁比特币，只是支付了交易费
>下图是一个铸币交易，其中包含两个交易，第二个交易的输出脚本就是开头为return，只是仅仅想要往其中写入内容。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/9953d185035c4a958b6942d992a559c9_1740930908650.png)
下图是一个转账交易，输出脚本也是以return开头的，输入是0.05BTC，输出是0，说明输入金额全部用来支付交易费，这个交易并没有销毁任何比特币，只是将输入的费用作为交易费转给挖到矿的矿工了
这种交易永远不会兑现，所以矿工不会将其保存在UTXO中，对全节点比较友好。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/2f2ed288b0e328cb986ea7c12c3c6c2c_1740930916135.png)

**实际中的脚本，都需要加上OP前缀，如：CHECKSIG应该为OP_CHECKSIG,这里仅仅为了学习友好，就删去了该前缀**

## 总结
BTC系统中使用的脚本语言非常简单，简单到没有一个专门的名称，我们就称其为”比特币脚本语言“。而在后文的以太坊的智能合约中，则比此复杂得多。实际上，该脚本语言甚至连一般语言中的循环都不支持，但设计简单却也有其用意。
如果不支持循环，也就永远不会出现死循环，也就不用担心停机问题。而在以太坊中，由于其语言图灵完备，所以要依赖于汽油费机制来防止其陷入死循环。此外，该脚本语言虽然在某些方面功能很有限，但另外一些方面功能却很强大(密码学相关功能很强大)
例如，前文提到的CHECKMULTISIG用一条语句便实现了检查多重签名的功能。这一点与很多通用编程语言相比，是很强大的。
