

+++

title = "Algorithm"
description = "Algorithm"
tags = ["it", "Algorithm"]

+++



# 算法



## DP



- dp：dp 问题的一般形式就是**求最值**，求解dp的核心问题是**穷举**。因为要求最值，肯定要把所有可行的答案穷举出来，然后在其中找最值。
  - **重叠子问题**：动态规划的穷举有点特别，因为这类问题**存在「重叠子问题」**，如果暴力穷举的话效率会极其低下，所以需要「备忘录」即「**DP table**」来优化穷举过程，避免不必要的计算
  - **最优子结构**：动态规划问题一定会**具备「最优子结构」**，才能通过子问题的最值得到原问题的最值。**要符合「最优子结构」，子问题间必须互相独立**。子问题之间不能相互影响其结果
    - 比如考试，每门科目的成绩都是互相独立的。你的原问题是考出最高的总成绩，那么你的子问题就是要把语文考到最高，数学考到最高…… 为了每门课考到最高，你要把每门课相应的选择题分数拿到最高，填空题分数拿到最高…… 当然，最终就是你每门课都是满分，这就是最高的总成绩。
    - 得到了正确的结果：最高的总成绩就是总分。因为这个过程符合最优子结构，“每门科目考到最高”这些子问题是互相独立，互不干扰的。
    - 但是，如果加一个条件：你的语文成绩和数学成绩会互相制约，数学分数高，语文分数就会降低，反之亦然。这样的话，显然你能考到的最高总成绩就达不到总分了，按刚才那个思路就会得到错误的结果。因为子问题并不独立，语文数学成绩无法同时最优，所以最优子结构被破坏。
  - **状态转移方程**：虽然动态规划的核心思想就是穷举求最值，但是问题可以千变万化，穷举所有可行解其实并不是一件容易的事，只有列出**正确的「状态转移方程」**，才能正确地穷举。**写出状态转移方程是最困难的**

### 标准动态规划

- **状态转移方程定义步骤**
  - **base case**：先明确基本情况，一般是问题最小时的确定结果。base case将作为状态转移方程的最小状态的结果
  - **状态**：找到「状态」，状态会不断向base case靠近，是原问题和子问题中会变化的变量，一般来说就是问题给出变量中的某个或某些。状态将作为状态转移方程的参数
  - **选择**：导致「状态」产生变化的行为
    - 选择不一定是线性的，比如【凑零钱】中
  - **定义dp数组/函数**：综合上面3项，化为可以将父问题化为子问题的规律，按照规律写出状态转移方程式
  - 构造路径：如果需要构造路径，只需要在过程中随`dp[W][N]`用**完全一致的流程**构造一个`pathDp[W][N]`即可
- 代码实现步骤：虽说有**自顶向下递归**、**自底向上循环**两种方式，但其步骤是基本一致的，不过标准的dp一般采用**自底向上循环**的方式
  - dp table：初始化一个 dp table。根据状态转移方程参数的范围初始化 dp table 的大小
    - 有时在这个大小的基础上+1，可以有各种用途，比如下面【斐波那契】【凑零钱】【最大公共子串长度】中
  - extreme：初始化最值。一般采用一个单独的变量专门记录最值
    - 当然也有其它方法，比如下面【凑零钱】中
  - base case：写出，在**自顶向下递归**中表现为**递归结束条件**，在**自底向上循环**中表现为 **DP table 最前内容**
  - 状态转移：如下

```go
# 初始化 base case
dp[0][0][...] = base
# 进行状态转移
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 求最值(选择1，选择2...)
```



##### 斐波那契

- 斐波那契数列没有求最值，所以严格来说不是动态规划问题，但是可以很好的表示重叠子问题，状态转移方程也很简单，即斐波那契本身 `f(n)=1,n=1,2`；`f(n)=f(n-1)+f(n-2),n>2` ，状态即斐波那契的函数参数 n。让dp初始化为n+1长度，单纯就是让下标跟n对应，方便写代码

自顶向下递归

```go
func fib(n int) {
    if (n < 1) {
        return 0
    }
    // 备忘录初始化
    memo := make([]int, n+1)
    return helper(meno, n)
}

func helper(memo []int, n int) int {
    // base case
    if n == 1 || n == 2 {
        return 1
    }
    // 先从备忘录获取，存在子问题结果就直接返回
    if memo[n] != 0 {
        return memo[n]
    }
    // 状态转移方程。记录结果到备忘录，再返回
    memo[n] = helper(memo, n-1) + helper(memo, n-2)
    return memo[n]
}
```

自底向上循环

```go
func fib(n int) int {
    // 状态范围锁定
    if n < 0 {
        return 0
    }
    // dp table
    dp := make([]int, n+1)
    // base case
    dp[1], dp[2] = 1, 1
    // 自底向上循环
    for i := 3; i <= n; i++ {
        // 状态转移方程
        dp[i] = dp[i-1] + dp[i-2]
    }
    return dp[n]
}
```

