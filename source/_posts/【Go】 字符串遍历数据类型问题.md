---
title: 【Go】字符串遍历数据类型问题
toc: true
date: 2024-07-09 00:36:24
tags: golang 开发语言
categories: Golang
---

​点击阅读更多查看文章内容<!--more-->

## 字符串遍历问题


在使用for i,v:=range str遍历字符串时
- str[i]是unit8（byte）类型，返回的是单个字节
字符串在Go中是以字节序列的形式存储的，而 str[i] 直接访问了这个字节序列中的第 i 个字节。如果字符串中的字符是单字节的ASCII字符，那么 s[i] 就足以表示该字符。但是，如果字符是多字节的Unicode字符，那么 s[i] 就只是该字符的第一个字节，而不是整个字符。

- v是int32（rune）类型，返回的是字符的unicode编码
```go
func main() {
	str := "hello,world!你好，世界！"
	for i, _ := range str {
		fmt.Print(str[i], " ")
	}
	//104 101 108 108 111 44 119 111 114 108 100 33 228 229 239 228 231 239
	fmt.Println()
	for _, v := range str {
		fmt.Print(v, " ")
	}
	//104 101 108 108 111 44 119 111 114 108 100 33 20320 22909 65292 19990 30028 65281
}
```
