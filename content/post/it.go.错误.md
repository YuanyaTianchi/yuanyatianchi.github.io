

+++

title = "go.错误"
description = "go.错误"
tags = ["it", "go"]

+++



## 错误

### error

Go 语言通过内置的错误接口提供了非常简单的错误处理机制

```go
/*error的定义*/
type error interface {
	Error() string
}
/*error创建*/
func main() {
	error1 := errors.New("error1Message")
	fmt.Printf("%T\n", error1) //*errors.errorString
	fmt.Println(error1) //error1Message

	error2 := fmt.Errorf("%s", "error2Message")
	fmt.Printf("%T\n", error2) //*errors.errorString
	fmt.Println(error2) //error2Message
}
```



#### 错误处理方式

```go
func ErrorHandle() {
	file, err := os.OpenFile("./demo.txt", os.O_RDONLY, os.ModePerm)
	
	/*一般采用先输出后return的方式*/
	if err != nil {
		fmt.Println("Error:", err) //打印arr实际上就是打印err.Error()
		return
	}
	if err != nil {
		//os.OpenFile注释最后一句If there is an error, it will be of type *PathError.如果出错，应该是一个*PathError
		pathError, ok := err.(*os.PathError) //类型动态转换/查询。只有对接口对象才能执行
		if ok {
			fmt.Println(pathError.Op, pathError.Path, pathError.Err) //对于知道确切类型的Error，可以选择针对处理
		} else {
			panic(err) //panic，简单粗暴，只是报错不太好看
			//os.Exit(1) //os退出程序
			//log.Fatal(err)//log打印并结束程序
		}
		return
	}
	
	defer file.Close()
}
```

##### 统一错误处理

```go
func main() {
	http.HandleFunc("/", errWrapper(HandleFileList))
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		panic(err)
	}
}

type userError interface { //定义一个可以给用户看到的error
	error
	Message() string
}

type appHandler func(writer http.ResponseWriter, request *http.Request) error

/*包装handler(把错误统一处理掉),返回一个无错的handler*/
func errWrapper(handler appHandler) func(http.ResponseWriter, *http.Request) {
	return func(writer http.ResponseWriter, request *http.Request) {
        
		defer func() { //我们自己写一个recover,就不会需要被httpServer去recover了
			if r := recover(); r != nil {
				log.Printf("Panic: %v\n", r)
				http.Error(writer, http.StatusText(http.StatusInternalServerError), http.StatusInternalServerError)
			}
		}()
        
		err := handler(writer, request)
		if err != nil {
			log.Println(err.Error())

			if userErr, ok := err.(userError); ok {
				http.Error(writer, userErr.Message(), http.StatusBadRequest)
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
			http.Error(writer, http.StatusText(code), code)
		}
	}
}

type userError2 string

func (e userError2) Error() string {
	return e.Message()
}
func (e userError2) Message() string {
	return string(e)
}

/*所有错误都返回,只做对的事情*/
func HandleFileList(writer http.ResponseWriter, request *http.Request) error {
	const prefix = "/file"
	if strings.Index(request.URL.Path, prefix) != 0 {
		return userError2("path must start with " + prefix) //这是可以给用户看的信息
	}
    
	path := request.URL.Path[len(prefix):]
	file, err := os.Open("." + path)
	if err != nil {
		/*恐慌,但是http.server中panic是被recover()了的,不会down掉程序
		panic(err)*/
		/*err.Error()的内容将写入响应体.但是我们不应该给用户显示内部的错误
		http.Error(writer, err.Error(), http.StatusInternalServerError)
		return*/
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



#### 自定义错误

在编码中通过实现 error 接口类型来生成错误信息

```go
/*自定义异常*/
type DivideError struct { //自定义除零异常
	dividend int //被除数
	divisor int //除数
}

func NewDivideError(dividend int, divisor int) *DivideError { //DivideError构造方法
	return &DivideError{dividend: dividend, divisor: divisor}
}

func (divideErrorPtr *DivideError) Error() string { // 实现 error 接口
	message := `divisor is zero. dividend: %d, divisor: %d`
	return fmt.Sprintf(message, divideErrorPtr.dividend, divideErrorPtr.divisor) //Sprintf组合模板字符串与变量
}

func main() {
	result, err := Divide(9, 0)
	if err!=nil {
		fmt.Println("error：", err) //error： divisor is zero. dividend: 9, divisor: 0
	} else {
		fmt.Println("result = ", result)
	}
}

func Divide(dividend int, divisor int) (result int, error error) { //除法函数
	error = nil
	if divisor == 0 {
		error = NewDivideError(dividend, divisor)
	} else {
		result = dividend / divisor
	}
	return
}
```

### panic

- panic：
  - 停止当前函数执行
  - 一直向上返回，执行每一层的defer
  - 如果没有遇见recover，程序退出

- recover
  - 仅在defer调用中使用
  - 获取panic的值
  - 如果无法处理，可重新panic

```go
func PanicTest() {
	//设置recover
	defer func() {                    //无论panic是否发生都会执行
		if r := recover(); r != nil { //recover()函数catch到panic的信息并以errorString返回
			fmt.Printf("%T\n", r)         //*errors.errorString
			if err, ok := r.(error); ok { //断言r是一个error才进行处理
				fmt.Printf("%T, %s\n", err, err) //*errors.errorString, this is a panic test
			} else {
				panic(r) //如果不是一个error,鬼知道是什么,再次恐慌!
			}
		}
	}() //匿名函数调用

	//fmt.Println("this is a panic test")       //无panic
	//panic("this is a panic test")             //显式调用panic函数
	panic(errors.New("this is a panic test")) //显式调用panic函数
}
```



### error vs panic

- 能err处理尽量不panic
- 意料之中的：使用error。如:文件打不开
- 意料之外的：使用panic。如:数组越界

