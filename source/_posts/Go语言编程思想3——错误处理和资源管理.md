---
title: Go语言编程思想3——错误处理和资源管理
toc: true
date: 2022-01-15 01:02:47
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

# Go语言编程思想3——错误处理和资源管理
>资源管理：及时关闭文件，及时释放资源，如果打开的文件还未关闭就因为出错而在中间跳出，就无法保证有效的资源管理，因此在这里两者一起进行考虑
# 一、defer调用
- 调用在函数结束时发生（在return/panic之前执行）
- 参数在defer语句时计算
- defer列表为先进后出（先defer的后执行）
```go
func tryDefer() {
	for i := 0; i < 100; i++ {
		defer fmt.Print(i," ")
		if i == 30 {
			panic("printed too many")
		}
	}
}
//执行结果:30 29 28 27 26 25 ... 3 2 1 0
//退出的时候i是30，但不会全部输出30， i是在执行defer语句时的值
```


Ex. 将前20个斐波那契数列输出到文件
```go
func writeFile(filename string) {

	file, err := os.Create(filename) //新建文件
	if err != nil {
		panic(err)
	}
	defer file.Close()              //关闭文件
	writer := bufio.NewWriter(file) //新建bufio.Newwriter
	defer writer.Flush()            //writer需要Flush
	//先运行writer.Flush(),再运行file.close()
	f := fib.Fibonacci()
	for i := 0; i < 20; i++ {
		fmt.Fprintln(writer, f())
	}
}
```

**何时使用defer调用**
- Open/Close
- Lock/Unlock
- PrintHeader/PrintFooter

# 二、错误处理

- 尽量用error不用panic
- 意料之中的：使用error。如：文件打不开
- 意料之外的：使用panic。如：数组越界，如开了大小为n的数组，明明循环最大到n，但是结果越界，出现了意料之外的错误，这时用panic


error的定义
```go
type error interface {
	Error() string
}
```

将error当做普通的值类型来处理即可

panic会把程序挂掉，尽量少用panic，遇到错误时可以输出提示语句后return
```go
func writeFile(filename string) {
	// 注释表明，OpenFile如果出错那么一定是*PathError
	// 所以对Error进行判断，如果不是*PathError那么就报panic
	file, err := os.OpenFile(filename, os.O_EXCL|os.O_CREATE, 0666)
	if err != nil {
		if pathError, ok := err.(*os.PathError); !ok {
			panic(err)
		} else {
			fmt.Printf("%s,%s,%s\n",
				pathError.Op,
				pathError.Path,
				pathError.Err)
		}
		return
	}
	defer file.Close()              //关闭文件
	writer := bufio.NewWriter(file) //新建bufio.Newwriter
	defer writer.Flush()            //writer需要Flush
	//先运行writer.Flush(),再运行file.close()
	f := fib.Fibonacci()
	for i := 0; i < 20; i++ {
		fmt.Fprintln(writer, f())
	}
}
```

自建error
```go
	err=errors.New("this is a custom error")
```

## 服务器统一错误处理

```go
package filelisting

import (
	"io/ioutil"
	"net/http"
	"os"
	"strings"
)

const prefix = "/list/"

//字符串实现接口
type userError string

func (e userError) Error() string {
	return e.Message()
}
func (e userError) Message() string {
	return string(e)
}

func HandleFileList(writer http.ResponseWriter,
	request *http.Request) error {
	if strings.Index(request.URL.Path, prefix) != 0 {

		return userError("path must start " + "with " + prefix)
	}
	path := request.URL.Path[len(prefix):] // /list/fib.txt
	file, err := os.Open(path)
	if err != nil {
		return err
	}
	defer file.Close()

	all, err := ioutil.ReadAll(file)
	if err != nil {
		return err
	}
	writer.Write(all)
	return nil
}

```


```go
package main

import (
	"learngo/errhandling/filelistingserver/filelisting"
	"log"
	"net/http"
	"os"
)

type appHandler func(writer http.ResponseWriter,
	request *http.Request) error

//函数式编程，将输入的函数包装成输出函数来输出
func errWrapper(
	handler appHandler) func(
	http.ResponseWriter, *http.Request) {
	return func(writer http.ResponseWriter,
		request *http.Request) {

		defer func() {
			if r := recover(); r != nil {
				log.Printf("Panic:%v", r)
				http.Error(writer,
					http.StatusText(http.StatusInternalServerError),
					http.StatusInternalServerError)
			}
		}()

		err := handler(writer, request)
		//错误处理
		if err != nil {
			log.Printf("Error occurred"+
				"handling request:%s",
				err.Error())
			if userErr,ok:=err.(userError);ok{
				http.Error(writer,
					userErr.Message(),
					http.StatusBadRequest)
				return
			}


			code := http.StatusOK
			switch {
			case os.IsNotExist(err):
				code = http.StatusNotFound
			case os.IsPermission(err):
				code = http.StatusForbidden
			default:
				code = http.StatusInternalServerError
			}
			http.Error(writer,
				http.StatusText(code), code)
		}
	}
}

type userError interface {
	error //给系统看的
	Message() string  //给用户看的
}

func main() {
	http.HandleFunc("/",
		errWrapper(filelisting.HandleFileList),
	)
	err := http.ListenAndServe(":8888", nil)
	if err != nil {
		panic(err)
	}
}

```


