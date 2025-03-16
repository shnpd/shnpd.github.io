---
title: Hyperledger Fabric学习笔记——3.测试网络的启动分析
toc: true
date: 2022-04-29 17:34:56
tags: fabric 学习 docker
categories: Fabric
---

​​点击阅读更多查看文章内容<!--more-->

# 1. 启动网络

**执行以下指令均要以管理员身份运行，请首先执行su root命令**

1.  查看目录
`cd /home/gopath/src/github.com/hyperledger/fabric-samples/first-network`
.env:存储一些环境变量
base:存储docker-compose的一些公共服务
byfn.sh:执行脚本
configtx.yaml和crypto-config.yaml:根据之前生成的2个工具，生成相应的配置文件，用来启动网络，放到当前目录的channel-artifacts和crypto-config里面
docker-compose:用来启动网络
scripts:存放测试脚本，做的事：创建通道、加入通道、安装链码、实例化链码、链码交互 
2.  生成配置 `./byfn.sh -m generate -i 1.0.0 `

>注意！！！！
一定要记得把gopath中的bin目录添加到环境变量中，否则可能无法调用刚才编译好的configtxgen和cryptogen工具
可以通过 export PATH=$PATH:/home/gopath/bin 来临时添加或者使用vim ~/.bash_profile修改PATH 一行，之后使用source ~/.bash_profile生效。

3. 启动网络 `./byfn.sh -m up -i 1.0.0`
4. 关闭网络，自动清除配置和docker进程 `./byfn.sh -m down -i 1.0.0`

# 2.分析网络

1.  查看crypto-config配置
peer与order分离
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/27403303791f3b61625a7b162ca08539_1740930633267.png)
peer又按照组织或主体分离
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e691e9b1b4e4fc652a921b5c9f861870_1740930633267.png)
每个组织生成ca(存储证书和私钥),msp(存储管理员证书和中间证书),peers(存储每一个peer相关的证书),users(存储每一个用户的证书)
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e0a92bb3fcb56dc19e80f3e8edf22a86_1740930633267.png)
users的内容，最少包含两个用户——你创建的用户和admin用户
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/7765f7a0eef8e66c805df3f3a55f5e13_1740930633267.png)
peers的内容，组织1中存储的peer0和peer1
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/9def732ff6938784c4cb3b549acbe367_1740930639686.png)
2.  查看channel-artifacts配置
genesis.block:整个网络的创世区块
channel.tx:创建的通道的配置
Org1MSPanchors.tx和Org2MSPanchors.tx:两个主体的锚节点的配置
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/9d2ac386bc2cac722e2ff31efb456728_1740930639686.png)
3.  启动网络，分析日志
` ./byfn.sh -m up -i 1.0.0 `
4.  启动网络
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f2c0d10ac08dbd6982b0ba74000f7df5_1740930639686.png)
5.  指定通道名称和一些变量，通道创建完成
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/e6c43919e801390df07a7a41f9abb930_1740930639686.png)
6.  4个peer加入通道
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/3b01991707d5fae8aa3173fa0eabe629.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/81476609c54c2eaf36dab1fbc79cf482.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a00fd339fa086de4b8bfc93ca03f1eb2.png)![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/8ccaef772f408acb5492810b1675ad60_1740930639686.png)
7.  组织中的锚节点在通道update成功
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ddf1652494d414cff35aabf1894ec42.png)![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/39434bfaddb4a7e4238602c6721e4d70_1740930646022.png)
8.  链码安装到peer
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aaedbadff9a4f4fb5ec0ebb6657e39a1.png)![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/6358a19dc30c704c91e56425c8ecf0e8_1740930646022.png)
9.  链码实例化
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/58686d0c1a4ee70d5c7b17c15ee461a5_1740930646022.png)
10.  在peer0进行查询操作，成功，查询结果为100
 ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/df1ac26541b8889819a834d17047ced1_1740930646022.png)
11.  进行修改操作，返回200，修改成功
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/79d4e98d0e7a144a26c3e71e4248017b_1740930646022.png)
12.  再次查询结果为90
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/bec60235648547cd73b109db1f471afe_1740930651823.png)
13.  查看docker容器，3个dev开头的就是链码，4个peer开头的就是每一个peer
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/d60551623088be64a878d0225d0a39d8_1740930651823.png)
3个链码会生成3个image
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b9a5c7fac1dd4d94c7e16d2056875577_1740930651823.png)
14.  查看脚本
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/58929c99b51f91b8972881e04efbe69f.png)![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/dad45eeaf228719dcdb1e202ee6c5e32_1740930651823.png)
找到脚本对应位置的链码
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/412e2471a932eecbda2f65bfea09b3c4_1740930651823.png)


15.  查看go的代码
实现了Init和Invoke接口，就代表是一个fabric智能合约
Init：首先获取参数，不为4就报错，然后把参数存入数据库中
Invoke：设置了3个方法（invoke，delete，query）
invoke：转账操作
delete：从数据库删除
query：查询操作，以JSON形式返回 
16.  查看脚本
a初始有100元，b初始有200元
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/aa236a37bf999178b68504673347749e_1740930659019.png)
查询a有多少钱，所以打印了100
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f9594d503f96ef73c8a1ca43c12a6f59_1740930659019.png)
a给b转10元
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/a5188e46d3e7df07b37e0ffb6b20d0c6_1740930659019.png)
17.  总结
根据下面配置运行一个网络
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f420844b3cc12a524ed3b4d46886daff_1740930659019.png)
网络执行了一个链码，实现了初始化、查询、删除、转账等操作
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/5a061467060c72916ca7c21e7e842885_1740930659019.png)按照下面的脚本执行，首先进行初始化，然后查询a账户余额，然后a给b转账10元，然后再执行一个查询a账户余额的操作
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/10b4bbef8f5de643550d5f6dc57d0904_1740930666930.png)
18.  关闭网络，清除image和容器
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b4de741b3edc1e2f9e63422ea0e99fac_1740930666930.png)

