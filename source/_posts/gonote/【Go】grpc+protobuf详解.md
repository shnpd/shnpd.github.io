---
title: 【Go】grpc+protobuf详解
toc: true
date: 2025-03-03 00:00:00
tags: 
  - go
categories: 
  - Golang

---

点击阅读更多查看文章内容<!--more-->

## Protobuf详解

官方地址： https://developers.google.com/protocol-buffers/docs/proto3

数据类型

| .proto Type | Notes                                                        | Go Type |
| :---------- | :----------------------------------------------------------- | :------ |
| double      |                                                              | float64 |
| float       |                                                              | float32 |
| int32       | 使用变长编码，对于负值的效率很低，如果你的域有可能有负值，请使用sint64替代 | int32   |
| uint32      | 使用变长编码                                                 | uint32  |
| uint64      | 使用变长编码                                                 | uint64  |
| sint32      | 使用变长编码，这些编码在负值时比int32高效的多                | int32   |
| sint64      | 使用变长编码，有符号的整型值。编码时比通常的int64高效。      | int64   |
| fixed32     | 总是4个字节，如果数值总是比总是比228大的话，这个类型会比uint32高效。 | uint32  |
| fixed64     | 总是8个字节，如果数值总是比总是比256大的话，这个类型会比uint64高效。 | uint64  |
| sfixed32    | 总是4个字节                                                  | int32   |
| sfixed64    | 总是8个字节                                                  | int64   |
| bool        |                                                              | bool    |
| string      | 一个字符串必须是UTF-8编码或者7-bit ASCII编码的文本。         | string  |
| bytes       | 可能包含任意顺序的字节数据。                                 |         |

---

### **默认值**

当一个消息被解析的时候，如果被编码的信息不包含一个特定的singular元素，被解析的对象所对应的域被设置位一个默认值，对于不同类型指定如下：

- 对于strings，默认是一个空string
- 对于bytes，默认是一个空的bytes
- 对于bools，默认是false
- 对于数值类型，默认是0
- 对于枚举，默认是第一个定义的枚举值，必须为0;

---

### **option go_package**

指明生成文件的目录以及包名，option go_package=“.;proto”，在当前目录下生成文件，文件package为proto，使用go_package后无需为proto文件再单独定义package

---

### **编号问题**

客户端定义

```protobuf
message Data{
	string name = 1;
	string url = 2;
}
```

服务端定义

```protobuf
message Data{
	string name = 2;
	string url = 1;
}
```

如果两个文件的序号不对应那么最终的结果是，客户端发送的name在服务端被解析为url，客户端发送的url在服务端被解析为name

>  假设name为tom，url为tom.com，那么实际传输的数据类似1(序号)3(长度)tom27tom.com

---

### **import其它的proto文件**

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024222.png" alt="image-20250204230746070" style="zoom: 80%;" />

![image-20250204230825975](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024223.png)

自定义文件在代码中使用时需要生成import的proto的代码（两个proto文件都在同一目录下直接import一个package即可）

内置文件需要import文件中的go_package

empty.proto

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024224.png" alt="image-20250204234153497" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024225.png" alt="image-20250204234215359" style="zoom:67%;" />

---

### **嵌套的message对象**

message只放在需要它的message里面，避免公共message的爆炸式增长

```protobuf
message HelloReply {
    string message = 1;

    message Result {
        string name = 1;
        string url = 2;
    }

    repeated Result data = 2;
}
```

生成代码调用时可以通过 `proto.HelloReply_Result` 对其进行实例化

---

### **枚举类型**

proto文件定义

```protobuf
enum Gender{
    MALE = 0;
    FEMALE = 1;
}
```

生成代码使用：`proto_bak.Gender_MALE`

生成的Gender是int32类型，对每个值都会生成对应的常量

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024226.png" alt="image-20250204232609929" style="zoom:67%;" />

---

### **map类型**

proto文件定义：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024227.png" alt="image-20250204232920851" style="zoom:67%;" />

生成代码：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024228.png" alt="image-20250204232947785" style="zoom:67%;" />

使用：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024229.png" alt="image-20250204233003733" style="zoom:67%;" />



---

### **内置timestamp类型**

proto文件定义：

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024230.png" alt="image-20250204233944576" style="zoom:67%;" />

---

## grpc详解

## 什么是rpc

实现rpc调用主要解决三个问题：

