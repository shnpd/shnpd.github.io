---
title: Hyperledger Fabric学习笔记——7.链码安装、实例化、执行
toc: true
date: 2022-04-29 18:24:50
tags: fabric 学习 区块链
categories: Fabric
---

​​点击阅读更多查看文章内容<!--more-->

# 1.智能合约

- 执行环境：以太坊虚拟智能合约执行环境EVM，fabric执行环境是docker
- 链码 
  - 是应用层和区块链底层的中间点
  - 每一个链码执行环境是一个独立的docker
  - 使用GRPC协议与背书节点通信，只有背书节点才能运行智能合约
- 链码的生命周期 
  - 打包：智能合约的编写和编译
  - 安装：将打包好的文件，上传到背书节点
  - 实例化：实际安装，执行Init方法，只执行一次，构造函数
  - 升级：升级和修复链码
  - 交互：自己定义的方法的调用
- 链码的交互流程
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f34f5de51ebf50de8035c9accf240836_1740930613908.png)

- 系统链码（CC：chaincode） 
  - LSCC：管理链码的生命周期
  - CSCC：配置管理链码，管理链的配置
  - QSCC：查询账本存储，是一个区块索引的外部服务
  - ESCC：交易背书的链码，交易执行后的链码进行封装签名，给客户端返回背书交易结果
  - VSCC：交易验证的链码
- 链码编程的接口 
  - Init()：链码初始化，只执行一次
  - Invoke() ：链码的业务逻辑的编写
  - 上面2个方法参数一样，参数是SDK的接口
- 链码SDK的接口 
  - 写代码再看
- 一些注意点 
  - 分布式多机多节点执行，链码会执行多次
  - 不写随机函数，交易会无效，多次执行不一样
  - 不写系统时间，多机时间不一定一样

# 2.网络搭建配置的实现

- crypto-config.yaml：用于配置节点的个数，参考firstnetwork编写
- 编写好后，传到linux对应目录
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b68496b5548e65c1b7e4a949951d686f_1740930613908.png)
- 进入deploy目录，设置工作目录为当前目录
`export FABRIC_CFG_PATH=$GOPATH/src/fabric_asset/deploy`
- 指定按照yaml文件生成配置
`cryptogen generate --config=./crypto-config.yaml`
---
- configtx.yaml：用于区块联盟中的组织信息，配置名字和证书等的位置，参考firstnetwork编写
- 编写好后，传到linux对应目录
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f76447f4da085d3127e818047ad97ad1_1740930619743.png)
- 创建用于存放配置的目录
`mkdir config`
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ca322ad4ea35789600fb2903e953760e_1740930619743.png)

- 生成系统链的创世区块
-profile：指定联盟配置
-outputBlock：指定存放的位置
`configtxgen -profile OneOrgsOrdererGenesis -outputBlock ./config/genesis.block`
- 生成通道的创世交易
-profile：指定业务联盟
-outputCreateChannelTx：指定存放路径
-channelID：指定创建名字
`configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./config/mychannel.tx -channelID mychannel`
- 生成两个组织锚节点的交易信息
`configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./config/Org0MSPanchors.tx -channelID mychannel -asOrg Org0MSP`
`configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./config/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP`
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/c91940e59beee81f01ad3548cb9fe541_1740930619743.png)
---
-  新建docker-compose.yaml文件并移动到deploy目录下

# 3.启动网络

- 启动docker，后台运行（要以管理员身份运行）
`docker-compose up -d`
- 查看orderer节点的运行日志
`docker logs orderer.example.com`
- 与客户端交互操作
`docker exec -it cli bash`

- 创建通道
-o：指定与哪个orderer节点通信
-c：指定创建的通道名称
-f：指定使用的文件
`peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/config/mychannel.tx`
- 加入通道
`peer channel join -b mychannel.block`
- 查看peer加入的通道列表
`peer channel list`
- 指定主节点
`peer channel update -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/config/Org1MSPanchors.tx`
---
- 安装链码
-n：安装的名字
-v: version
-l：使用语言
-p：path
`peer chaincode install -n badexample -v 1.0.0 -l golang -p github.com/chaincode/badexample`
- 克隆一个会话，交互执行peer0，查看安装的链码
`docker exec -it peer0.org1.example.com bash`
`cd /var/hyperledger/production/chaincodes/`
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f4e8ef8f91be1fb6320bd350772ca47e_1740930619743.png)
- 链码实例化
`peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n badexample -l golang -v 1.0.0 -c '{"Args":["init"]}'`
- 链码交互执行
`peer chaincode query -C mychannel -n badexample -c '{"Args":[]}'`
- 多次执行查询，得到的结果不同，因为invoke()中使用了随机数，不要这么做
# 4.网络关闭
- 退出客户端
`exit`
- 在deploy目录下关闭docker
`docker-compose down`
