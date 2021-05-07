

- 反射
  - 反射是一种让程序-主要是通过类型一 理解其自 身结构的一种能力
  - 在计算机科学中，反射是指计算机程序在运行时(Run time)可以访问、检测和修改它本身状态或行为的一种能力。用比喻来说，反射就是程序在运行的时候能够“观察"并且修改自己的行为。
  - Go语言提供了一种机制在运行时更新变量和检查它们的值、调用它们的方法，但是在编译时并不知道这些变
    量的具体类型，这称为反射机制。
  - 静态类型：编译时就知道的变量类型。入变量声明时的赋予的类型：var i int。
  - 动态类型：运行时才知道的变量类型。比如运行时给一个接口变量赋值，这个值的类型(如果值为nil的时候没有动态类型)：var a interface{} 此时interface{}为静态类型，a=10赋值后有int为动态类型。一个变量的动态类型在运行时可能改变，这主要依赖于它的赋值(前提是这个变量是接口类型)
- go语言特点
  - go语言是静态：编译时类型已经确定，比如对已基本数据类型的再定义后的类型，反射时候需要确认返回的
    类型语言是何种类型。
  - 空接口interface{}：go的反射机制是要通过接口来进行的，而类似于Java的Object的空接口可以和任何类型进行
    交互，因此对基本数据类型等的反射也直接利用了这一特点
- Go语言的类型: 
  - 变量包括(type, value)两部分，理解这一-. 点就知道为什么nil!= nil了
  - type包括static type和coqcrete type.简单来说static type是你在编码是看见的类型(如int. string),concrete type是runtime系统看见的类型
  - 类型断言能否成功，取决于变量的concrete type, 而不是static type。 因此，-个reader变量如果它的concrete type也实现了write方法的话，它也可以被类型断言为writer.
- Go语言的反射就是建立在类型之.上的，Golang的指定类型的变量的类型是静态的(也就是指定int. string这 些的
  变量，它的type是static type)，在创建变量的时候就已经确定，反射主要与Golang的interface类型相关(它的
  type是concrete type)， 只有interface类型才有反射一 -说。
- 使用建议：除非必要不要使用反射，反射对性能比普通代码低一两个数量级，期且反射相关代码在编译时无法检测错误，可能运行很久之后panic导致程序终止，且使用反射代码可读性很差

## reflect

#### pair

在Golang的实现中，每个interface变量都有一 个对应pair, pair中记录了实际变量的值和类型：(value, type)
value是实际变量值，type是 实际变量的类型。-个interface{}类型的变量包含了2个指针，- -个指针指向值的类型[对应concrete type]，另外-个指针指向实际的值[对应value] 。

```go
func main() {
	file, err := os.OpenFile("demo.txt", os.O_RDONLY, os.ModePerm)
	fmt.Println(err)
	var r io.Reader
	r = file //将变量file赋值给接口变量r，则变量r的pair(对)中将记录这样的信息：(file, *os.File)
	var w io.Writer
	w = r.(io.Writer) //pair在接口变量的连续赋值过程中是不变的，仍然是：(file, *os.File)
	//反射就是用来检测存储在接口变量内部(值valge;类型concrete type) pair对的一种机制。
	reflect.TypeOf(w) //获取pair中的type
	reflect.ValueOf(w) //获取pair中的value
}
```

```go
func main() {
	var num float64 = 2.3
	t := reflect.TypeOf(num)
	fmt.Println("type:", t)
	v := reflect.ValueOf(num)
	fmt.Println("value:", v)
	fmt.Println("kind==float64?", v.Kind() == reflect.Float64) //kind种类，type类型，两者完全不同，如type Pseron struct{}，kind是struct，type是person
	fmt.Println("type:", v.Type())
	fmt.Println("value:", v.Float())
}
```



根据Go官方关于反射的博客，反射有三大定律: .

