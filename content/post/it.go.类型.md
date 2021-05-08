

+++

title = "go.类型"
description = "go.类型"
tags = ["it", "go"]

+++



# 类型



- 类型
  - byte：字节，8bit
  - (u)int，(u)int8，(u)int16，(u)int32，(u)int64：整数，int长度默认与操作系统位数一致，加u表示无符号整数
  - rune：int32的别名，很适合表示字符，因为go规范源代码为UTF-8编码，UTF-8中很多字符都是3字节
  - uintptr：指针的整数表现类型，本质上等同于无符号整数
  - float32，float64：浮点数
  - complex64，complex128：复数，complex64实部和虚部是float32，complex128实虚部是float64
  - bool：布尔
  - string：字符串
- 传递：Golang所有类型的参数传递都是值传递

### int

#### 位运算

```go
func DecimalToBinary(num int) string { //十进制转二进制
	return DecimalTo(num, 1, 1)
}
func DecimalToOctonary(num int) string { //十进制转八进制
	return DecimalTo(num, 7, 3)
}
func DecimalToHexadecimal(num int) string { //十进制转十六进制
	return DecimalTo(num, 15, 4)
}
func DecimalTo(num, base, offset int) string { //10进制转n进制。base=n-1，offset=log2(n)
	if num == 0 {
		return "0"
	}
	result := ""
	chars := []rune{'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'}
	for ; num != 0; num = int(uint(num) >> offset) { //golang中没有无符号右移，可以先将num转成uint64进行位运算，实现无符号右移，再转回int即可
		temp := num & base
		result = string(chars[temp]) + result
	}
	return result
}

func runePrint() {
	fmt.Printf("%#v\n", []rune("世界")) //打印：[]int32{19990, 30028}。可以发现实际还是[]int32
}
```

### uintptr

- 指针的整数表现类型，其值一般是地址值，但本质上是一个无符号整数
- 永远不要试图引入一个uintptr类型的临时变量，因为它可能会破坏代码的安全性
  - uintptr变量值被改变、uintptr变量所指向的数据的地址发生改变，都将使它指向无效地址空间，使用它将摧毁程序

### complex

```go
func complex() {
	c1 := 1 + 1i
	c2 := 2 + 2i
	c3 := c1 + c2
	fmt.Println(c3) //(3+3i)
}
```

#### cmplx

欧拉公式：
$$
e^{ix}=cosx+isinx <=> e^{iπ}+1=0
$$

```go
func cmplxPackage() {
	fmt.Println(cmplx.Abs(3 + 4i)) //对复数c求模

	/*欧拉公式*/
	res := cmplx.Pow(math.E, 1i*math.Pi) + 1   //Pow实现一个指数，参数1为底数
	fmt.Println(res)                           //(0+1.2246467991473515e-16i)
	res = cmplx.Exp(1i*math.Pi) + 1            //Exp实现一个e的指数
	fmt.Println(res)                           //(0+1.2246467991473515e-16i)。实部为0
	fmt.Printf("%.3f\n" /*.3表示只取小数点后3位*/, res) //(0.000+0.000i)
}
```





### 指针：*

指针变量 * 和地址值 & 的区别：指针变量保存的是一个地址值，会分配独立的内存来存储一个整型数字。当变量前面有 * 标识时，才等同于 & 的用法，否则会直接输出一个整型数字

```go
func pointer() {
	//栈：存放动态变量，变量运行时分配内存。堆：存放静态变量，变量编译时分配内存
	//变量，有内存和内存地址，内存默认保存该类型零值
	var a int                     //有 内存a 和 内存地址&a，a就是盒子，&a就是盒子的编号，0就是盒子里放的东西
	fmt.Println("a =", a)         //a = 0
	fmt.Println("&a =", &a)       //&a = 0xc00007a090。""
	fmt.Println("*&a =", *&a)     //*&a = 10
	fmt.Println("&*&a =", &*&a)   //&*&a = 0xc00007a090
	fmt.Println("*&*&a =", *&*&a) //*&*&a = 0
	//"&"无法连续运算，即无法&&a。"&"可以对任意变量使用，只要是变量，就存在地址值，包括指针变量
	//"*"无法连续运算，即无法**a。"*"只能对地址值使用

	var ptr *int                  //指针变量，是变量，也有自己的内存和内存地址，内存默认保存<nil>值
	fmt.Println("ptr =", ptr)     //ptr = <nil>。ptr的内存保存<nil>值，没有保存地址值时无法被"*"运算
	fmt.Println("&ptr =", &ptr)   //ptr = 0xc00009c020
	fmt.Println("*&ptr =", *&ptr) //ptr = <nil>

	ptr = new(int)              //分配内存，内存中保存指定类型的缺省值，并返回该内存地址值
	fmt.Println("*ptr =", *ptr) //*ptr = 0

	ptr = &a                      //ptr
	fmt.Println("ptr =", ptr)     //ptr = 0xc00007a090
	fmt.Println("*ptr =", *ptr)   //ptr = 0
	fmt.Println("&*ptr =", &*ptr) //ptr = 0xc00007a090

	b := 1
	c := b
	fmt.Println("b =", &b, "c =", &c) //b = 0xc00007a0b0 c = 0xc00007a0b8
}
```



### string

- string：字符串，是**只读字节数组**，长度固定，但长度不是字符串类型的一部分。**所有在字符串变量上的写操作都是通过拷贝其结构体实现**，即复制数据地址和对应长度，不涉及底层字节数组的复制



##### 结构

```go
// reflect.StringHeader
type StringHeader struct {
    Data uintptr // 指向字节数组的第一个字符
    Len  int     // 字节数组长度
}
```



##### 只读

- 只读：字符串会分配到只读的内存空间，不可修改。因为不可变特性可以保证我们不会引用到意外发生改变的值，Java、Python 以及很多编程语言的字符串也都是不可变的。Golang 的字符串可以作为哈希的键，所以如果哈希的键是可变的，不仅会增加哈希实现的复杂度，还可能会影响哈希的比较

- `SRODATA`：string read only data。代码中存在的字符串，编译器会将其标记成只读数据 `SRODATA`

通过`go tool compile -S xxx.go`将这里代码编译成汇编语言，可以看到到 `hello` 字符串的 `SRODATA` 标记

```go
package main

func main() {
	str := "hello"
	println([]byte(str))
}
```

```bash
$ go tool compile -S main.go
...
go.string."hello" SRODATA dupok size=5
	0x0000 68 65 6c 6c 6f                                   hello
...
```



##### 修改

- 修改：在 Golang 中只是不支持直接修改 string 变量的内存空间，仍然可以通过在 string、[]byte 类型之间反复转换实现修改这一目的
  - 先将这段内存拷贝到堆或者栈上；
  - 将变量的类型转换成 `[]byte` 后并修改字节数据；
  - 将修改后的字节数组转换回 `string`；



##### 解析//todo

解析器会在[词法分析](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-lexer-and-parser/)阶段解析字符串，词法分析阶段会对源文件中的字符串进行切片和分组，将原有无意义的字符流转换成 Token 序列。我们可以使用两种字面量方式在 Go 语言中声明字符串，即双引号和反引号:

```go
str1 := "this is a string"
str2 := `this is another
string`
```

Go

使用双引号声明的字符串和其他语言中的字符串没有太多的区别，它只能用于单行字符串的初始化，如果字符串内部出现双引号，需要使用 `\` 符号避免编译器的解析错误，而反引号声明的字符串可以摆脱单行的限制。当使用反引号时，因为双引号不再负责标记字符串的开始和结束，我们可以在字符串内部直接使用 `"`，在遇到需要手写 JSON 或者其他复杂数据格式的场景下非常方便。

```go
json := `{"author": "draven", "tags": ["golang"]}`
```

Go

两种不同的声明方式其实也意味着 Go 语言编译器需要能够区分并且正确解析两种不同的字符串格式。解析字符串使用的扫描器 [`cmd/compile/internal/syntax.scanner`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner) 会将输入的字符串转换成 Token 流，[`cmd/compile/internal/syntax.scanner.stdString`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner.stdString) 方法是它用来解析使用双引号的标准字符串：

```go
func (s *scanner) stdString() {
	s.startLit()
	for {
		r := s.getr()
		if r == '"' {
			break
		}
		if r == '\\' {
			s.escape('"')
			continue
		}
		if r == '\n' {
			s.ungetr()
			s.error("newline in string")
			break
		}
		if r < 0 {
			s.errh(s.line, s.col, "string not terminated")
			break
		}
	}
	s.nlsemi = true
	s.lit = string(s.stopLit())
	s.kind = StringLit
	s.tok = _Literal
}
```

Go

从这个方法的实现我们能分析出 Go 语言处理标准字符串的逻辑：

1. 标准字符串使用双引号表示开头和结尾；
2. 标准字符串需要使用反斜杠 `\` 来逃逸双引号；
3. 标准字符串不能出现如下所示的隐式换行 `\n`；

```go
str := "start
end"
```

Go

使用反引号声明的原始字符串的解析规则就非常简单了，[`cmd/compile/internal/syntax.scanner.rawString`](https://draveness.me/golang/tree/cmd/compile/internal/syntax.scanner.rawString) 会将非反引号的所有字符都划分到当前字符串的范围中，所以我们可以使用它支持复杂的多行字符串：

```go
func (s *scanner) rawString() {
	s.startLit()
	for {
		r := s.getr()
		if r == '`' {
			break
		}
		if r < 0 {
			s.errh(s.line, s.col, "string not terminated")
			break
		}
	}
	s.nlsemi = true
	s.lit = string(s.stopLit())
	s.kind = StringLit
	s.tok = _Literal
}
```

Go

无论是标准字符串还是原始字符串都会被标记成 `StringLit` 并传递到语法分析阶段。在语法分析阶段，与字符串相关的表达式都会由 [`cmd/compile/internal/gc.noder.basicLit`](https://draveness.me/golang/tree/cmd/compile/internal/gc.noder.basicLit) 方法处理：

```go
func (p *noder) basicLit(lit *syntax.BasicLit) Val {
	switch s := lit.Value; lit.Kind {
	case syntax.StringLit:
		if len(s) > 0 && s[0] == '`' {
			s = strings.Replace(s, "\r", "", -1)
		}
		u, _ := strconv.Unquote(s)
		return Val{U: u}
	}
}
```

Go

无论是 `import` 语句中包的路径、结构体中的字段标签还是表达式中的字符串都会使用这个方法将原生字符串中最后的换行符删除并对字符串 Token 进行 Unquote，也就是去掉字符串两边的引号等无关干扰，还原其本来的面目。

[`strconv.Unquote`](https://draveness.me/golang/tree/strconv.Unquote) 处理了很多边界条件导致实现非常复杂，其中不仅包括引号，还包括 UTF-8 等编码的处理逻辑，这里也就不展开介绍了。

##### 拼接//todo

Go 语言拼接字符串会使用 `+` 符号，编译器会将该符号对应的 `OADD` 节点转换成 `OADDSTR` 类型的节点，随后在 [`cmd/compile/internal/gc.walkexpr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.walkexpr) 中调用 [`cmd/compile/internal/gc.addstr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.addstr) 函数生成用于拼接字符串的代码：

```go
func walkexpr(n *Node, init *Nodes) *Node {
	switch n.Op {
	...
	case OADDSTR:
		n = addstr(n, init)
	}
}
```

[`cmd/compile/internal/gc.addstr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.addstr) 能帮助我们在编译期间选择合适的函数对字符串进行拼接，该函数会根据带拼接的字符串数量选择不同的逻辑：