状态压缩：这里空间复杂度从 o(n) 压缩到了 o(1)。一般来说是把一个二维的 DP table 压缩成一维，即 o(n^2) 压缩到 o(n)

```go
func fib(n int) (res int) {
    // 状态范围锁定
    if n < 0 {
        return 0
    }
    // 只需要用2个变量记录上一次结果和上上次结果即可
    pre, prePre = 1, 1
    // 自底向上循环
    for i := 3; i <= n; i++ {
        // 状态转移方程
        res = pre + prePre
        // 更新记录
        prePre, pre = pre, res
    }
    return res
}
```



##### 凑零钱

- 标准的动态规划问题，并且选择不是线性的

- 有 `k` 种面值的硬币，面值分别为 `c1, c2 ... ck`，每种硬币的数量无限，再给一个总金额 `amount`，问你**最少**需要几枚硬币凑出这个金额，如果不可能凑出，算法返回 -1 。比如说 `k = 3`，面值分别为 1，2，5，总金额 `amount = 11`。那么最少需要 3 枚硬币凑出，即 11 = 5 + 5 + 1。算法的函数签名如下

```
// coins 中是可选硬币面值，amount 是目标金额
int coinChange(int[] coins, int amount);
```

- 这个问题符合最优子结构：比如你想求 `amount = 11` 时的最少硬币数（原问题），如果你知道凑出 `amount = 10` 的最少硬币数（子问题），你只需要把子问题的答案加一（再选一枚面值为 1 的硬币）就是原问题的答案。因为硬币的数量是没有限制的，所以子问题之间没有相互制，是互相独立的。（如果存在硬币数量限制，好像就可以使用贪心的思想解题了？）

- 状态转移方程
  - base case：目标金额 `amount` 为 0 时算法返回 0
  - 状态：由于硬币数量无限，硬币的面额也是题目给定的，只有目标金额会不断地向 base case 靠近，所以唯一的「状态」就是目标金额 `amount`。
  - 选择：目标金额为什么变化呢，因为你在选择硬币，你每选择一枚硬币，就相当于减少了目标金额。所以说所有硬币的面值，就是你的「选择」
  - dp定义：`dp[n]=-1,n<0`；`dp[n]=0,n=0`；`dp[n]=min(res, dp[n-coin in coins]),n>0`，`res` 是问题迭代过程中记录最小值的变量

```python
# 伪码框架
def coinChange(coins: List[int], amount: int):

    # 定义：要凑出金额 n，至少要 dp(n) 个硬币
    def dp(n):
        # 做选择，选择需要硬币最少的那个结果
        for coin in coins:
            res = min(res, 1 + dp(n - coin))
        return res

    # 题目要求的最终结果是 dp(amount)
    return dp(amount)
```

无备忘录穷举

```python
def coinChange(coins: List[int], amount: int):

    def dp(n):
        # base case
        if n == 0: return 0
        if n < 0: return -1
        # 求最小值，所以初始化为正无穷
        res = float('INF')
        for coin in coins:
            subproblem = dp(n - coin)
            # 子问题无解，跳过
            if subproblem == -1: continue
            res = min(res, 1 + subproblem)

        return res if res != float('INF') else -1

    return dp(amount)
```

带备忘录递归

```python
def coinChange(coins: List[int], amount: int):
    # 备忘录
    memo = dict()
    def dp(n):
        # 查备忘录，避免重复计算
        if n in memo: return memo[n]
        # base case
        if n == 0: return 0
        if n < 0: return -1
        res = float('INF')
        for coin in coins:
            subproblem = dp(n - coin)
            if subproblem == -1: continue
            res = min(res, 1 + subproblem)

        # 记入备忘录
        memo[n] = res if res != float('INF') else -1
        return memo[n]

    return dp(amount)
```

带备忘录循环： `dp` 数组初始化大小和值为 `amount + 1` ，是因为凑成 `amount` 金额的硬币数最多只可能等于 `amount`（全用 1 元面值的硬币），所以初始化为 `amount + 1` 就相当于初始化为正无穷，便于后续取最小值，这也是可以这样写`dp[i] = min(dp[i], 1 + dp[i - coin])`的原因。

```python
int coinChange(vector<int>& coins, int amount) {
    // 数组大小为 amount + 1，初始值也为 amount + 1
    vector<int> dp(amount + 1, amount + 1);
    // base case
    dp[0] = 0;
    // 外层 for 循环在遍历所有状态的所有取值
    for (int i = 0; i < dp.size(); i++) {
        // 内层 for 循环在求所有选择的最小值
        for (int coin : coins) {
            // 子问题无解，跳过
            if (i - coin < 0) continue;
            dp[i] = min(dp[i], 1 + dp[i - coin]); //
        }
    }
    return (dp[amount] == amount + 1) ? -1 : dp[amount];
}
```

### 背包

##### 01背包