1. **Call ID映射**。我们怎么告诉远程机器我们要调用add，而不是sub或者Foo呢？在本地调用中，函数体是直接通过函数指针来指定的，我们调用add，编译器就自动帮我们调用它相应的函数指针。但是在远程调用中，函数指针是不行的，因为两个进程的地址空间是完全不一样的。所以，在RPC中，所有的函数都必须有自己的一个ID。这个ID在所有进程中都是唯一确定的。客户端在做远程过程调用时，必须附上这个ID。然后我们还需要在客户端和服务端分别维护一个 {函数 <–> Call ID} 的对应表。两者的表不一定需要完全相同，但相同的函数对应的Call ID必须相同。当客户端需要进行远程调用时，它就查一下这个表，找出相应的Call ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。
2. **序列化和反序列化**。客户端怎么把参数值传给远程的函数呢？在本地调用中，我们只需要把参数压到栈里，然后让函数自己去栈里读就行。但是在远程过程调用时，客户端跟服务端是不同的进程，不能通过内存来传递参数。甚至有时候客户端和服务端使用的都不是同一种语言（比如服务端用C++，客户端用Java或者Python）。这时候就需要客户端把参数先转成一个字节流，传给服务端后，再把字节流转成自己能读取的格式。这个过程叫序列化和反序列化。同理，从服务端返回的值也需要序列化反序列化的过程。
3. **网络传输**。远程调用往往用在网络上，客户端和服务端是通过网络连接的。所有的数据都需要通过网络传输，因此就需要有一个网络传输层。网络传输层需要把Call ID和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。只要能完成这两者的，都可以作为传输层使用。因此，它所使用的协议其实是不限的，能完成传输就行。尽管大部分RPC框架都使用TCP协议，但其实UDP也可以，而gRPC干脆就用了HTTP2。Java的Netty也属于这层的东西。

#### rpc开发的四大要素

RPC技术在架构设计上有四部分组成，分别是：**客户端、客户端存根、服务端、服务端存根。**

> **通过stub屏蔽掉网络通信的内容，使得在客户端调用远程的函数就跟调用本地函数一样**

- **客户端(Client)：**服务调用发起方，也称为服务消费者。
- **客户端存根(Client Stub)：**该程序运行在客户端所在的计算机机器上，主要用来存储要调用的服务器的地址，另外，该程序还负责将客户端请求远端服务器程序的数据信息打包成数据包，通过网络发送给服务端Stub程序；其次，还要接收服务端Stub程序发送的调用结果数据包，并解析返回给客户端。
- **服务端(Server)：**远端的计算机机器上运行的程序，其中有客户端要调用的方法。
- **服务端存根(Server Stub)：**接收客户Stub程序通过网络发送的请求消息数据包，并调用服务端中真正的程序功能方法，完成功能调用；其次，将服务端执行调用的结果进行数据处理打包发送给客户端Stub程序。

了解完了RPC技术的组成结构我们来看一下具体是如何实现客户端到服务端的调用的。实际上，如果我们想要在网络中的任意两台计算机上实现远程调用过程，要解决很多问题，比如：

- 两台物理机器在网络中要建立稳定可靠的通信连接。
- 两台服务器的通信协议的定义问题，即两台服务器上的程序如何识别对方的请求和返回结果。也就是说两台计算机必须都能够识别对方发来的信息，并且能够识别出其中的请求含义和返回含义，然后才能进行处理。这其实就是通信协议所要完成的工作。

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125810860.png" alt="image-20250130232829575" style="zoom:67%;" />

在上述图中，通过1-10的步骤图解的形式，说明了RPC每一步的调用过程。具体描述为：

- 1、客户端想要发起一个远程过程调用，首先通过调用本地客户端Stub程序的方式调用想要使用的功能方法名；
- 2、客户端Stub程序接收到了客户端的功能调用请求，**将客户端请求调用的方法名，携带的参数等信息做序列化操作，并打包成数据包。**
- 3、客户端Stub查找到远程服务器程序的IP地址，调用Socket通信协议，通过网络发送给服务端。
- 4、服务端Stub程序接收到客户端发送的数据包信息，并**通过约定好的协议将数据进行反序列化，得到请求的方法名和请求参数等信息。**
- 5、服务端Stub程序准备相关数据，**调用本地Server对应的功能方法进行，并传入相应的参数，进行业务处理。**
- 6、服务端程序根据已有业务逻辑执行调用过程，待业务执行结束，将执行结果返回给服务端Stub程序。
- 7、服务端Stub程序**将程序调用结果按照约定的协议进行序列化，**并通过网络发送回客户端Stub程序。
- 8、客户端Stub程序接收到服务端Stub发送的返回数据，**对数据进行反序列化操作，**并将调用返回的数据传递给客户端请求发起者。
- 9、客户端请求发起者得到调用结果，整个RPC调用过程结束。

