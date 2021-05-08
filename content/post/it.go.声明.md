

+++

title = "go.声明"
description = "go.声明"
tags = ["it", "go"]

+++



## 声明

### 本包：package

- 包命名：自己开发的程序，一般采用域名作为顶级包名，这样就不会重复了，没有域名可以使用`github.com/<username>`作为顶级路径
- main包：go build 同时要满足`main`包和包含`main()`函数，才能编译成一个可执行文件

```go
//申明本文件所属包。go语言以包作为管理单位，每个文件必须先声明包
package main
```



### 导包：import

- `GOROOT`是安装Go的路径，比如`c:/it/go/Go`；`GOPATH`是我们自己定义的开发者个人的工作空间，比如`c:/it/go/GoProject`，编译器会使用这两个路径，再加上`import`导入的相对全路径来查找磁盘上的包，比如导入的`fmt`包，编译器最终找到的是`c:/it/go/Go`这个位置。
- 对于包的查找，是有优先级的，编译器会优先在`GOROOT`里搜索，其次是`GOPATH`,一旦找到，就会马上停止搜索。如果最终都没找到，就报编译异常了。
- 从版本控制系统（GitHub）上导入包时，会先在`GOPATH`下搜索这个包，如果没有找到，就会使用`go get`工具从版本控制系统（GitHub）获取，并把获取到的源代码存储在`GOPATH`目录下对应URL的目录里，以供编译使用
- 对于go语言来讲，其实并不关心你的代码是内部还是外部的，总之都在GOPATH里，任何import包的路径都是从GOPATH开始的，唯一的区别， 就是内部依赖的包是开发者自己写的，外部依赖的包是go get下来的。
- 如果有两个包重名，为他们取别名即可

```go
import "fmt"    //导包
import ."fmt"   //省略调用。调用的时候只需要Println()，不需要fmt.Println()，一般不建议使用
import _"fmt"   //忽略包，不使用其中内容。只是为了调用其中init()函数，一个包可以有多个init函数。
import io "fmt" //取别名。可io.Println()
import ( //多导入
	"io"
	"math"
)
import "project/controller" //"项目名/包名"，从其它项目导包。go语言以gopath作为根目录，必须要保证该项目在gopath目录下
```

tip：一个非main包在通过编译后会生成一个.a文件（在临时目录下生成，使用go install安装到$GOROOT或$GOPATH下，否则你看不到.a)，用于后续可执行程序链接使用。比如Go标准库中的包对应的源码部分路径在: $GOROOT/src， 而标准库中包编译后的.a文件路径在$GOR00T/pkg/darwin_ _amd64 下。

- 导入的包必须要使用，否则会包编译错误，使用`_`空标志符来导入一个包的目的，是执行这个包里的init函数

- 以数据库的驱动为例，Go语言为了统一关于数据库的访问，使用`databases/sql`抽象了一层数据库的操作，可以满足我们操作MYSQL、Postgre等数据库，这样不管我们使用这些数据库的哪个驱动，编码操作都是一样的，想换驱动的时候，就可以直接换掉，而不用修改具体的代码。

  这些数据库驱动的实现，就是具体的，可以由任何人实现的，它的原理就是定义了init函数，在程序运行之前，把实现好的驱动注册到sql包里，这样我们就使用使用它操作数据库了

  剩下针对的数据库的操作，都是使用的`database/sql`标准接口，如果我们想换一个mysql的驱动的话，只需要换个导入就可以了，灵活方便，这也是面向接口编程的便利

```go
package mysql

import (
	"database/sql"
)

func init() {
	sql.Register("mysql", &MySQLDriver{})
}
```

```go
import "database/sql"
import _ "github.com/go-sql-driver/mysql"

db, err := sql.Open("mysql", "user:password@/dbname")
```



### 函数：func

- 函数是 Go 语言的一等公民

#### 调用惯例

- 调用惯例是调用方和被调用方对于参数和返回值传递的约定

##### C

