

+++

title = "go.测试"
description = "go.测试"
tags = ["it", "go"]

+++



# 测试

Debugging Sucks !  Testing Rocks !

```go
/*单元测试文件名必须以 _test.go 为后缀，前缀一般为要测试的文件名*/
func TestMain(m *testing.M) { //TestMain 在测试函数执行前后执行其它内容，类似AOP。TestMain(m *testing.M)为固定写法
	fmt.Println("before m.Run()")
	m.Run() //执行的测试函数
	fmt.Println("after m.Run()")
}

func TestXxx(t *testing.T) { //单元测试函数名必须以 Test 为前缀
	t.Run("TestXxx Run testXxx", testXxx) //执行子测试程序
	//testXxx(t) //也可以直接调用
}

func testXxx(t *testing.T) { //不以Test开头无法执行，但可以通过*testing.T的Run方法执行
	fmt.Println("testXxx")
}
```

- 注意：测试方法与被测试方法在一个目录中时，`go testdemo_test.go`或者goland点击的运行只会运行测试方法所在文件，所以会找不到被测试方法
  - 可以通过命令执行进行测试，`go test .`会带上目录内的所有文件、`go test -v demo_test.go demo.go`带上被测试文件，-v可以使运行结果被看到
  - 在goland中可以将测试文件单独放到一个目录，通过包名调用被测试方法，因为有包导入，可以成功运行
  - `go clean -n`查看`go clean`清理内容，`go clean`清理所有缓存，`go clean cache`清理一般缓存，`go clean testcache`清理test缓存

## 表格驱动测试

```go
/*表格驱动测试：数据与测试过程分离，*/
func TestTriangle(t *testing.T) {
	tests := []struct{ a, b, c int }{ //{测试数据..., 期望结果...}
		//一般数据
		{3, 4, 5},
		{5, 12, 13},
		{7, 24, 25},
		{9, 40, 41},

		//特殊数据
		{30000, 40000, 50000},
	}
	for _, tt := range tests {
		if actual := CalcTriangle(tt.a, tt.b); actual != tt.c {
			t.Errorf("calcTriangle(%d, %d); got %d; expected %d", tt.a, tt.b, actual, tt.c)
		}
	}
}

func CalcTriangle(a, b int) int {
	c := int(math.Sqrt(float64(a*a + b*b)))
	return c
}
```

## 代码覆盖率

命令行也可以运行：go  test可以直接运行当前目录下的测试文件

go test -coverprofile=c.out：执行测试，测试顺利同通过则生成名为c.out的代码覆盖率文件，直接打开看不懂

go tool cover -html=c.out：在html打开，绿色部分是测试到的内容，红色部分是未测试到内容

## 性能测试

```go
/*go test -bench：性能测试。可以看到平均每次循环的执行时间
go test -bench . -cpuprofile cpu.out：
go tool pprof cpu.out：出现交互式命令
web：在网页显示，是一张cpu执行时间分布图。需要安装www.graphviz.org*/
func BenchmarkSubstr(b *testing.B) {
	test := []int{3, 4, 5}
	b.ResetTimer() //重置计时器，以使此前一些准备性质的内容不作计算
	for i := 0; i < b.N; i++ { //循环次数由b.N决定
		if actual := CalcTriangle(test[0], test[1]); actual != test[2] {
			b.Errorf("calcTriangle(%d, %d); got %d; expected %d", test[0], test[1], actual, test[2])
		}
	}
}

func CalcTriangle(a, b int) int {
	c := int(math.Sqrt(float64(a*a + b*b)))
	return c
}
```



## HttpServer测试

```go
/*测试数据*/
var tests = []struct {
	//输入的参数
	h appHandler

	//返回的结果
	code    int
	message string
}{
	{noError, 200, "no error"},
	{errPanic, 500, "Internal Server Error"}, //不知道的结果，可以先随便写，通过首次执行测试来获取
	{errUserError, 400, "user error"},
	{errNotFound, 404, "Not Found"},
}

func TestErrWrapper(t *testing.T) {
	for _, tt := range tests {
		f := ErrWrapper(tt.h) //要测试ErrWrapper，但其返回的是一个函数，则测试该返回的函数能否正确执行
		//手动创建的response和request实例
		response := httptest.NewRecorder()
		request := httptest.NewRequest(http.MethodGet, "https://www.baidu.com", nil)
		f(response, request)
		verifyResponse(response.Result(), tt.code, tt.message, t)
	}
}

func TestErrWrapperInServer(t *testing.T) {
	for _, tt := range tests {
		f := ErrWrapper(tt.h)
		server := httptest.NewServer(http.HandlerFunc(f)) //启动了一个真的httpServer
		response, _ := http.Get(server.URL)               //发送一个真的httpRequest
		verifyResponse(response, tt.code, tt.message, t)
	}
}

func verifyResponse(response *http.Response, expectedCode int, expectedMessage string, t *testing.T) {
	body, _ := ioutil.ReadAll(response.Body)
	bodyStr := strings.Trim(string(body), "\n")
	if response.StatusCode != expectedCode || bodyStr != expectedMessage {
		t.Errorf("expect(%d,%s); got(%d, %s)", expectedCode, expectedMessage, response.StatusCode, bodyStr)
	}
}

func errPanic(writer http.ResponseWriter, request *http.Request) error {
	panic(123)
}

type testingUserError string

func (e testingUserError) Error() string {
	return e.Message()
}
func (e testingUserError) Message() string {
	return string(e)
}

func errUserError(writer http.ResponseWriter, request *http.Request) error {
	return testingUserError("user error")
}

func errNotFound(writer http.ResponseWriter, request *http.Request) error {
	return os.ErrNotExist
}

/*...*/

func noError(writer http.ResponseWriter, request *http.Request) error {
	fmt.Fprintln(writer, "no error")
	return nil
}
```



