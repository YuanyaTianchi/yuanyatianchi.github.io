

+++

title = "go.cmd"
description = "go.cmd"
tags = ["it", "go"]

+++



# go.cmd



## shell

### go

Go语言中包含了大量用于处理Go语言代码的命令和工具。其中，go命令就是最常用的一个，它有许多子命令。这些子命令都拥有不同的功能，如下所示。

- build：用于编译给定的代码包或Go语言源码文件及其依赖包。
- clean：用于清除执行其他go命令后遗留的目录和文件。
- doc：用于执行godoc命令以打印指定代码包。
- env：用于打印Go语言环境信息。
- fix：用于执行go tool fix命令以修正给定代码包的源码文件中包含的过时语法和代码调用。
- fmt：用于执行gofmt命令以格式化给定代码包中的源码文件。
- get：用于下载和安装给定代码包及其依赖包(提前安装git或hg)。
- list：用于显示给定代码包的信息。
- run：用于编译并运行给定的命令源码文件。
- install：编译包文件并编译整个程序。
- test：用于测试给定的代码包。
- tool：用于运行Go语言的特殊工具。
- version：用于显示当前安装的Go语言的版本信息



#### get

```shell
go get github.com/go-sql-driver/mysql #命令可以下载依赖包
go get github.com/go-sql-driver/mysql@v1.4.1 #指定下载版本，或升级到指定的版本号version
go get -u github.com/go-sql-driver/mysql@v1.4.1 #将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号)
go get -u=patch #将会升级到最新的修订版本
```



#### module

Go语言从v1.5开始开始引入`vendor`模式，如果项目目录下有vendor目录，那么go工具链会优先使用`vendor`内的包进行编译、测试等。godep是一个通过vender模式实现的Go语言的第三方依赖管理工具，类似的还有由社区维护准官方包管理工具dep

`go module`是Go1.11版本后官方推出的版本管理工具，从Go1.13版本开始，`go module`作为Go默认的依赖管理工具

| 命令     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| download | download modules to local cache(下载依赖包，默认为$GOPATH/pkg/mod) |
| edit     | edit go.mod from tools or scripts（编辑go.mod)               |
| graph    | print module requirement graph (打印模块依赖图)              |
| init     | initialize new module in current directory（初始化当前目录为mod） |
| tidy     | add missing and remove unused modules(拉取缺少的模块，移除不用的模块) |
| vendor   | make vendored copy of dependencies(将依赖复制到vendor下)     |
| verify   | verify dependencies have expected content (验证依赖是否正确） |
| why      | explain why packages or modules are needed(解释为什么需要依赖) |



```shell
go env -w GO111MODULE=on #on启用模块支持；off禁用模块支持；auto当项目在$GOPATH/src外，且项目根目录有go.mod文件时，开启模块支持。on时就无所谓在不在gopath中创建项目了

go mod init <模块名> #初始化模块，将在当前目录下生成go.mod文件

go mod edit -droprequire=<模块名> #移除模块
go help mod edit #查看go mod edit用法
```

###### go.mod

```go
module github.com/yuanya/package1 //定义本模块名，"github.com/<花名>/"作前缀防止模块名冲突

go 1.14  //golang版本

require "github.com/yuanya/package2" v0.0.0 //定义依赖包及版本
replace "github.com/yuanya/package2" => "../package2" //以相对路径导入其它模块
```

go mod支持语义化版本号，比如`go get foo@v1.2.3`，也可以跟git的分支或tag，比如`go get foo@master`，当然也可以跟git提交哈希，比如`go get foo@e3702bed2`。关于依赖的版本支持以下几种格式：

```go
gopkg.in/tomb.v1 v1.0.0-20141024135613-dd632973f1e7
gopkg.in/vmihailenco/msgpack.v2 v2.9.1
gopkg.in/yaml.v2 <=v2.2.1
github.com/tatsushid/go-fastping v0.0.0-20160109021039-d7bb493dee3e
latest
```

在国内访问golang.org/x上的包都需要翻墙，可以在go.mod中使用replace替换成github上对应的库

```go
replace (
	golang.org/x/text v0.3.0 => github.com/golang/text v0.3.0
	golang.org/x/crypto v0.0.0-20180820150726-614d502a4dac => github.com/golang/crypto v0.0.0-20180820150726-614d502a4dac
)
```

