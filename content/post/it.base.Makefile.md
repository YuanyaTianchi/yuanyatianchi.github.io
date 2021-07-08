

+++

title = "Makefile"
description = "Makefile"
tags = ["it", "base"]

+++



# Makefile

make 是一个自动化构建工具，会在当前目录下寻找 `Makefile` 或 `makefile` 文件。如果存在相应的文件，它就会依据其中定义好的规则完成构建任务。

命名为

makefile基本规则。注意：**规则前必须使用键盘"TAB"键，即制表符\t，否则将无法识别到command**，从而报错 `make: 对“command”无需做任何事`。

```makefile
[target] ... : [prerequisites] ...
	<tab>[command]
    ...
    ...
```

- targets：规则的目标
- prerequisites：可选的要生成 targets 需要的文件或者是目标。
- command：make 需要执行的命令（任意的 shell 命令）。可以有多条命令，每一条命令占一行。

```makefile
# 伪目标定义。不创建目标文件，而是去执行这些目标下面的命令。如 make clean
.PHONY: all build run gotool clean help

# 变量定义
BINARY="main"

all: gotool build

build:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ${BINARY}

run:
	@go run ./

gotool:
	go fmt ./
	go vet ./

clean:
	@if [ -f ${BINARY} ] ; then rm ${BINARY} ; fi

help:
	@echo "make - 格式化 Go 代码, 并编译生成二进制文件"
	@echo "make build - 编译 Go 代码, 生成二进制文件"
	@echo "make run - 直接运行 Go 代码"
	@echo "make clean - 移除二进制文件和 vim swap files"
	@echo "make gotool - 运行 Go 工具 'fmt' and 'vet'"
```