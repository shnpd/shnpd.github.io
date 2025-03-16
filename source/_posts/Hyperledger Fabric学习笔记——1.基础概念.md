---
title: Hyperledger Fabric学习笔记——1.基础概念
toc: true
date: 2022-04-29 16:48:14
tags: 区块链
categories: Fabric
---

​​点击阅读更多查看文章内容<!--more-->

>hyperledger官网：https://www.hyperledger.org/
# hyperledger与数字货币
-  都是基于区块链技术实现的
-  比特币1秒7笔交易，以太坊1分钟几百笔，hyperledger理论上一分钟50万笔交易
-  hyperledger因为不用挖矿，不需要很强的硬件支持，也不耗费资源
-  hyperledger没有51%攻击问题，加入链中的成员要经过CA认证，是有许可的网络

# fabric是什么

-  目标：做企业级**联盟链**的基础设施 
  	-  公有链：全网公开，没有类似CA的用户验证
  	-  联盟链：只针对某个特定群体的成员和有限的第三方，内部指定多个预选节点为记账人，每个块的生成由所有的预选节点共同决定
  	-  私有链：所有网络中的结点都掌握在一家机构手中
  	-  联盟链和私有链可统称为许可链
-  可插拔的共识机制（solo和kafka等）
-  多链多通道隔离，可以做业务隔离，保护业务数据隐私

# fabric的重要组件

-  fabric CA 
  	- 自动生成认证证书
  	- 创建账户
  	- 是一个工具集
- fabric Peer 
  	- 可以有很多Peer，是Ledger和blockchain存储的位置
  	- 一个peer可以加入不同的channel
- fabric ordering service 
  	- 提供排序服务，用来做共识
  	-  创建block区块
  	- 使用solo排序，配置成使用kafka排序（优先状态机）

# fabric的开发语言

-  智能合约 
  	- go
  	- java
- SDK 
  	- java
  	- node.js（官方推荐，效率）
  	- go（大坑，支持是最差的）
  	- python

# fabric的channel

-  每个channel可以理解成独立的fabric实例
- 不同的channel是私有的子网，类似于微信群，隔离业务数据
- peer是微信里的人，peer可以加入不同的channel
- 还可以设置允许什么人加入

# fabric的chaincode

-  chaincode(链码)就是智能合约，是一个应用程序
- 用于更新账本数据，由peer去执行chaincode
- 在fabric里，chaincode是数据唯一的更新方式
- chaincode属于某一个channel
- chaincode的生命周期 
  	- 安装链码
  	- 实例化（调用init方法）
  	- 调用使用（调用invoke方法）
- 每个chaincode有不同的背书策略（如何去达成共识） 
  	- 可能有的chaincode是所有人都同意才可以
  	- 可能有的chaincode是至少有一个人同意才可以
  	- 可能有的chaincode是有4个人同意才可以
  	- 适应企业复杂应用场景

# fabric的msp
- 是一组重要的密码学签名工具
- 定义了你是谁，你在哪（在哪个channel中）
- CA去颁发证书

# 术语回顾
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b94247b508fa69cfe8240ba39be6edd7_1740930666930.png)
- channel数据通道，独立的fabric实例，不同channel数据是隔离的
- world state 是世界状态，是ledger里面存放的数据，是KV（key-value）数据的存储，可以用leveldb和couchdb存储
- ledger：账本，记录当前所有的世界状态，从底层架构上保证了数据一致性，不可篡改性
- chaincode：链码，编写智能合约，ledger的变化只能通过调用链码实现
- peer：是整个网络的基础，每一个peer的可以持有一个或多个ledger，每一个peer也可以有一个或多个chaincode
- network：是有peer组成的网络，形成区块链网络，在同一个网络中peer实现同步记账，保证了peer的数据一致性
- ordering service：排序服务，进行排序和验证，验证通过的数据，写入peer节点的ledger，具体还要看背书策略
- msp：peer节点的认证和标识