- 问题：有n个重量和价值分别为wi，vi的物品，从这些物品中挑选出总重量不超过W的物品，求所有挑选方案中价值总和的最大值。因为对每个物品只有选和不选两种情况，所以这个问题称为01背包
  - 限制：1≤n≤100。1<wi，vi≤100。1≤W≤10000
  - 输入：n=4，(w,v)={(2,3),(1,2),(3,4),(2,2)}，W=5。4个物品，重量和价值分别如(w,v)描述，总重W描述
  - 输出：7(选择第0、1、3号物品)
- 状态：总重量W、物品N
- 选择：选或不选第N件物品
- 状态转移方程：`dp[W][N]`表示，总重量不超过W，在前N件物品中进行选择，能选到的最大价值
  -  `dp[W][N] = 0, W==0 || N==0`：base case
  - `dp[W][N] = max(dp[W][N-1], dp[W-ws[N-1]][N-1]+vs[N-1]), W>0 && N>0`：如果不选择第N件物品`dp[W][N]`可以转化为`dp[W][N-1]`问题，如果选择第N件物品`dp[W][N]`可以转化为`dp[W-ws[N-1]][N-1]+vs[N-1]`问题，择其大即可得到`dp[W][N]`的结果

```go
// 循环
func OneZeroPackage(W, N int, ws, vs []int) [][]int {
	// 默认所有值都是0，base case已就位
	dp := make([][]int, W+1)
	for i := 0; i < len(dp); i++ {
		dp[i] = make([]int, N+1)
	}

	for i := 1; i <= W; i++ {
		for j := 1; j <= N; j++ {
			// 重量不足时只能不选
			if i < ws[j-1] {
				dp[i][j] = dp[i][j-1]
				continue
			}

			a := dp[i][j-1]
			b := dp[i-ws[j-1]][j-1] + vs[j-1]
			// 重量足够谁大选谁
			if a > b {
				dp[i][j] = a
			} else {
				dp[i][j] = b
			}
		}
	}
	return dp
}
```

```go
// 递归
func OneZeroPackage(W, N int, ws, vs []int) [][]int {
	var trance []int

	// 默认所有值都是0，base case已就位
	dp := make([][]int, W+1)
	for i := 0; i < len(dp); i++ {
		dp[i] = make([]int, N+1)
	}

	for i := 1; i <= W; i++ {
		recursion(i, N, ws, vs, &dp, trance)
	}
	return dp
}

func recursion(W, N int, ws, vs []int, dp *[][]int) (res int) {
	if (*dp)[W][N] != 0 {
		return (*dp)[W][N]
	}
	if W == 0 || N == 0 {
		return 0
	}
	if W < ws[N-1] {
		res = recursion(W, N-1, ws, vs, dp)
	} else {
		a := recursion(W, N-1, ws, vs, dp)
		b := recursion(W-ws[N-1], N-1, ws, vs, dp) + vs[N-1]
		if a > b {
			res = a
		} else {
			res = b
		}
	}
	(*dp)[W][N] = res
	return (*dp)[W][N]
}
```

###### 路径追踪

- 只需要在过程中随`dp[W][N]`用**完全一致的流程**构造一个`pathDp[W][N]`即可。这里用`[]int`类型元素作为路径

```go
func OneZeroPackage(W, N int, ws, vs []int) ([][]int, [][][]int) {
	// 默认所有值都是0，base case已就位
	dp := make([][]int, W+1)
	for i := 0; i < len(dp); i++ {
		dp[i] = make([]int, N+1)
	}
	pathDp := make([][][]int, W+1)
	for i := 0; i < len(pathDp); i++ {
		pathDp[i] = make([][]int, N+1)
	}

	for i := 1; i <= W; i++ {
		for j := 1; j <= N; j++ {
			// 重量不足时只能不选
			if i < ws[j-1] {
				pathDp[i][j] = pathDp[i][j-1]
				dp[i][j] = dp[i][j-1]
				continue
			}

			a := dp[i][j-1]
			b := dp[i-ws[j-1]][j-1] + vs[j-1]
			// 重量足够谁大选谁
			if a > b {
				pathDp[i][j] = pathDp[i][j-1]
				dp[i][j] = a
			} else {
				pathDp[i][j] = append(pathDp[i-ws[j-1]][j-1], j)
				dp[i][j] = b
			}
		}
	}
	return dp, pathDp
}
```



### 贪心

##### 区间调度问题

- 给出很多形如 `[start, end]` 的闭区间，设计一个算法，**算出这些区间中最多有几个互不相交的区间**
  - `intvs = [[1,3], [2,4], [3,6]]`，这些区间最多有 2 个区间互不相交，即 `[[1,3], [3,6]]`，算法应该返回 2
  - 注意边界相同并不算相交
  - 这个问题在生活中的应用广泛，比如你今天有好几个活动，每个活动都可以用区间 `[start, end]` 表示开始和结束的时间，请问你今天**最多能参加几个活动呢？**显然你一个人不能同时参加两个活动，所以说这个问题就是求这些时间区间的最大不相交子集。