1. Reflection goes from interface value to reflection object：反射可以从接口值得到反射对象

   反射是一-种检测存储在interface中的类型和值机制。这可以通过TypeOf函数和ValueOf函数得到。

2. Reflection goes from reflection object to interface value：反射可以从反射对象获得接口值

   它将ValueOf的返回值通过Interface()函数反向转变成interface变量。前两条就是说接口型变量和反射类型对象可以相互转化，反射类型对象实际上就是指的前面说的reflect.Type和reflect.Value.

3. To modify a reflection object, the value must be settable：如果需要操作一个反射变量,则其值必须可以修改

  反射变量可设置的本质是它存储了原变量本身，这样对反射变量的操作，就会反映到原变量本身;反之，如果反射变量不能代表原变量，那么操作了反射变量，不会对原变量产生任何影响，这会给使用者带来疑惑。所以第二种情况在语言层面是不被允许的。

当执行reflect.ValueOfinterface)之后，就得到了-个类型为"relfect.value"变量，可以通过它本身的Interface()方
法获得接口变量的真实内容，然后可以通过类型判断进行转换，转换为原有真实类型。不过,我们可能是已知原有
类型，也有可能是未知原有类型，因此，下面分两种情况进行说明。

```go
//已知类型的情况
func main() {
	var num float64 = 2.3
	v := reflect.ValueOf(num) //"接口类型变量"->"反射类型对象"
	convertValue := v.Interface().(float64) //"反射类型对象"->"接口类型变量"，强制转换。前提是已知其类型为float64
	fmt.Println(convertValue) //2.3

	vPtr := reflect.ValueOf(&num)
	convertValuePtr := vPtr.Interface().(*float64)
	fmt.Println(convertValuePtr) //一个地址
}
```

```go
//反射修改值
func main() {
	var v float64 = 2.3
	vPtrRef := reflect.ValueOf(&v)          //只有传入指针获取的reflect.Value，才能实际修改原变量的值
	vRef := vPtrRef.Elem()                  //Elem返回接口v包含的值或指针v指向的值。如果v的类型不是interface或ptr,则panic。如果v为零，则返回零值。v表示反射对象，这里指vPtrRef，reflect.ValueOf中传入的是地址值，vPtrRef才会是一个ptr
	fmt.Println("type:", vRef.Type())       //type: float64
	fmt.Println("是否可以修改数据:", vRef.CanSet()) //是否可以修改数据: true。如果不是传入的指针，则将是false
	//如果你的变量v是一个指针、map、slice、 channel. Array。 那么你可以直接使用reflect.TypeOf(v).Elem()来确定包含的类型
	vRef.SetFloat(3.14) //重新赋值
	fmt.Println(v)      //3.14
}
```

