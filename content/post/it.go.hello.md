

+++

title = "go.hello"
description = "go.hello"
tags = ["it", "go"]

+++



# go.hello



- 中文官网：https://golang.google.cn/
- 中文文档：https://studygolang.com/pkgdoc

## 环境

### linux

- 安装：https://golang.google.cn/dl/ ，下载解压版直接解压即可



- 环境变量：`/it/go/go/bin:/it/go/gopath/bin`。一些插件可能需要用到比如protoc-gen-go

```shell
vim /etc/profile
# 写入如下内容
export PATH=$PATH:/it/go/go/bin:/it/go/gopath/bin
# 刷新环境变量
source /etc/profile
```

- 配置

```shell
#必要配置
go env -w GOROOT=D:\it\go\Go
go env -w GOPATH=D:\it\go\GoPath
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct #七牛云代理。"goproxy.io"、"goproxy.cn"、"mirrors.aliyun.com/goproxy/" 等任选其一

#可选配置
go env -w GOPRIVATE=*.gitlab.com,*.gitee.com #设置GOPRIVATE来跳过私有库，比如常用的Gitlab或Gitee，中间使用逗号分隔
go env -w GOSUMDB=off #如果在运行go mod vendor时，提示Get https://sum.golang.org/lookup/xxxxxx: dial tcp 216.58.200.49:443: i/o timeout，则是因为Go 1.13设置了默认的GOSUMDB=sum.golang.org，这个网站是被墙了的，用于验证包的有效性，可以通过这个命令关闭。私有仓库自动忽略验证
go env -w GOSUMDB="sum.golang.google.cn" #可以设置 GOSUMDB="sum.golang.google.cn"， 这个是专门为国内提供的sum 验证服务
```

### goland

1. 下载并解压：https://www.jetbrains.com/go/download/other.html

```sh
# 软连接
ln -s /it/go/goland/bin/goland.sh /usr/bin/goland.sh

# 别名。使 goland nohup，并将其产生的 stdout 重定向到 /dev/null
vim /etc/profile
#添加如下内容，goland将不输出任何信息，也不会产生 nohup.out
alias goland='nohup goland.sh >/dev/null & 2>&1'
```

2. 破解

   1. 打开goland，选择试用
   2. 下载破解文件 jetbrains-agent.jar 2020版本，将 jetbrains-agent.jar 拖入goland窗口，会提示restart，点击restart即可完成破解。适用于1.2及其之前的版本

3. 配置

   1. go

   2. go mod：开启即可

   3. proto import

      1. Goland插件Protocol Buffer Editor：Configure → Setting → Protocol Buffers → 取消勾选Configure automatically → 添加所需目录
         1. 一般是当前project或module的上一级目录，以使生成的go文件依然保持正确的包导入路径，且前缀统一更具可读性；D:/it/go/GoProject/pkg/mod
         2. 如果用到go mod导入的内容，还需要添加go mod包所在目录。D:/it/go/GoProject/pkg/mod
      2. 如果是`protoc`命令工具：则添加 `-I` 参数指定目录

   4. 编码：Configure → Setting → Editor → File Encodings

   5. 全选为UTF-8，with NO BOM

      2. Transparent native-to-ascii conversion自动转换ASCII编码：建议勾选。可能意思是，在文件中输入文字时他会自动的转换为Unicode编码，然后在idea中发开文件时他会自动转回文字来显示

   6. 注解生效激活：Configure → Setting → Compiler → Annotation Processors

      1. Enable annotation processing

   7. 文件过滤：Configure → Setting → Editor → File Types

      1. ActionScript → Ignore files and folders：添加 *.idea;\*.iml; 进去

   8. 字体：Configure → Setting → Editor → Font

   9. 主题：Configure → Setting → Editor → Color Scheme
   
   10. 背景图：Ctrl+Shift+A，搜索set background Image
   
   11. 参数名提示：Ctrl+Shift+A，搜索show parameter name hints
   
   12. 文档模板注释：
   
       ```java
          /* 类文档注释模板配置
          1.file - settings - editor - file and data templates
          2.includs - file header - 右空白框内复制下面类文档注释模板
          3.新建类时自动生成文档注释
          */
          /** 类文档模板
          @title: ${NAME}
          @projectName ${PROJECT_NAME}
          @description: TODO
          @author ${USER}
          @date ${DATE}${TIME}
           */
          public class DocumentAnnotationTemplate {
              /* 方法文档注释模板配置
              1.file - settings - editor - live templates
              2.右+号 - template group - 名aaa(在最前比较方便)
              3.选中aaa - live template - 名ann - 描述ann - 空白框内复制下面方法注释模板 - define - everywhere
              3.新建方法后在方法上面输入ann - 按tab补全
              */
              /** 方法模板
              @description: TODO
              @param ${tags}
              @return ${return_type}
              @throws
              @author ${USER}
              @date ${DATE}${TIME}
               */
              public void function() { }
       }
       ```
   
   13. evaluate expression：https://www.cnblogs.com/mrmoo/p/9942605.html



### windows

#### go

- 安装：https://golang.google.cn/dl/ ，下载解压版直接解压即可
- 环境变量：`D:\it\go\GoPath\bin`。一些插件可能需要用到比如protoc-gen-go
- 配置：与linux一致

### goland

1. 下载并解压、破解、配置 都与linux一致


## HelloWorld

```go
package main

import "fmt"

func main() {
	fmt.Println("HelloWorld") //go语言语句结尾没有分号
}
```





## 工程结构

1) 配置GOPATH环境变量， 配置src同级目录的绝对路径
	如：C:\Users\superman\Desktop\code\go\src\25_工程管理：不同目录
	
2) 自动生成bin或pkg目录，需要使用go install命令(了解)
	除了要配置GOPATH环境变量，还要配置GOBIN环境变量

- src：

  - main1.go

    ```go
    //申明包名可以与所在文件夹名不一致，除main包外建议一致，main包一般在src下
    //一个可执行程序必须有且仅有一个main包
    package main
    
    import "other"
    
    //每个go文件，都可以有且仅有一个main函数
    func main{
        main2func1() //同包中的方法可以直接调用
        other.Other1() //不同包中的方法，只有大写
    }
    ```

  - main2.go

    ```go
    //同文件夹下的文件，申明包名必须一致
    package main
    
    //除了main函数，同包中不能有同名函数
    func main{}
    func main2func1{}
    ```

  - other

    - other1.go

      ```go
      package other //整个src中，不同文件夹，包名必须不一样，
      
      //首字母小写，无法被其它包调用
      func other1() {}
      //首字母大写，可以被其它包调用
      func Other1() {}
      ```

      如果想使用其他包的结构体、结构体成员，类型名、成员名，也要首字母大写才可见

- bin: 放可执行程序
- pkg: 放平台相关的库