## grpc

gRPC 是一个高性能、开源和通用的 RPC 框架，面向移动和 HTTP/2 设计。目前提供 C、Java 和 Go 语言版本，分别是：[grpc](https://github.com/grpc/grpc), [grpc-java](https://github.com/grpc/grpc-java), [grpc-go](https://github.com/grpc/grpc-go). 其中 C 版本支持 [C](https://github.com/grpc/grpc), [C++](https://github.com/grpc/grpc/tree/master/src/cpp), [Node.js](https://github.com/grpc/grpc/tree/master/src/node), [Python](https://github.com/grpc/grpc/tree/master/src/python), [Ruby](https://github.com/grpc/grpc/tree/master/src/ruby), [Objective-C](https://github.com/grpc/grpc/tree/master/src/objective-c), [PHP](https://github.com/grpc/grpc/tree/master/src/php) 和 [C#](https://github.com/grpc/grpc/tree/master/src/csharp) 支持.

[grpc项目地址](https://github.com/grpc/grpc)

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125810861.png" alt="image-20250201183151401" style="zoom:67%;" />

### grpc的四种数据流

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125810863.png" alt="image-20250201234141191" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125810864.png" alt="image-20250201234152361" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125810865.png" alt="image-20250201234207109" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303125810866.png" alt="image-20250201234221126" style="zoom:67%;" />

### proto

```protobuf
syntax = "proto3";//声明proto的版本 只能 是3，才支持 grpc

//声明 包名
option go_package=".;proto";

//声明grpc服务
service Greeter {
    /*
    以下 分别是 服务端 推送流， 客户端 推送流 ，双向流。
    */
    rpc GetStream (StreamReqData) returns (stream StreamResData){}
    rpc PutStream (stream StreamReqData) returns (StreamResData){}
    rpc AllStream (stream StreamReqData) returns (stream StreamResData){}
}


//stream请求结构
message StreamReqData {
    string data = 1;
}
//stream返回结构
message StreamResData {
    string data = 1;
}
```



### 服务端

```go
package main

import (
	"fmt"
	"google.golang.org/grpc"
	"log"
	"net"
	"start/new_stream/proto"
	"sync"
	"time"
)

const PORT  = ":50052"

type server struct {
}

//服务端 单向流
func (s *server)GetStream(req *proto.StreamReqData, res proto.Greeter_GetStreamServer) error{
	i:= 0
	for{
		i++
		res.Send(&proto.StreamResData{Data:fmt.Sprintf("%v",time.Now().Unix())})
		time.Sleep(1*time.Second)
		if i >10 {
			break
		}
	}
	return nil
}

//客户端 单向流
func (s *server) PutStream(cliStr proto.Greeter_PutStreamServer) error {

	for {
		if tem, err := cliStr.Recv(); err == nil {
			log.Println(tem)
		} else {
			log.Println("break, err :", err)
			break
		}
	}

	return nil
}

//客户端服务端 双向流
func(s *server) AllStream(allStr proto.Greeter_AllStreamServer) error {

	wg := sync.WaitGroup{}
	wg.Add(2)
	go func() {
		for {
			data, _ := allStr.Recv()
			log.Println(data)
		}
		wg.Done()
	}()

	go func() {
		for {
			allStr.Send(&proto.StreamResData{Data:"ssss"})
			time.Sleep(time.Second)
		}
		wg.Done()
	}()

	wg.Wait()
	return nil
}

func main(){
	//监听端口
	lis,err := net.Listen("tcp",PORT)
	if err != nil{
		panic(err)
		return
	}
	//创建一个grpc 服务器
	s := grpc.NewServer()
	//注册事件
	proto.RegisterGreeterServer(s,&server{})
	//处理链接
	err = s.Serve(lis)
	if err != nil {
		panic(err)
	}
}
```



### 客户端

```go
package main

import (
	"google.golang.org/grpc"

	"context"
	_ "google.golang.org/grpc/balancer/grpclb"
	"log"
	"start/new_stream/proto"
	"time"
)

const (
	ADDRESS = "localhost:50052"
)


func main(){
	//通过grpc 库 建立一个连接
	conn ,err := grpc.Dial(ADDRESS,grpc.WithInsecure())
	if err != nil{
		return
	}
	defer conn.Close()
	//通过刚刚的连接 生成一个client对象。
	c := proto.NewGreeterClient(conn)
	//调用服务端推送流
	reqstreamData := &proto.StreamReqData{Data:"aaa"}
	res,_ := c.GetStream(context.Background(),reqstreamData)
	for {
		aa,err := res.Recv()
		if err != nil {
			log.Println(err)
			break
		}
		log.Println(aa)
	}
	//客户端 推送 流
	putRes, _ := c.PutStream(context.Background())
	i := 1
	for {
		i++
		putRes.Send(&proto.StreamReqData{Data:"ss"})
		time.Sleep(time.Second)
		if i > 10 {
			break
		}
	}
	//服务端 客户端 双向流
	allStr,_ := c.AllStream(context.Background())
	go func() {
		for {
			data,_ := allStr.Recv()
			log.Println(data)
		}
	}()

	go func() {
		for {
			allStr.Send(&proto.StreamReqData{Data:"ssss"})
			time.Sleep(time.Second)
		}
	}()

	select {
	}

}
```



### **grpc的metadata机制**

grpc中的metadata类似于http中的header，可以用来存放一些元信息（如token）

gRPC让我们可以像本地调用一样实现远程调用，对于每一次的RPC调用中，都可能会有一些有用的数据，而这些数据就可以通过metadata来传递。metadata是以key-value的形式存储数据的，其中key是`string`类型，而value是`[]string`，即一个字符串数组类型。metadata使得client和server能够为对方提供关于本次调用的一些信息，metadata的生命周期就是一次RPC调用。

**新建metadata**

MD 类型实际上是map，key是string，value是string类型的slice。

```go
type MD map[string][]string
```

创建的时候可以像创建普通的map类型一样使用new关键字进行创建：

```go
//第一种方式
md := metadata.New(map[string]string{"key1": "val1", "key2": "val2"})
//第二种方式 key不区分大小写，会被统一转成小写。
md := metadata.Pairs(
    "key1", "val1",
    "key1", "val1-2", // "key1" will have map value []string{"val1", "val1-2"}
    "key2", "val2",
)
```

 **发送metadata**：将metadata添加到context中

```go
md := metadata.Pairs("key", "val")

// 新建一个有 metadata 的 context
ctx := metadata.NewOutgoingContext(context.Background(), md)

// 单向 RPC
response, err := client.SomeRPC(ctx, someRequest)
```

**接收metadata**：从context中取出metadata

```go
func (s *server) SomeRPC(ctx context.Context, in *pb.SomeRequest) (*pb.SomeResponse, err) {
    md, ok := metadata.FromIncomingContext(ctx)
    // do something with metadata
}
```

---

### **grpc拦截器**

gRPC 拦截器主要分为两种：客户端拦截器（ClientInterceptor），服务端拦截器（ServerInterceptor），顾名思义，分别于请求的两端执行相应的前拦截处理。

- 客户端拦截器

1、作用时机？

请求被分发出去之前。

2、可以做什么？

a)、请求日志记录及监控

b)、添加请求头数据、以便代理转发使用

