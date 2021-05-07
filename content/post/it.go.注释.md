# 文档注释

## go doc

go会自动把我们的注释转换为文档

go doc：显示当前文件夹下的文档

go doc Queue：显示指定type的文档

go doc Pop：显示指定方法

## godoc

- go1.13之后官方安装不再包含godoc，通过go get golang.org/x/tools/cmd/godoc 获取，可以将gopath下bin中生成的godoc.exe复制一份到%goroot%/bin中

- 命令：godoc：运行godocServer，默认http://127.0.0.1:6060，可将运行时所在的项目(文件夹)中的代码也输出文档

godocument.go

```go
type GoDocument string

/*这是一个文档测试*/
//无缩进开头的连续多个注释，会被合并到一起。/**/和//没有区别
//	q.Push(123) //以缩进开头的注释，会被文档纳入文字框内
func (gd GoDocument) Print() {
	fmt.Println(gd) //方法中的注释不会被带到文档中
}
```

godocument.go

```go
//Example测试
//必须要有"//Output:"注释才允许被运行，"//Output:"后用注释写下期望内容，当输出内容与期望内容一致时通过测试
//且该测试代码将被godoc测试，通过测试则将被对应写到godoc文档中对应方法之下作为实例代码
func ExampleGoDocument_Print() {
	gd1:= GoDocument("GoDocument实例1")
	fmt.Println(gd1)
	gd2:= GoDocument("GoDocument实例2")
	fmt.Println(gd2)

	// Output:
	// GoDocument实例1
	// GoDocument实例2
}
```

## swagger

```shell
go get -u -v github.com/swaggo/swag/cmd/swag #将下载到gopath/bin下
```



# 特殊注释

## //go:linkname

详见 /go/cmd/compile

## //go:generate

详见 /go/cmd/go