- 思路其实很简单，可以分为以下三步：
  1. 从区间集合 intvs 中选择一个区间 x，这个 x 是在当前所有区间中**结束最早的**（end 最小）。
  2. 把所有与 x 区间相交的区间从区间集合 intvs 中删除。
  3. 重复步骤 1 和 2，直到 intvs 为空为止。之前选出的那些 x 就是最大不相交子集。



### 原来的

分治：原问题的解即子问题的解的合并，树式

贪心：遵循当前最优，适合链式

动态规划：局部最优解来推导全局最优解，树式

##### 经典动态规划

- 动态规划和贪心算法都是一种递推算法，均用局部最优解来推导**全局最优解**，用于子问题之间不完全独立的情况，是对**遍历解空间**（暴力遍历）的一种优化，当问题具有**最优子结构**时，可用动规，把问题结构解析成树
- 贪心，考虑的不是局部最优，而是当前最优，当前最优不一定是全局最优，所以可能要考虑扩大范围，取更大一个范围的局部当前最优。把问题（数据）结构解析成链表
- 动态规划（dp，dynamic programming）：代表了这一类问题（最优子结构、重叠子问题、子问题最优性（贪心））的—般解法，是设计方法或者策略，不是具体算法
- 本质是**递推**，核心是找到状态转移的方式，写出dp方程
- 形式:
  - 记忆型递归（从大往小拆）
  - 递推（从小往大推）：**通过逐步扩大（即动态）问题条件的范围（即规划）**来逐步挑选最优解，大范围条件的解是小范围条件的解集合中的最优解
- 举例:
  - 01背包问题
  - 钢条切割问题
  - 数字三角形问题（滚动数组)
  - 最长公共子序列问题
  - 完全背包问题
  - 最长上升子序列问题
- 重叠（交叉）子问题
  - 发现：当用一个传统的dfs或者递归求导一个问题最优解的时候，发现递归树展开时，不同节点问题上有重叠的解。比如斐波那契 f(n)=f(n-1)+f(n-2)，又f(n-1)=f(n-2)+f(n-3)，所以f(n-2)就是一个重叠，即解决f(n-1)的时候已经解决了f(n-2)的这一子问题，所以如果斐波那契能把哪些计算过的值都利用起来，将变成一个O(n)的问题，而不是O(!n)
  - 记忆型递归：即计算过的值都用一个map映射存储，空间换时间
  - 递推
  - 当然Fibonacci不能算是一个恰当的动态规划例子，因为它太特殊了，是一个子问题完全重叠的特例

##### 01背包

- 有n个重量和价值分别为wi，vi的物品，从这些物品中挑选出总重量不超过W的物品，求所有挑选方案中价值总和的最大值。因为对每个物品只有选和不选两种情况，所以这个问题称为01背包
  - 限制：1≤n≤100。1<wi，vi≤100。1≤W≤10000
  - 输入：n=4，(w,v)={(2,3),(1,2),(3,4),(2,2)}，W=5。4个物品，重量和价值分别如(w,v)描述，总重W描述
  - 输出：7(选择第0、1、3号物品)

##### 贪心策略

- 贪心是动态规划的一个特例
- 遵循某种规则，不断(贪心地)选取当前最优策略，最终找到最优解
- 难点：当前最优未必是整体最优

##### 硬币问题

有1、5、10、50、100、500元的硬币各c1、c5、c10、c50、c100、c500（输入变量）枚。现在要用这些硬币来支付A元，最少需要多

少枚硬币？假定本题至少存在一种支付方案00

0<ci<10^9，0<A<10^9

输入：第一行有六个数字，分别代表从小到大6种面值的硬币的个数；第二行为A，代表需支付的A元

样例

输入

3 2 1 3 0 2

620

输出

6

分析：如果通过遍历解空间去计算用多少张500，是 i=1、2、3、4...去试 i * 500<=A时各种情况，而贪心只考虑眼前最优，直接取最大值A/500，然后继续用剩下的去考虑同样策略最优

```go
var (
	coinValues = []int{1, 5, 10, 50, 100, 500} //硬币面值
	coinCounts = [6]int{3, 2, 1, 3, 0, 2}      //各面值硬币个数，索引与硬币面值对应。来自输入
)

func main() {
	fmt.Println(f(620, 5))
}

func f(a int, i int) int {
	if a <= 0 {
		return 0
	}
	if i == 0 {
		return a
	}
	coinValue := coinValues[i] //当前硬币面值
	n := a / coinValue         //需要多少个当前coin
	if n > coinCounts[i] {     //是否有足够的硬币数
		n = coinCounts[i]
	}
	n += f(a-coinValue*n, i-1) //用剩下的面值继续处理
	return n
}
```



##### 快速渡河

POJ-1700，北大POJ一个题

N个人渡船，1艘船，一次只能载2人，但是得有1人再把船划回来，每个人划船速度不同，每2人划船速度由慢的那个人决定，需要一种策略使最短时间内所有人都渡河

输入：

1

人数：4

每个人渡河时间：1 2 5 10