```go
type User struct {
	Username string
	Password string
}

func (u User) SendMessage(tittle string, content string) {
	fmt.Println("SendMessage:", tittle, content)
}

func (u User) PrintInfo() {
	fmt.Println(u)
}

func main() {
	u1 := User{"username1", "password1"}
	getMessage(u1)

	//反射修改u1
	vPtr := reflect.ValueOf(&u1)
	if vPtr.Kind() == reflect.Ptr { //必须要是一个指针类型
		newValue := vPtr.Elem()
		fmt.Println(newValue.CanSet()) //true
		username := newValue.FieldByName("Username")
		username.SetString("username11")
		password := newValue.FieldByName("Password")
		password.SetString("password11")
		fmt.Println(u1) //{username11 password11}
	}

	v := reflect.ValueOf(u1)
	methodValue1 := v.MethodByName("PrintInfo")
	fmt.Println("kind:", methodValue1.Kind(), ", type:", methodValue1.Type()) //kind: func , type: func()
	methodValue1.Call(nil)                                                    //{username11 password11}
	// 通过Call方法调用，传入一个存放参数反射对象的切片[]reflect.Value。没有参数传入nil或者空切片即可

	methodValue2 := v.MethodByName("SendMessage")
	fmt.Println("kind:", methodValue2.Kind(), ", type:", methodValue2.Type()) //kind: func , type: func(string, string)
	args := []reflect.Value{reflect.ValueOf("标题"), reflect.ValueOf("内容")}
	methodValue2.Call(args) //SendMessage: 标题 内容

	f := fun
	funValue := reflect.ValueOf(f)
	fmt.Println("kind:", funValue.Kind(), ", type:", funValue.Type()) //kind: func , type: func(int, string) string
	funReturn := funValue.Call([]reflect.Value{reflect.ValueOf(123), reflect.ValueOf("fun")})
	fmt.Println(funReturn)                                //[123fun]，返回的是一个切片
	fmt.Println(funReturn[0].Kind(), funReturn[0].Type()) //string string
	fmt.Printf("%T\n", funReturn[0])                      //reflect.Value

	funReturnStr := funReturn[0].Interface().(string) //反射类型对象->接口类型变量
	fmt.Println(funReturnStr)                         //123fun
	fmt.Printf("%T\n", funReturnStr)                  //string
}

func fun(i int, s string) string {
	fmt.Println("fun被调用了")
	return strconv.Itoa(i) + s
}

func getMessage(input interface{}) {
	fmt.Println(input)

	t := reflect.TypeOf(input)
	fmt.Println("type:", t.Name()) //type: User
	fmt.Println("kind:", t.Kind()) //kind: struct

	v := reflect.ValueOf(input)
	fmt.Println("value:", v) //value: {username1 password1}

	//反射字段
	for i := 0; i < t.NumField(); i++ {
		typeField := t.Field(i)                            //类型信息。
		value := v.Field(i).Interface()                    //值。结构体中的字段必须是大写字母开头，否则panic
		fmt.Println(typeField.Name, typeField.Type, value) //Username string username1 \n Password string password1
	}
	//反射方法
	for i := 0; i < t.NumMethod(); i++ {
		method := t.Method(i)                 //reflect.Method。定义的方法名为大写字母开头才能被反射获取，否则获取不到
		fmt.Println(method.Name, method.Type) //PrintInfo func(main.User) \n SendMessage func(main.User)
	}
}
```

## canset

### 什么是可设置（ CanSet ）

首先需要先明确下，可设置是针对 reflect.Value 的。普通的变量要转变成为 reflect.Value 需要先使用 reflect.ValueOf() 来进行转化。

那么为什么要有这么一个“可设置”的方法呢？比如下面这个例子：

```
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println(v.CanSet()) // false
```

golang 里面的所有函数调用都是值复制，所以这里在调用 reflect.ValueOf 的时候，已经复制了一个 x 传递进去了，这里获取到的 v 是一个 x 复制体的 value。那么这个时候，我们就希望知道我能不能通过 v 来设置这里的 x 变量。就需要有个方法来辅助我们做这个事情：CanSet()

但是, 非常明显，由于我们传递的是 x 的一个复制，所以这里根本无法改变 x 的值。这里显示的就是 false。

那么如果我们把 x 的地址传递给里面呢？下面这个例子：

```
var x float64 = 3.4
v := reflect.ValueOf(&x)
fmt.Println(v.CanSet()) // false
```

我们将 x 变量的地址传递给 reflect.ValueOf 了。应该是 CanSet 了吧。但是这里却要注意一点，这里的 v 指向的是 x 的指针。所以 CanSet 方法判断的是 x 的指针是否可以设置。指针是肯定不能设置的，所以这里还是返回 false。

那么我们下面需要可以通过这个指针的 value 值来判断的是，这个指针指向的元素是否可以设置，所幸 reflect 提供了 Elem() 方法来获取这个“指针指向的元素”。

```
var x float64 = 3.4
v := reflect.ValueOf(&x)
fmt.Println(v.Elem().CanSet()) // true
```

