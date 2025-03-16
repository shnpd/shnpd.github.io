---
title: Go语言编程思想4——测试与性能调优
toc: true
date: 2022-01-25 12:22:54
tags: golang 开发语言 后端
categories: Golang
---

​​点击阅读更多查看文章内容<!--more-->

# Go语言编程思想4——测试与性能调优
>Debugging Sucks! Testing Rocks!
>多做测试，少做调试


Go语言使用**表格驱动测试** 
# 一、传统测试
正确结果在前，函数结果在后，判断是否相等

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f29b06fa1d7cafa136dc63e2b2b0d84a_1740930687223.png)
- 测试逻辑和测试数据混在一起
- 出错信息不明确
- 一旦一个数据出错测试全部结束

# 二、表格驱动测试

将测试数据写在struct中，a + b = c

卸载for循环中判断add(a,b)是否等于c，若不等于再进一步处理

**测试文件命名：测试的东西_test（若不按照标准命名则无法执行测试函数），本例文件命名为Triangle_test**

**测试函数命名需要为Test+测试名称**

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/3b78e6aab969a79e7102fd610ed36f95_1740930695031.png)
- 分离的测试数据和测试逻辑
- 明确的出错信息
- 可以部分失败
- go语言的语法使得我们更易实践表格驱动测试

```go

func calcTriangle(a, b int) int {
	var c int
	c = int(math.Sqrt(float64(a*a + b*b)))
	return c
}

func TestTriangle(t *testing.T) {
	tests := []struct{ a, b, c int }{
		{3, 4, 5},
		{5, 12, 13},
		{8, 15, 17},
		{12, 35, 37},
		{30000, 40000, 50000},
	}

	for _, tt := range tests {
		if actual := calcTriangle(tt.a, tt.b); actual != tt.c {
			t.Errorf("calcTriangle(%d,%d);"+
				"got %d;expected %d",
				tt.a, tt.b, actual, tt.c)
		}
	}
}
```


>在**命令行**中运行测试，进入test文件目录，运行`go test .`
>![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/5fc14b44fa27d10d0b8fc2e6d7697cd4_1740930695031.png)

# 三、代码覆盖率
查看测试代码的代码覆盖率

运行测试函数：run '...' with Coverage
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/30d4b0c68003b06210680ab5eb14fb56_1740930695031.png)

被测试的代码中，绿色的是覆盖到的，红色是没有覆盖到的，点击绿色部分可以看到覆盖了多少次
   
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/a4c80fd2a3e64b829c875799f7711c57_1740930695031.png)

>命令行
> `go test -coverprofile  c.out`
>`go tool cover -html  c.out`
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/2c25841838b6e9137b19d1b1b9feca18_1740930695031.png)

# 四、性能测试

在Triangle_test文件中继续创建下列函数

注意函数命名需要为Benchmark+测试名称

b.N为测试次数

```go
func BenchmarkTriangle(b *testing.B) {
	aa := 30000
	bb := 40000
	cc := 50000
	for i := 0; i < b.N; i++ {
		actual := calcTriangle(aa, bb)
		if actual != cc {
			b.Errorf("calcTriangle(%d,%d);"+
				"got %d;expected %d",
				aa, bb, actual, cc)
		}
	}
}
```
结果：![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ae518685dbbfef1d321aaf87b510f739_1740930702861.png)

>命令行 `go test -bench .`
>![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/fa6b545b88b55dbcefe73b328308333f_1740930702861.png)



# 五、pprof性能调优
**命令行命令：**
>1. `go test -bench . -cpuprofile cpu.out` 获得cpu性能的日志文件cpu.out
>2. `go tool pprof cpu.out` 查看日志文件
>3. 执行完pprof后会进入命令行，再执行`web`命令可以进入web页面可视化日志文件，这里需要先安装Graphviz，链接放在下面直接下载安装即可。
> [Graphviz](https://graphviz.org/download/)


**可视化结果：**
>方框越大，箭头越粗，耗时越长
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b9a62d4f0b778aa1f62c3ecd519f7396_1740930702861.png)


**优化map的方法**
>map是哈希表实现会有判重等操作
>可以使用空间换时间
>```a:=make([]int,0xffff)```
>假设中文字符最大就是0xFFFF，这里开一个0xFFFF大小的数组可以存储所有字符
>使用：a['e']=1（实质：a[0x65]=1）;a[‘课’]=1（实质：a[0x8BFE]=1）