输出：

最短渡河总时间17

- 分析：可以发现，不同的思路，在不同的情况下也不一定某一种思路更优
  - 一般思路：为了回程更快，始终让最快的带其它所有人。
    - 组合为：去1、2，回1，去1、5，回1，去1、10，总时间为：2+1+5+1+10=19
    - 如果是1、5、6、7的一组人，则总时间为5+1+6+1+7=20
    - 如果是a、b、c、d（小大顺序）的一组人，总时间为2a+b+c+d
  - 另辟蹊径：因为速度由慢的人决定，让最慢的两人一起渡河，让最快的两个其一回程
    - 组合为：去1、2，回1，去5、10，回2，去1、2。总时间为2+1+10+2+2=17
    - 如果是1、5、6、7的一组人，总时间为：5+1+7+5+5=23
    - 如果是a、b、c、d（小大顺序）的一组人，总时间为a+3b+d
  - 比较：2a+b+c+d、a+3b+d，抵消相同部分即比较a+c、b，根据两者大小选择不同方案
  - 扩展n人：假设n个人渡河时间按顺序放在一个数组p中，
    - 一般思路：很简单，每次返回都是最快的p[0]
      - 虽然是每来回1次就已经成为一个循环，一个来回时间为p[0]+p[n]
      - 但是为了与下面的思路比较时间，以2次来回计算1次时间进行比较即可，时间为2*p[0]+p[n-1]+p[n]
    - 始终保持让最慢的两人一起渡河，让最快的两个其一回程，其实n个人与4个人是一样的流程：
      - 初始状态是：有n个人都在左边未渡河
      - 去p[0]、p[1]，回p[0]，去p[n]、p[n-1]，回p[1]，使用时间p[0]+2*p[1]+p[n]
      - 目前状态是，p[0]-p[n-2]都在左边未渡河，可以发现，其实经过这么一轮循环，结果是渡河了2个最慢的人然后我们又回到了n-2个人渡河的状态，以同样的方式迭代即可，每次减少2个最慢的人
    - 2\*p[0]+p[n-1]+p[n]、p[0]+2\*p[1]+p[n]比较，即p[0]+p[n-1]、2\*p[1]
    - 每2个人渡河进行比较选取其中当前的最优思路即可，完成贪心

```go
var (
   times = []int{} //每个人渡河时间
)

func main() {
   fmt.Println(f(times))
}

func f(times []int) int {
   left := len(times) //未渡河人数
   timeSum := 0       //渡河总时间
   for left > 0 {
      if left == 1 {
         timeSum += times[0]
      } else if left == 2 {
         timeSum += times[1]
      } else if left == 3 {
         timeSum += times[0] + times[1] + times[2]
      } else {
         /*始终由时间最短的人回程*/
         time1 := 2*times[0] + times[left-1] + times[left-2] //2*p[0]+p[n-1]+p[n]
         time2 := times[0] + 2*times[1] + times[left-1] //p[0]+2*p[1]+p[n]
         if time1 < time2 {
            timeSum += time1
         } else {
            timeSum += time2
         }
      }
   }
   return timeSum
}
```

##### 区间问题

###### 区间调度

有n项工作，每项工作分别在 si 时间开始,在 ti 时间结束.
对于每项工作，你都可以选择参与与否.如果选择了参与，那么自始至终都必须全程参与。
此外，参与工作的时间段不能重复(即使是开始的瞬间和结束的瞬间的重叠也是不允许的)，
你的目标是参与尽可能多的工作， 那么最多能参与多少项工作呢？

思路：总是选结束时间最早的工作

代码思路：因为工作不一定是有序的，用一个job结构体封装job的开始时间和结束时间，实现比较方法，使其可以通过结束时间排序，排好序后按**总是选结束时间最早的工作**这个思路来即可

###### 区间选点

POJ1201

有一些区间，问最少的点命中所有区间

思路：也是按区间结束值进行排序，总是选区间结束值，下一次再选择开始值大于该结束值且结束值最近的点即可

加条件：除了开始时间、结束时间，每个区间追加区间命中数这个属性，使每个区间必须被指定数量的点命中





##### 最大公共子串长度

```go
// `dp[i][j]= (a[i]==b[j]?dp[i-1][j-1]+1:0)`
// 这样是正确的，但是当i=0或者j=0时，dp会下标越界，这种时候一般让i、j使用的时候都比原来+1即可，即dp[1][1]来存a[0]和b[0]的结果即可
// 就变成dp[i][j]= (a[i-1]==b[j-1]?dp[i-1][j-1]+1:)，到代码中体现就是让dp的长宽+1，i、j从1开始遍历，a、b串取元素的时候用i-1、j-1即可
// dp[0][0]本来初始就是0不影响长度加算，所以就是让dp[0][0]作为基本情况，如果在其它问题中有初始值也可以用最尾或者最末来初始化一个值这样用
func f(a, b string) (res int) {
    var (
        aLen, bLen = len(a)+1, len(b)+1
        dp = make([][]int, aLen)
    )
    for i:=0; i<aLen; i++{
		dp[i] = make([]int, bLen)
	}
    
    for i:=1; i<aLen; i++ {
        for j:=1; j<bLen; j++ {
            if a[i-1]==b[j-1] {
                dp[i][j] = dp[i-1][j-1]+1
                if dp[i][j]>res{
                    res = dp[i][j]
                }
            }
        }
    }
    return
}
```