c)、请求或者结果重写

3、实现

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"time"

	"start/grpc_interceptor/proto"
)

func interceptor(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
	start := time.Now()
	err := invoker(ctx, method, req, reply, cc, opts...)
	fmt.Printf("method=%s req=%v rep=%v duration=%s error=%v\n", method, req, reply, time.Since(start), err)
	return err
}

func main(){
	//stream
	var opts []grpc.DialOption

	opts = append(opts, grpc.WithInsecure())
	// 指定客户端interceptor
	opts = append(opts, grpc.WithUnaryInterceptor(interceptor))

	conn, err := grpc.Dial("localhost:50051", opts...)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)
	r, err := c.SayHello(context.Background(), &proto.HelloRequest{Name:"bobby"})
	if err != nil {
		panic(err)
	}
	fmt.Println(r.Message)
}
```

- 服务端拦截器

1、作用时机？

请求被具体的Handler相应前。

2、可以做什么？

a）访问认证

b）请求日志记录及监控

c）代理转发

3、实现

```go
package main

import (
	"context"
	"fmt"
	"net"

	"google.golang.org/grpc"

	"start/grpc_interceptor/proto"
)


type Server struct{}

func (s *Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply,
	error){
	return &proto.HelloReply{
		Message: "hello "+request.Name,
	}, nil
}


