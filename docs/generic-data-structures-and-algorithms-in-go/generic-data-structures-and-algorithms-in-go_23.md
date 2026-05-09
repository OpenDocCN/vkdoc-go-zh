# 15. 动态规划

上一章介绍了生态模拟的并发实现，其中使用了本书前面介绍的许多技术。

本章将焦点从数据结构转向算法设计。

我们将介绍一种用于解决优化问题的算法技术——动态规划，并将该技术应用于几个问题。

正如你将在本章中看到的，“如果你不能记住过去，你就注定会重蹈覆辙。”

在下一节中，我们将介绍一个动态规划的简单示例：计算第 n 个斐波那契数。我们将探讨两种动态规划方法。

## 15.1 动态规划示例：第 n 个斐波那契数

动态规划的核心机制是用更小的子问题来表示问题的解，每个子问题都有最优解。每个子问题都是原问题的一个更小版本。通过存储较小问题的结果，我们可以高效地获得较大问题的结果。

一个简单的例子涉及第 n 个斐波那契数的计算。

**Fib(n) = Fib(n-1) + Fib(n-2)，其中 n > 1**

序列中的前两个数是 0 和 1。

斐波那契数列的初始序列为：

[0, 1, 1, 2, 3, 5, 8, 13, 21, …]

我们考察三种计算第 n 个斐波那契数的替代算法。前两种涉及动态规划，第三种涉及递归。

### 自上而下的动态规划

考虑如下给出的函数 `FibonacciTopDown` 及其辅助函数 `computeFromCache`：

```go
func FibonacciTopDown(n int) int64 {
	firstTwoCases := map[int]int64{
		0: 0,
		1: 1,
	}
	return computeFromCache(n, firstTwoCases)
}

func computeFromCache(n int, cache map[int]int64) int64 {
	// 如果 n 的答案已经找到，则直接返回
	if val, found := cache[n]; found {
		return val
	}
	cache[n] = computeFromCache(n-1, cache) +
		computeFromCache(n-2, cache)
	return cache[n]
}
```

在 `computeFromCache` 中使用了一个 `map`，如果某个解已经计算过，则直接返回。

变量 `cache` 存储键值对（n 与第 n 个斐波那契数）。

这是动态规划，因为大小为 n 的问题是通过大小为 n-1 和 n-2 的问题来计算的。

这种自上而下方法的计算复杂度是 **O(n)**。空间复杂度也是 O(n)，因为使用了 map 来存储之前的计算结果。

### 自下而上的动态规划

考虑如下给出的函数 `FibonacciBottomUp`：

```go
func FibonacciBottomUp(n int) int64 {
	table := []int64{0, 1}
	for i := 2; i <= n; i++ {
		table = append(table, table[i-1]+
			table[i-2])
	}
	return table[n]
}
```

我们从 **0** 到 **n** 自下而上地构建变量 `table`。

该解决方案的计算复杂度也是 **O(n)**。空间复杂度为 **O(1)**。



### 递归解法

函数 `Fib` 如下所示是一个递归解法。但其计算复杂度为 **O(2^n)**，这是难以处理的。

```go
func Fib(n int64) int64 {
if n < 2 {
return n
}
return Fib(n - 1) + Fib(n - 2)
}
```

清单 15-1 展示了三种方法以及一个用于进行时间分析的主驱动程序。

```go
package main
import (
"fmt"
"time"
)
func FibonacciTopDown(n int) int64 {
firstTwoCases := map[int]int64{
0: 0,
1: 1,
}
return computeFromCache(n, firstTwoCases)
}
func computeFromCache(n int, cache map[int]int64) int64 {
// 如果 n 的结果已存在，直接返回
if val, found := cache[n]; found {
return val
}
cache[n] = computeFromCache(n - 1, cache) +
computeFromCache(n - 2, cache)
return cache[n]
}
func FibonacciBottomUp(n int) int64 {
table := []int64{0, 1}
for i := 2; i <= n; i++ {
table = append(table, table[i - 1] +
table[i - 2])
}
return table[n]
}
func Fib(n int64) int64 {
if n < 2 {
return n
}
return Fib(n - 1) + Fib(n - 2)
}
func main() {
fmt.Println("fib(7) = ", FibonacciTopDown(7))
start := time.Now()
fib40 := FibonacciTopDown(40)
elapsed := time.Since(start)
fmt.Println("Value of FibonacciTopDown(40): ", fib40)
fmt.Println("Computation time: ", elapsed)
fmt.Println("fib(7) = ", FibonacciBottomUp(7))
start = time.Now()
fib40 = FibonacciBottomUp(40)
elapsed = time.Since(start)
fmt.Println("\nValue of FibonacciBottomUp(40): ", fib40)
fmt.Println("Computation time: ", elapsed)
fmt.Println("fib(7) = ", Fib(7))
start = time.Now()
fib40 = Fib(40)
elapsed = time.Since(start)
fmt.Println("\nValue of Fib(40): ", fib40)
fmt.Println("Computation time: ", elapsed)
}
/* 输出
fib(7) =  13
Value of FibonacciTopDown(40):  102334155
Computation time:  36.136μs
fib(7) =  13
Value of FibonacciBottomUp(40):  102334155
Computation time:  7.377μs
fib(7) =  13
Value of Fib(40):  102334155
Computation time:  424.44211ms
*/
清单 15-1
斐波那契数
```

