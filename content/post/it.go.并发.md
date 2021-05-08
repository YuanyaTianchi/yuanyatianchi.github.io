

+++

title = "go.并发"
description = "go.并发"
tags = ["it", "go"]

+++



# 并发

并发是一个cpu核心快速分配时间片交替执行多个任务，并行是多个cpu核心分别执行自己的任务。核心数量受限于硬件，这时候就需要程序设计提高并发量，即并发编程

## 并发模型

- https://www.bilibili.com/video/av70488008?p=116
- 操作系统体系架构：无论语言层面何种并发模型，到了操作系统层面，一定是以线程的形态存在的，而操作系统根据资源访问权限的不同，体系架构可分为用户空间和内核空间
  - 内核空间：主要操作访问CPU资源、I/0资源、内存资源等硬件资源，为上层应用程序提供最基本的基础资源
  - 用户空间：上层应用程序的固定活动空间，用户空间不可以直接访问资源，必须通过“系统调用"、“库函数”或"Shell脚本"来调用内核空间提供的资源
- 计算机语言中的线程：现在的计算机语言，可以狭义的认为是一种"软件"， 它们中所谓的"线程"，往往是用户态的线程，和操作系统本身内核态的线程(简称KSE) ，是有区别的
- 线程实现模型：Go并发编程模型在底层是由操作系统所提供的线程库支撑的，因此还是得从线程实现模型说起。线程可以视为进程中的控制流。一个进程至少会包含一个线程，因为其中至少会有一个控制流持续运行。因而，一个进程的第一个线程会随着这个进程的启动而创建，这个线程称为该进程的主线程。当然，一个进程也可以包含多个线程。这些线程都是由当前进程中已存在的线程创建出来的，创建的方法就是调用系统调用，更确切地说是调用pthread create函数。拥有多个线程的进程可以并发执行多个任务，并且即使某个或某些任务被阻塞,也不会影响其它任务正常执行，这可以大大改善程序的响应时间和吞吐量。另一方面， 线程不可能独立于进程存在。它的生命周期不可能逾越其所属进程的生命周期。线程的实现模型主要有3个，分别是:用户级线程模型、内核级线程模型和两级线程模型。它们之间最大的差异就在于线程与内核调度实体( Kernel Scheduling Entity,简称KSE)之间的对应关系上。顾名思义，内核调度实体就是可以被内核的调度器调度的对象。在很多文献和书中，它也称为内核级线程，是操作系统内核的最小调度单元。
  - 内核级线程模型：用户线程与KSE是1对1关系(1:1)。大部分编程语言的线程库(如linux的pthread，Java的java.lang.Thread，C++11的td:thread等等)都是对操作系统的线程(内核级线程)的一层封装，创建出来的每个线程与一个不同的KSE静态关联，因此其调度完全由OS调度器来做。这种方式实现简单，直接借助OS提供的线程能力，并且不同用户线程之间一般也不会相互影响。但其创建，销毁以及多个线程之间的上下文切换等操作都是直接由OS层面亲自来做，在需要使用大量线程的场景下对OS的性能影响会很大。
  - 用户级线程模型：用户线程与KSE是多对1关系(M:1)，这种线程的创建，销毁以及多个线程之间的协调等操作都是由用户自己实现的线程库来负责，对OS内核透明，-一个进程中所有创建的线程都与同一个KSE在运行时动态关联。现在有许多语言实现的协程基本上都属于这种方式。这种实现方式相比内核级线程可以做的很轻量级，对系统资源的消耗会小很多，因此可以创建的数量与上下文切换所花费的代价也会小得多。但该模型有个致命的缺点，如果我们在某个用户线程上调用阻塞式系统调用(如用阻塞方式read网络I0),那么-. 旦KSE因阻塞被内核调度出CPU的话，剩下的所有对应的用户线程全都会变为阻塞状态(整个进程挂起)。所以这些语言的协程库会把自己一些阻塞的操作重新封装为完全的非阻塞形式，然后在以前要阻塞的点上，主动让出自己，并通过某种方式通知或唤醒其他待执行的用户线程在该KSE上运行,从而避免了内核调度器由于KSE阻塞而做上下文切换，这样整个进程也不会被阻塞了。
  - 两级线程模型：用户线程与KSE是多对多关系(M:N),这种实现综合了前两种模型的优点，为-个进程中创建多个KSE,并且线程可以与不同的KSE在运行时进行动态关联，当某个KSE由于其上工作的线程的阻塞操作被内核调度出CPU时，当前与其关联的其余用户线程可以重新与其他KSE建立关联关系。当然这种动态关联机制的实现很复杂，也需要用户自己去实现，这算是它的一个缺点吧。Go语言中的并发就是使用的这种实现方式，Go为了实现该模型自己实现了一个运行时调度器来负责Go中的"线程”与KSE的动态关联。此模型有时也被称为混合型线程模型,即用户调度器实现用户线程到KSE的"调度"，内核调度器实现KSE到CPU上的调度。

