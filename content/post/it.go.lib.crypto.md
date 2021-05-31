

+++

title = "crypto"
description = "crypto"
tags = ["it","go","lib"]

+++



# it.go.lib.crypto

## rand

```go
func main() {
	//设置种子，设置一次后，单次程序运行期间可以随机生成不同的数
	//如果种子参数不变，多次运行程序，每次运行程序产生的随机数集的值和顺序都一样
	nano := time.Now().UnixNano() //当前系统时间戳
	rand.Seed(nano) //以当前系统时间作为种子参数，保证每次运行产生的随机数集不一样
	for i:=0; i<10; i++ {
		fmt.Println("rand = ", rand.Int())     //随机Int数，Int数范围内
		fmt.Println("rand = ", rand.Intn(100)) //随机Int内数，100范围内
	}
}
```