### 代码讨论

动态规划的自底向上方法大约比动态规划的自顶向下方法快五倍。两种方法都显著快于递归方法。

在下一节中，我们将研究算法设计中的一个经典问题：0/1 背包问题。

## 15.2 另一个应用：0/1 背包问题

假设给定一组物品。我们希望将这些物品的一个子集装入一个具有指定重量限制的背包中。每个被考虑的物品如果被装入背包，都有一个指定的重量和利润。我们希望选择一个能最大化利润的物品子集。

作为一个小例子，假设有四个潜在的物品，其重量分别为 **4, 6, 2, 8**，利润分别为 **12, 15, 9, 21**。假设背包的重量限制为 10。

让我们枚举总重量 <= 10 的物品组合。

物品 1 + 物品 2 (总重量 10)，利润 27

物品 1 + 物品 3 (总重量 6)，利润 21

物品 2 + 物品 3 (总重量 8)，利润 24

物品 3 + 物品 4 (总重量 10)，利润 30

最优解是将物品 3 和物品 4 装入背包。

### 暴力解法

暴力解法枚举重量和利润的所有子集组合。考虑以下函数：

```go
// 暴力解法
func KnapSackBF(weightLimit int, weights []int, profits []int, n int) int {
if n == 0 || weightLimit == 0 {
return 0
}
if weights[n - 1] > weightLimit {
return KnapSackBF(weightLimit, weights, profits, n - 1)
} else {
// 假设我们包含物品 n - 1
value1 := profits[n - 1] +
KnapSackBF(weightLimit –
weights[n - 1], weights, profits, n - 1)
// 假设我们不包含物品 n - 1
value2 := KnapSackBF(weightLimit, weights,
profits, n - 1)
if value1 >= value2 {
return value1
} else {
return value2
}
}
}
```

### 代码讨论

如果 `weights[n – 1]` 处的重量超过 `weightLimit`，则递归调用 `KnapSackBF`，并将 `n` 替换为 `n – 1`。

否则，计算 `value1`（假设包含物品 `n – 1`）和 `value2`（假设排除物品 `n – 1`）。在此递归层级返回 `value1` 和 `value2` 中的较大值。

这个暴力算法是我们遇到的第一个计算上难以处理的例子。

由于此递归函数涵盖了所有子集，并且众所周知，大小为 `n` 的集合的子集数量为 `2^n`，因此我们得出这个暴力方法是 **O(2^n)** 的。计算时间呈指数级增长，是渐近的。

### 动态规划解法

该问题的动态规划解法如下所示：

```go
// 动态规划解法
func KnapSackDP(weightLimit int, weights []int, profits
[]int) int {
n := len(weights)
if weightLimit < 0 {
return 0
}
table := make([][]int, n + 1)
for i := 0; i <= n; i++ {
table[i] = make([]int, weightLimit + 1)
}
for i := 1; i <= n; i++ {
for w := 1; w <= weightLimit; w++ {
if weights[i - 1] <= w {
// 包含物品
profit1 := profits[i - 1] + table[i - 1][w - weights[i - 1]]
// 排除物品
profit2 := table[i - 1][w]
if profit1 > profit2 {
table[i][w] = profit1
} else {
table[i][w] = profit2
}
} else {
// 排除物品
table[i][w] = table[i - 1][w]
}
}
}
return table[n][weightLimit]
}
```

### 代码讨论

我们使用一个二维切片来实现动态规划。让我们通过一个小例子来了解该算法的工作原理。

假设我们的重量和利润数组如下：

Weights = [3, 5, 1]

Profits = [10, 20, 1]

WeightLimit = 5

我们计算出 `n` 等于 2。

我们创建一个 4 × 6 的表。

在 `KnapsackDP` 中生成如下表格：

**0   0   0      0      0     0**

**0   0   0      0   10   10**

**0   0   0   10   10   20**

**0   1   1   10   11   20**

由于动态规划解法是通过两个嵌套循环找到的，因此计算复杂度为 **O(n x L)**，其中 L 是重量限制。

在清单 15-2 中，我们比较了解决 0/1 背包问题的每种算法的计算时间。

