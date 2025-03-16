---
title: Goland连接服务器/虚拟机远程编译开发
toc: true
date: 2023-10-30 14:36:43
tags: 服务器 运维
categories: 杂项
---

​​点击阅读更多查看文章内容<!--more-->

# 创建SSH连接
>SSH用于与远程服务器建立连接

Settings -> Tools -> SSH Configurations
添加新的ssh连接，Host为ip地址，Username为用户名，认证方式这里选择密码验证
全部填完后可以点击Test Connection测试连接是否成功
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b92bed1bcf69261ad88559078106a479_1740930208396.png%20=600x)


# 创建Deployment
>Deployment用于构建本地与远程服务器的路径映射

Settings -> Build,Execution,Deployment -> Deployment
添加新的Deployment，Type选择SFTP，SSH选择刚刚配置的SSH连接，根目录，URL如图所示
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/d9fd3e599a4a8c217bdb8c191c3416d1_1740930208396.png%20=600x)
选择Mappings设置路径映射，Local path为项目本地目录，Deployed path是在远程服务器上的目录
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/3ed56d907c5b176bcff111ec516a6160_1740930208396.png%20=600x)
将本地的文件上传到服务器
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/784482c46b21ce9a743f5fa79e08c309_1740930214881.png%20=600x)
设置每次保存文件时都上传到服务器，在删除本地文件时也同步删除服务器文件
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/034570290e10f97eef9f3becec1aad22_1740930214881.png%20=600x)
# 使用服务器环境编译
>有的时候我们本地的环境与服务器不一致，导致项目在本地无法运行，在配置完上述步骤后可以修改为在服务器上进行编译

在Run/Debug Configurations中将Run on设置为ssh连接的服务器，同时勾选Build on remote target
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/8e6a5f050f081622bce88635f8905ca7_1740930214881.png%20=600x)

