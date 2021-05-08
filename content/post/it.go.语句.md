

+++

title = "go.语句"
description = "go.语句"
tags = ["it", "go"]

+++



# 语句

控制语句：control statement



## 条件

### if

```go
func ifSentence() {
	content1, err1 := ioutil.ReadFile("if.txt")
	if err1 != nil {
		fmt.Println(err1)
	} else {
		fmt.Printf("%s\n", content1)
	}

	if content2, err2 := ioutil.ReadFile("if.txt"); err2 != nil { //if后的变量只在if的块中存活
		fmt.Println(err2)
	} else {
		fmt.Printf("%s\n", content2)
	}
}
```

### swich

```go
func switchSentence1(a, b int, op string) int {
	var result int
	switch op { //switch的每个case自动break，在case中使用fallthrough才会继续执行下一个case。
	case "+":
		result = a + b
	case "-":
		result = a - b
	case "*":
		result = a * b
	case "/":
		result = a / b
	default:
		panic("unsupported operator:" + op)
	}
	return result
}

func switchSentence2(score int) string {
	g := ""
	switch { //无变量
	case score < 0 || score > 100:
		panic(fmt.Sprintf("Wrong score: %d", score))
	case score < 60:
		g = "C"
	case score < 90:
		g = "B"
	case score <= 100:
		g = "A"
	}
	return g
}
```

##### 类型断言

```go
/*context.parentCancelCtx*/
func parentCancelCtx(parent Context) (*cancelCtx, bool) {
	for {
		switch c := parent.(type) {
		case *cancelCtx:
			return c, true
		case *timerCtx:
			return &c.cancelCtx, true
		case *valueCtx:
			parent = c.Context
		default:
			return nil, false
		}
	}
}
```

### select

```go
func selectSentence {
	select {
	//每个 case 都必须是一个通信。
	//所有 channel 表达式都会被求值。所有被发送的表达式都会被求值。
	//如果任意某个通信可以进行，它就执行，其他被忽略。
	//如果有多个 case 都可以运行，Select 会随机公平地选出一个执行，其他不会执行。
	//如果没有case可以执行，有default则执行default，没有default则select阻塞，直到某个通信可以运行，Go 不会重新对 channel 或值进行求值
	//select 会循环检测条件，如果有满足则执行并退出，否则一直循环检测
	case communication clause:
		statement(s)
	case communication clause:
		statement(s)
	/* 你可以定义任意数量的 case */
	default: /* 可选。有了default，即使得chan的获取变成了非阻塞式 */
		statement(s)
	}
}

```



## 循环

### for

```go
func forSentence() {
	fmt.Println(DecimalToBinary(13))
}

func DecimalToBinary(n int) string {
	result := ""
	for ; n>0; n/=2 { //for的三个条件都可以省略
		lsb := n%2
		result = strconv.Itoa(lsb) + result
	}
	return result
}

for key, value := range oldMap { //for range可以对array、slice、map、string、channel等进行迭代循环
    newMap[key] = value
}

```

### range

Go 语言中 range 关键字用于 for 循环中迭代数组(array)、切片(slice)、通道(channel)或集合(map)的元素。在数组和切片中它返回元素的索引和索引对应的值，在集合中返回 key-value 对

```go
nums := []int{2, 3, 4}
sum := 0
for _, num := range nums { //在数组上使用range将传入index和值两个变量,这里用不到index，所以用空白符"_"省略
    sum += num
}
fmt.Println("sum:", sum)

```