```go
package main
import (
"fmt"
"time"
)
// 暴力解法 - 略
// 动态规划解法 - 略
func main() {
weights := []int{4, 6, 2, 8}
profits := []int{12, 15, 9, 21}
fmt.Println("Solution 1 = ", KnapSackBF(10, weights, profits, 4))
weights1 := []int{4, 6, 2, 8, 1, 17, 23, 10, 4, 8}
profits1 := []int{12, 15, 9, 21, 5, 8, 20, 6, 1, 15}
result := KnapSackBF(20, weights1, profits1, 10)
fmt.Println("Solution 2 = ", result)
weights2 := []int{}
for i := 0; i < 800; i++ {
weights2 = append(weights2, 2 * i)
}
profits2 := []int{}
for i := 0; i < 800; i++ {
profits2 = append(profits2, 3 * i)
}
start := time.Now()
result2 := KnapSackBF(400, weights2, profits2, 800)
elapsed := time.Since(start)
fmt.Println("Solution 3 = ", result2)
fmt.Println("Time for solution3 (brute force): ",
elapsed)
start = time.Now()
result3 := KnapSackDP(400, weights2, profits2)
elapsed = time.Since(start)
fmt.Println("Solution 3 = ", result3)
fmt.Println("Time for solution3 (dynamic programming): ", elapsed)
}
/* 输出
Solution 1 = 30
Solution 2 = 57
Solution 3 = 600
Time for solution3 (brute force): 1m10.248200934s
Solution 3 = 600
Time for solution3 (dynamic prograamming): 1.621038ms
*/
清单 15-2
0/1 背包计算时间
```

### 代码讨论

对于涉及 800 个重量和利润的问题，动态规划解法的速度是暴力解法的 43,000 倍。如果此问题中的重量限制增加，暴力解法的计算时间将变得难以处理。

在下一节中，我们将应用动态规划来寻找两个 DNA 字符串中的最长子序列。



## 15.3 DNA 子序列

DNA 字符串是由字母表 {A, C, G, T} 中的字符组成的序列。此类字符串的一个示例是“C**GT**T**A**C**AA**TTT**G**C**G**”。

我们将字符串的**子序列**定义为按顺序（不一定是连续顺序）从原始字符串中从左到右扫描时选取的字符序列。

对于前面的字符串，一个子序列是“GTAAAGG”。该序列取自原始字符串中以粗体显示的字符。

在计算遗传学中，一个重要的问题是找到两个 DNA 字符串的最长公共子序列。这就是最长子序列问题。

一种暴力方法是枚举`string1`的所有子序列，然后针对`string2`逐一测试它们。如果`string1`有 **n** 个字符，`string2`有 **m** 个字符，那么这种暴力算法将需要`O(2^(n)m)`的时间。对于大型的`string1`，这在计算上是棘手的。

我们使用动态规划来解决这个问题。

我们将`Length(j, k)`定义为既是`X[0:j]`的子序列又是`Y[0:k]`的子序列的最长字符串的长度。

需要考虑两种情况。第一种情况是`X[j-1]`等于`y[k-1]`。

由此可得`Length(j, k) = 1 + Length(j-1, k–1)`。

如果`x[j-1]`不等于`y[k-1]`，我们无法得到包含`x[j-1]`和`y[k-1]`的子序列。

那么我们设`Length(j, k) = max{Length(j-1, k), Length(j, k – 1)}`。

对于`j = 0, 1, …, n`，`Length(j, 0)`为`0`；对于`k = 0, 1, …, m`，`Length(0, k)`为`0`。

这些递推关系引出了一个动态规划解决方案。

我们创建一个`(n + 1) x (m + 1)`的二维切片`L`。我们将此表初始化为`0`。我们迭代地构建`L`直到得到`L[n,m]`。

清单 15-3 给出了最长公共子序列问题的动态规划解决方案，以及一个带有两个测试用例的主驱动程序。

```
package main
import (
"fmt"
)
func max(value1, value2 int) int {
if value1 >= value2 {
return value1
} else {
return value2
}
}
func reverse(x []rune) []rune {
result := []rune{}
for index := len(x) - 1; index >= 0; index-- {
result = append(result, x[index])
}
return result
}
func longestCommonSubsequenceTable(x, y []rune) (LCS [][]int) {
// Return matrix so that LCS[j][k] is longest
// common sequence for x[0:j] and y[0:k]
n := len(x)
m := len(y)
// Initialize LCS table of size (n + 1 x m + 1)
LCS = make([][]int, n + 1)
for row := 0; row = table[j][k - 1] {
j -= 1
k -= 1
}
}
return string(reverse(result))
}
func main() {
x := "CGTTACAATTTGCG"
y := "TTTTAAACGTGCG"
lcs := LongestCommonSequence([]rune(x), []rune(y))
fmt.Println(lcs)
x = "ATCGAATTCCGGTAGTCGT"
y = "CGATAGTTCAGCCAG"
lcs = LongestCommonSequence([]rune(x), []rune(y))
fmt.Println(lcs)
}
/* Output
TTAATGCG
TAGC
*/
清单 15-3
最长公共子序列
```

### 代码讨论

为了能够访问`x`和`y`输入字符串中的单个字符，我们需要将每个字符串转换为`rune`切片。

前面描述的递推关系构成了函数`longestCommonSubsequenceTable`细节的基础。

在函数`LongestCommonSequence`中，我们从表的右侧开始，向左工作直到表的开头。因此，我们需要反转结果，并将反转后的`rune`切片转换为字符串。

该动态规划解决方案的计算复杂度为`O(n x m)`。

## 15.4 总结

本章介绍了动态规划这一算法设计技术。它被应用于几个问题，包括斐波那契数、0/1 背包问题和 DNA 子序列。在所有这三个问题中，动态规划都提供了高效的解决方案。

在下一章中，我们将把注意力转向图结构以及一些利用图的经典算法。