终于返回 true 了。但是这个 Elem() 使用的时候有个前提，这里的 value 必须是指针对象转换的 reflect.Value。（或者是接口对象转换的 reflect.Value）。这个前提不难理解吧，如果是一个 int 类型，它怎么可能有指向的元素呢？所以，使用 Elem 的时候要十分注意这点，因为如果不满足这个前提，Elem 是直接触发 panic 的。

在判断完是否可以设置之后，我们就可以通过 SetXX 系列方法进行对应的设置了。

```
var x float64 = 3.4
v := reflect.ValueOf(&x)
if v.Elem().CanSet() {
    v.Elem().SetFloat(7.1)
}
fmt.Println(x)
```

### 更复杂的类型

对于复杂的 slice， map， struct， pointer 等方法，我写了一个例子：

```
package main

import (
 "fmt"
 "reflect"
)

type Foo interface {
 Name() string
}

type FooStruct struct {
 A string
}

func (f FooStruct) Name() string {
 return f.A
}

type FooPointer struct {
 A string
}

func (f *FooPointer) Name() string {
 return f.A
}

func main() {
 {
  // slice
  a := []int{1, 2, 3}
  val := reflect.ValueOf(&a)
  val.Elem().SetLen(2)
  val.Elem().Index(0).SetInt(4)
  fmt.Println(a) // [4,2]
 }
 {
  // map
  a := map[int]string{
   1: "foo1",
   2: "foo2",
  }
  val := reflect.ValueOf(&a)
  key3 := reflect.ValueOf(3)
  val3 := reflect.ValueOf("foo3")
  val.Elem().SetMapIndex(key3, val3)
  fmt.Println(val) // &map[1:foo1 2:foo2 3:foo3]
 }
 {
  // map
  a := map[int]string{
   1: "foo1",
   2: "foo2",
  }
  val := reflect.ValueOf(a)
  key3 := reflect.ValueOf(3)
  val3 := reflect.ValueOf("foo3")
  val.SetMapIndex(key3, val3)
  fmt.Println(val) // &map[1:foo1 2:foo2 3:foo3]
 }
 {
  // struct
  a := FooStruct{}
  val := reflect.ValueOf(&a)
  val.Elem().FieldByName("A").SetString("foo2")
  fmt.Println(a) // {foo2}
 }
 {
  // pointer
  a := &FooPointer{}
  val := reflect.ValueOf(a)
  val.Elem().FieldByName("A").SetString("foo2")
  fmt.Println(a) //&{foo2}
 }
}
```

上面的例子如果都能理解，那基本上也就理解了 CanSet 的方法了。

特别可以关注下，map，pointer 在修改的时候并不需要传递指针到 reflect.ValueOf 中。因为他们本身就是指针。

所以在调用 reflect.ValueOf 的时候，我们必须心里非常明确，我们要传递的变量的底层结构。比如 map， 实际上传递的是一个指针，我们没有必要再将他指针化了。而 slice， 实际上传递的是一个 SliceHeader 结构，我们在修改 Slice 的时候，必须要传递的是 SliceHeader 的指针。这点往往是需要我们注意的。

## CanAddr

在 reflect 包里面可以看到，除了 CanSet 之外，还有一个 CanAddr 方法。它们两个有什么区别呢？

CanAddr 方法和 CanSet 方法不一样的地方在于：对于一些结构体内的私有字段，我们可以获取它的地址，但是不能设置它。

比如下面的例子：

```
package main

import (
 "fmt"
 "reflect"
)

type FooStruct struct {
 A string
 b int
}


func main() {
 {
  // struct
  a := FooStruct{}
  val := reflect.ValueOf(&a)
  fmt.Println(val.Elem().FieldByName("b").CanSet())  // false
  fmt.Println(val.Elem().FieldByName("b").CanAddr()) // true
 }
}
```

所以，CanAddr 是 CanSet 的必要不充分条件。一个 Value 如果 CanAddr, 不一定 CanSet。但是一个变量如果 CanSet，它一定 CanAddr。