- 使用 [gcc](https://gcc.gnu.org/)[1](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-function-call/#fn:1) 或者 [clang](https://clang.llvm.org/)[2](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-function-call/#fn:2) 将 C 语言编译成汇编代码是分析其调用惯例的最好方法，从汇编语言中可以了解函数调用的具体过程。gcc 和 clang 编译相同 C 语言代码可能会生成不同的汇编指令，不过生成的代码在结构上不会有太大的区别，所以对只想理解调用惯例的人来说没有太多影响。这里选择使用 gcc 编译器来编译 C 语言

```bash
$ gcc --version
gcc (Ubuntu 4.8.2-19ubuntu1) 4.8.2
Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

- 假设有以下的 C 语言代码，代码中只包含两个函数，其中一个是主函数 `main`，另一个是我们定义的函数 `my_function`

```c
// ch04/my_function.c
int my_function(int arg1, int arg2) {
    return arg1 + arg2;
}

int main() {
    int i = my_function(1, 2);
}
```

- 可以使用 `cc -S my_function.c` 命令将上述文件编译成如下所示的汇编代码。按照调用前、调用时以及调用后的顺序分析上述调用过程：
  - 在 `my_function` 调用前，调用方 `main` 函数将 `my_function` 的两个参数分别存到 edi 和 esi 寄存器中；
  - 在 `my_function` 调用时，它会将寄存器 edi 和 esi 中的数据存储到 eax 和 edx 两个寄存器中，随后通过汇编指令 `addl` 计算两个入参之和；
  - 在 `my_function` 调用后，使用寄存器 eax 传递返回值，`main` 函数将 `my_function` 的返回值存储到栈上的 `i` 变量中；

```c
main:
	pushq	%rbp
	movq	%rsp, %rbp
	subq	$16, %rsp
	movl	$2, %esi  // 设置第二个参数
	movl	$1, %edi  // 设置第一个参数
	call	my_function
	movl	%eax, -4(%rbp)
my_function:
	pushq	%rbp
	movq	%rsp, %rbp
	movl	%edi, -4(%rbp)    // 取出第一个参数，放到栈上
	movl	%esi, -8(%rbp)    // 取出第二个参数，放到栈上
	movl	-8(%rbp), %eax    // eax = esi = 1
	movl	-4(%rbp), %edx    // edx = edi = 2
	addl	%edx, %eax        // eax = eax + edx = 1 + 2 = 3
	popq	%rbp
```

- 当 `my_function` 函数的入参增加至八个时，重新编译当前程序可以会得到不同的汇编代码

```c
int my_function(int arg1, int arg2, int ... arg8) {
    return arg1 + arg2 + ... + arg8;
}
```

`main` 函数调用 `my_function` 时，前六个参数会使用 edi、esi、edx、ecx、r8d 和 r9d 六个寄存器传递。寄存器的使用顺序也是调用惯例的一部分，函数的第一个参数一定会使用 edi 寄存器，第二个参数使用 esi 寄存器，以此类推。最后的两个参数与前面的完全不同，调用方 `main` 函数通过栈传递这两个参数

```c
main:
	pushq	%rbp
	movq	%rsp, %rbp
	subq	$16, %rsp     // 为参数传递申请 16 字节的栈空间
	movl	$8, 8(%rsp)   // 传递第 8 个参数
	movl	$7, (%rsp)    // 传递第 7 个参数
	movl	$6, %r9d
	movl	$5, %r8d
	movl	$4, %ecx
	movl	$3, %edx
	movl	$2, %esi
	movl	$1, %edi
	call	my_function
```

- 图中展示了 `main` 函数在调用 `my_function` 前的栈信息。图中 rbp 寄存器会存储函数调用栈的基址指针，即属于 `main` 函数的栈空间的起始位置，而另一个寄存器 rsp 存储的是 `main` 函数调用栈结束的位置，这两个寄存器共同表示了函数的栈空间。

![](./img/c-function-call-stack.png)

- 在调用 `my_function` 之前，`main` 函数通过 `subq $16, %rsp` 指令分配了 16 个字节的栈地址，随后将第六个以上的参数按照从右到左的顺序存入栈中，即第八个和第七个，余下的六个参数会通过寄存器传递，接下来运行的 `call my_function` 指令会调用 `my_function` 函数：

  - `my_function` 会先将寄存器中的全部数据转移到栈上，然后利用 eax 寄存器计算所有入参的和并返回结果。

  ```c
  my_function:
  	pushq	%rbp
  	movq	%rsp, %rbp
  	movl	%edi, -4(%rbp)    // rbp-4 = edi = 1
  	movl	%esi, -8(%rbp)    // rbp-8 = esi = 2
  	...
  	movl	-8(%rbp), %eax    // eax = 2
  	movl	-4(%rbp), %edx    // edx = 1
  	addl	%eax, %edx        // edx = eax + edx = 3
  	...
  	movl	16(%rbp), %eax    // eax = 7
  	addl	%eax, %edx        // edx = eax + edx = 28
  	movl	24(%rbp), %eax    // eax = 8
  	addl	%edx, %eax        // edx = eax + edx = 36
  	popq	%rbp
  ```

- 我们可以将本节的发现和分析简单总结成 — 当我们在 x86_64 的机器上使用 C 语言中调用函数时，参数都是通过寄存器和栈传递的，其中：
  - 六个以及六个以下的参数会按照顺序分别使用 edi、esi、edx、ecx、r8d 和 r9d 六个寄存器传递；
  - 六个以上的参数会使用栈传递，函数的参数会以从右到左的顺序依次存入栈中；
- 而函数的返回值是通过 eax 寄存器进行传递的，由于只使用一个寄存器存储返回值，所以 C 语言的函数不能同时返回多个值。



##### Golang

-  `myFunction` 函数接受两个整数并返回两个整数，`main` 函数在调用 `myFunction` 时将 66 和 77 两个参数传递到当前函数中，

```go
package main

func myFunction(a, b int) (int, int) {
	return a + b, a - b
}

func main() {
	myFunction(66, 77)
}
```

- 使用 `go tool compile -S -N -l main.go` 编译上述代码可以得到如下所示的汇编指令（如果编译时不使用 `-N -l` 参数，编译器会对汇编代码进行优化，编译结果会有较大差别）

```go
"".main STEXT size=68 args=0x0 locals=0x28
	0x0000 00000 (main.go:7)	MOVQ	(TLS), CX
	0x0009 00009 (main.go:7)	CMPQ	SP, 16(CX)
	0x000d 00013 (main.go:7)	JLS	61
	0x000f 00015 (main.go:7)	SUBQ	$40, SP      // 分配 40 字节栈空间
	0x0013 00019 (main.go:7)	MOVQ	BP, 32(SP)   // 将基址指针存储到栈上
	0x0018 00024 (main.go:7)	LEAQ	32(SP), BP
	0x001d 00029 (main.go:8)	MOVQ	$66, (SP)    // 第一个参数
	0x0025 00037 (main.go:8)	MOVQ	$77, 8(SP)   // 第二个参数
	0x002e 00046 (main.go:8)	CALL	"".myFunction(SB)
	0x0033 00051 (main.go:9)	MOVQ	32(SP), BP
	0x0038 00056 (main.go:9)	ADDQ	$40, SP
	0x003c 00060 (main.go:9)	RET
```

- 根据 `main` 函数生成的汇编指令，我们可以分析出 `main` 函数调用 `myFunction` 之前的栈：

![](./img/golang-function-call-stack-before-calling.png)

`main` 函数通过 `SUBQ $40, SP` 指令一共在栈上分配了 40 字节的内存空间：**主函数调用栈**

| 空间          | 大小    | 作用                           |
| ------------- | ------- | ------------------------------ |
| SP+32 ~ BP    | 8 字节  | `main` 函数的栈基址指针        |
| SP+16 ~ SP+32 | 16 字节 | 函数 `myFunction` 的两个返回值 |
| SP ~ SP+16    | 16 字节 | 函数 `myFunction` 的两个参数   |

`myFunction` 入参的压栈顺序和 C 语言一样，都是从右到左，即第一个参数 66 在栈顶的 SP ~ SP+8，第二个参数存储在 SP+8 ~ SP+16 的空间中。

当我们准备好函数的入参之后，会调用汇编指令 `CALL "".myFunction(SB)`，这个指令首先会将 `main` 的返回地址存入栈中，然后改变当前的栈指针 SP 并执行 `myFunction` 的汇编指令：

```go
"".myFunction STEXT nosplit size=49 args=0x20 locals=0x0
	0x0000 00000 (main.go:3)	MOVQ	$0, "".~r2+24(SP) // 初始化第一个返回值
	0x0009 00009 (main.go:3)	MOVQ	$0, "".~r3+32(SP) // 初始化第二个返回值
	0x0012 00018 (main.go:4)	MOVQ	"".a+8(SP), AX    // AX = 66
	0x0017 00023 (main.go:4)	ADDQ	"".b+16(SP), AX   // AX = AX + 77 = 143
	0x001c 00028 (main.go:4)	MOVQ	AX, "".~r2+24(SP) // (24)SP = AX = 143
	0x0021 00033 (main.go:4)	MOVQ	"".a+8(SP), AX    // AX = 66
	0x0026 00038 (main.go:4)	SUBQ	"".b+16(SP), AX   // AX = AX - 77 = -11
	0x002b 00043 (main.go:4)	MOVQ	AX, "".~r3+32(SP) // (32)SP = AX = -11
	0x0030 00048 (main.go:4)	RET
```

从上述的汇编代码中我们可以看出，当前函数在执行时首先会将 `main` 函数中预留的两个返回值地址置成 `int` 类型的默认值 0，然后根据栈的相对位置获取参数并进行加减操作并将值存回栈中，在 `myFunction` 函数返回之间，栈中的数据如下图所示：

![](./img/golang-function-call-stack-before-return.png)

在 `myFunction` 返回后，`main` 函数会通过以下的指令来恢复栈基址指针并销毁已经失去作用的 40 字节栈内存：

```go
    0x0033 00051 (main.go:9)    MOVQ    32(SP), BP
    0x0038 00056 (main.go:9)    ADDQ    $40, SP
    0x003c 00060 (main.go:9)    RET
```

通过分析 Go 语言编译后的汇编指令，我们发现 Go 语言使用栈传递参数和接收返回值，所以它只需要在栈上多分配一些内存就可以返回多个值。

##### 对比

C 语言和 Go 语言在设计函数的调用惯例时选择了不同的实现。C 语言同时使用寄存器和栈传递参数，使用 eax 寄存器传递返回值；而 Go 语言使用栈传递参数和返回值。我们可以对比一下这两种设计的优点和缺点：

- C 语言的方式能够极大地减少函数调用的额外开销，但是也增加了实现的复杂度；
  - CPU 访问栈的开销比访问寄存器高几十倍[3](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-function-call/#fn:3)；
  - 需要单独处理函数参数过多的情况；
- Go 语言的方式能够降低实现的复杂度并支持多返回值，但是牺牲了函数调用的性能；
  - 不需要考虑超过寄存器数量的参数应该如何传递；
  - 不需要考虑不同架构上的寄存器差异；
  - 函数入参和出参的内存空间需要在栈上进行分配；

Go 语言使用栈作为参数和返回值传递的方法是综合考虑后的设计，选择这种设计意味着编译器会更加简单、更容易维护。



#### 参数传递

除了函数的调用惯例之外，Go 语言在传递参数时是传值还是传引用也是一个有趣的问题，不同的选择会影响我们在函数中修改入参时是否会影响调用方看到的数据。我们先来介绍一下传值和传引用两者的区别：

- 传值：函数调用时会对参数进行拷贝，被调用方和调用方两者持有不相关的两份数据；
- 传引用：函数调用时会传递参数的指针，被调用方和调用方两者持有相同的数据，任意一方做出的修改都会影响另一方。

不同语言会选择不同的方式传递参数，Go 语言选择了传值的方式，**无论是传递基本类型、结构体还是指针，都会对传递的参数进行拷贝**。本节剩下的内容将会验证这个结论的正确性。

##### 整型和数组

我们先来分析 Go 语言是如何传递基本类型和数组的。如下所示的函数 `myFunction` 接收了两个参数，整型变量 `i` 和数组 `arr`，这个函数会将传入的两个参数的地址打印出来，在最外层的主函数也会在 `myFunction` 函数调用前后分别打印两个参数的地址：

```go
func myFunction(i int, arr [2]int) {
	fmt.Printf("in my_funciton - i=(%d, %p) arr=(%v, %p)\n", i, &i, arr, &arr)
}

func main() {
	i := 30
	arr := [2]int{66, 77}
	fmt.Printf("before calling - i=(%d, %p) arr=(%v, %p)\n", i, &i, arr, &arr)
	myFunction(i, arr)
	fmt.Printf("after  calling - i=(%d, %p) arr=(%v, %p)\n", i, &i, arr, &arr)
}

$ go run main.go
before calling - i=(30, 0xc00009a000) arr=([66 77], 0xc00009a010)
in my_funciton - i=(30, 0xc00009a008) arr=([66 77], 0xc00009a020)
after  calling - i=(30, 0xc00009a000) arr=([66 77], 0xc00009a010)
```

当我们通过命令运行这段代码时会发现，`main` 函数和被调用者 `myFunction` 中参数的地址是完全不同的。

不过从 `main` 函数的角度来看，在调用 `myFunction` 前后，整数 `i` 和数组 `arr` 两个参数的地址都没有变化。那么如果我们在 `myFunction` 函数内部对参数进行修改是否会影响 `main` 函数中的变量呢？这里更新 `myFunction` 函数并重新执行这段代码：

```go
func myFunction(i int, arr [2]int) {
	i = 29
	arr[1] = 88
	fmt.Printf("in my_funciton - i=(%d, %p) arr=(%v, %p)\n", i, &i, arr, &arr)
}

$ go run main.go
before calling - i=(30, 0xc000072008) arr=([66 77], 0xc000072010)
in my_funciton - i=(29, 0xc000072028) arr=([66 88], 0xc000072040)
after  calling - i=(30, 0xc000072008) arr=([66 77], 0xc000072010)
```

Go

我们可以看到在 `myFunction` 中对参数的修改也仅仅影响了当前函数，并没有影响调用方 `main` 函数，所以能得出如下结论：**Go 语言的整型和数组类型都是值传递的**，也就是在调用函数时会对内容进行拷贝。需要注意的是如果当前数组的大小非常的大，这种传值的方式会对性能造成比较大的影响。

##### 结构体和指针

接下来我们继续分析 Go 语言另外两种常见类型 — 结构体和指针。下面这段代码中定义了一个结构体 `MyStruct` 以及接受两个参数的 `myFunction` 方法：

```go
type MyStruct struct {
	i int
}

func myFunction(a MyStruct, b *MyStruct) {
	a.i = 31
	b.i = 41
	fmt.Printf("in my_function - a=(%d, %p) b=(%v, %p)\n", a, &a, b, &b)
}

func main() {
	a := MyStruct{i: 30}
	b := &MyStruct{i: 40}
	fmt.Printf("before calling - a=(%d, %p) b=(%v, %p)\n", a, &a, b, &b)
	myFunction(a, b)
	fmt.Printf("after calling  - a=(%d, %p) b=(%v, %p)\n", a, &a, b, &b)
}

$ go run main.go
before calling - a=({30}, 0xc000018178) b=(&{40}, 0xc00000c028)
in my_function - a=({31}, 0xc000018198) b=(&{41}, 0xc00000c038)
after calling  - a=({30}, 0xc000018178) b=(&{41}, 0xc00000c028)
```

Go

从上述运行的结果我们可以得出如下结论：

- 传递结构体时：会拷贝结构体中的全部内容；
- 传递结构体指针时：会拷贝结构体指针；

修改结构体指针是改变了指针指向的结构体，`b.i` 可以被理解成 `(*b).i`，也就是我们先获取指针 `b` 背后的结构体，再修改结构体的成员变量。我们简单修改上述代码，分析一下 Go 语言结构体在内存中的布局：

```go
type MyStruct struct {
	i int
	j int
}

func myFunction(ms *MyStruct) {
	ptr := unsafe.Pointer(ms)
	for i := 0; i < 2; i++ {
		c := (*int)(unsafe.Pointer((uintptr(ptr) + uintptr(8*i))))
		*c += i + 1
		fmt.Printf("[%p] %d\n", c, *c)
	}
}

func main() {
	a := &MyStruct{i: 40, j: 50}
	myFunction(a)
	fmt.Printf("[%p] %v\n", a, a)
}

$ go run main.go
[0xc000018180] 41
[0xc000018188] 52
[0xc000018180] &{41 52}
```

Go

在这段代码中，我们通过指针修改结构体中的成员变量，结构体在内存中是一片连续的空间，指向结构体的指针也是指向这个结构体的首地址。将 `MyStruct` 指针修改成 `int` 类型的，那么访问新指针就会返回整型变量 `i`，将指针移动 8 个字节之后就能获取下一个成员变量 `j`。

如果我们将上述代码简化成如下所示的代码片段并使用 `go tool compile` 进行编译会得到如下的结果：

```go
type MyStruct struct {
	i int
	j int
}

func myFunction(ms *MyStruct) *MyStruct {
	return ms
}

$ go tool compile -S -N -l main.go
"".myFunction STEXT nosplit size=20 args=0x10 locals=0x0
	0x0000 00000 (main.go:8)	MOVQ	$0, "".~r1+16(SP) // 初始化返回值
	0x0009 00009 (main.go:9)	MOVQ	"".ms+8(SP), AX   // 复制引用
	0x000e 00014 (main.go:9)	MOVQ	AX, "".~r1+16(SP) // 返回引用
	0x0013 00019 (main.go:9)	RET
```

Go

在这段汇编语言中，我们发现当参数是指针时，也会使用 `MOVQ "".ms+8(SP), AX` 指令复制引用，然后将复制后的指针作为返回值传递回调用方。

![](./img/golang-pointer-as-argument.png)

所以将指针作为参数传入某个函数时，函数内部会复制指针，也就是会同时出现两个指针指向原有的内存空间，所以 Go 语言中传指针也是传值。

##### 传值

当我们验证了 Go 语言中大多数常见的数据结构之后，其实能够推测出 Go 语言在传递参数时使用了传值的方式，接收方收到参数时会对这些参数进行复制；了解到这一点之后，在传递数组或者内存占用非常大的结构体时，我们应该尽量使用指针作为参数类型来避免发生数据拷贝进而影响性能。

#### 小结

这一节我们详细分析了 Go 语言的调用惯例，包括传递参数和返回值的过程和原理。Go 通过栈传递函数的参数和返回值，在调用函数之前会在栈上为返回值分配合适的内存空间，随后将入参从右到左按顺序压栈并拷贝参数，返回值会被存储到调用方预留好的栈空间上，我们可以简单总结出以下几条规则：

1. 通过堆栈传递参数，入栈的顺序是从右到左，而参数的计算是从左到右；
2. 函数返回值通过堆栈传递并由调用者预先分配内存空间；
3. 调用函数时都是传值，接收方会对入参进行复制再计算；





#### 使用

- 函数：如果函数的调用参数相同，则永远返回相同的结果。它不依赖于程序执行期间函数外部任何状态或数据的变化，必须只依赖于其输入参数
  - 函数参数都是值传递
    - slice,map,channel会有传引用的错觉，原因：比如切片的数据结构中包含了指向了数组的指针，即便是传值，这个指针指向的地址任然是那个数组

```go
func Function() {
	f1 := Func1              //函数可以赋值给变量
	fmt.Println(f1(2, 3))    //6
	fmt.Println(Func2(2, 3)) //3 0
}

func Func1(n1 int, n2 int) int { //go中的返回值声明，就等于定义了一个具有所属类型默认值的变量，所以第一个()与第二个()总不能有同名变量
	return n1 * n2
}

func Func2(n1, n2 int) (n3, n4 int) { //返回值可以拥有名字，等于定义了变量，所以两个()中不能有同名变量
	n3, _ = n2, n1 //在返回类型()中定义过的变量，可以直接"="，无须":="
	return         //返回值有名字时，可以省略返回变量名，将自动返回函数中存在的同名变量，不存在的同名变量的返回值，将返回类型缺省值
	/*return n3, n1 //也可以返回非同名变量*/
}

func Func3(args ...int) { //不定参数，一定（只能）放在形参中的最后一个位置。传递的实参可以是零或多个
	fmt.Println("len(args) = ", len(args)) //获取用户传递参数的个数
	for i, data := range args {
		fmt.Printf("args[%d] = %d\n", i, data)
	}
}
```



##### main函数

- main函数：主函数
  - 定义：不能有返回值（通过os. Exit来返回状态），不能有参数（通过os.Args获取命令行参数）。main()只能唯一定义在main包中
  - 调用：只能由go程序自动调用，不可以被引用。
    - main包中的go文件默认总被执行
    - 在main()所在文件导入存在init()的包，init()会在main()之前执行

```go
func main() { //main函数，。入口函数，不能带参数，也不能定义返回值，"{"不能单独起行
    fmt.Println(os.Args)
    os.Exit(-1)
}
```



##### init函数

- init()：包初始化函数
  - 定义：不能有返回值，不能有参数
    - 可以多个定义在任意包中
  - 调用：只能由go程序自动调用，不可以被引用。
    - init()函数会在所在包被导入时执行，用于初始化包所需要的特定资料
    - 一个包可以被多个包导入，但是只能被init()一次
    - 多个init()调用顺序
      - 同文件：从上到下按顺序执行
      - 同包，异文件：按文件名字符串“从小到大”排序后，顺序调用各文件中的init()函数
      - 异包，包间无依赖：按照导入顺序调用
      - 异包，包间有依赖：最后被依赖的最先被初始化，如包1导入包2，包2导入包3，初始化顺序是3-2-1。注意避免循环依赖，否则死循环

```go

```



##### 闭包

- 闭包：一个函数"捕获"了和它在同一作用域的其它常量和变量。这就意味着当闭包被调用的时候，不管在程序什么地方调用，闭包能够使用这些常量或者变量。它不关心这些捕获了的变量和常量是否已经超出了作用域，所以只要闭包还在使用它，这些变量就还会存在。
  - 一个函数可以做另一个函数的返回值，配合局部变量的使用，形成闭包结构
  - 闭包组成理解：
    - 函数体：代码、局部变量、自由变量
    - 自由变(常)量的指向内容：即被"捕获"的那变(常)量
- ps
  - Go语言的赋值和函数传参规则很简单，除了闭包函数以引用的方式对外部变量访问之外，其它赋值和函数传参数都是以传值的方式处理

```go
func main() {
	add := adder(0) //add即一个闭包变量，闭包也是函数的一种
	for i := 0; i < 10; i++ {
		fmt.Println(add(i)) //执行函数体，自由变量所指向的内容存在于闭包之中、函数体之外，函数体执行完毕并不会释放改内容，所以可以完成累加
	}
}

func adder(sum int) func(int) int {
	return func(n int) int { //返回一个闭包
		sum += n //n是局部变量，sum是自由变量，sum指向的内容即一个int值
		return sum
	}
}
```

函数式编程实现：

```go
func main() {
	add := adder(0)
	for i := 0; i < 10; i++ {
		var s int
		s, add = add(i) //返回了一个新的adder方法实例，不存在自由变量了
		fmt.Println(s)
	}
}

type iAdder func(int) (int, iAdder)

func adder(sum int) iAdder {
	return func(n int) (int, iAdder) {
		return sum + n, adder(sum + n)
	}
}
```

##### Fibonacci

```go
func main() {
	fib := fibonacci()
	/*1, 1, 2, 3, 5, 8, 13 ...
	  a, b			//第一次调用
		 a, b		//第二次调用
			...		//...*/
	for i := 0; i < 10; i++ {
		fmt.Println(fib())
	}
	printContents(fib)
}

func fibonacci() intGenerator {
	a, b := 0, 1
	return func() int {
		a, b = b, a+b
		return a
	}
}

type intGenerator func() int //定义一个整数迭代器 

func (g intGenerator) Read(p []byte) (n int, err error) { //实现io.Reader接口
	next := g()
	if next > 10000 {
		return 0, io.EOF
	}
	s := fmt.Sprintf("%d\n", next)
	return strings.NewReader(s).Read(p) //在Fibonacci迭代较大以后p容量会不足，只需要对next和当前读到的内容做简单缓存即可
}

func printContents(reader io.Reader) {
	scanner := bufio.NewScanner(reader)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}
```

##### BinaryTree

```go
func main() {
	root := Node{Value: 3}
	root.Left = &Node{}
	root.Left.Right = CreateNode(2)
	root.Right = &Node{5, nil, nil}
	root.Right.Left = new(Node)
	root.Right.Left.SetValue(4)

	root.InorderTraversePrint()
	nodeCount := 0
	root.InorderTraverseFunc(func(node *Node) { //可以在遍历的时候做任何事情了
		nodeCount++
	})
	fmt.Println(nodeCount)
}

type Node struct {
	Value       int
	Left, Right *Node
}

func (node Node) Print() {
	fmt.Println(node.Value, " ")
}

func (node *Node) SetValue(value int) {
	if node == nil {
		fmt.Println("Setting Value to nil node. Ignored.")
		return
	}
	node.Value = value
}

func CreateNode(value int) *Node {
	return &Node{Value: value}
}

func (node *Node) InorderTraverseFunc(f func(*Node)) {
	if node == nil {
		return
	}
	node.Left.InorderTraverseFunc(f)
	f(node)
	node.Right.InorderTraverseFunc(f)
}

func (node *Node) InorderTraversePrint() {
	node.InorderTraverseFunc(func(node *Node) {
		node.Print()
	})
}
```

##### 匿名函数

- 匿名函数：不需要定义函数名的一种函数实现方式
  - 在Go语言里，**所有的匿名函数(Go语言规范中称之为函数字面量)都是闭包**

```go
func main() {
	n := 10
	s := "mike"
	v := 1
	//匿名函数，没有函数名字，函数定义，还没有调用
	f1 := func(num int, str string) { //:=自动推导类型
		fmt.Println("num = ", num, "str = ", str)
		fmt.Println("n = ", n, "s = ", s) //可以捕获和它在同一作用域的常量或变量
		v = 2
		fmt.Println("v = ", v) //v = 2
	}
	f1(n, s)
	fmt.Println("v = ", v) //v = 2，可见闭包中是直接以引用修改
	//给一个函数类型起别名
	type FuncType func(num int, str string) //函数没有参数， 没有返回值
	var f2 FuncType                         //声明变量
	f2 = f1
	f2(n, s)
	//定义匿名函数，同时调用，并有返回值
	n1, n2 := func(num1, num2 int) (n1, n2 int) {
		fmt.Printf("num1 = %d, num2 = %d \n", num1, num2)
		n1, n2 = num1, num2
		return
	}(1, 2) //后面的()代表调用此匿名函数
	fmt.Println("n1 =", n1, "n2 =", n2)
}
```



##### 回调函数

- 回调函数：在某个类A的函数f中调用其它某个类B的函数f3，且需要类B的函数f3(回调函数)中再回过来调用类A自己的某个函数f1(被回调的函数)
  - 一个函数做另一个函数的参数可以形成回调函数

```go
func main() {
	var a A
	a.Func1(2, 3) //Calling function: main.A.Func2-fm(2,3), return: 5 ... 2 0
}

type A struct{}

func (a A) Func1(n1 int, n2 int) {
	var b B
	fmt.Println(b.Func3(n1, n2, a.Func2, func(n1 int, n2 int) (n3 int, n4 int) {
		n3, _ = n1, n2
		return
	}))
}

func (a A) Func2(n1 int, n2 int) int {
	return n1 + n2
}

type B struct{}

func (b B) Func3(n1 int, n2 int, f1 func(int, int) int, f2 func(int, int) (int, int)) (int, int) {
	fPtr1 := reflect.ValueOf(f1).Pointer()
	fName1 := runtime.FuncForPC(fPtr1).Name()
	fmt.Printf("Calling function: %s(%d,%d), return: %d ... ", fName1, n1, n2, f1(n1, n2))
	return f2(n1, n2)
	/*具有多个返回值的函数无法与其它函数一起直接返回，如设置返回类型为(int, int, int)，return f1(n1, n2), f2(n1, n2)将编译报错*/
}
```



##### 延迟调用：defer

- 确保在函数结束时发生
- 参数在defer语句时计算
- Open/Close，Lock/Unlock，PrintHeader/PrintFooter

关键字defer 用于延迟-个的数或者方法(或者当前所创建的医名函数)的执行。注意，defer
语句只能出现在函数或方法的内部。

defer语句经常被用于处理成对的操作。如打开、关闭。连接、断开连接、加锁、释放锁。通过
defer机制，不论函数逻辑多复杂。都能保证在任何执行路径下，资源被释放。释放资源的defer
应该直按跟在请求资源的语句后。
如果一个所数中有多个defer语句，它们会以LIFO (后进先出)的顺序执行。哪怕函数或某个延
迟调用发生错误，这些调用依旧会被执行。

```go
func main() {
	deferNum := 1
	defer func(d int) {
		fmt.Println(deferNum)
	}(deferNum)                 //main 结束前调用，先进后出，并且参数deferNum已经传入，deferNum后续的改变将无法影响到函数内
	deferNum = 2                //这里的改变将不会影响到第一个defer+匿名函数中的输出值
	defer fmt.Println(deferNum) //main 结束前调用，后进先出，该语句将先于第一个defer执行
}
```



### 变量：var

```go
func Variate() {
	s, n1, n2 := "str", 2, 3   //":="：赋值操作符，左边必须至少有一个有未被声明过的新变量，且该变量在之后必须被使用过，否则编译报错
	n1, n2 = n2, n1            //交换
	n3, _ := Division(n1, n2)  //"_"：空白标识符，"_"对应位置的返回值将不被接收，在函数返回值时的使用
	fmt.Println(s, n1, n2, n3) //str 3 2 1
}

func Division(dividend int, divisor int) (int, error) {
	if dividend == 0 {
		return 0, errors.New("divide by zero")
	}
	return dividend / divisor, nil
}
```

### 常量：const

```go
func Constant() {
	/*在定义常量组时，如果不提供初始值，则表示将使用上一行的表达式。iota是常量组的一个值，初始为0，每定义一个常量自增1*/
	const ( //iota:=0
		b1 = true //b1=true;iota++
		b2        //b2=true;iota++
		n1 = iota //n1=iota;iota++
		_         // _=iota;iota++
		n2        //n2=iota;iota++
		n3        //n3=iota;iota++
	)
	fmt.Println(b1, b2, n1, n2, n3) //true true 2 4 5

	/*常量缺省类型不固定，且可以是任意数字类型，类似于只是文本替换，无需强制类型转换*/
	var n4 float64 = n2 + n3
	var n5 int64 = n2 + n3
	fmt.Println(n4, n5) //9 9
}
```

#### 枚举

```go
func Enums() {
	const (
		b = 1 << (10 * iota)
		kb
		mb
		gb
		tb
		pb
	)
	fmt.Println(b, kb, mb, gb, tb, pb) //1 1024 1048576 1073741824 1099511627776 1125899906842624
}
```



### 自定义类型：type

##### 方法绑定

```go
func main() {
	var myInt MyInt = 1

	result := myInt.add1(2)
	fmt.Println(myInt, result) //1 3
	result = (&myInt).add1(2)  //即使指针调用，该方法还是会转为内存调用并值传递
	fmt.Println(myInt, result) //1 3

	myInt.add2(2)      //即使内存调用，该方法还是会转为指针调用并引用传递
	fmt.Println(myInt) //3
	(&myInt).add2(2)
	fmt.Println(myInt) //5
}

type MyInt int //这里类似于给int取一个别名，但本质上还是一个自定义类型

/*方法绑定：将方法绑定到自定义类型，使之成为类的成员方法，通过类对象调用，并且将调用方法的对象本身作为方法的第一个参数，传递到方法中，这个参数叫接收者*/
func (myInt MyInt) add1(myInt1 MyInt) MyInt { //接收者类型可以是值，将值传递
	return myInt + myInt1
}

func (myInt *MyInt) add2(myInt1 MyInt) { //接收者类型可以是指针，将指针传递
	*myInt += myInt1
}

type IntPtr *int //自定义指针类型。这类自定义类型不能被方法绑定

```



#### 结构体：struct

```go
func main() {
	user1 := User{username: "username"} //未赋值成员为对应类型缺省值
	fmt.Println(user1)                  //{username }
	change(&user1)                      //想要真正改变user的内容需要传入指针，因为是值传递
	fmt.Println(user1)                  //{username password}
	fmt.Println(&user1)                 //&{username password}。非基本类型不再是输出具体地址值

	user2 := User{"username", "password"} //不指定key必须为全部成员赋值
	fmt.Println(user2 == user1)           //true
	fmt.Println(&user2 == &user1)         //false

	ptr1 := new(User)                                     //成员均为对应类型缺省值
	ptr1.username, ptr1.password = "username", "password" //通过指针操作成员与直接操作结构体变量一致
	fmt.Printf("%T\n", ptr1)                              //*main.user

	ptr2 := &User{username: "name"}
	change(ptr2)
	fmt.Println(*ptr2) //{username password}
	fmt.Println(ptr2)  //&{username password}

	vipUser1 := VipUser{User{"username", "password"}}
	fmt.Printf("%+v\n", vipUser1)       //{User:{username:username password:password}}
	fmt.Println(vipUser1.User)          //{username password}，可以通过结构体名直接调用
	fmt.Println(vipUser1.User.username) //username。显式调用
	fmt.Println(vipUser1.username)      //username。隐式调用。继承了User，即继承了User的成员，如果没有在直属成员中找到该该成员，则到继承的成员中去找。但若结构体中存在同名成员username时，将得到该成员的值，即重写，但不会影响到vipUser1.User.username。如果继承了多个结构体，继承的多个结构体中都有同名成员，且本结构体中无该名字的直属成员，则无法隐式调用

	func1 := vipUser1.ChangeUser
	func1("changeUsername1", "changePassword1") //已有接收者
	fmt.Println(vipUser1.User)                  //{changeUsername1 changePassword1}
	func2 := (*VipUser).ChangeUser
	func2(&vipUser1, "changeUsername2", "changePassword2") //第一个参数传入接收者
	fmt.Println(vipUser1.User)                             //{changeUsername2 changePassword2}
}

type User struct { //type 语句设定了结构体的名称，struct 语句定义一个新的数据结构
	username string
	password string
}

func (userPtr *User) ChangeUser(username string, password string) {
	userPtr.username = username
	userPtr.password = password
}

type VipUser struct {
	User //匿名字段，使VipUser继承了User。使用*User有同样的效果
	//username string
}

func change(user *User) {
	user.password = "password"
}
```

##### 内存对齐

- CPU要想从内存读取数据
  - 需要通过地址总线，把地址传输给内存
    - 如果地址总线只有8根，则地址就只有8位，可以表示256个地址，所以256byte就是8根地址总线最大的寻址空间，最多使用256byte的内存
    - 要使用更大的内存就需要更宽的地址总线，如32位地址总线就可以寻址4g内存
  - 内存准备好数据，输出到数据总线，交给CPU
    - 每次操作4byte需要32位（根）数据总线
    - 每次操作8byte就需要64位数据总线
    - 这里每次操作的字节数就是所谓的**机器字长**
- 如果内存就像我们逻辑上认为的，一个挨一个形成一个大矩阵，我们可以访问任意地址，并输出到总线。但是实际上为了实现更高的访问效率，典型的内存布局是，一个内存条的一面是一个 Rank ，一个 Rank 包含8个 Chip，一个 Chip 是8个**堆叠**的 Bank，到 Bank 这里就可以通过选择行列来定位一个地址
  - 这不像是我们逻辑上认为的那样连续的存在，但8个Chips共用同一个地址，各自选择同一个位置的一个字节，再组合起来作为我们逻辑上认为的连续8个字节。注意这里共用地址不代表8个Chips的同一个位置地址值相同，只是通过地址获取第一个Chip上的位置，然后后面7个Chip使用相同的行列位置，地址值依然是递增的，即地址值连续了，通过这样的并行操作，提高了内存访问效率
  - 但如果使用这种设计，用来获取数据的地址值就只能是8的倍数，如果非要错开一格，由于最后一个字节对应的位置与前7个字节不同，不能在一次操作中被同一个地址选中，所以这样的地址是不被硬件支持的。因为64位数据总线决定了一次性传输8byte，而地址值找到第二个Chip，后面6个Chip共用同一个位置，总共只能取出7byte，最后1byte在第一个Chip上与第二个Chip同位置的下一个位置上，与前7个字节位置不同，硬件是不支持这样一次性取的

![](./img/内存访问.png)

- 但是之所以有的CPU支持访问任意地址，是因为多做了处理，比如要读取地址为1-8这8个字节，CPU会分两次读，第一次从内存读出0-7字节，但是只取1-7这7个字节，第二次读8-15，但是只取第一个字节，把两次结果拼接起来拿到所需数据
- 但是这必然影响性能，所以为保证程序顺利高效的运行，**编译器会把各种类型的数据，安排到合适的地址，并占用合适的长度，这就是内存对齐**，每种类型的对齐值就是它的对齐边界
- 内存对齐要求数据存储地址以及占用的字节数都要是对其边界的倍数，所以这里的int32要错开2字节从4开始存，而不能接着从2开始

![](./img/内存对齐.png)

- **对齐边界**：与平台有关，常见的32位平台指针宽度和寄存器宽度都是4字节，64位平台都是8字节，而被golang称作寄存器宽度的值，就可以理解为机器字长，也是平台对应的最大对其边界，而数据类型的对其边界，是取类型大小与平台最大对其边界中较小的那个，所以同一个类型在不同平台上对齐边界可能不同
  - 比如int64在64位平台上对其边界为8字节，而在32位平台上是4字节
  - 之所以不同类型不使用统一的对齐边界，是为了减少内存浪费、提高读取性能
    - 比如一个int16数据只占用2字节，还要使用8字节的对其边界，就有6字节被浪费
    - 而如果使用1字节对其边界，可能会被存储成第一个字节在最后一个Chip上，第二个在第一个Chip上，就需要cpu读取内存两次再拼接，而影响性能

- 结构体的对齐边界：对结构体而言，首先要确定每个成员的对其边界，然后取其中最大，就是这个结构体类型的对齐边界。具体对齐方式是这样的：
  - 内存对齐的第一个要求：存储这个结构体的起始地址，是对齐边界的倍数。结构体的每个成员在存储时，都要把这个起始地址当作地址0，然后再用相对地址来决定自己该放哪
  - 内存对齐的第二个要求：结构体整体占用字节数，需要是类型对其边界的倍数，不够的话要向后扩张。
    - 为什么要这样限制？比如这个类型如下占用了22字节，如果不扩大到其类型对其边界的整数倍，而我们创建了一个长度为2的该类型数组，其中第二元素从位置22开始，就没有对齐内存了
    - 所以只有每个结构体的大小都是对齐值的整数倍，才能保证数组里的每一个元素都是内存对齐的。

![](./img/结构体内存对齐.png)

#### 接口：interface

##### 概述

- 在计算机科学中，接口是计算机系统中多个组件共享的边界，不同的组件能够在边界上交换信息[1](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/#fn:1)。如下图所示，接口的本质是引入一个新的中间层，调用方可以通过接口与具体实现分离，解除上下游的耦合，上层的模块不再需要依赖下层的具体模块，只需要依赖一个约定好的接口。
- 这种面向接口的编程方式有着非常强大的生命力，无论是在框架还是操作系统中我们都能够找到接口的身影。可移植操作系统接口（Portable Operating System Interface，POSIX)[2](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/#fn:2)就是一个典型的例子，它定义了应用程序接口和命令行等标准，为计算机软件带来了可移植性 — 只要操作系统实现了 POSIX，计算机软件就可以直接在不同操作系统上运行。
- 除了解耦有依赖关系的上下游，接口还能够帮助我们隐藏底层实现，减少关注点。《计算机程序的构造和解释》中有这么一句话：**代码必须能够被人阅读，只是机器恰好可以执行**[3](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/#fn:3)
- 人能够同时处理的信息非常有限，定义良好的接口能够隔离底层的实现，让我们将重点放在当前的代码片段中。SQL 就是接口的一个例子，当我们使用 SQL 语句查询数据时，其实不需要关心底层数据库的具体实现，我们只在乎 SQL 返回的结果是否符合预期。
- 计算机科学中的接口是比较抽象的概念，但是编程语言中接口的概念就更加具体。**Go 语言中的接口是一种内置的类型，它定义了一组方法的签名**，本节会介绍 Go 语言接口的几个基本概念以及常见问题，为后面的实现原理做铺垫。

###### 隐式接口

- 很多面向对象语言都有接口这一概念，例如 Java 和 C#。Java 的接口不仅可以定义方法签名，还可以定义变量，这些定义的变量可以直接在实现接口的类中使用，这里简单介绍一下 Java 中的接口：代码定义了一个必须实现的方法 `sayHello` 和一个会注入到实现类的变量 `hello`。

```java
public interface MyInterface {
    public String hello = "Hello";
    public void sayHello();
}
```

- `MyInterfaceImpl` 实现了 `MyInterface` 接口

```java
public class MyInterfaceImpl implements MyInterface {
    public void sayHello() {
        System.out.println(MyInterface.hello);
    }
}
```

- Java 中的类必须通过上述方式显式地声明实现的接口，但是在 Go 语言中实现接口就不需要使用类似的方式。首先简单了解一下在 Go 语言中如何定义接口。定义接口需要使用 `interface` 关键字，在接口中我们只能定义方法签名，不能包含成员变量，一个常见的 Go 语言接口是这样的：
  - 如果一个类型需要实现 `error` 接口，那么它只需要实现 `Error() string` 方法

```go
type error interface {
	Error() string
}
```

-  `RPCError` 结构体就是 `error` 接口的一个实现。代码根本就没有 `error` 接口的影子，这是为什么呢？Go 语言中**接口的实现都是隐式的**，我们只需要实现 `Error() string` 方法就实现了 `error` 接口。Go 语言实现接口的方式与 Java 完全不同：
  - 在 Java 中：实现接口需要显式地声明接口并实现所有方法；
  - 在 Go 中：实现接口的所有方法就隐式地实现了接口；
- 我们使用 `RPCError` 结构体时并不关心它实现了哪些接口，Go 语言只会在传递参数、返回参数以及变量赋值时才会对某个类型是否实现接口进行检查

```go
type RPCError struct {
	Code    int64
	Message string
}

func (e *RPCError) Error() string {
	return fmt.Sprintf("%s, code=%d", e.Message, e.Code)
}
```

- 这里举几个例子来演示发生接口类型检查的时机：

```go
func main() {
	var rpcErr error = NewRPCError(400, "unknown err") // typecheck1
	err := AsErr(rpcErr) // typecheck2
	println(err)
}

func NewRPCError(code int64, msg string) error {
	return &RPCError{ // typecheck3
		Code:    code,
		Message: msg,
	}
}

func AsErr(err error) error {
	return err
}
```

- Go 语言在[编译期间](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/)对代码进行类型检查，上述代码总共触发了三次类型检查：
  - 将 `*RPCError` 类型的变量赋值给 `error` 类型的变量 `rpcErr`；
  - 将 `*RPCError` 类型的变量 `rpcErr` 传递给签名中参数类型为 `error` 的 `AsErr` 函数；
  - 将 `*RPCError` 类型的变量从函数签名的返回值类型为 `error` 的 `NewRPCError` 函数中返回；
- 从类型检查的过程来看，编译器仅在需要时才检查类型，类型实现接口时只需要实现接口中的全部方法，不需要像 Java 等编程语言中一样显式声明。

###### 类型

- 接口也是 Go 语言中的一种类型，它能够出现在变量的定义、函数的入参和返回值中并对它们进行约束，不过 Go 语言中有两种略微不同的接口，一种是带有一组方法的接口，另一种是不带任何方法的 `interface{}`：
- Go 语言使用 [`runtime.iface`](https://draveness.me/golang/tree/runtime.iface) 表示带有一组方法的接口，使用 [`runtime.eface`](https://draveness.me/golang/tree/runtime.eface) 表示不包含任何方法的接口 `interface{}`，两种接口虽然都使用 `interface` 声明，但是由于后者在 Go 语言中很常见，所以在实现时使用了特殊的类型。
- 需要注意的是，与 C 语言中的 `void *` 不同，`interface{}` 类型**不是任意类型**。如果我们将类型转换成了 `interface{}` 类型，变量在运行期间的类型也会发生变化，获取变量类型时会得到 `interface{}`。
- 这里函数不接受任意类型的参数，只接受 `interface{}` 类型的值，在调用 `Print` 函数时会对参数 `v` 进行类型转换，将原来的 `Test` 类型转换成 `interface{}` 类型，本节会在后面介绍类型转换的实现原理。

```go
package main

func main() {
	type Test struct{}
	v := Test{}
	Print(v)
}

func Print(v interface{}) {
	println(v)
}
```

###### 指针和接口

- 在 Go 语言中同时使用指针和接口时会发生一些让人困惑的问题，接口在定义一组方法时没有对实现的接收者做限制，所以我们会看到某个类型实现接口的两种方式：

![](./img/golang-interface-and-pointer.png)

这是因为结构体类型和指针类型是不同的，就像我们不能向一个接受指针的函数传递结构体一样，在实现接口时这两种类型也不能划等号。虽然两种类型不同，但是上图中的两种实现不可以同时存在，Go 语言的编译器会在结构体类型和指针类型都实现一个方法时报错 “method redeclared”。

对 `Cat` 结构体来说，它在实现接口时可以选择接受者的类型，即结构体或者结构体指针，在初始化时也可以初始化成结构体或者指针。下面的代码总结了如何使用结构体、结构体指针实现接口，以及如何使用结构体、结构体指针初始化变量。

```go
type Cat struct {}
type Duck interface { ... }

func (c  Cat) Quack {}  // 使用结构体实现接口
func (c *Cat) Quack {}  // 使用结构体指针实现接口

var d Duck = Cat{}      // 使用结构体初始化变量
var d Duck = &Cat{}     // 使用结构体指针初始化变量
```

Go

实现接口的类型和初始化返回的类型两个维度共组成了四种情况，然而这四种情况不是都能通过编译器的检查：

|                      | 结构体实现接口 | 结构体指针实现接口 |
| :------------------: | :------------: | :----------------: |
|   结构体初始化变量   |      通过      |       不通过       |
| 结构体指针初始化变量 |      通过      |        通过        |

四种中只有使用指针实现接口，使用结构体初始化变量无法通过编译，其他的三种情况都可以正常执行。当实现接口的类型和初始化变量时返回的类型时相同时，代码通过编译是理所应当的：

- 方法接受者和初始化类型都是结构体；
- 方法接受者和初始化类型都是结构体指针；

而剩下的两种方式为什么一种能够通过编译，另一种无法通过编译呢？我们先来看一下能够通过编译的情况，即方法的接受者是结构体，而初始化的变量是结构体指针：

```go
type Cat struct{}

func (c Cat) Quack() {
	fmt.Println("meow")
}

func main() {
	var c Duck = &Cat{}
	c.Quack()
}
```

Go

作为指针的 `&Cat{}` 变量能够**隐式地获取**到指向的结构体，所以能在结构体上调用 `Walk` 和 `Quack` 方法。我们可以将这里的调用理解成 C 语言中的 `d->Walk()` 和 `d->Speak()`，它们都会先获取指向的结构体再执行对应的方法。

但是如果我们将上述代码中方法的接受者和初始化的类型进行交换，代码就无法通过编译了：

```go
type Duck interface {
	Quack()
}

type Cat struct{}

func (c *Cat) Quack() {
	fmt.Println("meow")
}

func main() {
	var c Duck = Cat{}
	c.Quack()
}

$ go build interface.go
./interface.go:20:6: cannot use Cat literal (type Cat) as type Duck in assignment:
	Cat does not implement Duck (Quack method has pointer receiver)
```

Go

编译器会提醒我们：`Cat` 类型没有实现 `Duck` 接口，`Quack` 方法的接受者是指针。这两个报错对于刚刚接触 Go 语言的开发者比较难以理解，如果我们想要搞清楚这个问题，首先要知道 Go 语言在[传递参数](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-function-call/)时都是传值的。

##### 使用

Go 语言提供了另外一种数据类型即接口，它把所有的具有共性的方法定义在一起，**任何其他类型只要实现了这些方法就是实现了这个接口**

```go
func main() {
	var people People
	fmt.Println(people)        //<nil>
	fmt.Println(&people)       //0xc0000301f0
	fmt.Printf("%T\n", people) //<nil>。内存中没有保存值，自然也不存在类型，但声明了只能保存People类型

	people = User{"username"}  //多态
	fmt.Printf("%T\n", people) //main.User。本质上还是User
	people.eat()               //user eat
	people.speak()             //user speak: 我的username是 username

	var i Biology = people         //空接口类型变量可以接收任何类型值。var i interface{}可以直接创建空接口变量
	fmt.Println("None is All：", i) //None is All：{username}

	/*类型断言.类型动态转换*/
	v1, b1 := i.(People)   //返回值1是调用对象的值，返回值2是调用对象的类型是否与指定类型一致的boolean值
	fmt.Println(v1, b1, i) //{username} true {username}。是User，但也是People
	v2, b2 := i.(int)
	fmt.Println(v2, b2, i)     //0 false {username}。类型不一致时返回值1返回指定类型的缺省值
	switch value := i.(type) { //i.(type)这种类型断言方式在switch之外无法使用。可以没有value接收对象值
	case User:
		fmt.Println("User：", value) //User： {username}
	}
}

/*接口实现：实现了某接口所有方法的类型，都是其实现类*/
type Biology interface { //空接口
	// 没有方法则意味着所有其它类型都已经实现了该接口的所有方法，所以空接口类型变量可以接收其它所有类型值(多态)
	// var i interface{}可以直接创建空接口变量
}

type Plant interface {
	otherFunc()
}

type Animal interface {
	eat()
}

type People interface {
	Animal //匿名，继承Animal接口
	speak()
}

type User struct { //当User实现了People的所有方法后产生实现关系
	username string
}

func (user User) eat() { //实现eat
	fmt.Println("user eat")
}
func (user User) speak() { //实现speak
	fmt.Println("user speak: 我的username是", user.username)
}
```

