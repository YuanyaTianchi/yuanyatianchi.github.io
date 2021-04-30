# 重构笔记

一个例子

```go
// 影片类型
type MovieType int

var (
	CHILD_RENTS MovieType = 2 // 儿童租用的
	REGULAR     MovieType = 0 // 普通的
	NEW_RELEASE MovieType = 1 // 新发布的
)

// 影片
type Movie struct {
	Title     string
	PriceCode MovieType
}
```

```go
// 租借记录
type Rental struct {
	Movie      *Movie
	DaysRented int // 租借天数
}
```

```go
// 顾客
type Customer struct {
	Name    string
	Rentals []*Rental
}

// 添加租借记录
func (cus *Customer) AddRental(rental *Rental) {
	cus.Rentals = append(cus.Rentals, rental)
}

// 生成详单
func (cus *Customer) Statement() string {
	var (
		totalAmount          float64 = 0                                      // 总价
		frequentRenterPoints         = 0                                      // 常客积分点
		result                       = "Rental Record for " + cus.Name + "\n" // 详单结果，页头内容
	)

	// 根据
	for _, item := range cus.Rentals {
		var thisAmount float64 = 0

		switch item.Movie.PriceCode {
		case REGULAR:
			thisAmount += 2
			if item.DaysRented > 2 {
				thisAmount += float64(item.DaysRented-2) * 1.5
			}
			break
		case NEW_RELEASE:
			thisAmount += float64(item.DaysRented) * 3
			break
		case CHILD_RENTS:
			thisAmount += 1.5
			if item.DaysRented > 3 {
				thisAmount += float64(item.DaysRented-3) * 1.5
			}
			break
		}

		frequentRenterPoints++
		// 新书租借超过两天额外增加积分
		if item.Movie.PriceCode == NEW_RELEASE && item.DaysRented > 1 {
			frequentRenterPoints++
		}
		totalAmount += thisAmount
		result += "\t" + item.Movie.Title + "\t" + strconv.Itoa(int(thisAmount)) // TODO: float64 转 string
	}

	// 详单结果，页脚内容
	result += "Amount owed is" + strconv.Itoa(int(totalAmount)) + "\n"
	result += "Your earned " + strconv.Itoa(frequentRenterPoints) + " frequent renter points" + "\n"
	return result
}
```

快速而随性的设计一个简单的程序并没有错，但如果这是复杂系统中具有代表性的一段，那对这个程序的信心就要动摇了。Customer 中长长的 Statement 做的事情太多，做了很多原本应该由其它结构体完成的事情

在这个例子里，希望以html格式输出详单（而不是字符串）



函数应该是可复用的，功能尽量单一的，且需要根据其使用的数据判定它应该属于哪个文件或者对象

第一步必须是为即将修改的代码构建一组可靠的测试环境，好的测试是重构的根本

提炼函数内的一块代码时，先找出这块代码所使用的局部变量和参数。任何不会被修改的变量都可以直接作为提炼函数的参数传入，被修改的变量可以作为提炼函数的返回值。

提炼函数中的变量因为上下文改变，需要的话可以考虑调整变量命名，使其更符合新的上下文语意

goland可以选中一块代码，通过 `右键-Refactor-Extract Method` 直接提取成单独方法

重构技术是以微小的步伐修改程序，这样即使犯下错误，也可以很容易测试发现，而不必花大量时间调试

可以选择减少或去除临时变量，它们可能会被到处传递并引用，会助长冗长而复杂的代码，去除它们的代价是也许会产生更多次的计算而牺牲一些性能，这需要根据临时变量的可维护性和计算临时变量的性能之间权衡。有些计算结果的过程中用到的临时变量，可以直接将该结果获取变为方法，就可以有效去除临时变量