func main(){
	var interceptor grpc.UnaryServerInterceptor
	interceptor = func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
		// 继续处理请求
		fmt.Println("接收到新请求")
		res, err := handler(ctx, req)
		fmt.Println("请求处理完成")
		return res, err
	}
	var opts []grpc.ServerOption
	opts = append(opts, grpc.UnaryInterceptor(interceptor))

	g := grpc.NewServer(opts...)
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:50051")
	if err != nil{
		panic("failed to listen:"+err.Error())
	}
	err = g.Serve(lis)
	if err != nil{
		panic("failed to start grpc:"+err.Error())
	}
}
```

 拦截器的应用场景：[go-grpc-middleware](https://github.com/grpc-ecosystem/go-grpc-middleware)

---

### **通过拦截器和metadata实现grpc的auth认证**

客户端：

```go
package  main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"start/token_interceptor/proto"
)

type customCredential struct{}

func (c customCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{
		"appid":  "101010",
		"appkey": "i am key",
	}, nil
}

func (c customCredential) RequireTransportSecurity() bool {
	return false
}


func main() {
	var opts []grpc.DialOption

	//opts = append(opts, grpc.WithUnaryInterceptor(interceptor))
	opts = append(opts, grpc.WithInsecure())
	opts = append(opts, grpc.WithPerRPCCredentials(new(customCredential)))
	// 指定客户端interceptor

	conn, err := grpc.Dial("localhost:50051", opts...)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)
	//rsp, _ := c.Search(context.Background(), &empty.Empty{})
	rsp, err := c.SayHello(context.Background(), &proto.HelloRequest{
		Name: "bobby",

	})
	if err != nil {
		panic(err)
	}
	fmt.Println(rsp.Message)
}
```

服务端

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
	"net"

	"google.golang.org/grpc"

	"start/token_interceptor/proto"
)


type Server struct{}

func (s *Server) SayHello(ctx context.Context, request *proto.HelloRequest) (*proto.HelloReply,
	error){
	return &proto.HelloReply{
		Message: "hello "+request.Name,
	}, nil
}


func main(){
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	var interceptor grpc.UnaryServerInterceptor
	interceptor = func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
		md, ok := metadata.FromIncomingContext(ctx)
		if !ok {
			return resp, status.Errorf(codes.Unauthenticated, "无Token认证信息")
		}

		var (
			appid  string
			appkey string
		)

		if val, ok := md["appid"]; ok {
			appid = val[0]
		}

		if val, ok := md["appkey"]; ok {
			appkey = val[0]
		}

		if appid != "imooc" || appkey != "bobby" {
			return resp, status.Errorf(codes.Unauthenticated, "Token认证信息无效: appid=%s, appkey=%s", appid, appkey)
		}

		// 继续处理请求
		return handler(ctx, req)
	}
	var opts []grpc.ServerOption
	opts = append(opts, grpc.UnaryInterceptor(interceptor))

	s := grpc.NewServer(opts...)
	ser :=& Server{}
	proto.RegisterGreeterServer(s, ser)
	s.Serve(lis)
}
```

---

### **grpc的验证器**