## 调度器结构

Go并发调度：G-P-M模型。在操作系统提供的内核线程之上，Go搭建了一个特有的两级线程模型。goroutine机制实现了 M:N的线程模型，goroutine机制是协程(coroutine) 的一种实现，golang内置的调度器，可以让多核CPU中每个CPU执行一个协程。详见并发模型笔记

GPM

- 
- 

## 调度算法

- 



## goroutine ：go

- goroutine ：go中对协程(coroutine)的命名
  - **非抢占式**多任务处理，由协程主动交出控制权。相比与线程，资源消耗更少
  - 编译器/解释器/虚拟机层面的多任务
  - 多个协程可能在一个或多个线程.上运行

主goroutine
封装main函数的goroutine称为主goroutine。主goroutine所做的事情并不是执行main函数那么简单。它首先要做的是:设定每一个goroutine所能 申请的栈空间的最大尺寸。在32位的计算机系统中此最大尺寸为250MB，而在64位的计算机系统中此尺寸为1GB。如果有某个goroutine的栈空间尺寸大于这个限制，那么运行时系统就会引发-个栈溢出(stack overflow)的运行时恐慌。随后，这个go程序的运行也会终止。
此后，主goroutine会进行 -系列的初始化工作,涉及的工作内容大致如下:
1.创建一个特殊的defer语句，用于在主goroutine退出时做必要的善 后处理。因为主goroutine也可能非正常的
结束
2.启动专用于在后台清扫内存垃圾的goroutine,并设置GC可用的标识
3.执行mian包中的init函数
4.执行main函数
执行完main函数后，它还会检查主goroutine是否引发 了运行时恐慌，并进行必要的处理。最后主goroutine
会结束自己以及当前进程的运行。

当一个goroutine发生阻塞，Go 会自动地把与该goroutine处于同一系统线程的其他goroutine转移到另一个系统线程上去，以使这些goroutine不阻塞。

python定义协程：async def。定义为异步方法实现协程。没有go灵活，通过go关键字可以将任何普通方法协程运行

```go
func main() {
	go func() { //go关键字开启一个goroutine
		for i:=0; i<9; i++ {
			fmt.Println("other") //将不能执行9次，因为程序将随着当main线程的结束而结束，所有其它线程将随着程序的结束而中断
		}
	}()
	fmt.Println("main")
}
```

#### 非抢占式

```go
func main() {
	var a [10] int
	for i := 0; i < 10; i++ {
		go func(i int) {
			for {
                a[i]++ //永远不会交出cpu控制权。io操作会交出控制权如fmt.Print，或者runtime.Gosched()主动交出控制权
			}
		}(i)
	}
	time.Sleep(time.Millisecond) //主线程虽然只Sleep了1秒，但是如果得不到cpu控制权也将永远停在这里
	fmt.Println(a)
}
/*经测试与上述不符，可能是我的cpu线程较多(16),也可能是新版本go中对goroutine进行了优化*/
```

#### data race：数据竞争

go -race Xxx.go：这样运行可以看到data race信息

```go
//i的竞争：主线程在i++写i的时候，其它协程在a[i]++中读i。因为会有out of range的报错，很容易注意到i的读写竞争
//a的竞争：当协程仍在a[i]++写a的时候，主线程在fmt.Println(a)中读a。可以通过channel或其它sync方法解决解决
func main() {
	var a [10] int
	for i := 0; i < 10; i++ { //最后一次判定i++到10，跳出循环
		go func() {
			for {
                a[i]++ //i++到10又被a[i]引用，a[10]将out of range
			}
		}()
	}
	time.Sleep(time.Millisecond)
	fmt.Println(a)
}
```

如下，运行不会有任何报错，但是通过 go -race Xxx.go 会发现仍然存在data rance

```go
func main() {
	var a [10] int
	for i := 0; i < 10; i++ {
		go func(i int) {
			for {
                a[i]++ //读a
			}
		}(i)
	}
	time.Sleep(time.Millisecond)
	fmt.Println(a) //写a
}
```

