---
title: fabric网络搭建
toc: true
date: 2022-11-12 18:47:49
tags: fabric docker 运维
categories: Fabric
---

​​点击阅读更多查看文章内容<!--more-->

## 通用命令
**进入终端1**
`docker exec -it cli1 bash`
**进入终端2**
`docker exec -it cli2 bash`
**退出终端**
`exit`

**关闭网络并清除配置**
`docker-compose down -v`


---

# 命令执行顺序
## 通道操作

**创建通道**
`peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/msp/tlscacerts/tlsca.example.com-cert.pem`

**cli1加入通道**
`peer channel join -b mychannel.block`

**将区块文件复制到cli2**
`exit`

`docker cp cli1:/opt/gopath/src/github.com/hyperledger/fabric/peer/mychannel.block ./ `

`docker cp ./mychannel.block cli2:/opt/gopath/src/github.com/hyperledger/fabric/peer`

**cli2加入通道**
`peer channel join -b mychannel.block`

**设置锚节点**
组织1
`peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem`
组织2
`peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem`


## 链码操作
首先从fabric-samples中复制一个链码到chaincode/go目录下以便测试
`exit`
`cd /home/haonan/hyperledger/fabric-samples/chaincode/sacc`
`cp sacc.go /home/haonan/twonodes/chaincode/go/`

回到工作目录，进入终端
`cd /home/haonan/twonodes/chaincode/go/`
`docker exec -it cli1 bash`


**移动到链码路径**
`cd /opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go`

**配置go语言环境**
`go env -w GOPROXY=https://goproxy.cn,direct`
`go mod init`
`go mod vendor`

**返回工作目录**
`cd /opt/gopath/src/github.com/hyperledger/fabric/peer/`

**打包链码**
`peer lifecycle chaincode package sacc.tar.gz --path /opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go/ --label sacc_1`

**将打包链码复制到cli2**
`exit`

`docker cp cli1:/opt/gopath/src/github.com/hyperledger/fabric/peer/sacc.tar.gz ./`

`docker cp sacc.tar.gz cli2:/opt/gopath/src/github.com/hyperledger/fabric/peer`

**安装链码（两组织都需要）**
`peer lifecycle chaincode install sacc.tar.gz`

**组织批准链码（两组织都需要）**
**注意这里的package-id是安装链码时最后出现的id，每个人都不同需要更改为自己的id**
`peer lifecycle chaincode approveformyorg --channelID mychannel --name sacc --version 1.0 --init-required --package-id sacc_1:614c42c332c76568f8d4e839bd77a3b319796003f4e526c10aa907628fe5c541 --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem`

**查看组织是否批准**
`peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name sacc --version 1.0 --init-required --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  --output json`

**提交commit（单个组织提交即可）**
`peer lifecycle chaincode commit -o orderer.example.com:7050 --channelID mychannel --name sacc --version 1.0 --sequence 1 --init-required --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt`

**cli1链码调用**
`peer chaincode invoke -o orderer.example.com:7050 --isInit --ordererTLSHostnameOverride orderer.example.com --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n sacc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["a","bb"]}'`

**cli2链码查询**
`peer chaincode query -C mychannel -n sacc -c '{"Args":["query","a"]}'`

**cli2再次调用**
`peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n sacc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["set","a","cc"]}'`

**cli1链码查询**
`peer chaincode query -C mychannel -n sacc -c '{"Args":["query","a"]}'`


# 区块链浏览器
首先启动上面搭建的区块链网络

随后启动区块链浏览器，移动到testbe目录下
`docker-compose up -d`