[protoc-gen_validate](https://github.com/envoyproxy/protoc-gen-validate)，验证proto中定义的message是否符合某个规则

生成源码

```shell
protoc -I .  --go_out=plugins=grpc:. --validate_out="lang=go:." helloworld.proto
```

1. 新建validate.proto文件内容从 https://github.com/envoyproxy/protoc-gen-validate/blob/master/validate/validate.proto 拷贝
2. 新建helloworl.proto文件

```protobuf
syntax = "proto3";

import "validate.proto";
option go_package=".;proto";

service Greeter {
    rpc SayHello (Person) returns (Person);
}

message Person {
    uint64 id    = 1 [(validate.rules).uint64.gt    = 999];

    string email = 2 [(validate.rules).string.email = true];
    string name  = 3 [(validate.rules).string = {
                      pattern:   "^[^[0-9]A-Za-z]+( [^[0-9]A-Za-z]+)*$",max_bytes: 256,}];

}
```

客户端：

```go
package  main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"start/pgv_test/proto"
)

type customCredential struct{}


func main() {
	var opts []grpc.DialOption

	//opts = append(opts, grpc.WithUnaryInterceptor(interceptor))
	opts = append(opts, grpc.WithInsecure())

	conn, err := grpc.Dial("localhost:50051", opts...)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	c := proto.NewGreeterClient(conn)
	//rsp, _ := c.Search(context.Background(), &empty.Empty{})
	rsp, err := c.SayHello(context.Background(), &proto.Person{
		Email: "bobby",
	})
	if err != nil {
		panic(err)
	}
	fmt.Println(rsp.Id)
}
```

服务端：通过validate方法就可以验证规则，不符合则返回error，直接定义一个包含validate方法的接口，validate方法实现在生成的validate文件中

```go
package main

import (
	"context"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"net"

	"google.golang.org/grpc"

	"start/pgv_test/proto"
)


type Server struct{}

func (s *Server) SayHello(ctx context.Context, request *proto.Person) (*proto.Person,
	error){
	return &proto.Person{
		Id: 32,
	}, nil
}

type Validator interface {
	Validate() error
}

func main(){
	var interceptor grpc.UnaryServerInterceptor
	interceptor = func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
		// 继续处理请求
		if r, ok := req.(Validator); ok {
			if err := r.Validate(); err != nil {
				return nil, status.Error(codes.InvalidArgument, err.Error())
			}
		}

		return handler(ctx, req)
	}
	var opts []grpc.ServerOption
	opts = append(opts, grpc.UnaryInterceptor(interceptor))

	g := grpc.NewServer(opts...)
	proto.RegisterGreeterServer(g, &Server{})
	lis, err := net.Listen("tcp", "0.0.0.0:50051")
	if err != nil{
		panic("failed to listen:"+err.Error())
	}
	err = g.Serve(lis)
	if err != nil{
		panic("failed to start grpc:"+err.Error())
	}
}
```

---

### **grpc的状态码**

https://github.com/grpc/grpc/blob/master/doc/statuscodes.md

---

### **grpc中的错误处理**

1. 服务端生成错误信息

```go
err :=  status.Errorf(codes.NotFound, "记录未找到：%s", request.Name)
```

2. 客户端获取错误中的状态信息

```go
st, ok := status.FromError(err)
if !ok {
    // Error was not a status error
}
st.Message()
st.Code()
```

---

### **grpc中的超时机制**

在客户端设置超时时间3s

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024231.png" alt="image-20250205233201931" style="zoom: 50%;" />

服务端等待10s

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024232.png" alt="image-20250205233221474" style="zoom:50%;" />

此时，客户端在发出请求3s后会直接返回错误

---

### **protoc生成的go的源码里面有什么？**

proto文件：

```protobuf
syntax = "proto3";
option go_package = ".;proto";
service Greeter {
    rpc SayHello (HelloRequest) returns (HelloReply);
}
message HelloRequest {
    string name = 1;
}
message HelloReply {
    string message = 1;
}
```

proto 中的 message 会生成 go 文件中的 struct

![image-20250205233739011](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024233.png)

![image-20250205233817761](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024234.png)

**服务端：**

service 生成了 server 的接口

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024235.png" alt="image-20250205233644501" style="zoom:67%;" />

生成了注册方法

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024236.png" alt="image-20250205233915970" style="zoom: 67%;" />

服务端调用时，实现对应的业务逻辑（SayHello）然后直接将实现业务逻辑的结构体注册即可

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024237.png" alt="image-20250205235101434" style="zoom: 50%;" />

**客户端：**

生成了client接口（与服务端参数不同）

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024238.png" alt="image-20250205234134892" style="zoom: 50%;" />

通过new方法（类似于其它语言中的构造函数）生成接口，new返回的是greeterClient进行了一层包装将cc包装进来

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024239.png" alt="image-20250205234512046" style="zoom:67%;" />

cc包装了调用的方法Invoke

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024241.png" alt="image-20250205234636412" style="zoom:67%;" />

greeterClient通过cc.Invoke实现了SayHello方法的调用

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024242.png" alt="image-20250205234659177" style="zoom:67%;" />

cc就是我们在客户端调用时生成的conn

<img src="https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/20250303130024243.png" alt="image-20250205234906535" style="zoom:67%;" />
