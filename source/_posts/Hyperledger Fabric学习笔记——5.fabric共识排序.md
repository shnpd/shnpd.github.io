---
title: Hyperledger Fabric学习笔记——5.fabric共识排序
toc: true
date: 2022-04-29 17:55:09
tags: 区块链 fabric 分布式账本
categories: Fabric
---

​​点击阅读更多查看文章内容<!--more-->

>fabric不需要依赖挖矿，通过交易排序达成共识

# 1.共识机制

- 达成共识需要3个阶段，交易背书，交易排序，交易验证
- 交易背书：模拟交易
- 交易排序：确定交易顺序，最终将排序好的交易打包区块分发
- 交易验证：区块存储前要进行一下交易验证

# 2.orderer节点的作用
- 交易排序 
  - 目的：保证系统的最终一致性（有限状态机）
  - solo：单节点排序
  - kafka：外置的分布式消息队列
- 区块分发 
  - orderer中的区块并不是最终持久化的区块
  - 是一个中间状态的区块
  - 包含了所有交易，不管是否有效，都会打包传给组织的锚节点
- 多通道的数据隔离
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f38e11f9438942b9bc096dc3d93df308_1740930626478.png%20=500x)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/30ef7e4f2e16b70c232c2e69025e1498_1740930626478.png%20=500x)

# 3. 源码目录
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/078d8dd04ffda939c78822d04c3f3189_1740930626478.png%20=400x)

- bccsp：与密码学相关的，加密、数字签名、证书，将密码学中的函数抽象成了接口，方便调用和扩展
- bddtests：行为驱动开发，从需求直接到开发
- common：公共库、错误处理、日志处理、账本存储、相关工具
- core：是fabric的核心库，子目录是各个模块的目录 
  - comm：网络通信相关
- devenv：官方提供的开发环境，使用的是Vagrant
- docs：文档
- events：事件监听机制
- examples：官方提供的例子程序
- gossip：通信协议，组织内部的通信，区块同步
- gotools：用于编译
- images：docker镜像相关
- msp：成员服务管理，member service provider，读取证书做签名
- orderer：排序节点
- peer：peer节点
- proposals：用于扩展，新功能的提案
- protos：数据结构的定义

# 4.共识机制源码

- main.go是入口，orderer节点的初始化
- manager.go是控制中枢，是对链的操作，拿到chainsupport对象
- chainsupport.go链对象的代理，与链是对应的
- cut()方法：区块切割
- solo和kafka相关的配置
- server.go有两个Handle，是交易收集和区块广播的方法
- 将共识简化为排序，达成最终一致性