- 如果小于或者等于 5 个，那么会调用 `concatstring{2,3,4,5}` 等一系列函数；
- 如果超过 5 个，那么会选择 [`runtime.concatstrings`](https://draveness.me/golang/tree/runtime.concatstrings) 传入一个数组切片；

```go
func addstr(n *Node, init *Nodes) *Node {
	c := n.List.Len()

	buf := nodnil()
	args := []*Node{buf}
	for _, n2 := range n.List.Slice() {
		args = append(args, conv(n2, types.Types[TSTRING]))
	}

	var fn string
	if c <= 5 {
		fn = fmt.Sprintf("concatstring%d", c)
	} else {
		fn = "concatstrings"

		t := types.NewSlice(types.Types[TSTRING])
		slice := nod(OCOMPLIT, nil, typenod(t))
		slice.List.Set(args[1:])
		args = []*Node{buf, slice}
	}

	cat := syslook(fn)
	r := nod(OCALL, cat, nil)
	r.List.Set(args)
	...

	return r
}
```

Go

其实无论使用 `concatstring{2,3,4,5}` 中的哪一个，最终都会调用 [`runtime.concatstrings`](https://draveness.me/golang/tree/runtime.concatstrings)，它会先对遍历传入的切片参数，再过滤空字符串并计算拼接后字符串的长度。

```go
func concatstrings(buf *tmpBuf, a []string) string {
	idx := 0
	l := 0
	count := 0
	for i, x := range a {
		n := len(x)
		if n == 0 {
			continue
		}
		l += n
		count++
		idx = i
	}
	if count == 0 {
		return ""
	}
	if count == 1 && (buf != nil || !stringDataOnStack(a[idx])) {
		return a[idx]
	}
	s, b := rawstringtmp(buf, l)
	for _, x := range a {
		copy(b, x)
		b = b[len(x):]
	}
	return s
}
```

Go

如果非空字符串的数量为 1 并且当前的字符串不在栈上，就可以直接返回该字符串，不需要做出额外操作。

但是在正常情况下，运行时会调用 `copy` 将输入的多个字符串拷贝到目标字符串所在的内存空间。新的字符串是一片新的内存空间，与原来的字符串也没有任何关联，一旦需要拼接的字符串非常大，拷贝带来的性能损失是无法忽略的。



##### 类型转换

- 类型转换：使用 Go 语言解析和序列化 JSON 等数据格式时，经常需要将数据在 `string` 和 `[]byte` 之间来回转换。无论从哪种类型转换到另一种都需要拷贝数据，而内存拷贝的性能损耗会随着字符串和 `[]byte` 长度的增长而增长。类型转换的开销并没有想象的那么小，我们经常会看到 [`runtime.slicebytetostring`](https://draveness.me/golang/tree/runtime.slicebytetostring) 等函数出现在 [Flame Graphs](http://www.brendangregg.com/flamegraphs.html)（火焰图，一种分析程序性能的手段）中，成为程序的性能热点

`runtime.stringtoslicebyte` 将 string 转 []byte，

```go
func stringtoslicebyte(buf *tmpBuf, s string) []byte {
	var b []byte
    // 会根据是否传入缓冲区做出不同的处理：当传入缓冲区时，它会使用传入的缓冲区存储 []byte；当没有传入缓冲区时，运行时会调用 runtime.rawbyteslice 创建新的字节切片并将字符串中的内容拷贝过去；
	if buf != nil && len(s) <= len(buf) {
		*buf = tmpBuf{}
		b = buf[:len(s)]
	} else {
		b = rawbyteslice(len(s))
	}
	copy(b, s)
	return b
}
```

`runtime.slicebytetostring` 将 []byte 转 string

```go
func slicebytetostring(buf *tmpBuf, b []byte) (str string) {
	l := len(b)
    // 先处理长度为 0 或 1 的情况
	if l == 0 {
		return ""
	}
	if l == 1 {
		stringStructOf(&str).str = unsafe.Pointer(&staticbytes[b[0]])
		stringStructOf(&str).len = 1
		return
	}
    
	var p unsafe.Pointer
    // 根据传入的缓冲区大小决定是否需要为新字符串分配一片内存空间
	if buf != nil && len(b) <= len(buf) {
		p = unsafe.Pointer(buf)
	} else {
		p = mallocgc(uintptr(len(b)), nil, false)
	}
    // `runtime.stringStructOf` 会将字符串指针转换成 `runtime.stringStruct` 结构体指针，然后设置结构体持有的字符串指针 `str` 和长度 `len`
	stringStructOf(&str).str = p
	stringStructOf(&str).len = len(b)
    // 最后通过 `runtime.memmove` 将原 `[]byte` 中的字节全部复制到新的内存空间中
	memmove(p, (*(*slice)(unsafe.Pointer(&b))).array, uintptr(len(b)))
	return
}
```



##### 编码//todo

Unicode字符集的编码方式以及码点、码元：https://www.cnblogs.com/benbenalin/p/6921553.html

UTF-8编码方式与字节序标记：https://www.cnblogs.com/benbenalin/p/6935162.html

UTF-8究竟是怎么编码的：https://www.cnblogs.com/benbenalin/p/6953847.html

> - ASCII：最早只有127个字符被编码到计算机里，也就是大小写英文字母、数字和一些符号，即ASCII编码
> - GB：处理中文显然一个字节是不够的（64*64=4096，64位的计算机最多也只能编码4096个中文），至少需要两个字节，而且还不能和ASCII编码冲突，所以，中国制定了GB2312编码
> - Unicode：世界上有太多种语言，各有各的编码必然产生冲突，Unicode把所有语言都统一到一套编码里，解决了乱码问题。
>   - Unicode标准不断发展，最常用的是用两个字节表示一个字符，非常偏僻的字符可能需要4个字节，包括英文字符也是使用两个字节，这就导致Unicode编码的纯英文文本比ASCII编码多耗费一倍的存储
> - UTF-8：Unicode演变为"可变长编码"，是Unicode演变的一种编码方式
>   - UTF-8编码把一个Unicode字符根据不同的数字大小编码成1-6个字节，常用的英文字母被编码成1个字节，汉字通常是3个字节，只有很生僻的字符才会被编码成4-6个字节
>   - 如果包含大量英文字符，则UTF-8比Unicode更节省存储，如果包含大量英文，则UTF-8比Unicode更耗费存储
>   - UTF-8编码有一个额外的好处，就是ASCII编码实际上可以被看成是UTF-8编码的一部分，所以，大量只支持ASCII编码的历史遗留软件可以在UTF-8编码下继续工作。
> - Unicode、UTF-8在计算机系统通用的字符编码工作方式
>   - 在计算机内存中，统一使用Unicode编码，当需要保存到硬盘或者需要传输的时候，就转换为UTF-8编码
>   - 用记事本编辑的时候，从文件读取的UTF-8字符被转换为Unicode字符到内存里，编辑完成后，保存的时候再把Unicode转换为UTF-8保存到文件
>   - 浏览网页的时候，服务器会把动态生成的Unicode内容转换为UTF-8再传输到浏览器



##### 使用

```go
func StringOption() {
	s := "hello, world"

	/*字符串长度：内置的len函数；reflect.StringHeader结构（仅演示，不推荐）*/
	len1 := len(s)
	len2 := (*reflect.StringHeader)(unsafe.Pointer(&s)).Len
	fmt.Println(len1, len2)

	/*字符串支持切片操作*/
	s1 := s[:5]
	s2 := s[7:]
	fmt.Println(s1, s2)
	/*不同位置的切片底层也访问的同一块内存数据（因为字符串是只读的，相同的字符串面值常量通常是对应同一个字符串常量）*/
	s3 := "hello, world"[:5]
	s4 := "hello, world"[7:]
	fmt.Println(s3, s4)
}
```



- Go语言规范源代码是UTF-8编码，所以Go源代码中出现的字符串面值常量一般也是UTF-8编码的（对于转义字符，则没有这个限制）
- 源代码中的文本字符串通常被解释为采用UTF-8编码的Unicode码点（rune）序列
- 可以用字符串表示GBK等非UTF-8编码的数据，不过这种时候将字符串看作是一个只读的二进制数组更准确，因为`for range`等语法并不能支持非UTF-8编码的字符串的遍历

```go
/*Go语言的字符串中可以存放任意的二进制字节序列
可以用内置的print调试函数或fmt.Print函数直接打印，也可以for range直接遍历UTF-8解码后的Unicode码点值*/
func StringEncoding() {
	/*转型为字节类型，查看字符底层对应的数据
	打印：[]byte{0xe4, 0xb8, 0x96, 0xe7, 0x95, 0x8c, 0x68, 0x69}*/
	fmt.Printf("%#v\n", []byte("世界hi"))

	/*字符串面值中可以直接指定UTF-8编码后的值。如果源文件中全部是ASCII码，可以避免出现多字节的字符
	打印：世界*/
	fmt.Println("\xe4\xb8\x96\xe7\x95\x8c")

	/*对于Go字符串一般都假设其对应的是一个合法的UTF-8编码的字符序列，但也可能会遇到坏的编码
	如果遇到一个错误的UTF-8编码输入，将生成一个特别的Unicode字符'\uFFFD'，通常表现为'�'，不同软件中显示效果可能不一样
	损坏第1字符的第2、3字节，第1字节将被打印为字符'�'，第2、3字节码点值是损坏后的0表现为空字符，且错误编码不会向后扩散
	打印：�界hi*/
	fmt.Println("\xe4\x00\x00\xe7\x95\x8chi")

	/*for range直接遍历string，将迭代获取UTF-8解码后的Unicode码点值
	迭代含有损坏的UTF-8字符串时，第1字符的第2、3字节依然会被单独迭代到，不过迭代的值是损坏后的0(表现为空字符)
	打印UTF-8解码后的Unicode码点值：0:65533, 1:0, 2:0, 3:30028, 6:104, 7:105, */
	for i, c := range "\xe4\x00\x00\xe7\x95\x8chi" {
		fmt.Print(i, ":", c, ", ")
	}
	fmt.Println()

	/*for range遍历强转成[]byte的string，将迭代获取原始的字节码
	string强转[]byte一般不会产生运行时开销
	打印：0:228, 1:184, 2:150, 3:231, 4:149, 5:140, 6:104, 7:105, */
	for i, c := range []byte("世界") {
		fmt.Print(i, ":", c, ", ")
	}
	fmt.Println()

	/*for i下标遍历，将迭代获取字符串的字节数组元素
	打印%x(16进制)：0:e4, 1:0, 2:0, 3:e7, 4:95, 5:8c, 6:68, 7:69, */
	const s5 = "\xe4\x00\x00\xe7\x95\x8chi"
	for i := 0; i < len(s5); i++ {
		fmt.Printf("%d:%x, ", i, s5[i])
	}
	fmt.Println()
    
	/*string和[]rune类型的相互转换被提供了特殊支持。rune用于表示每个Unicode码点，目前只使用了21个bit位*/
	fmt.Printf("%#v\n", []rune("世界"))             //string转[]rune。打印：[]int32{19990, 30028}
	fmt.Printf("%#v\n", string([]rune{'世', '界'})) //[]rune转string。打印：世界
}
```

**类型转换**

字符串相关的强制类型转换主要涉及到 []byte 和 []rune 两种类型。每个转换都可能隐含重新分配内存的代价，最坏的情况下它们的运算时间复杂度都是O(n)。字符串和[]rune的强制转换一般要求两个类型的底层内存结构要尽量一致，显然它们底层对应的[]byte和[]int32类型是完全不同的内部布局，因此这种转换可能隐含重新分配内存的操作。

```go
/*模拟实现for range迭代字符串
for range迭代字符串时，每次解码一个Unicode字符，然后进入for循环体遇到崩坏的编码并不会导致迭代停止*/
func forOnString(s string, forBody func(i int, r rune)) {
	for i := 0; len(s) > 0; {
		r, size := utf8.DecodeRuneInString(s) //解码字符串第一个UTF-8编码的码值，返回该码值和编码的字节数
		forBody(i, r)                         //代表在循环体内，没什么特殊意义？
		s = s[size:]
		i += size
	}
}

/*模拟实现string转[]byte
新建一个切片，将string的字符迭代复制到[]byte中，以保证string只读的语义
实际上，在将string转为[]byte时，如果转换后的变量并没有被修改的情形，编译器可能会直接返回原始的string对应的底层数据*/
func str2bytes(s string) []byte {
	p := make([]byte, len(s))
	for i := 0; i < len(s); i++ {
		c := s[i]
		p[i] = c
	}
	return p
}

/*模拟实现[]byte转string(bytes)。因为string只读，无法直接同构构造底层字节数组生成字符串
可以通过unsafe包获取了字符串的底层数据结构，将切片的数代复制给字符串，以保证字符串只读的语义不会受切片的影响
实际上，如果转换后的字符串在生命周期中原始的[]byte的变量并不会变化，编译器可能会直接基于[]byte底层的数据构建字符串。*/
func bytes2str(s []byte) (p string) {
	/*感觉这里复制[]byte好像没必要？传入的[]byte参数本来就已经是值传递的复制品了*/
	data := make([]byte, len(s))
	for i, c := range s {
		data[i] = c
	}
	hdr := (*reflect.StringHeader)(unsafe.Pointer(&p))
	hdr.Data = uintptr(unsafe.Pointer(&data[0])) //hdr.Data指向data[0]
	hdr.Len = len(s)
	return p
}

/*模拟实现string转[]rune
因为底层内存结构的差异，将不存在string与[]byte相互转换时的编译器优化情况
string转[]rune必然会导致[]rune内存空间的重新分配，然后依次解码并复制对应的Unicode码点值*/
func string2runes(s string) []rune {
	var p []int32
	for len(s) > 0 {
		r, size := utf8.DecodeRuneInString(s) //解码字符串第一个UTF-8编码的码点值，返回该码点值和编码的字节数
		p = append(p, int32(r))
		s = s[size:]
	}
	return []rune(p)
}

/*模拟实现[]rune转string。同样，[]rune转string也必然会导致字符串重构*/
func runes2string(s []int32) string {
	var p []byte
	buf := make([]byte, 3)
	for _, r := range s {
		n := utf8.EncodeRune(buf, r) //UTF-8编码r，并写入buf，返回写入的字节数
		p = append(p, buf[:n]...)
	}
	return string(p)
}
```



###### strings

```go
func stringsPackage() {
	index := strings.Index("hello go", "hello") //Index：返回子序列的位置。不包含时返回-1
	fmt.Println(index)                             //true

	contains := strings.Contains("hello go", "hello") //Contains：匹配子序列，返回boolean值
	fmt.Println(contains)                             //true

	trim := strings.Trim("   trim     ", " ") //Trim：去掉两端的指定字符
	fmt.Printf("#%s#\n", trim)                //#trim#

	join := strings.Join([]string{"ABC", "DEF", "GHI"}, "---") //Join：插空组合
	fmt.Println(join)                                          //ABC---DEF---GHI
	
	#替换
	replaceAll := strings.Replace(s, old, new, n) //n表示从前开始替换2次，n<0时表示替换全部
	replaceAll := strings.ReplaceAll(s, old, new) //替换全部

	split := strings.Split("aaa@bbb@ccc", "@") //Split：以指定字符串拆分
	fmt.Println(split)                         //[aaa bbb ccc]

	fields := strings.Fields("   aaa  bbb ccc     ") //去掉空格，把元素放入数组中
	fmt.Println(fields)                              //[aaa bbb ccc]

	repeat := strings.Repeat("Go", 3) //Repeat：重复组合
	fmt.Println(repeat)               //"GoGoGo"
}
```

###### strconv

```go
func strconvPackage() {
	/*append：追加字符串*/
	appendArr := make([]byte, 0, 1024)
	appendArr = strconv.AppendBool(appendArr, false)  //将参数2转换为字符串后追加到字节数组参数1中
	fmt.Println(appendArr)                            //[116 114 117 101]。ASCII码
	fmt.Println(string(appendArr))                    //false。转换string后打印
	appendArr = strconv.AppendQuote(appendArr, "asd") //引用追加
	fmt.Println(string(appendArr))                    //false"asd"
	appendArr = strconv.AppendInt(appendArr, 123, 16) //参数2为追加数，参数3指定进制
	fmt.Println(string(appendArr))                    //false"asd"7b

	/*转字符串*/
	var format string
	format = strconv.Itoa(123)                      //int转string
	fmt.Println(format)                             //123
	format = strconv.FormatFloat(3.14, 'f', -1, 64) //float转string。'f' 指打印格式，以小数方式，-1指小数点位数(紧缩模式)， 64以float64处理
	fmt.Println(format)                             //3.14
	format = strconv.FormatBool(false)              //boolean转string
	fmt.Println(format)                             //false

	/*字符串转*/
	a, _ := strconv.Atoi("123")             //string转int
	fmt.Println(a)                          //123
	parse, err := strconv.ParseBool("true") //string转boolean
	if err == nil {
		fmt.Println(parse) //true
	} else {
		fmt.Println(err)
	}
}
```

###### json

```go
func jsonPackage() {
	/*转json。json.Marshal方式*/
	user1 := User{"username", "password", []string{"唱", "跳", "rap", "篮球"}}
	userBuf1, _ := json.Marshal(user1) //struct转json
	fmt.Println(string(userBuf1))      //{"Username":"\"username\"","hobby":["唱","跳","rap","篮球"]}
	map1 := map[string]string{"1": "一", "2": "二", "3": "三"}
	mapBuf, _ := json.Marshal(map1) //map转json
	fmt.Println(string(mapBuf))     //{"1":"一","2":"二","3":"三"}

	/*转json。json.MarshalIndent方式，将格式化编码*/
	userBuf2, _ := json.MarshalIndent(user1, "", "	")
	fmt.Println(string(userBuf2))
	/*{
		"Username": "\"username\"",
		"hobby": [
			"唱",
			"跳",
			"rap",
			"篮球"
		]
	}*/

	/*json转。json.Unmarshal*/
	var userJson = `{"Username":"\"username\"","hobby":["唱","跳","rap","篮球"]}`
	var user2 User
	_ = json.Unmarshal([]byte(userJson), &user2) //json转struct
	fmt.Println(user2)                           //{username  [唱 跳 rap 篮球]}
	var map2 map[string]interface{}
	_ = json.Unmarshal([]byte(userJson), &map2) //json转map
	fmt.Println(map2)                           //map[Username:"username" hobby:[唱 跳 rap 篮球]]
}
```



###### 正则

```go
func regexpPackage() {
	reg := regexp.MustCompile(`a[0-9]c`) //解析正则表达式，如果成功返回解释器，否则返回nil
	if reg == nil {
		fmt.Println("regex err")
		return
	}
	buf := "abc azc a7c aac 888 a9c  tac"
	result := reg.FindAllStringSubmatch(buf, -1) //根据规则提取关键信息
	fmt.Println(result)                          //[[a7c] [a9c]]。
}
```





### array

- 数组：由相同类型元素的集合组成的数据结构
  - 数组元素可以被修改，用数组中元素的索引快速访问特定元素
  - 可以将数组看作一个特殊的结构体，结构的字段名对应数组的索引，同时结构体成员的数目是固定的。内置函数`len`可以用于计算数组的长度，`cap`函数可以用于计算数组的容量。不过对于数组类型来说，`len`和`cap`函数返回的结果始终是一样的，都是对应数组类型的长度。
  - 长度为0的数组在内存中并不占用空间。空数组虽然很少直接使用，但是可以用于强调某种特有类型的操作时避免分配额外的内存空间，比如用于管道的同步操作。不关心管道中传输数据的真实类型，其中管道接收和发送操作只是用于消息的同步。对于这种场景，我们用空数组来作为管道类型可以减少管道元素赋值时的开销。当然一般更倾向于用无类型的匿名结构体代替

##### 结构

- 数组是值语义的，一个数组变量其结构就是**一片连续内存**，而不是隐式的指向第一个元素的指针
  - 所以数组赋值和传参都是以整体复制的方式处理，而不是像切片、字符串一样复制上层结构体（指向数组的指针以及其它属性）。为了避免复制的开销，可以传递一个指向数组的指针。通过数组指针变量操作数组的方式与数组变量基本一致，且数组指针赋值和传递只会拷贝一个指针。但数组指针类型依然不够灵活，因为数组的长度是数组类型的组成部分，指向不同长度数组的数组指针类型也是完全不同的

##### 类型

- `cmd/compile/internal/types.Array` 是数组在编译时生成的类型，由元素类型和数组大小共同构成。所以不同长度的数组，其类型也不相同，自然也无法直接赋值

```go
// cmd/compile/internal/types.Array
type Array struct {
	Elem  *Type // element type
	Bound int64 // number of elements; <0 if unknown yet
}
```

- 编译时用于创建数组类型的函数

```go
// cmd/compile/internal/types.NewArray：编译时用于创建数组类型的函数
func NewArray(elem *Type, bound int64) *Type {
	if bound < 0 {
		Fatalf("NewArray: invalid bound %v", bound)
	}
	t := New(TARRAY)
	t.Extra = &Array{Elem: elem, Bound: bound}
	t.SetNotInHeap(elem.NotInHeap())
	return t
}
```



##### 初始化

- 数组初始化是有长度的，且有两种不同的创建方式，这两种声明方式在运行时是完全等价

```go
// 显式的指定数组大小
arr1 := [3]int{1, 2, 3}
// 编译期间通过源代码推导数组的大小
arr2 := [...]int{1, 2, 3}
```



###### [n]T

- 使用显示声明数组大小 `[n]T` 的方式，变量的类型在编译进行到**类型检查**阶段就会被提取出来，然后通过 `types.NewArray` 创建包含数组大小的 `types.Array` 结构体。

`cmd/compile/internal/types.NewArray`：元素类型 Elem 和数组的大小 Bound 共同构成了数组的类型，而当前数组是否应该在堆栈中初始化也在编译期就确定了

```go

```



###### [...]T上限推导

- 使用 `[...]T` 的方式声明数组时，编译器会在的 `gc.typecheckcomplit` 函数中对该数组的大小进行推导。

`cmd/compile/internal/gc.typecheckcomplit`：会调用 `typecheckarraylit` 通过遍历元素的方式来计算数组中元素的数量，然后仍然通过 `types.NewArray` 创建 `types.Array`

```go
func typecheckcomplit(n *Node) (res *Node) {
	...
	if n.Right.Op == OTARRAY && n.Right.Left != nil && n.Right.Left.Op == ODDD {
		n.Right.Right = typecheck(n.Right.Right, ctxType)
		if n.Right.Right.Type == nil {
			n.Type = nil
			return n
		}
		elemType := n.Right.Right.Type

		length := typecheckarraylit(elemType, -1, n.List.Slice(), "array literal")

		n.Op = OARRAYLIT
		n.Type = types.NewArray(elemType, length)
		n.Right = nil
		return n
	}
	...

	switch t.Etype {
	case TARRAY:
		typecheckarraylit(t.Elem(), t.NumElem(), n.List.Slice(), "array literal")
		n.Op = OARRAYLIT
		n.Right = nil
	}
}
```



###### 语句转换

- 对于一个由字面量组成的数组，根据数组元素数量的不同，编译器会在负责初始化字面量的 `gc.anylit` 函数中做两种不同的优化（不考虑逃逸分析的情况下）：
  1. 当元素数量小于或者等于 4 个时，所有的变量会直接将数组中的元素放置在栈上；
  2. 当元素数量大于 4 个时，会将数组中的元素放置到静态区并在运行时取出拷贝到栈上；
- 之后这些转换后的代码才会继续进入[中间代码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)和[机器码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-machinecode/)两个阶段，最后生成可以执行的二进制文件。

```go
// cmd/compile/internal/gc.anylit
func anylit(n *Node, var_ *Node, init *Nodes) { 
	t := n.Type
	switch n.Op {
	case OSTRUCTLIT, OARRAYLIT:
		if n.List.Len() > 4 {
            // 先获取一个唯一的 `staticname`
			vstat := staticname(t)
			vstat.Name.SetReadonly(true)

			fixedlit(inNonInitFunction, initKindStatic, n, vstat, init)

			a := nod(OAS, var_, vstat)
			a = typecheck(a, ctxStmt)
			a = walkexpr(a, init)
			init.Append(a)
			break
		}

		fixedlit(inInitFunction, initKindLocalCode, n, var_, init)
	...
	}
}
```

```go
// cmd/compile/internal/gc.fixedlit
func fixedlit(ctxt initContext, kind initKind, n *Node, var_ *Node, init *Nodes) {
	var splitnode func(*Node) (a *Node, value *Node)
	...

	for _, r := range n.List.Slice() {
		a, value := splitnode(r)
		a = nod(OAS, a, value)
		a = typecheck(a, ctxStmt)
		switch kind {
		case initKindStatic:
            // 在静态存储区初始化数组中的元素并将临时变量赋值给数组
			genAsStatic(a)
		case initKindLocalCode:
			a = orderStmtInPlace(a, map[string][]*Node{})
			a = walkstmt(a)
			init.Append(a)
		}
	}
}
```

数组中元素的个数小于或者等于四个时，`gc.fixedlit` 会负责在函数编译之前将 `[3]{1, 2, 3}` 转换成更加原始的语句，上面代码会将原有的初始化语句 `[3]int{1, 2, 3}` 拆分成一个声明变量的表达式和几个赋值表达式，这些表达式会完成对数组的初始化：

```go
var arr [3]int
arr[0] = 1
arr[1] = 2
arr[2] = 3
```

但是如果当前数组的元素大于四个，`gc.anylit` 会先获取一个唯一的 `staticname`，然后调用 `gc.fixedlit` 函数在静态存储区初始化数组中的元素并将临时变量赋值给数组，假设代码需要初始化 `[5]int{1, 2, 3, 4, 5}`，可以将上述过程理解成以下的伪代码：

```go
var arr [5]int
statictmp_0[0] = 1
statictmp_0[1] = 2
statictmp_0[2] = 3
statictmp_0[3] = 4
statictmp_0[4] = 5
arr = statictmp_0
```

- 最终都要到栈上，即使400个元素，直接在栈上初始化，会如何呢？就算慢点儿，这是在编译阶段，能有什么太大的影响呢?
  - 这个问题很好，我可能没有完全准确的答案，个人的理解是在栈上初始化需要对变量一个一个赋值，静态区的话可以直接把整片内存复制过去，在元素多的时候会节省很多时间。



##### 元素的访问和赋值//todo

- 无论是在栈上还是静态存储区，数组在内存中都是一连串的内存空间，我们通过指向数组开头的指针、元素的数量以及元素类型占的空间大小表示数组。如果我们不知道数组中元素的数量，访问时可能发生越界；而如果不知道数组中元素类型的大小，就没有办法知道应该一次取出多少字节的数据，无论丢失了那个信息，我们都无法知道这片连续的内存空间到底存储了什么数据

数组访问越界是非常严重的错误，Go 语言中可以在编译期间的静态类型检查判断数组越界，`gc.typecheck1` 会验证访问数组的索引：

1. 访问数组的索引是非整数时，报错 “non-integer array index %v”；
2. 访问数组的索引是负数时，报错 “invalid array index %v (index must be non-negative)"；
3. 访问数组的索引越界时，报错 “invalid array index %v (out of bounds for %d-element array)"；

```go
// cmd/compile/internal/gc.typecheck1
func typecheck1(n *Node, top int) (res *Node) {
	switch n.Op {
	case OINDEX:
		ok |= ctxExpr
		l := n.Left  // array
		r := n.Right // index
		switch n.Left.Type.Etype {
		case TSTRING, TARRAY, TSLICE:
			...
			if n.Right.Type != nil && !n.Right.Type.IsInteger() {
				yyerror("non-integer array index %v", n.Right)
				break
			}
			if !n.Bounded() && Isconst(n.Right, CTINT) {
				x := n.Right.Int64()
				if x < 0 {
					yyerror("invalid array index %v (index must be non-negative)", n.Right)
				} else if n.Left.Type.IsArray() && x >= n.Left.Type.NumElem() {
					yyerror("invalid array index %v (out of bounds for %d-element array)", n.Right, n.Left.Type.NumElem())
				}
			}
		}
	...
	}
}
```



数组和字符串的一些简单越界错误都会在编译期间发现，例如：直接使用整数或者常量访问数组；但是如果使用变量去访问数组或者字符串时，编译器就无法提前发现错误，我们需要 Go 语言运行时阻止不合法的访问。Go 语言运行时在发现数组、切片和字符串的越界操作会由运行时的 [`runtime.panicIndex`](https://draveness.me/golang/tree/runtime.panicIndex) 和 [`runtime.goPanicIndex`](https://draveness.me/golang/tree/runtime.goPanicIndex) 触发程序的运行时错误并导致崩溃退出：

```go
TEXT runtime·panicIndex(SB),NOSPLIT,$0-8
	MOVL	AX, x+0(FP)
	MOVL	CX, y+4(FP)
	JMP	runtime·goPanicIndex(SB)

func goPanicIndex(x int, y int) {
	panicCheck1(getcallerpc(), "index out of range")
	panic(boundsError{x: int64(x), signed: true, y: y, code: boundsIndex})
}
```

```go
arr[4]: invalid array index 4 (out of bounds for 3-element array)
arr[i]: panic: runtime error: index out of range [4] with length 3
```



当数组的访问操作 `OINDEX` 成功通过编译器的检查后，会被转换成几个 SSA 指令，假设我们有如下所示的 Go 语言代码，通过如下的方式进行编译会得到 ssa.html 文件：

```go
package check

func outOfRange() int {
	arr := [3]int{1, 2, 3}
	i := 4
	elem := arr[i]
	return elem
}
```

```shell
$ GOSSAFUNC=outOfRange go build array.go
dumped SSA to ./ssa.html
```

`start` 阶段生成的 SSA（Static Single Assignment，静态单赋值）代码就是优化之前的第一版中间代码，下面展示的部分是 `elem := arr[i]` 对应的中间代码，在这段中间代码中我们发现 Go 语言为数组的访问操作生成了判断数组上限的指令 `IsInBounds` 以及当条件不满足时触发程序崩溃的 `PanicBounds` 指令：

```go
b1:
    ...
    v22 (6) = LocalAddr <*[3]int> {arr} v2 v20
    v23 (6) = IsInBounds <bool> v21 v11
If v23 → b2 b3 (likely) (6)

b2: ← b1-
    v26 (6) = PtrIndex <*int> v22 v21
    v27 (6) = Copy <mem> v20
    v28 (6) = Load <int> v26 v27 (elem[int])
    ...
Ret v30 (+7)

b3: ← b1-
    v24 (6) = Copy <mem> v20
    v25 (6) = PanicBounds <mem> [0] v21 v11 v24
Exit v25 (6)
```

编译器会将 `PanicBounds` 指令转换成上面提到的 [`runtime.panicIndex`](https://draveness.me/golang/tree/runtime.panicIndex) 函数，当数组下标没有越界时，编译器会先获取数组的内存地址和访问的下标、利用 `PtrIndex` 计算出目标元素的地址，最后使用 `Load` 操作将指针中的元素加载到内存中。

当然只有当编译器无法对数组下标是否越界无法做出判断时才会加入 `PanicBounds` 指令交给运行时进行判断，在使用字面量整数访问数组下标时会生成非常简单的中间代码，当我们将上述代码中的 `arr[i]` 改成 `arr[2]` 时，就会得到如下所示的代码：

```go
b1:
    ...
    v21 (5) = LocalAddr <*[3]int> {arr} v2 v20
    v22 (5) = PtrIndex <*int> v21 v14
    v23 (5) = Load <int> v22 v20 (elem[int])
    ...
```

Go 语言对于数组的访问还是有着比较多的检查的，它不仅会在编译期间提前发现一些简单的越界错误并插入用于检测数组上限的函数调用，还会在运行期间通过插入的函数保证不会发生越界。

数组的赋值和更新操作 `a[i] = 2` 也会生成 SSA 生成期间计算出数组当前元素的内存地址，然后修改当前内存地址的内容，这些赋值语句会被转换成如下所示的 SSA 代码：

```go
b1:
    ...
    v21 (5) = LocalAddr <*[3]int> {arr} v2 v19
    v22 (5) = PtrIndex <*int> v21 v13
    v23 (5) = Store <mem> {int} v22 v20 v19
    ...
```

赋值的过程中会先确定目标数组的地址，再通过 `PtrIndex` 获取目标元素的地址，最后使用 `Store` 指令将数据存入地址中，从上面的这些 SSA 代码中我们可以看出 上述数组寻址和赋值都是在编译阶段完成的，没有运行时的参与。



##### 使用

```go
func array() {
	var a [3]int            //未赋值元素以所定义类型零值初始化，类型为[3]int。纯声明不能使用[...]
	a1 := [3]int{0, 1}      //顺序指定元素初始化值，类型为[3]int
	a2 := []int{0, 1, 2}    //编译器将在编译期间通过源代码推导数组大小，类型为[]int
	a3 := [...]int{0, 1, 2} //编译器将在编译期间通过源代码推导数组大小，类型为[3]int。所以一般使用[...]比较好
	a4 := [...]int{1: 1, 2} //可以以"索引:值"的方式赋值
	a5 := [...][3]int{1: {0, 1}, 2: {2: 2}}
	fmt.Println(a, "\n", a1, "\n", a2, "\n", a3, "\n", a4, "\n", a5)

	fmt.Println(a1 == a3) //数组是值语义的，类型相同的数组可以比较值

	//遍历数组。for range方式迭代的性能可能会更好一些，因为这种迭代可以保证不会出现数组越界的情形，每轮迭代对数组元素的访问时可以省去对下标越界的判断。
	for i, v := range a3 {
		fmt.Println(i, v)
	}
	for i := range a3 {
		fmt.Println(i, a3[i])
	}
	//第一维数组有长度，但是数组的元素[0]int大小是0，因此整个数组占用的内存大小依然是0。没有付出额外的内存代价，我们就通过for range方式实现了times次快速迭代
	var times [5][0]int
	for range times {
		fmt.Println("hello")
	}
	for i := 0; i < len(a3); i++ {
		fmt.Println(i, a3[i])
	}

	a6 := &a3              //赋值数组指针，避免复制数组带来的开销
	fmt.Println(a6, a6[2]) //&[0 1 2] 2
	for i, v := range a6 {
		fmt.Println(i, v)
	}

	//长度为0的数组在内存中并不占用空间。空数组虽然很少直接使用，但是可以用于强调某种特有类型的操作时避免分配额外的内存空间，比如用于管道的同步操作
	c1 := make(chan [0]int)
	go func() {
		fmt.Println("c1")
		c1 <- [0]int{}
	}()
	<-c1
	//匿名结构体
	c2 := make(chan struct{})
	go func() {
		fmt.Println("c2")
		c2 <- struct{}{} // struct{}部分是类型, {}表示对应的结构体值
	}()
	<-c2
}

// "..."更多用法
func maxDemo() {
    max([]int{1,2,3}...)
    max(a,b)
}
// 取数组中最大值
func max(a ...int) int {
	res := a[0]
	for _, v := range a[1:] {
		if v > res {
			res = v
		}
	}
	return res
}
```

### slice



- 切片即"动态数组"，长度不固定，可以添删元素，添加元素时可能使切片容量增大
- 值传递：切片赋值和函数传参数时也是将切片头信息部分按传值方式处理，但不会导致底层数组数据的复制
- 切片的结构和字符串结构类似，但是没有只读限制
- 介绍过编译器在编译期间简化了获取数组大小、读写数组中的元素等操作：因为数组的内存固定且连续，多数操作都会直接读写内存的特定位置。但是切片是运行时才会确定内容的结构，所有操作还需要依赖 Go 语言的运行时



##### 结构

```go
/*reflect.SliceHeader*/
type SliceHeader struct {
    Data uintptr // 指向数组的指针
    Len  int     // 切片长度（对应元素的个数）
    Cap  int     // 切片容量，Data指向数组的大小（对应元素的个数）
}
```



##### 类型

- `cmd/compile/internal/types.Slice`：切片在编译时生成的类型，仅包含元素类型。

```go
// cmd/compile/internal/types.Slice
type Slice struct {
	Elem *Type
}
```

- `cmd/compile/internal/types.NewSlice`：编译时用于创建切片类型的函数

```go
// cmd/compile/internal/types.NewSlice：编译时用于创建切片类型的函数
func NewSlice(elem *Type) *Type {
	if t := elem.Cache.slice; t != nil {
		if t.Elem() != elem {
			Fatalf("elem mismatch")
		}
		return t
	}

	t := New(TSLICE)
	t.Extra = Slice{Elem: elem} // 不包括长度
	elem.Cache.slice = t
	return t
}
```



##### 初始化

- 切片有 3 种初始化方式：通过下标的方式获得数组或者切片的一部分；使用字面量初始化新的切片

###### 下标

- 通过下标的方式获得数组或者切片的一部分：是最原始也最接近汇编语言的方式，它是所有方法中最为底层的一种
  1. 编译器会将 `arr[0:3]` 或者 `slice[0:3]` 等语句转换成 `OpSliceMake` 操作
  2. `SliceMake` 操作会接受四个参数创建新的切片，元素类型、数组指针、切片大小和容量，这也是我们在数据结构一节中提到的切片的几个字段 
- 使用下标初始化切片，不会拷贝原数组或原切片中的数据，只会创建一个指向原数组的切片结构体，所以修改新切片的数据也会修改原切片（原数组）。

```go
arr[0:3]
// or
slice[0:3]
```

通过 `GOSSAFUNC` 变量编译代这里码可以得到一系列 SSA 中间代码，其中 `slice := arr[0:1]` 语句在 “decompose builtin” 阶段对应的代码如下所示

```go
package opslicemake

func newSlice() []int {
	arr := [3]int{1, 2, 3}
	slice := arr[0:1]
	return slice
}
```

```go
v27 (+5) = SliceMake <[]int> v11 v14 v17

name &arr[*[3]int]: v11
name slice.ptr[*int]: v11
name slice.len[int]: v14
name slice.cap[int]: v17
```



###### 字面量

- 使用字面量初始化新的切片
  1. 

```go
slice := []int{1, 2, 3}
```

- [`cmd/compile/internal/gc.slicelit`](https://draveness.me/golang/tree/cmd/compile/internal/gc.slicelit) 函数会在编译期间将它展开成如下所示的代码片段：
  1. 根据切片中的元素数量对底层数组的大小进行推断并创建一个数组；
  2. 将这些字面量元素存储到初始化的数组中；
  3. 创建一个同样指向 `[3]int` 类型的数组指针；
  4. 将静态存储区的数组 `vstat` 赋值给 `vauto` 指针所在的地址；
  5. 通过 `[:]` 操作获取一个底层使用 `vauto` 的切片；这里的 `[:]` 就是使用下标创建切片的方法，从这一点我们也能看出 `[:]` 操作是创建切片最底层的一种方法

```go
var vstat [3]int
vstat[0] = 1
vstat[1] = 2
vstat[2] = 3
var vauto *[3]int = new([3]int)
*vauto = vstat
slice := vauto[:]
```



###### 关键字make

- 使用关键字 `make` 创建切片
  - 使用字面量的方式创建切片，大部分的工作都会在编译期间完成
  - 但是当我们使用 `make` 关键字创建切片时，很多工作都需要运行时的参与；调用方必须向 `make` 函数传入切片的大小以及可选的容量

```go
slice := make([]int, 10)
```

- 类型检查期间的 [`cmd/compile/internal/gc.typecheck1`](https://draveness.me/golang/tree/cmd/compile/internal/gc.typecheck1) 函数会校验入参
- 这个函数不仅会检查 `len` 是否传入，还会保证传入的容量 `cap` 一定大于或者等于 `len`。除了校验参数之外，当前函数会将 `OMAKE` 节点转换成 `OMAKESLICE`，中间代码生成的 [`cmd/compile/internal/gc.walkexpr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.walkexpr) 函数会依据下面两个条件转换 `OMAKESLICE` 类型的节点：

  1. 切片的大小和容量是否足够小；
  2. 切片是否发生了逃逸，最终在堆上初始化

```go
func typecheck1(n *Node, top int) (res *Node) {
	switch n.Op {
	...
	case OMAKE:
		args := n.List.Slice()

		i := 1
		switch t.Etype {
		case TSLICE:
			if i >= len(args) {
				yyerror("missing len argument to make(%v)", t)
				return n
			}

			l = args[i]
			i++
			var r *Node
			if i < len(args) {
				r = args[i]
			}
			...
			if Isconst(l, CTINT) && r != nil && Isconst(r, CTINT) && l.Val().U.(*Mpint).Cmp(r.Val().U.(*Mpint)) > 0 {
				yyerror("len larger than cap in make(%v)", t)
				return n
			}

			n.Left = l
			n.Right = r
			n.Op = OMAKESLICE
		}
	...
	}
}
```

- 如果当前的切片不会发生逃逸并且切片非常小的时候，`make([]int, 3, 4)` 会被直接转换成如下所示的代码：初始化数组并通过下标 `[:3]` 得到数组对应的切片，这两部分操作都会在编译阶段完成，编译器会在栈上或者静态存储区创建数组并将 `[:3]` 转换成上一节提到的 `OpSliceMake` 操作。

```go
var arr [4]int
n := arr[:3]
```

- 当切片发生逃逸或者非常大时，运行时需要 [`runtime.makeslice`](https://draveness.me/golang/tree/runtime.makeslice) 在堆上初始化切片。用于创建切片的运行时函数 [`runtime.makeslice`](https://draveness.me/golang/tree/runtime.makeslice)，实现很简单
  - 主要工作是计算切片占用的内存空间并在堆上申请一片连续的内存，它使用如下的方式计算占用的内存：内存空间=切片中元素大小×切片容量内存空间=切片中元素大小×切片容量
  - 最后调用的 [`runtime.mallocgc`](https://draveness.me/golang/tree/runtime.mallocgc) 是用于申请内存的函数，这个函数的实现还是比较复杂，如果遇到了比较小的对象会直接初始化在 Go 语言调度器里面的 P 结构中，而大于 32KB 的对象会在堆上初始化，我们会在后面的章节中详细介绍 Go 语言的内存分配器，这里就不展开分析了。

```go
func makeslice(et *_type, len, cap int) unsafe.Pointer {
	mem, overflow := math.MulUintptr(et.size, uintptr(cap))
	if overflow || mem > maxAlloc || len < 0 || len > cap {
		mem, overflow := math.MulUintptr(et.size, uintptr(len))
		if overflow || mem > maxAlloc || len < 0 {
			panicmakeslicelen()
		}
		panicmakeslicecap()
	}

	return mallocgc(mem, et, true)
}
```

- 虽然编译期间可以检查出很多错误，但是在创建切片的过程中如果发生了以下错误会直接触发运行时错误并崩溃：
  1. 内存空间的大小发生了溢出；
  2. 申请的内存大于最大可分配的内存；
  3. 传入的长度小于 0 或者长度大于容量；



- 在之前版本的 Go 语言中，数组指针、长度和容量会被合成一个 [`runtime.slice`](https://draveness.me/golang/tree/runtime.slice) 结构，但是从 [cmd/compile: move slice construction to callers of makeslice](https://github.com/golang/go/commit/020a18c545bf49ffc087ca93cd238195d8dcc411#diff-d9238ca551e72b3a80da9e0da10586a4) 提交之后，构建结构体 [`reflect.SliceHeader`](https://draveness.me/golang/tree/reflect.SliceHeader) 的工作就都交给了 [`runtime.makeslice`](https://draveness.me/golang/tree/runtime.makeslice) 的调用方，该函数仅会返回指向底层数组的指针，调用方会在编译期间构建切片结构体：

```go
func typecheck1(n *Node, top int) (res *Node) {
	switch n.Op {
	...
	case OSLICEHEADER:
	switch 
		t := n.Type
		n.Left = typecheck(n.Left, ctxExpr)
		l := typecheck(n.List.First(), ctxExpr)
		c := typecheck(n.List.Second(), ctxExpr)
		l = defaultlit(l, types.Types[TINT])
		c = defaultlit(c, types.Types[TINT])

		n.List.SetFirst(l)
		n.List.SetSecond(c)
	...
	}
}
```

`OSLICEHEADER` 操作会创建我们在上面介绍过的结构体 [`reflect.SliceHeader`](https://draveness.me/golang/tree/reflect.SliceHeader)，其中包含数组指针、切片长度和容量，它是切片在运行时的表示：

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

正是因为大多数对切片类型的操作并不需要直接操作原来的 [`runtime.slice`](https://draveness.me/golang/tree/runtime.slice) 结构体，所以 [`reflect.SliceHeader`](https://draveness.me/golang/tree/reflect.SliceHeader) 的引入能够减少切片初始化时的少量开销，该改动不仅能够减少 ~0.2% 的 Go 语言包大小，还能够减少 92 个 [`runtime.panicIndex`](https://draveness.me/golang/tree/runtime.panicIndex) 的调用，占 Go 语言二进制的 ~3.5%[1](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array-and-slice/#fn:1)。



##### 访问

- `len(slice)` 或者 `cap(slice)` ：编译器中是 `OLEN` 和 `OCAP`操作，[`cmd/compile/internal/gc.state.expr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.expr) 函数会在 [SSA 生成阶段](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)阶段将它们分别转换成 `OpSliceLen` 和 `OpSliceCap`。在一些情况下，编译时会直接替换成切片的长度或者容量，不需要在运行时获取，
- 元素访问：访问切片中元素使用的 `OINDEX` 操作也会在中间代码生成期间转换成对地址的直接访问
- 切片的操作基本都是在编译期间完成的，除了访问切片的长度、容量或者其中的元素之外，编译期间也会将包含 `range` 关键字的遍历转换成形式更简单的循环，我们会在后面的章节中介绍使用 `range` 遍历切片的过程。



`cmd/compile/internal/gc.state.expr`：会在 [SSA 生成阶段](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)阶段将 `OLEN` 和 `OCAP`分别转换成 `OpSliceLen` 和 `OpSliceCap`

```go
// cmd/compile/internal/gc.state.expr
func (s *state) expr(n *Node) *ssa.Value {
	switch n.Op {
	case OLEN, OCAP:
		switch {
		case n.Left.Type.IsSlice():
			op := ssa.OpSliceLen
			if n.Op == OCAP {
				op = ssa.OpSliceCap
			}
			return s.newValue1(op, types.Types[TINT], s.expr(n.Left))
		...
		}
	...
	}
}
```

- 访问切片中的字段可能会触发 “decompose builtin” 阶段的优化，`len(slice)` 或者 `cap(slice)` 在一些情况下会直接替换成切片的长度或者容量，不需要在运行时获取：

```go
(SlicePtr (SliceMake ptr _ _ )) -> ptr
(SliceLen (SliceMake _ len _)) -> len
(SliceCap (SliceMake _ _ cap)) -> cap
```



- 除了获取切片的长度和容量之外，访问切片中元素使用的 `OINDEX` 操作也会在中间代码生成期间转换成对地址的直接访问：

```go
func (s *state) expr(n *Node) *ssa.Value {
	switch n.Op {
	case OINDEX:
		switch {
		case n.Left.Type.IsSlice():
			p := s.addr(n, false)
			return s.load(n.Left.Type.Elem(), p)
		...
		}
	...
	}
}
```



##### 追加和扩容

- 使用 `append` 关键字向切片中追加元素，中间代码生成阶段的 [`cmd/compile/internal/gc.state.append`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.append) 方法会根据返回值是否会覆盖原变量，选择进入两种流程
- 如果 `append` 返回的新切片不需要赋值回原有的变量，就会进入如下的处理流程：
  - 会先解构切片结构体，获取它的数组指针、大小、容量
  - 如果追加元素后切片的len在cap范围内，就直接将新元素依次加入切片；如果在追加元素后切片的大小大于容量，那么就会调用 [`runtime.growslice`](https://draveness.me/golang/tree/runtime.growslice) 对切片进行扩容并将新的元素依次加入切片

```go
// append(slice, 1, 2, 3)
ptr, len, cap := slice
newlen := len + 3
if newlen > cap {
    ptr, len, cap = growslice(slice, newlen)
    newlen = len + 3
}
*(ptr+len) = 1
*(ptr+len+1) = 2
*(ptr+len+2) = 3
return makeslice(ptr, newlen, cap)
```

- 如果使用 `slice = append(slice, 1, 2, 3)` 语句，即 `append` 返回的新切片需要赋值回原有的变量，那么 `append` 后的切片会覆盖原切片，这时 [`cmd/compile/internal/gc.state.append`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.append) 方法会使用另一种方式展开关键字：

```go
// slice = append(slice, 1, 2, 3)
a := &slice
ptr, len, cap := slice
newlen := len + 3
if uint(newlen) > uint(cap) {
   newptr, len, newcap = growslice(slice, newlen)
   vardef(a)
   *a.cap = newcap
   *a.ptr = newptr
}
newlen = len + 3
*a.len = newlen
*(ptr+len) = 1
*(ptr+len+1) = 2
*(ptr+len+2) = 3
```

- 是否覆盖原变量的逻辑其实差不多，最大的区别在于得到的新切片是否会赋值回原变量。如果我们选择覆盖原有的变量，就不需要担心切片发生拷贝影响性能，因为 Go 语言编译器已经对这种常见的情况做出了优化。



- 已经知道，当切片的容量不足时，会调用 [`runtime.growslice`](https://draveness.me/golang/tree/runtime.growslice) 函数为切片扩容，扩容是为切片分配新的内存空间并拷贝原切片中元素的过程，
  - 在分配内存空间之前需要先确定新的切片容量，运行时根据切片的当前容量选择不同的策略进行扩容：
    - 如果期望容量（原容量+追加元素数）大于当前容量的两倍就会使用期望容量；
    - 如果当前切片的长度小于 1024 就会将容量翻倍；
    - 如果当前切片的长度大于 1024 就会每次增加 25% 的容量，直到新容量大于期望容量；

```go
func growslice(et *_type, old slice, cap int) slice {
	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
```

- 对齐内存：上述代码片段仅会确定切片的大致容量，下面还需要根据切片中的元素大小对齐内存，当数组中元素所占的字节大小为 1、8 或者 2 的倍数时，运行时会使用如下所示的代码对齐内存：

```go
	var overflow bool
	var lenmem, newlenmem, capmem uintptr
	switch {
	case et.size == 1:
		lenmem = uintptr(old.len)
		newlenmem = uintptr(cap)
		capmem = roundupsize(uintptr(newcap))
		overflow = uintptr(newcap) > maxAlloc
		newcap = int(capmem)
	case et.size == sys.PtrSize:
		lenmem = uintptr(old.len) * sys.PtrSize
		newlenmem = uintptr(cap) * sys.PtrSize
		capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
		overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
		newcap = int(capmem / sys.PtrSize)
	case isPowerOfTwo(et.size):
		...
	default:
		...
	}
```

- 内存向上取整：[`runtime.roundupsize`](https://draveness.me/golang/tree/runtime.roundupsize) 函数会将待申请的内存向上取整，取整时会使用 [`runtime.class_to_size`](https://draveness.me/golang/tree/runtime.class_to_size) 数组，使用该数组中的整数可以提高内存的分配效率并减少碎片，我们会在内存分配一节详细介绍该数组的作用：

```go
var class_to_size = [_NumSizeClasses]uint16{
    0,
    8,
    16,
    32,
    48,
    64,
    80,
    ...,
}
```

- 内存溢出：在默认情况下，我们会将目标容量和元素大小相乘得到占用的内存。如果计算新容量时发生了内存溢出或者请求内存超过上限，就会直接崩溃退出程序，不过这里为了减少理解的成本，将相关的代码省略了。



- 拷贝：如果切片中元素不是指针类型，那么会调用 [`runtime.memclrNoHeapPointers`](https://draveness.me/golang/tree/runtime.memclrNoHeapPointers) 将超出切片当前长度的位置清空并在最后使用 [`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 将原数组内存中的内容拷贝到新申请的内存中。这两个方法都是用目标机器上的汇编指令实现的，这里就不展开介绍了。
- [`runtime.growslice`](https://draveness.me/golang/tree/runtime.growslice) 函数最终会返回一个新的切片，其中包含了新的数组指针、大小和容量，这个返回的三元组最终会覆盖原切片。

```go
	var overflow bool
	var newlenmem, capmem uintptr
	switch {
	...
	default:
		lenmem = uintptr(old.len) * et.size
		newlenmem = uintptr(cap) * et.size
		capmem, _ = math.MulUintptr(et.size, uintptr(newcap))
		capmem = roundupsize(capmem)
		newcap = int(capmem / et.size)
	}
	...
	var p unsafe.Pointer
	if et.kind&kindNoPointers != 0 {
		p = mallocgc(capmem, nil, false)
		memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
	} else {
		p = mallocgc(capmem, et, true)
		if writeBarrier.enabled {
			bulkBarrierPreWriteSrcOnly(uintptr(p), uintptr(old.array), lenmem)
		}
	}
	memmove(p, old.array, lenmem)
	return slice{p, old.len, newcap}
}
```

- 简单总结一下扩容的过程，当我们执行这里代码时，会触发 [`runtime.growslice`](https://draveness.me/golang/tree/runtime.growslice) 函数扩容 `arr` 切片并传入期望的新容量 5，这时期望分配的内存大小为 40 字节；不过因为切片中的元素大小等于 `sys.PtrSize`，所以运行时会调用 [`runtime.roundupsize`](https://draveness.me/golang/tree/runtime.roundupsize) 向上取整内存的大小到 48 字节，所以新切片的容量为 48 / 8 = 6。

```go
var arr []int64
arr = append(arr, 1, 2, 3, 4, 5)
```

##### 拷贝

- 使用 `copy(a, b)` 的形式对切片进行拷贝时，编译期间的 [`cmd/compile/internal/gc.copyany`](https://draveness.me/golang/tree/cmd/compile/internal/gc.copyany) 也会分两种情况进行处理拷贝操作

- 如果当前 `copy` 不是在运行时调用的，`copy(a, b)` 会被直接转换成下面的代码：代码中的 [`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 会负责拷贝内存

```go
n := len(a)
if n > len(b) {
    n = len(b)
}
if a.ptr != b.ptr {
    memmove(a.ptr, b.ptr, n*sizeof(elem(a))) 
}
```

- 如果拷贝是在运行时发生的，例如：`go copy(a, b)`，编译器会使用 [`runtime.slicecopy`](https://draveness.me/golang/tree/runtime.slicecopy) 替换运行期间调用的 `copy`，该函数的实现很简单：

```go
func slicecopy(to, fm slice, width uintptr) int {
	if fm.len == 0 || to.len == 0 {
		return 0
	}
	n := fm.len
	if to.len < n {
		n = to.len
	}
	if width == 0 {
		return n
	}
	...

	size := uintptr(n) * width
	if size == 1 {
		*(*byte)(to.array) = *(*byte)(fm.array)
	} else {
		memmove(to.array, fm.array, size)
	}
	return n
}
```

- 无论是编译期间拷贝还是运行时拷贝，两种拷贝方式都会通过 [`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 将整块内存的内容拷贝到目标的内存区域中：



##### 使用

- 切片的很多功能都是由运行时实现的，无论是初始化切片，还是对切片进行追加或扩容都需要运行时的支持，需要注意的是在遇到大切片扩容或者复制时可能会发生大规模的内存拷贝，一定要减少类似操作避免影响程序的性能。	

```go
func sliceOption() {
   var a []int             //nil切片，和nil相等，一般用来表示一个不存在的切片
   a1 := []int{}           //空切片（len=0，0长度切片），和nil不相等，一般用来表示一个空的集合
   a2 := []int{1, 2, 3}    //len=cap=3。直接初始化
   a3 := a2[0:2:cap(a2)]   //len=2，cap=3。[头:尾:容量]，含头不含尾
   a4 := a2[1:2]           //len=1，cap=cap(a2)=3
   a5 := a2[:0]            //len=0，cap=cap(a2)=3
   a6 := a2[1:]            //len=2，cap=cap(a2)=3
   a7 := a2[:]             //len=len(a2)=3，cap=cap(a2)=3
   a8 := make([]int, 0, 3) //len=0，cap=3。make([]类型, 长度, 容量)
   a9 := make([]int, 3)    //len=3，cap=3
   fmt.Println(a, a == nil, a1, a1 == nil, a2, a3, a4, a5, a6, a7, a8, a9)

   /*可以通过reflect.SliceHeader结构访问切片的信息（只是为了说明切片的结构，并不是推荐的做法）*/
   fmt.Println(len(a2)) //内置的len函数返回切片中有效元素的长度
   fmt.Println(cap(a2)) //内置的cap函数返回切片容量大小

   /*切片可以和nil进行比较，只有当切片底层数据指针为空时切片本身为nil，这时候切片的长度和容量信息将是无效的
   如果有切片的底层数据指针为空，但是长度和容量不为0的情况，那么说明切片本身已经被损坏了
   （比如直接通过reflect.SliceHeader或unsafe包对切片作了不正确的修改）*/

   /*除了遍历之外，只要是切片的底层数据指针、长度和容量没有发生变化的话，对切片的遍历、元素的读取和修改都和数组是一样的
   切片本身赋值或参数传递时，和数组指针的操作方式类似，只是复制切片头信息，并不会复制底层的数据
   对于类型，和数组的最大不同是，切片的类型和长度信息无关，只要是相同类型元素构成的切片均对应相同的切片类型*/

   /*添加元素
   容量不足的情况下，append的操作会导致重新分配内存，可能导致巨大的内存分配和复制数据代价
   即使容量足够，依然需要用append函数的返回值来更新切片本身，因为新切片的长度已经发生了变化*/
   a = append(a, 0)                 //向后添加元素
   a = append(a, 1, 2, 3)           //向后多个元素，手写解包方式
   a = append(a, []int{4, 5, 6}...) //向后添加切片, 切片需要解包
   fmt.Println(a)
   /*在开头一般都会导致内存的重新分配，而且会导致已有的元素全部复制1次
   因此，从切片的开头添加元素的性能一般要比从尾部追加元素的性能差很多*/
   a = append([]int{-1}, a...)         //向前添加元素
   a = append([]int{-4, -3, -2}, a...) //向前添加切片
   fmt.Println(a)
   /*由于append函数返回新的切片，也就是它支持链式操作。我们可以将多个append操作组合起来，实现在切片中间插入元素
   链式操作中的第二个append调用都会创建一个临时切片，并将a[i:]的内容复制到临时切片，然后将临时切片再追加到a[:i]*/
   a = append(a[:5], append([]int{1, 2, 3}, a[5:]...)...) //在第5个位置插入切片
   fmt.Println(a)
   /*可以用copy和append组合可以避免创建临时切片，append本质是用于追加元素，扩展切片容量只是append的一个副作用*/
   a = append(a, 0)     //扩展切片1个空间，为要插入的元素留出空间
   copy(a[5+1:], a[5:]) //a[i:]向后移动1个位置，将要插入位置开始之后的元素向后挪动一个位置
   a[5] = 1             //设置要添加的元素
   fmt.Println(a)
   /*用copy和append组合也可以实现在中间位置插入多个元素(也就是插入一个切片)*/
   a = append(a, a2...)       //为切片扩展足够的空间
   copy(a[6+len(a2):], a[6:]) //a[i:]向后移动len(x)个位置
   copy(a[6:], a2)            //复制新添加的切片
   fmt.Println(a)

   /*删除元素。删除开头的元素和删除尾部的元素都可以认为是删除中间元素操作的特殊情况
   原地完成：在原有的切片数据对应的内存区间内完成，不会导致内存空间结构的变化*/
   i, n := 0, 3
   a = a[:len(a)-n]               //删除尾部n个元素。即修改Len
   a = a[n:]                      //删除头部n个元素。即直接移动Data指针
   a = append(a[:0], a[n:]...)    //删除头部n个元素。不移动数据指针，将后面的数据向头部移动，原地完成
   a = a[:copy(a, a[n:])]         //删除头部n个元素。copy完成删除开头的元素，原地完成
   a = append(a[:i], a[i+n:]...)  //删除中间n个元素。原地完成
   a = a[:i+copy(a[i:], a[i+n:])] //删除中间n个元素。原地完成
   fmt.Println(a)
}

func capTest() {
	s := make([]int, 0, 7)
	oldCap := cap(s)
	for i := 0; i < 1000; i++ {
		s = append(s, i)
		if newCap := cap(s); oldCap < newCap {
			fmt.Printf("cap: %d ===> %d\n", oldCap, newCap)
			oldCap = newCap
		}
	}
	/*结果：
	cap: 7 ===> 14
	cap: 14 ===> 28
	cap: 28 ===> 56
	cap: 56 ===> 112
	cap: 112 ===> 224
	cap: 224 ===> 512 //可以发现存在例外，这里不是2倍
	cap: 512 ===> 1024*/
}
```

**内存**

内存技巧：切片高效操作的要点是要降低内存分配的次数，尽量保证`append`操作不会超出`cap`的容量，降低触发内存分配的次数和每次分配内存大小

```go
/*类似[0]int的空数组一般很少用到。但对于切片，len为0但是cap容量不为0的切片则是非常有用的特性
当然，如果len和cap都为0的话，则变成一个真正的空切片，虽然它并不是一个nil值的切片
在判断一个切片是否为空时，一般通过len获取切片的长度来判断，一般很少将切片和nil值做直接的比较。
比如下面的TrimSpace函数用于删除[]byte中的空格。函数实现利用了0长切片的特性，实现高效而且简洁*/
func TrimSpace(bytes []byte) []byte {
	s := bytes[:0]
	for _, b := range bytes {
		if b != ' ' {
			s = append(s, b)
		}
	}
	return s
}

/*其实类似的根据过滤条件原地删除切片元素的算法都可以采用类似的方式处理（因为是删除操作不会出现内存不足的情形）*/
func Filter(s []byte, fn func(x byte) bool) []byte {
	b := s[:0]
	for _, x := range s {
		if !fn(x) {
			b = append(b, x)
		}
	}
	return b
}
```

避免内存泄漏：切片操作并不会复制底层的数据。底层的数组会被保存在内存中，直到它不再被引用。但是有时候可能会因为一个小的内存引用而导致底层整个数组处于被使用的状态，这会延迟自动内存回收器对底层数组的回收。当然，如果切片存在的周期很短的话，可以不用刻意处理这个问题。因为如果切片本身已经可以被GC回收的话，切片对应的每个元素自然也就是可以被回收的了

```go
/*例如：函数加载整个文件到内存，然后搜索第一个出现的电话号码，最后结果以切片方式返回
这段代码返回的[]byte指向保存整个文件的数组。因为切片引用了整个原始数组，导致自动垃圾回收器不能及时释放底层数组的空间。
一个小的需求可能导致需要长时间保存整个文件数据。这虽然这并不是传统意义上的内存泄漏，但是可能会拖慢系统的整体性能。*/
func FindPhoneNumber1(filename string) []byte {
	b, _ := ioutil.ReadFile(filename)
	return regexp.MustCompile("[0-9]+").Find(b)
}

/*要修复这个问题，可以将感兴趣的数据复制到一个新的切片中
（数据的传值是Go语言编程的一个哲学，虽然传值有一定的代价，但是换取的好处是切断了对原始数据的依赖）*/
func FindPhoneNumber2(filename string) []byte {
	b, _ := ioutil.ReadFile(filename)
	b = regexp.MustCompile("[0-9]+").Find(b)
	return append([]byte{}, b...)
}

/*类似的问题，在删除切片元素时可能会遇到。假设切片里存放的是指针对象，那么下面删除末尾的元素后，
被删除的元素依然被切片底层数组引用，从而导致不能及时被自动垃圾回收器回收（这要依赖回收器的实现方式）*/
func f1() {
	var a []*int
	a = a[:len(a)-1] //被删除的最后一个元素依然被引用, 可能导致GC操作被阻碍
}

/*保险的方式是先将需要自动内存回收的元素设置为nil，保证自动回收器可以发现需要回收的对象，然后再进行切片的删除操作*/
func f2() {
	var a []*int
	a[len(a)-1] = nil //GC回收最后一个元素内存
	a = a[:len(a)-1]  //从切片删除最后一个元素
}
```

**类型转换**

```go
/*为了安全，当两个切片类型[]T和[]Y的底层原始切片类型不同时，Go语言是无法直接转换类型的。
不过安全都是有一定代价的，有时候这种转换是有它的价值的——可以简化编码或者是提升代码的性能。
比如在64位系统上，需要对一个[]float64切片进行高速排序，我们可以将它强制转为[]int整数切片，
然后以整数的方式进行排序（因为float64遵循IEEE754浮点数标准特性，当浮点数有序时对应的整数也必然是有序的）*/
var a = []float64{4, 2, 5, 7, 2, 1, 88, 1}

func SortFloat64FastV1(a []float64) {
	/*强制类型转换
	先将切片数据的开始地址转换为一个较大的数组的指针，然后对数组指针对应的数组重新做切片操作
	中间需要unsafe.Pointer来连接两个不同类型的指针传递
	需要注意的是，Go语言实现中非0大小数组的长度不得超过2GB，因此需要针对数组元素的类型大小计算数组的最大长度范围
	（[]uint8最大2GB，[]uint16最大1GB，以此类推，但是[]struct{}数组的长度可以超过2GB）*/
	var b []int = ((*[1 << 20]int)(unsafe.Pointer(&a[0])))[:len(a):cap(a)] //
	sort.Ints(b)                                                           //以int方式给float64排序
}

func SortFloat64FastV2(a []float64) {
	/*通过 reflect.SliceHeader 更新切片头部信息实现转换
	分别取到两个不同类型的切片头信息指针，任何类型的切片头部信息底层都是对应reflect.SliceHeader结构，
	然后通过更新结构体方式来更新切片信息，从而实现a对应的[]float64切片到c对应的[]int类型切片的转换*/
	var c []int
	aHdr := (*reflect.SliceHeader)(unsafe.Pointer(&a))
	cHdr := (*reflect.SliceHeader)(unsafe.Pointer(&c))
	*cHdr = *aHdr

	// 以int方式给float64排序
	sort.Ints(c)
}

/*通过基准测试，我们可以发现用sort.Ints对转换后的[]int排序的性能要比用sort.Float64s排序的性能好一点。
不过需要注意的是，这个方法可行的前提是要保证[]float64中没有NaN和Inf等非规范的浮点数
（因为浮点数中NaN不可排序，正0和负0相等，但是整数中没有这类情形）*/
```

```go
type AutoGenerated struct {
	Age   int    `json:"age"`
	Name  string `json:"name"`
	Child []int  `json:"child"`
}
func main() {
	a := AutoGenerated{}
	jsonStr1 := `{"age": 14,"name": "potter", "child":[1,2,3]}`
	json.Unmarshal([]byte(jsonStr1), &a)
	b := a.Child
	fmt.Println(b) //[1 2 3]
	fmt.Println(cap(a.Child), b) //[1 2 3]
	jsonStr2 := `{"age": 12,"name": "potter", "child":[4,5,6,7,8,9]}`
	json.Unmarshal([]byte(jsonStr2), &a)
	fmt.Println(b) //
	/*解析：切片b被初始化后len=3，与a.Child指向同一个底层数组
	JSON 数组解码到切片（slice）中，Unmarshal 将切片长度重置为零，然后将每个元素 append 到切片中。
	特殊情况，如果将一个空的 JSON 数组解码到一个切片中，Unmarshal 会用一个新的空切片替换该切片
	a.Child的cap原本只有4，现在append6个元素，添加4个之后需要经历一次扩容，即会memmove()一次，将原底层数组的元素拷贝一份到新的数组中
	b仍然指向原来的数组，并且len仍是3，所以b是[4 5 6]*/
}
```



### map

- map：哈希表，采用拉链法解决hash冲突，数组（桶数组）+数组（桶元素数组）实现，桶大小为8，超过由溢出桶装元素，通过位操作获取桶编号（低位hash），通过高8位hash快速访问桶内数组，访问扩容中的map元素会到old桶中获取，删除扩容中的map元素会待分流完再到新桶去删除

##### 哈希函数

- 哈希函数的选择在很大程度上能够决定哈希表的读写性能。在理想情况下，哈希函数应该能够将不同键映射到不同的索引上，这要求**哈希函数的输出范围大于输入范围**，但是由于键的数量会远远大于映射的范围，所以在实际使用时，这个理想的效果是不可能实现的。
- 比较实际的方式是让哈希函数的结果能够尽可能的均匀分布，然后通过工程上的手段解决哈希碰撞的问题。哈希函数映射的结果一定要尽可能均匀，结果不均匀的哈希函数会带来更多的哈希冲突以及更差的读写性能。使用结果分布较为均匀的哈希函数，那么哈希的增删改查的时间复杂度为 O(1)O(1)；但是如果哈希函数的结果分布不均匀，那么所有操作的时间复杂度可能会达到 O(n)

##### 解决冲突

- 哈希函数输入的范围一定会远远大于输出的范围，所以在使用哈希表时一定会遇到冲突，哪怕我们使用了完美的哈希函数，当输入的键足够多也会产生冲突。然而多数的哈希函数都是不够完美的，所以仍然存在发生哈希碰撞的可能，这时就需要一些方法来解决哈希碰撞的问题，常见方法的就是开放寻址法和拉链法。
- 需要注意的是，这里提到的哈希碰撞不是多个键对应的哈希完全相等，可能是多个哈希的部分相等，例如：两个键对应哈希的前四个字节相同。

###### 开放寻址法

- 核心思想是**依次探测和比较数组中的元素以判断目标键值对是否存在于哈希表中**，使用开放寻址法来实现哈希表底层的数据结构就是数组，不过因为数组的长度有限，向哈希表写入 (author, draven) 这个键值对时会从这样的索引开始遍历：`index := hash("author") % array.len`，即对键进行hash，然后去余数组的长度，可以均匀的分布数据
- 添加数据：如果发生了冲突，就会将键值对写入到下一个索引不为空的位置
- 查找数据：会从索引的位置开始线性探测数组，一一对比key，直到找到目标键值对或者空内存
- 装载因子：开放寻址法中的**装载因子**是**数组中元素的数量**与**数组大小**的比值，对其性能影响最大，随着装载因子的增加，线性探测的平均用时就会逐渐增加，这会影响哈希表的读写性能。当装载率超过 70% 之后，哈希表的性能就会急剧下降，而一旦装载率达到 100%，整个哈希表就会完全失效，这时查找和插入任意元素的时间复杂度都是 O(n) 的，这时需要遍历数组中的全部元素，所以在实现哈希表时一定要关注装载因子的变化。

###### 拉链法

- 拉链法：是哈希表最常见的实现方法。实现比较开放地址法稍微复杂一些，但是平均查找的长度也比较短，各个用于存储节点的内存都是动态申请的，可以节省比较多的存储空间
- 使用数组加上链表作为哈希底层的数据结构，不过一些编程语言会在拉链法的哈希中引入红黑树以优化性能，
- 更新数据：一个键值对写入哈希表时，键值对中的键先经过一个哈希函数，哈希函数返回的哈希会帮助我们选择一个桶，和开放地址法一样，选择桶的方式是直接对哈希返回的结果取模：`index := hash("author") % array.len`。选择桶后就可以遍历当前桶中的链表了：
  - 找到键相同的键值对 — 更新键对应的值；
  - 没有找到键相同的键值对 — 在链表的末尾追加新的键值对；
- 装载因子：装载因子=元素数量÷桶数量。装载因子越大，哈希的读写性能就越差。使用拉链法的哈希表装载因子都不会超过 1，当哈希表的装载因子较大时会触发哈希的扩容，创建更多的桶来存储哈希中的元素，保证性能不会出现严重的下降。如果有 1000 个桶的哈希表存储了 10000 个键值对，它的性能是保存 1000 个键值对的 1/10，但是仍然比在链表中直接读写好 1000 倍

##### 结构

- Go 语言运行时同时使用了多个数据结构组合表示哈希表
  - `count` 表示当前哈希表中的元素数量；
  - `B` 表示当前哈希表持有的 `buckets` 数量，但是因为哈希表中桶的数量都 2 的倍数，所以该字段会存储对数，也就是 `len(buckets) == 2^B`；
  - `hash0` 是哈希的种子，它能为哈希函数的结果引入随机性，这个值在创建哈希表时确定，并在调用哈希函数时作为参数传入；
  - `oldbuckets` 是哈希在扩容时用于保存之前 `buckets` 的字段，它的大小是当前 `buckets` 的一半，即2^(B-1)；
  - `buckets` 指向底层数组 `[]bmap`
    -  [`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap)即桶的结构，是长度最多为8的链表，即每一个 [`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap) 只能存储 8 个键值对，
  - `extra`：用来存储额外的数据，当冲突过多，单个桶已经装满时就会使用 `extra.nextOverflow` 中的桶存储溢出的数据，这样的桶可以暂时称作溢出桶。
    - `extra.nextOverflow` 就是指向了 `extra.overflow` 这个桶数组中，下一个还有额外空间可以装数据的桶，也就是，glang map的底层hash数组，是由 `buckets` 与 `extra.overflow` 两个 `[]bmap` 共同组成的，而实际上 `buckets` 与 `extra.overflow` 在内存中是也是连续存储的，可以看作两个拼接的数组，`buckets` 在前，`extra.overflow` 在后

```go
// runtime.hmap
type hmap struct {
	count     int
	flags     uint8
	B         uint8
	noverflow uint16
	hash0     uint32

	buckets    unsafe.Pointer
	oldbuckets unsafe.Pointer
	nevacuate  uintptr

	extra *mapextra
}

type mapextra struct {
	overflow    *[]*bmap
	oldoverflow *[]*bmap
	nextOverflow *bmap
}
```

- 桶的结构体 [`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap) 在 Go 语言源代码中的定义只包含一个简单的 `tophash` 字段，`tophash` 存储了键的哈希的高 8 位，通过比较不同键的哈希的高 8 位可以减少访问键值对次数以提高性能。随着哈希表存储的数据逐渐增多，会扩容哈希表或者使用额外的桶存储溢出的数据，不会让单个桶中的数据超过 8 个，但是创建过多的溢出桶最终也会导致哈希的扩容

```go
// runtime.bmap
type bmap struct {
	tophash [bucketCnt]uint8
}
```

- 运行期间，[`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap) 结构体其实不止包含 `tophash` 字段，因为哈希表中可能存储不同类型的键值对，而且 Go 语言也不支持泛型，所以键值对占据的内存空间大小只能在编译时进行推导。[`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap) 中的其他字段在运行时也都是通过计算内存地址的方式访问的，所以它的定义中就不包含这些字段，不过我们能根据编译期间的 [`cmd/compile/internal/gc.bmap`](https://draveness.me/golang/tree/cmd/compile/internal/gc.bmap) 函数重建它的结构
- 这里也可以看到，golang map的拉链法实际上是用**数组+数组**的方式实现的

```go
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}
```



##### 初始化

###### 字面量

```go
hash := map[string]int{"1": 2, "3": 4, "5": 6}
```

- 使用字面量初始化的方式最终都会通过 [`cmd/compile/internal/gc.maplit`](https://draveness.me/golang/tree/cmd/compile/internal/gc.maplit) 初始化：分为少于等于25和大于25两种情况，但无论是那种情况，使用字面量初始化的过程，都会使用 Go 语言中的关键字 `make` 来创建新的哈希并通过最原始的 `map[key]=val` 语法向哈希追加元素

```go
func maplit(n *Node, m *Node, init *Nodes) {
	a := nod(OMAKE, nil, nil)
	a.Esc = n.Esc
	a.List.Set2(typenod(n.Type), nodintconst(int64(n.List.Len())))
	litas(m, a, init)

	entries := n.List.Slice()
	if len(entries) > 25 {
		...
		return
	}

	// Build list of var[c] = expr.
	// Use temporaries so that mapassign1 can have addressable key, elem.
	...
}
```

- 当哈希表中的元素数量少于或者等于 25 个时，编译器会将字面量初始化的结构体转换成以下的代码，将所有的键值对一次加入到哈希表中。这种初始化的方式与的[数组](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array/)和[切片](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array-and-slice/)几乎完全相同

```go
hash := make(map[string]int, 3)
hash["1"] = 2
hash["3"] = 4
hash["5"] = 6
```

- 一旦哈希表中元素的数量超过了 25 个，编译器会创建两个数组分别存储键和值，这些键值对会通过如下所示的 for 循环加入哈希
  - 当然这里展开的两个切片 `vstatk` 和 `vstatv` 还会被编辑器继续展开，即前面切片的初始化展开方式

```go
hash := make(map[string]int, 26)
vstatk := []string{"1", "2", "3", ... ， "26"}
vstatv := []int{1, 2, 3, ... , 26}
for i := 0; i < len(vstak); i++ {
    hash[vstatk[i]] = vstatv[i]
}
```

- `//TODO` 需要注意的是：编译器对于分配在栈上和堆上的 map 处理的方式是不同的，分配在了栈上的 map 不会调用 `runtime.makemap`

###### 运行时

- 当创建的哈希被分配到栈上并且其容量小于 `BUCKETSIZE = 8` 时，Go 语言在编译阶段会使用如下方式快速初始化哈希，这也是编译器对小容量的哈希做的优化

```go
var h *hmap
var hv hmap
var bv bmap
h := &hv
b := &bv
h.buckets = b
h.hash0 = fashtrand0()
```

- 除了上述特定的优化之外，无论 `make` 是从哪里来的，只要我们使用 `make` 创建哈希，Go 语言编译器都会在[类型检查](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/)期间将它们转换成 [`runtime.makemap`](https://draveness.me/golang/tree/runtime.makemap)，使用字面量初始化哈希也只是语言提供的辅助工具，最后调用的都是 [`runtime.makemap`](https://draveness.me/golang/tree/runtime.makemap)。
  - 计算哈希占用的内存是否溢出或者超出能分配的最大值；
  - 调用 [`runtime.fastrand`](https://draveness.me/golang/tree/runtime.fastrand) 获取一个随机的哈希种子；
  - 根据传入的 `hint` 计算出需要的最小需要的桶的数量；
  - 使用 [`runtime.makeBucketArray`](https://draveness.me/golang/tree/runtime.makeBucketArray) 创建用于保存桶的数组；

```go
func makemap(t *maptype, hint int, h *hmap) *hmap {
	mem, overflow := math.MulUintptr(uintptr(hint), t.bucket.size)
	if overflow || mem > maxAlloc {
		hint = 0
	}

	if h == nil {
		h = new(hmap)
	}
	h.hash0 = fastrand()

	B := uint8(0)
	for overLoadFactor(hint, B) {
		B++
	}
	h.B = B

	if h.B != 0 {
		var nextOverflow *bmap
		h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
		if nextOverflow != nil {
			h.extra = new(mapextra)
			h.extra.nextOverflow = nextOverflow
		}
	}
	return h
}
```

- [`runtime.makeBucketArray`](https://draveness.me/golang/tree/runtime.makeBucketArray) 会根据传入的 `B` 计算出的需要创建的桶数量并在内存中分配一片连续的空间用于存储数据：这里就可以发现正常桶和溢出桶在内存中的存储空间是连续的，只是被 [`runtime.hmap`](https://draveness.me/golang/tree/runtime.hmap) 中的不同字段 `buckets`、`extra.overflow` 引用，当溢出桶数量较多时会通过 [`runtime.newobject`](https://draveness.me/golang/tree/runtime.newobject) 创建新的溢出桶。
  - 当桶的数量小于 2^4 时，由于数据较少、使用溢出桶的可能性较低，会省略创建的过程以减少额外开销；
  - 当桶的数量多于 2^4 时，会额外创建 2^(B−4) 个溢出桶；

```go
func makeBucketArray(t *maptype, b uint8, dirtyalloc unsafe.Pointer) (buckets unsafe.Pointer, nextOverflow *bmap) {
	base := bucketShift(b)
	nbuckets := base // 总桶==正常桶
	if b >= 4 {
		nbuckets += bucketShift(b - 4) // 总桶+=溢出桶
		sz := t.bucket.size * nbuckets
		up := roundupsize(sz)
		if up != sz {
			nbuckets = up / t.bucket.size
		}
	}

	buckets = newarray(t.bucket, int(nbuckets)) //包含了溢出桶数量newarray的
	if base != nbuckets {
		nextOverflow = (*bmap)(add(buckets, base*uintptr(t.bucketsize)))
		last := (*bmap)(add(buckets, (nbuckets-1)*uintptr(t.bucketsize)))
		last.setoverflow(t, (*bmap)(buckets))
	}
	return buckets, nextOverflow
}
```

##### 操作

- 读：我们在代码中，对哈希表的访问一般都是通过下标或者遍历进行的，两者都能读取哈希表的数据，但是使用的函数和底层原理完全不同。前者需要知道哈希的键并且一次只能获取单个键对应的值，而后者可以遍历哈希中的全部键值对，访问数据时也不需要预先知道哈希的键。这里介绍第一种方式，第二种访问方式会在 `range` 一节中详细分析

```go
_ = hash[key]

for k, v := range hash {
    // k, v
}
```

- 写：数据结构的写一般指的都是增加、删除和修改，增加和修改字段都使用索引和赋值语句，而删除字典中的数据需要使用关键字 `delete`

```go
hash[key] = value
hash[key] = newValue
delete(hash, key)
```

###### 访问

- 在编译的[类型检查](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/)期间，`hash[key]` 以及类似的操作都会被转换成哈希的 `OINDEXMAP` 操作，[中间代码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)阶段会在 [`cmd/compile/internal/gc.walkexpr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.walkexpr) 函数中将这些 `OINDEXMAP` 操作转换成如下的代码
- 赋值语句左侧接受参数的个数会决定使用的运行时方法：
  - 当接受一个参数时，会使用 [`runtime.mapaccess1`](https://draveness.me/golang/tree/runtime.mapaccess1)，该函数仅会返回一个指向目标值的指针；
  - 当接受两个参数时，会使用 [`runtime.mapaccess2`](https://draveness.me/golang/tree/runtime.mapaccess2)，除了返回目标值之外，它还会返回一个用于表示当前键对应的值是否存在的 `bool` 值

```go
v     := hash[key] // => v     := *mapaccess1(maptype, hash, &key)
v, ok := hash[key] // => v, ok := mapaccess2(maptype, hash, &key)
```

- [`runtime.mapaccess1`](https://draveness.me/golang/tree/runtime.mapaccess1) 会先通过哈希表设置的哈希函数、种子获取当前键对应的哈希，再通过 [`runtime.bucketMask`](https://draveness.me/golang/tree/runtime.bucketMask) 和 [`runtime.add`](https://draveness.me/golang/tree/runtime.add) 拿到该键值对所在的桶序号和哈希高位的 8 位数字。
  - 在 `bucketloop` 循环中，哈希会依次遍历正常桶和溢出桶中的数据，它会先比较哈希的高 8 位和桶中存储的 `tophash`，后比较传入的和桶中的值以加速数据的读写。用于选择桶序号的是哈希的最低几位（因为桶序号是取模或者位操作获得的，得到的本来就是最低几位），而用于加速访问（桶内）的是哈希的高 8 位（一个桶8个数据，8*8=64bit，可以一次性取出所有的tophash），这种设计能够减少同一个桶中有大量相等 `tophash` 的概率影响性能。

```go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	alg := t.key.alg
	hash := alg.hash(key, uintptr(h.hash0))
	m := bucketMask(h.B)
	b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
	top := tophash(hash)
bucketloop:
	for ; b != nil; b = b.overflow(t) {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if alg.equal(key, k) {
				v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
				return v
			}
		}
	}
	return unsafe.Pointer(&zeroVal[0])
}
```

![](./img/map_read_1.png)



- 另一个同样用于访问哈希表中数据的 [`runtime.mapaccess2`](https://draveness.me/golang/tree/runtime.mapaccess2) 只是在 [`runtime.mapaccess1`](https://draveness.me/golang/tree/runtime.mapaccess1) 的基础上多返回了一个标识键值对是否存在的 `bool` 值。使用 `v, ok := hash[k]` 的形式访问哈希表中元素时，我们能够通过这个布尔值更准确地知道当 `v == nil` 时，`v` 到底是哈希中存储的元素还是表示该键对应的元素不存在，所以在访问哈希时，更推荐使用这种方式判断元素是否存在。

```go
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool) {
	...
bucketloop:
	for ; b != nil; b = b.overflow(t) {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if alg.equal(key, k) {
				v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
				return v, true
			}
		}
	}
	return unsafe.Pointer(&zeroVal[0]), false
}
```

- 上面的过程是在正常情况下，访问哈希表中元素时的表现，然而与数组一样，哈希表可能会在装载因子过高或者溢出桶过多时进行扩容，哈希表扩容并不是原子过程，在扩容的过程中保证哈希的访问是比较有意思的话题，我们在这里也省略了相关的代码，后面的小节会展开介绍

###### 写入

- 当形如 `hash[k]` 的表达式出现在赋值符号左侧时，该表达式会在编译期间转换成 [`runtime.mapassign`](https://draveness.me/golang/tree/runtime.mapassign) 函数的调用，该函数与 [`runtime.mapaccess1`](https://draveness.me/golang/tree/runtime.mapaccess1) 比较相似，我们将其分成几个部分依次分析
- 首先是函数会根据传入的键拿到对应的哈希和桶：

```go
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	alg := t.key.alg
	hash := alg.hash(key, uintptr(h.hash0))

	h.flags ^= hashWriting

again:
	bucket := hash & bucketMask(h.B)
	b := (*bmap)(unsafe.Pointer(uintptr(h.buckets) + bucket*uintptr(t.bucketsize)))
	top := tophash(hash)
```

- 然后通过遍历比较桶中存储的 `tophash` 和键的哈希，如果找到了相同结果就会返回目标位置的地址。其中 `inserti` 表示目标元素的在桶中的索引，`insertk` 和 `val` 分别表示键值对的地址，获得目标地址之后会通过算术计算寻址获得键值对 `k` 和 `val`： 
  - for 循环会依次遍历定位到的正常桶和溢出桶中存储的数据，整个过程会分别判断 `tophash` 是否相等、`key` 是否相等，遍历结束后会从循环中跳出。

```go
	var inserti *uint8
	var insertk unsafe.Pointer
	var val unsafe.Pointer
bucketloop:
	for {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if isEmpty(b.tophash[i]) && inserti == nil {
					inserti = &b.tophash[i]
					insertk = add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
					val = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
				}
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			if !alg.equal(key, k) {
				continue
			}
			val = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
			goto done
		}
		ovf := b.overflow(t)
		if ovf == nil {
			break
		}
		b = ovf
	}
```

- 如果当前桶已经满了，哈希会调用 [`runtime.hmap.newoverflow`](https://draveness.me/golang/tree/runtime.hmap.newoverflow) 创建新桶或者使用 [`runtime.hmap`](https://draveness.me/golang/tree/runtime.hmap) 预先在 `noverflow` 中创建好的桶来保存数据，新创建的桶不仅会被追加到已有桶的末尾，还会增加哈希表的 `noverflow` 计数器。

```go
	if inserti == nil {
		newb := h.newoverflow(t, b)
		inserti = &newb.tophash[0]
		insertk = add(unsafe.Pointer(newb), dataOffset)
		val = add(insertk, bucketCnt*uintptr(t.keysize))
	}

	typedmemmove(t.key, insertk, key)
	*inserti = top
	h.count++

done:
	return val
}
```

- 如果当前键值对在哈希中不存在，哈希会为新键值对规划存储的内存地址，通过 [`runtime.typedmemmove`](https://draveness.me/golang/tree/runtime.typedmemmove) 将键移动到对应的内存空间中并返回键对应值的地址 `val`。如果当前键值对在哈希中存在，那么就会直接返回目标区域的内存地址，哈希并不会在 [`runtime.mapassign`](https://draveness.me/golang/tree/runtime.mapassign) 这个运行时函数中将值拷贝到桶中，该函数只会返回内存地址，真正的赋值操作是在编译期间插入的
  - [`runtime.mapassign_fast64`](https://draveness.me/golang/tree/runtime.mapassign_fast64) 与 [`runtime.mapassign`](https://draveness.me/golang/tree/runtime.mapassign) 函数的逻辑差不多，我们需要关注的是后面的三行代码，其中 `24(SP)` 是该函数返回的值地址，我们通过 `LEAQ` 指令将字符串的地址存储到寄存器 `AX` 中，`MOVQ` 指令将字符串 `"88"` 存储到了目标地址上完成了这次哈希的写入。

```go
00018 (+5) CALL runtime.mapassign_fast64(SB)
00020 (5) MOVQ 24(SP), DI               ;; DI = &value
00026 (5) LEAQ go.string."88"(SB), AX   ;; AX = &"88"
00027 (5) MOVQ AX, (DI)                 ;; *DI = AX
```



###### 扩容

- 简单总结一下哈希表扩容的设计和原理，哈希在存储元素过多时会触发扩容操作，每次都会将桶的数量翻倍，扩容过程不是原子的，而是通过 [`runtime.growWork`](https://draveness.me/golang/tree/runtime.growWork) 增量触发的，在扩容期间访问哈希表时会使用旧桶，向哈希表写入数据时会触发旧桶元素的分流。除了这种正常的扩容之外，为了解决大量写入、删除造成的内存泄漏问题，哈希引入了 `sameSizeGrow` 这一机制，在出现较多溢出桶时会整理哈希的内存减少空间的占用。
- 前面在介绍哈希的写入过程时其实省略了扩容操作，随着哈希表中元素的逐渐增加，哈希的性能会逐渐恶化，所以需要更多的桶和更大的内存保证哈希的读写性能： 
- [`runtime.mapassign`](https://draveness.me/golang/tree/runtime.mapassign) 函数会在以下两种情况发生时触发哈希的扩容：
  1. 装载因子已经超过 6.5；
  2. 哈希使用了太多溢出桶；

```go
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	...
	if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
		hashGrow(t, h)
		goto again
	}
	...
}
```

- 如果扩容是溢出的桶太多导致的，那么这次扩容就是等量扩容 `sameSizeGrow`，`sameSizeGrow` 是一种特殊情况下发生的扩容，当我们持续向哈希中插入数据并将它们全部删除时，如果哈希表中的数据量没有超过阈值，就会不断积累溢出桶造成缓慢的内存泄漏[4](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/#fn:4)。[runtime: limit the number of map overflow buckets](https://github.com/golang/go/commit/9980b70cb460f27907a003674ab1b9bea24a847c) 引入了 `sameSizeGrow` 通过复用已有的哈希扩容机制解决该问题，一旦哈希中出现了过多的溢出桶，它会创建新桶保存数据，垃圾回收会清理老的溢出桶（即结构中的`extra.oldoverflow`）并释放内存[5](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/#fn:5)。
- 扩容的入口是 [`runtime.hashGrow`](https://draveness.me/golang/tree/runtime.hashGrow)。哈希在扩容的过程中会通过 [`runtime.makeBucketArray`](https://draveness.me/golang/tree/runtime.makeBucketArray) 创建一组新桶和预创建的溢出桶，随后将原有的桶数组设置到 `oldbuckets` 上并将新的空桶设置到 `buckets` 上，溢出桶也使用了相同的逻辑更新

```go
func hashGrow(t *maptype, h *hmap) {
	bigger := uint8(1)
	if !overLoadFactor(h.count+1, h.B) {
		bigger = 0
		h.flags |= sameSizeGrow
	}
	oldbuckets := h.buckets
	newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)

	h.B += bigger
	h.flags = flags
	h.oldbuckets = oldbuckets
	h.buckets = newbuckets
	h.nevacuate = 0
	h.noverflow = 0

	h.extra.oldoverflow = h.extra.overflow
	h.extra.overflow = nil
	h.extra.nextOverflow = nextOverflow
}
```

- 在 [`runtime.hashGrow`](https://draveness.me/golang/tree/runtime.hashGrow) 中还看不出来等量扩容和翻倍扩容的太多区别，等量扩容创建的新桶数量只是和旧桶一样，该函数中只是创建了新的桶，并没有对数据进行拷贝和转移。哈希表的数据迁移的过程在是 [`runtime.evacuate`](https://draveness.me/golang/tree/runtime.evacuate) 中完成的，它会对传入桶中的元素进行再分配。

```go
func evacuate(t *maptype, h *hmap, oldbucket uintptr) {
	b := (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))
	newbit := h.noldbuckets()
	if !evacuated(b) {
		var xy [2]evacDst
		x := &xy[0]
		x.b = (*bmap)(add(h.buckets, oldbucket*uintptr(t.bucketsize)))
		x.k = add(unsafe.Pointer(x.b), dataOffset)
		x.v = add(x.k, bucketCnt*uintptr(t.keysize))

		y := &xy[1]
		y.b = (*bmap)(add(h.buckets, (oldbucket+newbit)*uintptr(t.bucketsize)))
		y.k = add(unsafe.Pointer(y.b), dataOffset)
		y.v = add(y.k, bucketCnt*uintptr(t.keysize))
```

- 如果这是等量扩容，那么旧桶与新桶之间是一对一的关系，所以两个 [`runtime.evacDst`](https://draveness.me/golang/tree/runtime.evacDst) 只会初始化一个；而当哈希表的容量翻倍时，每个旧桶的元素会都分流到新创建的两个桶中，所以它会创建两个用于保存分配上下文的 [`runtime.evacDst`](https://draveness.me/golang/tree/runtime.evacDst) 结构体，这两个结构体分别指向了一个新桶

```go
		for ; b != nil; b = b.overflow(t) {
			k := add(unsafe.Pointer(b), dataOffset)
			v := add(k, bucketCnt*uintptr(t.keysize))
			for i := 0; i < bucketCnt; i, k, v = i+1, add(k, uintptr(t.keysize)), add(v, uintptr(t.valuesize)) {
				top := b.tophash[i]
				k2 := k
				var useY uint8
				hash := t.key.alg.hash(k2, uintptr(h.hash0))
				if hash&newbit != 0 {
					useY = 1
				}
				b.tophash[i] = evacuatedX + useY
				dst := &xy[useY]

				if dst.i == bucketCnt {
					dst.b = h.newoverflow(t, dst.b)
					dst.i = 0
					dst.k = add(unsafe.Pointer(dst.b), dataOffset)
					dst.v = add(dst.k, bucketCnt*uintptr(t.keysize))
				}
				dst.b.tophash[dst.i&(bucketCnt-1)] = top
				typedmemmove(t.key, dst.k, k)
				typedmemmove(t.elem, dst.v, v)
				dst.i++
				dst.k = add(dst.k, uintptr(t.keysize))
				dst.v = add(dst.v, uintptr(t.valuesize))
			}
		}
		...
}
```

![](./img/hashmap-evacuate-destination.png)

- 只使用哈希函数是不能定位到具体某一个桶的，哈希函数只会返回很长的哈希，例如：`b72bfae3f3285244c4732ce457cca823bc189e0b`，还需一些方法将哈希映射到具体的桶上。一般都会使用取模或者位操作来获取桶的编号，假如当前哈希中包含 4 个桶，那么它的桶掩码就是 0b11(3)，使用位操作就会得到 3， 我们就会在 3 号桶中存储该数据： 

```ruby
0xb72bfae3f3285244c4732ce457cca823bc189e0b & 0b11 #=> 0
```

- 如果新的哈希表有 8 个桶，在大多数情况下，原来经过桶掩码 `0b11` 结果为 3 的数据会因为桶掩码增加了一位变成 `0b111` 而分流到新的 3 号和 7 号桶，所有数据也都会被 [`runtime.typedmemmove`](https://draveness.me/golang/tree/runtime.typedmemmove) 拷贝到目标桶中：
- [`runtime.evacuate`](https://draveness.me/golang/tree/runtime.evacuate) 最后会调用 [`runtime.advanceEvacuationMark`](https://draveness.me/golang/tree/runtime.advanceEvacuationMark) 增加哈希的 `nevacuate` 计数器并在所有的旧桶都被分流后清空哈希的 `oldbuckets` 和 `oldoverflow`：

```go
func advanceEvacuationMark(h *hmap, t *maptype, newbit uintptr) {
	h.nevacuate++
	stop := h.nevacuate + 1024
	if stop > newbit {
		stop = newbit
	}
	for h.nevacuate != stop && bucketEvacuated(t, h, h.nevacuate) {
		h.nevacuate++
	}
	if h.nevacuate == newbit { // newbit == # of oldbuckets
		h.oldbuckets = nil
		if h.extra != nil {
			h.extra.oldoverflow = nil
		}
		h.flags &^= sameSizeGrow
	}
}
```

- 之前在分析哈希表访问函数 [`runtime.mapaccess1`](https://draveness.me/golang/tree/runtime.mapaccess1) 时其实省略了扩容期间获取键值对的逻辑，当哈希表的 `oldbuckets` 存在时，会先定位到旧桶并在该桶没有被分流时从中获取键值对。因为旧桶中的元素还没有被 [`runtime.evacuate`](https://draveness.me/golang/tree/runtime.evacuate) 函数分流，其中还保存着我们需要使用的数据，所以旧桶会替代新创建的空桶提供数据

```go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	...
	alg := t.key.alg
	hash := alg.hash(key, uintptr(h.hash0))
	m := bucketMask(h.B)
	b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
	if c := h.oldbuckets; c != nil {
		if !h.sameSizeGrow() {
			m >>= 1
		}
		oldb := (*bmap)(add(c, (hash&m)*uintptr(t.bucketsize)))
		if !evacuated(oldb) {
			b = oldb
		}
	}
bucketloop:
	...
}
```

- 在 [`runtime.mapassign`](https://draveness.me/golang/tree/runtime.mapassign) 函数中也省略了一段逻辑，当哈希表正在处于扩容状态时，每次向哈希表写入值时都会触发 [`runtime.growWork`](https://draveness.me/golang/tree/runtime.growWork) 增量拷贝哈希表中的内容：

```go
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	...
again:
	bucket := hash & bucketMask(h.B)
	if h.growing() {
		growWork(t, h, bucket)
	}
	...
}
```

- 当然除了写入操作之外，删除操作也会在哈希表扩容期间触发 [`runtime.growWork`](https://draveness.me/golang/tree/runtime.growWork)，触发的方式和代码与这里的逻辑几乎完全相同，都是计算当前值所在的桶，然后拷贝桶中的元素。 



###### 删除

- 要删除哈希中的元素，就需要使用 Go 语言中的 `delete` 关键字，这个关键字的唯一作用就是将某一个键对应的元素从哈希表中删除，无论是该键对应的值是否存在，这个内建的函数都不会返回任何的结果。

- 在编译期间，`delete` 关键字会被转换成操作为 `ODELETE` 的节点，而 [`cmd/compile/internal/gc.walkexpr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.walkexpr) 会将 `ODELETE` 节点转换成 [`runtime.mapdelete`](https://draveness.me/golang/tree/runtime.mapdelete) 函数簇中的一个，包括 [`runtime.mapdelete`](https://draveness.me/golang/tree/runtime.mapdelete)、`mapdelete_faststr`、`mapdelete_fast32` 和 `mapdelete_fast64`：

```go
func walkexpr(n *Node, init *Nodes) *Node {
	switch n.Op {
	case ODELETE:
		init.AppendNodes(&n.Ninit)
		map_ := n.List.First()
		key := n.List.Second()
		map_ = walkexpr(map_, init)
		key = walkexpr(key, init)

		t := map_.Type
		fast := mapfast(t)
		if fast == mapslow {
			key = nod(OADDR, key, nil)
		}
		n = mkcall1(mapfndel(mapdelete[fast], t), nil, init, typename(t), map_, key)
	}
}
```

- 这些函数的实现其实差不多，挑选其中的 [`runtime.mapdelete`](https://draveness.me/golang/tree/runtime.mapdelete) 分析一下。哈希表的删除逻辑与写入逻辑很相似，只是触发哈希的删除需要使用关键字，如果在删除期间遇到了哈希表的扩容，就会分流桶中的元素，分流结束之后会找到桶中的目标元素完成键值对的删除工作。

```go
func mapdelete(t *maptype, h *hmap, key unsafe.Pointer) {
	...
	if h.growing() {
		growWork(t, h, bucket)
	}
	...
search:
	for ; b != nil; b = b.overflow(t) {
		for i := uintptr(0); i < bucketCnt; i++ {
			if b.tophash[i] != top {
				if b.tophash[i] == emptyRest {
					break search
				}
				continue
			}
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
			k2 := k
			if !alg.equal(key, k2) {
				continue
			}
			*(*unsafe.Pointer)(k) = nil
			v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
			*(*unsafe.Pointer)(v) = nil
			b.tophash[i] = emptyOne
			...
		}
	}
}
```

- 其实只需要知道 `delete` 关键字在编译期间经过[类型检查](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/)和[中间代码生成](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-ir-ssa/)阶段被转换成 [`runtime.mapdelete`](https://draveness.me/golang/tree/runtime.mapdelete) 函数簇中的一员，用于处理删除逻辑的函数与哈希表的 [`runtime.mapassign`](https://draveness.me/golang/tree/runtime.mapassign) 几乎完全相同，不太需要刻意关注。



##### 使用

```go
func main() {
	/*创建集合。m := map[string]string{"1": "一", "2": "二", "3": "三"}*/
	m := make(map[string]string) //编译器会调用runtime.makemap，该方法实际返回一个指针
	m["1"] = "一"
	m["2"] = "二"
	m["3"] = "三"
	/*取值*/
	v, b := m["1"]
	fmt.Println(v, b)
	/*遍历。多次运行之间的结果是不定的，一次运行中的多次遍历结果是固定的*/
	for k, v := range m {
		fmt.Println(k, "：", v)
	}
	for k := range m {
		fmt.Println(k, "：", m [k])
	}
	/*删除*/
	delete(m, "3")
	/*传参*/
	fmt.Printf("map的内存地址是：%p\n", m) //map的内存地址是：0xc000062330
	modify1(m) //modify1接收到的map的内存地址是：0xc000062330
	modify2(m) //modify2接收到的map的内存地址是：0xc000062330
	fmt.Println(m) //map[1:壹 2:贰]
}
func modify1(m map[string]string){
	fmt.Printf("modify1接收到的map的内存地址是：%p\n", m)
	m["1"] = "壹"
}
func modify2(p interface{}){ //接口变量接收，类似于多态
	fmt.Printf("modify2接收到的map的内存地址是：%p\n", p)
	m := p.(map[string]string)
	m["2"] = "贰"
}

```

基于 go 实现简单 HashMap，暂未做 key 值的校验

```go
package main

import (
    "fmt"
)

type HashMap struct {
    key string
    value string
    hashCode int
    next *HashMap
}

var table [16](*HashMap)

func initTable() {
    for i := range table{
        table[i] = &HashMap{"","",i,nil}
    }
}

func getInstance() [16](*HashMap){
    if(table[0] == nil){
        initTable()
    }
    return table
}

func genHashCode(k string) int{
    if len(k) == 0{
        return 0
    }
    var hashCode int = 0
    var lastIndex int = len(k) - 1
    for i := range k {
        if i == lastIndex {
            hashCode += int(k[i])
            break
        }
        hashCode += (hashCode + int(k[i]))*31
    }
    return hashCode
}

func indexTable(hashCode int) int{
    return hashCode%16
}

func indexNode(hashCode int) int {
    return hashCode>>4
}

func put(k string, v string) string {
    var hashCode = genHashCode(k)
    var thisNode = HashMap{k,v,hashCode,nil}

    var tableIndex = indexTable(hashCode)
    var nodeIndex = indexNode(hashCode)

    var headPtr [16](*HashMap) = getInstance()
    var headNode = headPtr[tableIndex]

    if (*headNode).key == "" {
        *headNode = thisNode
        return ""
    }

    var lastNode *HashMap = headNode
    var nextNode *HashMap = (*headNode).next

    for nextNode != nil && (indexNode((*nextNode).hashCode) < nodeIndex){
        lastNode = nextNode
        nextNode = (*nextNode).next
    }
    if (*lastNode).hashCode == thisNode.hashCode {
        var oldValue string = lastNode.value
        lastNode.value = thisNode.value
        return oldValue
    }
    if lastNode.hashCode < thisNode.hashCode {
        lastNode.next = &thisNode
    }
    if nextNode != nil {
        thisNode.next = nextNode
    }
    return ""
}

func get(k string) string {
    var hashCode = genHashCode(k)
    var tableIndex = indexTable(hashCode)

    var headPtr [16](*HashMap) = getInstance()
    var node *HashMap = headPtr[tableIndex]

    if (*node).key == k{
        return (*node).value
    }

    for (*node).next != nil {
        if k == (*node).key {
            return (*node).value
        }
        node = (*node).next
    }
    return ""
}

//examples 
func main() {
    getInstance()
    put("a","a_put")
    put("b","b_put")
    fmt.Println(get("a"))
    fmt.Println(get("b"))
    put("p","p_put")
    fmt.Println(get("p"))
}

```





### 类型转换

go语言只有强制类型转换

```go
func TypeConversion() {
	a, b := 1, 2
	var c float64 = float64(a + b) //格式type_name(expression)
	fmt.Println(c)
}
```

