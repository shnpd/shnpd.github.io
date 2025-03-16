---
title: Hyperledger Fabric学习笔记——6.账本存储
toc: true
date: 2022-04-29 18:11:40
tags: 区块链 fabric 学习
categories: Fabric
---

​​点击阅读更多查看文章内容<!--more-->

# 1.账本存储概念
- peer节点做账本存储
- orderer是临时存储区块，peer节点是账本存储的持久化，会改变世界状态
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20c719852ee1edf54799f7889136bd91_1740930619743.png%20=500x)

- 文件系统：区块是存储为文件的
- 区块索引：用于查询区块，是用levelDB实现的
- 状态数据库：一般存放区块链最新状态，数据不需要HA，可以从文件系统再次获取，couchDB支持模糊查询

# 2. 交易读写集

- 交易流程 
  - 交易模拟 
    - 在背书节点执行模拟时，最终返回交易读写集（RWset），告诉区块链在交易中读写了哪些数据
  - 交易排序
  - 交易验证，交易验证后，更新世界状态，更新的就是读写集中的写集
- 读写集的3个概念 
  - 读集：包含键的列表，键的提交版本，读取对应的值，返回的是已提交的状态的值（读已提交的内容，不能读取当前交易写入的数据）
  - 写集：包含键的列表，写入的数据的值，如果多次写入，以最后一次为准
  - 版本号：用区块高度和交易编号组成
- 交易验证阶段是对读写集进行验证（验证读集） 
  - 验证读集的版本号是否等于世界状态的版本号

# 3.账本存储相关概念

- 世界状态 
  - 交易执行后，所有键的最新值
- 历史数据索引（可选）
- 区块存储 
  - 按照文件存储 blocfile_xxxxxx
  - 文件大小是64M，若修改，需要重新编译peer源码
  - 账本最大容量64M*
- 区块读取 
  - 区块文件流
  - 区块流
  - 迭代器
- 区块索引 
  - 键：区块高度、区块哈希、交易哈希
  - 值：区块文件编号、文件内的偏移量、区块数据的长度
- 区块提交 
  - 保存到文件

# 4. 账本存储相关源码
- 从4方面看，读写集、状态数据、历史数据、区块文件
- 可以先从core/ledger下的ledger_interface.go中看大体结构
- 读写集，分为交易读写集生成和交易读写集验证两个部分去看 
  - 交易读写集生成 
    - core/ledger/kvledger/txmgmt/txmgr/lockbasedtxmgr/lockbased_tx_simulator.go
  - 交易读写集验证 
    - core/ledger/kvledger/txmgmt/validator/statebasedval/state_based_validator.go
- 状态数据库：看levelDB的实现 
  - core/ledger/kvledger/txmgmt/statedb/stateleveldb/stateleveldb.go
- 历史数据库：看levelDB的实现 
  - core/ledger/kvledger/history/historydb/historyleveldb/historyleveldb.go
- 区块文件读取 
  - common/ledger/blkstorage/fsblkstorage/fs_blockstore.go
  - 区块流：common/ledger/blkstorage/fsblkstorage/block_stream.go

# 5.账本存储相关源码总结

- 首先看了账本存储的根目录的接口，从宏观上把握
- 交易读写集的生成和验证
- 数据库增删改查
- 还有一个没有实现的方法，数据裁剪的接口，可能去解决数据无限增长的处理