#### go语言的调度器

调度协程的在线程上运行，一个线程上可以运行多个协程

- 任何函数只需加上go就能送给调度器运行
- 不需要在定义时区分是否是异步函数
- 调度器在合适的点进行切换
  - 切换点：仅作参考，不能保证一定切换，不能保证在其他地方一定不切换
    - I/O，select
    - channel
    - 函数调用(有时)
    - runtime.Gosched()
    - 等待锁

### time

```go
func main() {
	time.Sleep(time.Second*1)
	fmt.Println("Sleep时间到")

	timeChannel := time.After(time.Second*1) //定时1s，1s后，创建channel，向其中发送实时时间数据值，并返回该channel
	fmt.Println("当前时间：", time.Now()) //当前时间： 2019-12-03 11:02:01.9846649 +0800 CST m=+1.004076201
	fmt.Println("After时间到", <-timeChannel) //After时间到 2019-12-03 11:02:02.9851912 +0800 CST m=+2.004602501

	/*定时器。一定时间后，向timer对象自身的channel类型成员C发送实时时间数据值，仅一次*/
	timer1 := time.NewTimer(time.Second*1)
	fmt.Println("当前时间：", time.Now()) //当前时间： 2019-12-03 11:02:02.9852482 +0800 CST m=+2.004659501
	fmt.Println("接收时间：", <-timer1.C) //接收时间： 2019-12-03 11:02:03.986297 +0800 CST m=+3.005708301
	timer2 := time.NewTimer(time.Second*10)
	timer2.Reset(time.Second*1) //定时重设为1s，返回重设成功与否的boolean值
	timer2.Stop() //停止定时器，同时关闭其channel

	/*断续器。每隔一段时间，向timer对象自身的channel类型成员C发送实时时间数据值*/
	ticker1 := time.NewTicker(time.Second*1)
	i := 0
	for {
		if i == 3 {
			ticker1.Stop() //停止断续器，同时关闭其channel
			break
		}
		fmt.Println(i, <-ticker1.C)
		i++
	}
}
```



### runtime

```go
func main() {
	fmt.Println(runtime.GOROOT()) //gorrot目录
	fmt.Println(runtime.GOOS) //操作系统
	fmt.Println(runtime.NumCPU()) //逻辑cpu的数量
	n := runtime.GOMAXPROCS(runtime.NumCPU()) //设置go程序执行的最大cpu核心数量，1-256。一般init中设置
	fmt.Println(n)
	runtime.Gosched() //使当前goroutine让出cpu时间片，先让别的goroutine执行
	runtime.Goexit() //终止所在goroutine
	fmt.Println(n)
}
```



## channel

同函数一样，channel也是一等公民，可以作为参数和返回值

- 临界资源安全问题：临界资源指并发环境中多个进程/线程/协程共享的资源。在并发编程中对临界资源的处理不当，往往会导致数据不一致的问题。
- Go的并发编程：不要以共享内存的方式去通信，而要以通信的方式去共享内存。
  - 要以通信的方式去共享内存：鼓励通过channeI将共享状态或共享状态的变化，在各个Goroutine之间传递，这样同样能像用锁一样保证在同一的时间只有一个Goroutine访问共享状态。
  - 不要以共享内存的方式去通信：不鼓励用锁保护共享状态的方式在不同的Goroutine中分享信息。在主流的编程语言中为了保证多线程之间共享数据安全性和一致性，都会提供一套基本的同步工具集， 如锁，条件变量,原子操作等等。Go语言标准库也毫不意外的提供了这些同步机制，使用方式也和其他语言也差不多。
- channel：Go语言中，要传递某个数据给另- -个goroutine(协程),可以把这个数据封装成-个对象，然后把这个对象的指针传入某个channel中，另外一个goroutine从这个channel中读出这个指针， 并处理其指向的内存对象。Go从语言层面保证同-一个时间只有一-个goroutine能够访问channel里面的数据，为开发者提供了-种优雅简单的工具，所以Go的做法就是使用channel来通信，通过通信来传递内存数据，使得内存数据在不同的goroutine中传递，而不是使用共享内存来通信。