**性能调优的步骤**
>1. -cpuprofile：获取性能数据
>2. go tool pprof：查看性能数据（web可视化）
>3. 分析慢在哪里（哪个框最大）
>4. 优化代码
>5. 再用-cpuprofile获取性能数据，查看优化结果，继续优化

# 六、http测试
两种方法
- 通过使用假的Request/Response
- 通过起服务器

>测试的函数参考之前笔记7中的服务器统一错误处理
```go

func errPanic(writer http.ResponseWriter,
	request *http.Request) error {
	panic(123)
}

type testingUserError string

func (e testingUserError) Error() string {
	return e.Message()
}
func (e testingUserError) Message() string {
	return string(e)
}

func errUserError(writer http.ResponseWriter,
	request *http.Request) error {
	return testingUserError("user error")
}

func errNotFound(writer http.ResponseWriter,
	request *http.Request) error {
	return os.ErrNotExist
}
func errNotPermission(writer http.ResponseWriter,
	request *http.Request) error {
	return os.ErrPermission
}
func errUnknown(writer http.ResponseWriter,
	request *http.Request) error {
	return errors.New("unknown error")
}
func noError(writer http.ResponseWriter,
	request *http.Request) error {
	fmt.Fprintln(writer, "no error")
	return nil
}
var tests = []struct {
	h       appHandler
	code    int
	message string
}{
	{errPanic, 500, "Internal Server Error"},
	{errUserError, 400, "user error"},
	{errNotFound, 404, "Not Found"},
	{errNotPermission, 403, "Forbidden"},
	{errUnknown, 500, "Internal Server Error"},
	{noError, 200, "no error"},
}

//用假的request，response，速度快
func TestErrWrapper(t *testing.T) {
	for _, tt := range tests {
		f := errWrapper(tt.h)
		response := httptest.NewRecorder()
		request := httptest.NewRequest(
			http.MethodGet, "http://www.imooc.com", nil)
		f(response, request)
		verifyResponse(response.Result(), tt.code, tt.message, t)
	}
}

//真正起一个server，测试力度更大
func TestErrWrapperInServer(t *testing.T) {
	for _, tt := range tests {
		f := errWrapper(tt.h)
		server := httptest.NewServer(http.HandlerFunc(f))
		resp, _ := http.Get(server.URL)
		verifyResponse(resp, tt.code, tt.message, t)
	}
}

func verifyResponse(resp *http.Response, expectedCode int, expectMsg string, t *testing.T) {
	b, _ := ioutil.ReadAll(resp.Body)
	body := strings.Trim(string(b), "\n")
	if resp.StatusCode != expectedCode || body != expectMsg {
		t.Errorf("expected(%d,%s);"+
			"got(%d,%s)",
			expectedCode, expectMsg,
			resp.StatusCode, body)
	}
}

```
# 七、生成文档
用注释写文档
在测试中加入Example
使用go doc/godoc 来查看/生成文档

```godoc -http :6060```生成文档，通过 http://localhost:6060 访问


**注释**

```go

// An FIFO queue.
type Queue []int

// Pushes the element into the queue.
//  e.g. q.Push(123)
func (q *Queue) Push(v int) {
	*q = append(*q, v)
}

// Pops element from head.
func (q *Queue) Pop() int {
	head := (*q)[0]
	*q = (*q)[1:]
	return head
}

// Returns if the queue is empty or not.
func (q *Queue) IsEmpty() bool {
	return len(*q)==0
}
```

**示例代码：**
也可以当做测试来做

函数命名为ExampleQueue_+函数名

注释为标准答案，先写Output，格式必须严格一致


 ```go
func ExampleQueue_Pop() {
	q := Queue{1}
	q.Push(2)
	q.Push(3)
	
	fmt.Println(q.Pop())
	fmt.Println(q.Pop())
	fmt.Println(q.IsEmpty())
	fmt.Println(q.Pop())
	fmt.Println(q.IsEmpty())

	// Output:
	// 1
	// 2
	// false
	// 3
	// true
}
```

生成文档：

![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/cd4791df6f4a5b2c2d563b3e7c9e9415_1740930702861.png)
>Example可以看作是特别的test，可以执行函数进行测试（注释为想要得到的结果），同时也可以生成文档的example
>若将注释中的false删掉e，执行函数得到以下结果
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/845e93a9e16798e49433688e678504c4_1740930702861.png)