## DFS

- DFS：深度优先遍历，也叫回溯算法。不像动态规划存在重叠子问题可以优化，回溯算法就是纯暴力穷举，复杂度一般都很高。某种程度上来看，动态规划的暴力求解阶段就是回溯算法



- 过程：**解决一个回溯问题，实际上就是一个决策树的遍历过程**
  - 路径：记录已经做过的选择
  - 选择列表：当前节点可以做的选择。
  - 结束条件：到达决策树底层，无法再做选择的条件。
- 代码实现步骤：**其核心是 for 循环里面的递归，在递归调用之前「做选择」，在递归调用之后「撤销选择」**
  - 参数：将「选择列表」和「路径」作为递归函数的参数
  - 结束条件：也表现为递归的结束条件，到达决策树底层，将路径添加到结果集
  - 循环递归选择：**核心就是 for 循环中，在递归调用之前「做选择」，在递归调用之后「撤销选择」**
    - 进入循环时，如果「当前选择」已经在「路径」中了（可以用单独写一个方法遍历判断），表示已经选择过了，直接continue跳过即可。比如【全排列】【N皇后】中
    - 或者也可以，「做选择」时，先将该选择从选择列表移除，然后将选择添加到路径；「撤销选择」时，先从路径中移除选择，再将该选择再加入选择列表
    - 当然还有有更好的方法，通过交换元素达到目的

```python
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件，到达决策树底层，无法再做选择的条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        if 路径 包含 选择:
            continue

        路径.add(选择)
        backtrack(路径, 选择列表)
        路径.remove(选择)
```



##### 全排列

- 列出`n` 个不重复的数的全排列
- 比方说给三个数 `[1,2,3]`，你肯定不会无规律地乱穷举，一般是这样：先固定第一位为 1，然后第二位可以是 2，那么第三位只能是 3；然后可以把第二位变成 3，第三位就只能是 2 了；然后就只能变化第一位，变成 2，然后再穷举后两位……，就形成了一个决策树，为啥说这是决策树呢，因为你在每个节点上其实都在做决策，**定义的** **`backtrack`** **函数其实就像一个指针，在这棵树上游走，同时要正确维护每个节点的属性，每当走到树的底层，其「路径」就是一个全排列**。
  - 路径：记录在 track 中
  - 选择列表：nums 中不存在于 track 的那些元素
  - 结束条件：nums 中的元素全都在 track 中出现

```java
List<List<Integer>> res = new LinkedList<>();

/* 主函数，输入一组不重复的数字，返回它们的全排列 */
List<List<Integer>> permute(int[] nums) {
    // 记录「路径」
    LinkedList<Integer> track = new LinkedList<>();
    backtrack(nums, track);
    return res;
}

void backtrack(int[] nums, LinkedList<Integer> track) {
    // 触发结束条件
    if (track.size() == nums.length) {
        res.add(new LinkedList(track));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        // 排除不合法的选择，这里稍微做了些变通，没有显式记录「选择列表」，而是通过 nums 和 track 推导出当前的选择列表，所以后面不需要从选择列表删除元素和添加回来
        if (track.contains(nums[i]))
            continue;
        // 做选择
        track.add(nums[i]);
        // 进入下一层决策树
        backtrack(nums, track);
        // 取消选择
        track.removeLast();
    }
}
```

##### N皇后

-  N×N 的棋盘防止N个皇后，使得它们不能互相攻击，问有多少种放法。皇后可以攻击 同一行、同一列、左上、左下、右上、右下 8个方向任意距离的单位
- 本质上跟全排列问题差不多，决策树的每一层表示棋盘上的每一行；每个节点可以做出的选择是，在该行的任意一列放置一个皇后
  - 路径：board 中小于 row 的那些行都已经成功放置了皇后
  - 选择列表：第 row 行的所有列都是放置皇后的选择
  - 结束条件：row 超过 board 的最后一行
- 最坏时间复杂度仍然是 O(N^(N+1))，而且无法优化