```go
/*channel是用来传递数据的一个数据结构，队列，先进先出*/
func main() {
    var c chan int //chan关键字声明通道。c==nil,不可用
	c1 := make(chan int) //初始化channel。缺省缓冲区容量为0，为阻塞式通道。发送端(线程)会阻塞，直到接收端(线程)从通道中接收值；接收端会阻塞，直到发送端发送数据。
	c2 := make(chan int, 1024)//参数2设置缓冲区大小。带缓冲区的通道，允许发送端数据发送和接收端数据接收处于异步状态，即发送端发送的数据可以存放在缓冲区中，等待接收端接收数据，而无须接收端立刻接收数据。缓存区满，发送端阻塞，直到某个接收端从缓冲区中取走某部分值使缓冲区不满；缓冲区非空非满，发送端会阻塞，直到发送的值被拷贝到缓冲区中；缓冲区空，接收端阻塞，直到缓冲区有值可以接收
    
	go func() { //
		v := 1
		time.Sleep(time.Second*1)
		c1 <- v //把v值发送到通道ch
	}()
    
	go func(c1 <- chan int, c2 chan <- int) { //channel声明时可以控制读写权限，c chan int可读写，为双向通道，c <- chan int只读，c chan <- int只写，为单向通道
		v, b := <-c1 //返回1是数据值，返回2表示通道状态的Boolean值，false表示从一个一已关闭的通道读取数据，类似于map，v,b=map[k]
		fmt.Println(v, b) //1 true
		c2 <- v+1
		for i:=0; i<=9; i++ { //带缓冲区的channel无须立刻接收值，可以发送多个数据保存到channel缓冲区
			time.Sleep(time.Second*1)
			c2 <- i
		}
		close(c2) //关闭channel。只能由发送方关闭。channel和其中数据仍然存在，只关闭写，不关闭读，只是让接收端线程读完channel数据之后，不再阻塞等待新数据写入。接收端可以从关闭的chan中无限次取出chan中数据类型缺省值，可以用v, b := <-c1中的b进行判断缓存是否已经取完，或者通过range chan来取
	}(c1, c2)
	v1, v2 := <-c2+1, <-c2
	fmt.Println(v1, v2) //3 0
	for v := range c2 { //遍历channel。如果channel不关闭，则range函数不会结束，当channel被取空时，主线程阻塞(此时只剩下主线程)，所有线程阻塞，形成死锁，报错
		fmt.Print(v, " ") //1 2 3 4 5 6 7 8 9。一秒一输出
	}
}
```

```go
//作返回值
func createworker(id int) chan<- int {
	C := make(chan int )
	go func() { //直接<-c会阻塞，则启动一个goroutine来<-c
		for {
			fmt. Printf ("Worker %d received %c\n", id, <-c)
		}
	}()
	return C
}
```





#### 线程同步

```go
func main() {
	c := make(chan int) //引入通道作为同步工具
	go func() {
		for i:=0; i<9; i++ {
			fmt.Println("other") //可以执行9次
		}
		c <- 0 //发送端
	}()
	fmt.Println("main")
	fmt.Println(<-c) //接收端，将阻塞等待，直到发送端线程执行完循环发送数据
}
```

```go
// 各启一个goroutine，按序输出 "a"、"b"、"c" 各100次
func main() {
	var (
		wg  sync.WaitGroup
		ch1 = make(chan struct{}, 1) // 防止协程 g3 最后一次循环没有 ch1 的接收者而阻塞
		ch2 = make(chan struct{})
		ch3 = make(chan struct{})
	)
	wg.Add(3)

	go pr(ch1, ch2, "a", &wg)
	go pr(ch2, ch3, "b", &wg)
	go pr(ch3, ch1, "c", &wg) // g3

	ch1 <- struct{}{}
	wg.Wait()
}

func pr(cur, next chan struct{}, text string, wg *sync.WaitGroup) {
	var counter int32 = 100
	for {
		if counter < 1 {
			break
		}
		<-cur
		fmt.Println(text)
		counter--
		next <- struct{}{}
	}
	wg.Done()
}
```

#### fibonacci

```go
func main() {
	c := make(chan int) //数字通信
	b := make(chan bool) //程序是否结束
	go func() {
		for i := 0; i < 8; i++ {
			num := <-c
			fmt.Println(num)
		}
		b <- true //控制fibonacci中循环停止条件
	}()
	fibonacci(c, b)
}
func fibonacci(c chan<- int, b <-chan bool) {
	x, y := 1, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case flag := <-b:
			fmt.Println("flag = ", flag)
			return
		}
	}
}
```

#### 超时

