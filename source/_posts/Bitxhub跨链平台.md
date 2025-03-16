---
title: Bitxhub跨链平台
toc: true
date: 2023-10-11 19:00:45
tags: 区块链
categories: 区块链
---

​​点击阅读更多查看文章内容<!--more-->

 # BitXHub跨链平台
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/d0fd2158fcb516099c6c44fa2dd10dbf_1740930214881.png)


# 跨链系统架构
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/281e78fa1b54c8d495d796f483f28b5d_1740930214881.png)
**过程**
1. 在跨链合约中调用统一写好的Broker合约
2. Broker合约抛出事件由Plugin捕获到
3. 封装成平台统一的数据结构提交到中继链中
4. 目的链的跨链网关从中继链中同步IBTP数据结构
5. 网关将该数据结构通过Plugin提交到目的链

# 中继链体系架构
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e84db2efc64bb6182415e8f7b06ea6ed_1740930221785.png)
# 中继链的模块和流程
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/2c6e718a745f3e6cf3483ba2f222540b_1740930221785.png)
 
# 跨链网关
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/a0a16c346ee61f9cb9d41b559d3f2aee_1740930221785.png)
- 通过动态加载插件的形式适配应用链并随时进行热更新
- 无需保存跨链状态，重新启动时可直接从应用链和中继链恢复跨链交易相应的状态信息

# IBTP协议
>IBTP（Inter Blockchain Transfer Protocol）：由平台提出的一种通用的跨链交互的消息传输协议。

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/4c88d6d17fa3c61f4b664f3508b35cdf_1740930221785.png)
- 所有异构链的跨链交易都可以封装成统一格式的IBTP信息
- 调用信息（Payload）+证明信息（Proof）字段可以适配所有异构链
	
# 跨链验证引擎
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e7d7e98271e5aaab65bf19017c245b03_1740930221785.png)
- 应用链在加入时需要为自己编写跨链交易相关的验证规则提前部署到虚拟机中
- 每次交易都会调用验证规则对proof进行相应的验证


# 跨链实战
跨链系统启动流程
- 启动中继链
- 快速启动应用链和部署跨链合约
- 启动对接应用链的跨链网关
- 发送跨链交易进行跨链交互

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/039a609f5271a83c7e55e256a1042c39_1740930230463.png)