```c
vector<vector<string>> res;

/* 输入棋盘边长 n，返回所有合法的放置 */
vector<vector<string>> solveNQueens(int n) {
    // '.' 表示空，'Q' 表示皇后，初始化空棋盘。
    vector<string> board(n, string(n, '.'));
    backtrack(board, 0);
    return res;
}

void backtrack(vector<string>& board, int row) {
    // 触发结束条件
    if (row == board.size()) {
        res.push_back(board);
        return;
    }

    int n = board[row].size();
    for (int col = 0; col < n; col++) {
        // 排除不合法选择
        if (!isValid(board, row, col)) 
            continue;
        // 做选择
        board[row][col] = 'Q';
        // 进入下一行决策
        backtrack(board, row + 1);
        // 撤销选择
        board[row][col] = '.';
    }
}

/* 是否可以在 board[row][col] 放置皇后？ */
bool isValid(vector<string>& board, int row, int col) {
    int n = board.size();
    // 检查列是否有皇后互相冲突
    for (int i = 0; i < n; i++) {
        if (board[i][col] == 'Q')
            return false;
    }
    // 检查右上方是否有皇后互相冲突
    for (int i = row - 1, j = col + 1; 
            i >= 0 && j < n; i--, j++) {
        if (board[i][j] == 'Q')
            return false;
    }
    // 检查左上方是否有皇后互相冲突
    for (int i = row - 1, j = col - 1;
            i >= 0 && j >= 0; i--, j--) {
        if (board[i][j] == 'Q')
            return false;
    }
    return true;
}
```

- 有的时候，我们并不想得到所有合法的答案，只想要一个答案。比如解数独的算法，找所有解法复杂度太高，只要找到一种解法就可以。只要稍微修改一下回溯算法的代码

```c
// 函数找到一个答案后就返回 true
bool backtrack(vector<string>& board, int row) {
    // 触发结束条件
    if (row == board.size()) {
        res.push_back(board);
        return true;
    }
    ...
    for (int col = 0; col < n; col++) {
        ...
        board[row][col] = 'Q';

        if (backtrack(board, row + 1))
            return true;

        board[row][col] = '.';
    }

    return false;
}
```



## BFS

- 常见场景：**问题的本质就是让你在一幅「图」中找到从起点** **`start`** **到终点** **`target`** **的最近距离，这个例子听起来很枯燥，但是 BFS 算法问题其实都是在干这个事儿**
  - 广义的描述可以有各种变体，比如走迷宫，有的格子是围墙不能走，从起点到终点的最短距离是多少？如果这个迷宫带「传送门」可以瞬间传送呢？
  - 比如说两个单词，要求你通过某些替换，把其中一个变成另一个，每次只能替换一个字符，最少要替换几次？
  - 比如说连连看游戏，两个方块消除的条件不仅仅是图案相同，还得保证两个方块之间的最短连线不能多于两个拐点。你玩连连看，点击两个坐标，游戏是如何判断它俩的最短连线有几个拐点的？
- **BFS 找到的路径一定是最短的，但代价就是空间复杂度比 DFS 大很多**
  - BFS 可以找到最短距离，但是空间复杂度高。处理二叉树问题的例子，假设给你的这个二叉树是满二叉树，节点数为 `N`，对于 DFS 算法来说，空间复杂度无非就是递归堆栈，最坏情况下顶多就是树的高度，也就是 `O(logN)`，
  - DFS 不能找最短路径吗？其实也是可以的，但是时间复杂度相对高很多。你想啊，DFS 实际上是靠递归的堆栈记录走过的路径，你要找到最短路径，肯定得把二叉树中所有树杈都探索完才能对比出最短的路径有多长




- 过程：就是在一个图中从一个start节点开始**扩散**，直到找到target节点，通过一个**队列**存储下一步要扩散的节点，通过一个**集合**存储所有已经走过的点，以避免走回头路
  - **要扩散的节点**：一般用一个队列存储，也可以是链表，或者其它的方便增删元素的结构
  - **已扩散的节点**：

- 代码实现步骤：
  - 声明队列q、集合visited、扩散步数step，将start节点加入 q 和 visited，因为直到start就要扩散了，就直接加入到已扩散的节点无妨，当然改变一下位置在循环中再加入visited也无妨
  - 在q.len=0之前一直循环：
    - 遍历q：
      - 移除元素，表示已扩散
      - 判断是否到是target节点，到达就返回step
      - 遍历元素的相邻节点
        - 如果不在已扩散的节点中，则将该相邻节点加入

```c++
// 计算从起点 start 到终点 target 的最近距离
int BFS(Node start, Node target) {
    Queue<Node> q; // 核心数据结构
    Set<Node> visited; // 避免走回头路，即避免再次走到走过的节点

    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0; // 记录扩散的步数

    while (q not empty) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            Node cur = q.poll();
            /* 划重点：这里判断是否到达终点 */
            if (cur is target)
                return step;
            /* 将 cur 的相邻节点加入队列。cur.adj() 泛指 cur 相邻的节点 */
            for (Node x : cur.adj())
                if (x not in visited) {
                    q.offer(x);
                    visited.add(x);
                }
        }
        /* 划重点：更新步数在这里 */
        step++;
    }
}
```



##### 二叉树的最小深度

https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/

- **显然起点就是** **`root`** **根节点，终点就是最靠近根节点的那个「叶子节点」嘛**，叶子节点就是两个子节点都是 `null` 的节点

