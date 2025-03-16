---
title: Hyperledger Fabric学习笔记——2.开发环境搭建
toc: true
date: 2022-04-29 16:57:02
tags: 区块链
categories: Fabric
---

​​点击阅读更多查看文章内容<!--more-->

# 1.需要的环境
- docker
- docker-compose
- go
- JDK
- npm和node.js

# 2.下载fabric组件的Docker镜像
[下载地址](https://hub.docker.com/u/hyperledger)

baseos下载0.3.1，其余都下载1.0.0，标记为latest

- baseos: `docker pull hyperledger/fabric-baseos:x86_64-0.3.1`
- tools: `docker pull hyperledger/fabric-tools:x86_64-1.0.0`
- peer: `docker pull hyperledger/fabric-peer:x86_64-1.0.0`
- ccenv: `docker pull hyperledger/fabric-ccenv:x86_64-1.0.0`
- ca: `docker pull hyperledger/fabric-ca:x86_64-1.0.0`
- orderer: `docker pull hyperledger/fabric-orderer:x86_64-1.0.0`

镜像下载完成之后注意要给每个镜像添加一个latest标签

添加标签的方法：`docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`

```bash
docker tag hyperledger/fabric-tools:x86_64-1.0.0 hyperledger/fabric-tools:latest

docker tag hyperledger/fabric-ccenv:x86_64-1.0.0 hyperledger/fabric-ccenv:latest

docker tag hyperledger/fabric-ca:x86_64-1.0.0 hyperledger/fabric-ca:latest

docker tag hyperledger/fabric-orderer:x86_64-1.0.0 hyperledger/fabric-orderer:latest

docker tag hyperledger/fabric-peer:x86_64-1.0.0 hyperledger/fabric-peer:latest

docker tag hyperledger/fabric-baseos:x86_64-0.3.1 hyperledger/fabric-baseos:latest
```

# 3.下载fabric源码库

[github项目地址](https://github.com/hyperledger/fabric)

1. **下载**：在github.com的hyperledger目录下
git clone git://github.com/hyperledger/fabric.git
2. **切换版本**：cd fabric；git checkout release-1.0
3. **进入目录** ：cd /home/gopath/src/github.com/hyperledger/fabric/common/configtx/tool/configtxgen(其中workstation是gopath目录)
4. **编译工具** ：go install --tags=nopkcs11
5. **进入目录** ：cd /home/gopath/src/github.com/hyperledger/fabric/common/tools/cryptogen
6. **编译工具**： go install --tags=nopkcs11， 编译好的工具在gopath的bin目录下

>编译工具时可能会出现找不到文件的错误，这里需要检查一下gopath设置是否正确，目录结构是否正确，有时候即使通过 go en查到的gopath与命令实际查找的gopath不一致，建议通过以下方法再设置一遍：
在shell里面输入“sudo vi /etc/environment”，在打开的文件末尾加入：“export GOPATH=/home/gopath”。注意：这个目录是我选中的目录，替换成你使用的目录！

# 4.下载fabric-samples
[github项目地址](https://github.com/hyperledger/fabric-samples)
1. 下载： cd /home/gopath/src/github.com/hyperledger;
git clone https://github.com/hyperledger/fabric-samples.git
2. 进入目录：cd fabric-samples
3. 切换版本：git checkout release-1.0
4. 备份一下