```go
func main() {
	ch := make(chan int)
	quit := make(chan bool)
	go func() {
		for {
			select {
			case num := <-ch:
				fmt.Println("num = ", num)
			case <-time.After(time.Second*3):
				fmt.Println("超时")
				quit <- true
			}
		}
	}()
	for i := 0; i < 3; i++ {
		ch <- i
		time.Sleep(time.Second)
	}
	<-quit
	fmt.Println("程序结束")
}
```

#### 多任务多并发

```go
func main() {
	TaskSend(3)
}

type Worker struct {
	In   chan string
	Done chan bool
}

func TaskSend(n int) {
	var workers [10]Worker
	for i := 0; i < len(workers); i++ {
		workers[i] = CreateWorker()
	}
	for i := 0; i < n; i++ { //发布n轮任务
		for j, worker := range workers { //每个工作者每轮收到1个任务
			worker.In <- "第" + strconv.Itoa(i+1) + "轮任务，工作者" + strconv.Itoa(j) + "号"
		}
	} //循环结束后，每个工作者都收到了n个任务吧
	for _, worker := range workers {
		for i := 0; i < n; i++ { //每个工作者完成所有任务
			<-worker.Done
		}
	}
}

func CreateWorker() Worker {
	w := Worker{
		In:   make(chan string),
		Done: make(chan bool),
	}
	go DoWork(w)
	return w
}

func DoWork(worker Worker) { //done用来通知外面，事情做完了
	for s := range worker.In {
		fmt.Println(s)
		go func() { worker.Done <- true }() //如果不在开一个goroutine来done，那么任务第一轮结束就会死锁，所有goroutine都阻塞在这里
	}
}
```

加入等待组与重构

```go
func main() {
	TaskSend(3)
}

type Worker struct {
	In   chan string
	Done func()
}

func TaskSend(n int) {
	var wg sync.WaitGroup //等待组
	var workers [10]Worker
	for i := 0; i < len(workers); i++ {
		workers[i] = CreateWorker(&wg)
	}
	wg.Add(n * len(workers)) //等待组添加共计 n * len(workers) 个任务

	for i := 0; i < n; i++ { //n轮任务
		for j, worker := range workers { //每个工作者每轮收到1个任务
			worker.In <- "第" + strconv.Itoa(i+1) + "轮任务，工作者" + strconv.Itoa(j) + "号"
		}
	} //循环结束后，每个工作者都收到了n个任务吧

	wg.Wait()
}

func CreateWorker(wg *sync.WaitGroup) Worker {
	worker := Worker{
		In: make(chan string),
		Done: func() {
			wg.Done()
		},
	}
	go DoWork(worker, wg)
	return worker
}

func DoWork(worker Worker, wg *sync.WaitGroup) { //done用来通知外面，事情做完了
	for s := range worker.In {
		fmt.Println(s)
		worker.Done()
	}
}
```

select，timer，ticker，nil chan的使用

```go
func main() {
	var c1, c2 = generator(), generator()
	var worker = createWorker(0)

	var values []int //如果生成器生成速度大于消费速度，那么会有中间数字被跳过消费，所以需要一个队列存储生成数
	tc := time.After(10 * time.Second)
	ticker := time.Tick(time.Second) //定时器，每隔指定时间送一个值到通道中
	for {
		var activeWorker chan<- int //nil chan，永远不会被case选中
		var activeValue int
		if len(values) > 0 {
			activeWorker = worker //非nil时，就能被选中了
			activeValue = values[0]
		}
		select {
		case n := <-c1:
			values = append(values, n)
		case n := <-c2:
			values = append(values, n)
		case activeWorker <- activeValue:
			values = values[1:]
		case <-time.After(800 * time.Millisecond): //800毫秒之内没有数据生成，则timeout一下
			fmt.Println("timeout")
		case <-ticker:
			fmt.Println("queue len:", len(values))
		case <-tc:
			fmt.Printf("结束\n")
			return
		}
	}

}

func generator() chan int {
	out := make(chan int)
	go func() {
		i := 0
		for {
			time.Sleep(time.Duration(rand.Intn(1500)) * time.Millisecond)
			out <- i
			i++
		}
	}()
	return out
}

func createWorker(id int) chan<- int {
	c := make(chan int)
	go worker(id, c)
	return c
}

func worker(id int, c chan int) {
	for n := range c {
		time.Sleep(1 * time.Second)
		fmt.Printf("Worker %d received %d\n", id, n)
	}
}
```


