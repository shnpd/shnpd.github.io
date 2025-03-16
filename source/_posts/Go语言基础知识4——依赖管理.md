---
title: Go语言基础知识4——依赖管理
toc: true
date: 2022-01-13 16:00:26
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

>- 依赖：别人写的库，依赖其进行编译
>- 依赖管理的三个阶段：GOPATH，GOVENDOR，go mod
>- GOPATH和GOVENDOR正在向go mod迁移

# 一、GOPATH
GOPATH是一个环境，就是一个目录

- 默认在~/go（unix，linux），%USERPROFILE%\go（windows）
- 管理方式：给一个目录，所有的依赖都到GOPATH下去找
- GOPATH要求在目录下必须有一个src目录，文件都放在$GOPATH/src目录下
- 问题：所有的项目都在GOPATH目录下面，依赖库也放在GOPATH下面，导致GOPATH越来越大

>**注意**：
>1. 在使用GOPATH和GOVENDOR管理依赖时GO111MODULE要设置为off
在终端输入命令：set GO111MODULE=off（Windows下）
>2. 在运行的时候在要进行如下设置，否则他还是会按照on来运行
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/fe11392cd971505c48b88016a027b460_1740930874000.png)


>GO111MODULE=auto 在$GOPATH/src 外面且根目录有go.mod 文件时，开启模块支持
>GO111MODULE=off 无模块支持，go会从GOPATH 和 vendor 文件夹寻找包
>GO111MODULE=on 模块支持，go会忽略GOPATH和vendor文件夹，只根据go.mod下载依赖

# 二、GOVENDOR
>所有项目都放在一个path下它们所依赖的库会有冲突（如：不同项目依赖同一个库的不同版本）

- 每个项目都有自己的vendor目录，存放第三方库
GOVENDOR有大量第三方依赖管理工具：glide，dep，go dep

 
 **从下图可以看到对依赖会依次查找vendor、GOROOT和GOPATH目录**
 
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/55735c230d77f45705be4178d52c71b6_1740930874000.png)

# 三、go mod

>**项目会新建go.mod文件，依赖包到go.mod下找**

以zap为例，main包如下
```go
package main

import "go.uber.org/zap"

func main() {
	logger, _ := zap.NewProduction()
	logger.Warn("warning test")
}
```
go.mod如下

```go
module awesomeProject1

go 1.17

require (
	go.uber.org/atomic v1.10.0 // indirect
	go.uber.org/multierr v1.8.0 // indirect
	go.uber.org/zap v1.23.0 // indirect
)
```

zap.NewProduction()目录为：`$GOPATH/pkg/mod/go.uber.org/zap@1.23.0/logger.go`

---

- 与项目不在同一目录下，由go命令统一的管理，用户不必关心目录结构

- 初始化：
go mod init

- 增加依赖：
使用go get命令拉取
在代码中直接import

- 更新依赖：
go get [@v...] (不加版本号默认为最新版本)，
go mod tidy(清除多余文件)

- 项目迁移到go mod：
go mod init
go build ./... （当前目录以及其所有子目录全部build一遍）


>**目录整理**
一个目录下只能有一个main函数，如果目录下有多个main函数在go build时会报错，因此在目录整理时，应该把各个main函数都分配到不同的目录下