```go
int minDepth(TreeNode root) {
    if (root == null) return 0;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);
    // root 本身就是一层，depth 初始化为 1
    int depth = 1;

    while (!q.isEmpty()) {
        int sz = q.size();
        /* 将当前队列中的所有节点向四周扩散 */
        for (int i = 0; i < sz; i++) {
            TreeNode cur = q.poll();
            /* 判断是否到达终点 */
            if (cur.left == null && cur.right == null) 
                return depth;
            /* 将 cur 的相邻节点加入队列 */
            if (cur.left != null)
                q.offer(cur.left);
            if (cur.right != null) 
                q.offer(cur.right);
        }
        /* 这里增加步数 */
        depth++;
    }
    return depth;
}


```



##### 打开转盘锁

https://leetcode-cn.com/problems/open-the-lock/

```go
int openLock(String[] deadends, String target) {
    // 记录需要跳过的死亡密码
    Set<String> deads = new HashSet<>();
    for (String s : deadends) deads.add(s);
    
    // 记录已经穷举过的密码，防止走回头路
    Set<String> visited = new HashSet<>();
    Queue<String> q = new LinkedList<>();
    
    // 从起点开始启动广度优先搜索
    int step = 0;
    q.offer("0000");
    visited.add("0000");

    while (!q.isEmpty()) {
        int sz = q.size();
        /* 将当前队列中的所有节点向周围扩散 */
        for (int i = 0; i < sz; i++) {
            String cur = q.poll();

            /* 判断是否到达终点 */
            if (deads.contains(cur))
                continue;
            if (cur.equals(target))
                return step;

            /* 将一个节点的未遍历相邻节点加入队列 */
            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                if (!visited.contains(up)) {
                    q.offer(up);
                    visited.add(up);
                }
                String down = minusOne(cur, j);
                if (!visited.contains(down)) {
                    q.offer(down);
                    visited.add(down);
                }
            }
        }
        /* 在这里增加步数 */
        step++;
    }
    // 如果穷举完都没找到目标密码，那就是找不到了
    return -1;
}
```



##### 双向 BFS 优化

- **无论传统 BFS 还是双向 BFS，无论做不做优化，从 Big O 衡量标准来看，时间复杂度都是一样的**，只能说双向 BFS 是一种 trick，算法运行的速度会相对快一点
- **传统的 BFS 框架就是从起点开始向四周扩散，遇到终点时停止；而双向 BFS 则是从起点和终点同时开始扩散，当两边有交集的时候停止**。
- 双向 BFS 还是遵循 BFS 算法框架的，只是**不再使用队列，而是使用 HashSet 方便快速判断两个集合是否有交集**。另外的一个技巧点就是 **while 循环的最后交换** **`q1`** **和** **`q2`** **的内容**，所以只要默认扩散 `q1` 就相当于轮流扩散 `q1` 和 `q2`。
- **不过，双向 BFS 也有局限，因为你必须知道终点在哪里**。二叉树最小高度的问题，你一开始根本就不知道终点在哪里，也就无法使用双向 BFS；但是第二个密码锁的问题，是可以使用双向 BFS 算法来提高效率的

```go
int openLock(String[] deadends, String target) {
    Set<String> deads = new HashSet<>();
    for (String s : deadends) deads.add(s);
    // 用集合不用队列，可以快速判断元素是否存在
    Set<String> q1 = new HashSet<>();
    Set<String> q2 = new HashSet<>();
    Set<String> visited = new HashSet<>();

    int step = 0;
    q1.add("0000");
    q2.add(target);

    while (!q1.isEmpty() && !q2.isEmpty()) {
        // 哈希集合在遍历的过程中不能修改，用 temp 存储扩散结果
        Set<String> temp = new HashSet<>();

        /* 将 q1 中的所有节点向周围扩散 */
        for (String cur : q1) {
            /* 判断是否到达终点 */
            if (deads.contains(cur))
                continue;
            if (q2.contains(cur))
                return step;
            visited.add(cur);

            /* 将一个节点的未遍历相邻节点加入集合 */
            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                if (!visited.contains(up))
                    temp.add(up);
                String down = minusOne(cur, j);
                if (!visited.contains(down))
                    temp.add(down);
            }
        }
        /* 在这里增加步数 */
        step++;
        // temp 相当于 q1
        // 这里交换 q1 q2，下一轮 while 就是扩散 q2
        q1 = q2;
        q2 = temp;
    }
    return -1;
}
```

- 双向 BFS 还有一个优化，就是在 while 循环开始时做一个判断：因为按照 BFS 的逻辑，队列（集合）中的元素越多，扩散之后新的队列（集合）中的元素就越多；在双向 BFS 算法中，如果我们每次都选择一个较小的集合进行扩散，那么占用的空间增长速度就会慢一些，效率就会高一些

```go
// ...
while (!q1.isEmpty() && !q2.isEmpty()) {
    if (q1.size() > q2.size()) {
        // 交换 q1 和 q2
        temp = q1;
        q1 = q2;
        q2 = temp;
    }
    // ...
```

