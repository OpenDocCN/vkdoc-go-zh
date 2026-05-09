# 使用协程计算斐波那契数列

接下来的示例（清单 1-16）展示了如何使用协程序输出斐波那契数列。

```go
package main
import "fmt"
func fibonacci(c chan<- int, quit <-chan bool) {
x, y := 0, 1
for {
select {
case c <- x:
x, y = y, x + y // 生成数列
case <- quit:
fmt.Println("quit")
return
}
}
}
func main() {
c := make(chan int)
quit := make(chan bool)
go func() {
for i := 0; i < 20; i++ {
fmt.Println(<-c)
}
quit <- true
}()
fibonacci(c, quit)
}
/* Output

quit
*/
清单 1-16
使用协程计算斐波那契数列
```

函数 `fibonacci` 的第一个参数 `c` 用于向通道中放入信息，第二个参数 `quit` 则从通道中取出信息。

协程在 `main` 函数中作为内部函数启动。`Println(<-c)` 语句会阻塞，直到 `fibonacci` 函数将值 `x` 放入整数通道 `c` 中。

函数 `fibonacci` 中的 `select` 语句要么将 `x` 的下一个值放入通道 `c`，要么在 `quit` 变为 `true` 时立即结束程序。实际的斐波那契数列计算逻辑，体现在 `case c <- x` 语句中的第二行。

在下一节中，我们将探讨通过使用并发所能获得的潜在性能提升。

## 1.4 基准测试并发应用程序

使用并发的目标是加速程序执行。然而，部署协程会带来开销，因此有时使用并发会适得其反。由于调试并发代码具有挑战性，且处理死锁和竞态条件有时颇为棘手，在设计并发解决方案时需要格外谨慎。对并发应用程序进行测试，并将其性能与非并发解决方案进行比较，是很有价值的。

在本节中，我们将展示几个应用程序，对并发解决方案与非并发解决方案的性能进行比较。由于基准测试结果受处理器和内存、机器的环境工作负载（后台运行进程数量）以及机器架构的影响，在推广结论或可能从基准测试结果中得出错误结论时必须小心谨慎。

请参考清单 1-17 中的程序。在此，我们对比了在无并发和并发情况下，构建包含 1 亿个浮点数的切片并求其和所需的时间。

```go
package main
import "fmt"
import "time"
import "sync"
var output1 float64
var output2 float64
var output3 float64
var output4 float64
var wg sync.WaitGroup
func worker1() {
defer wg.Done()
var output []float64
sum := 0.0
for index := 0; index < 100_000_000; index++ {
output = append(output, 89.6)
sum += 89.6
}
output1 = sum
}
func worker2() {
defer wg.Done()
var output []float64
sum := 0.0
for index := 0; index < 100_000_000; index++ {
output = append(output, 64.8)
sum += 64.8
}
output2 = sum
}
func worker3() {
defer wg.Done()
var output []float64
sum := 0.0
for index := 0; index < 100_000_000; index++ {
output = append(output, 956.8)
sum += 956.8
}
output3 = sum
}
func worker4() {
defer wg.Done()
var output []float64
sum := 0.0
for index := 0; index < 100_000_000; index++ {
output = append(output, 1235.8)
sum += 1235.8
}
output4 = sum
}
func main() {
wg.Add(8)
// 无并发处理的计算时间
start := time.Now()
worker1()
worker2()
worker3()
worker4()
elapsed := time.Since(start)
fmt.Println("\n4 个 worker 串行执行时间: ", elapsed)
fmt.Printf("Output1: %f \nOutput2: %f  \nOutput3: %f  \nOutput4: %f\n",
output1, output2, output3, output4)
// 并发处理的计算时间
start = time.Now()
go worker1()
go worker2()
go worker3()
go worker4()
wg.Wait()
elapsed = time.Since(start)
fmt.Println("\n4 个 worker 并行执行时间: ", elapsed)
fmt.Printf("Output1: %f \nOutput2: %f  \nOutput3: %f  \nOutput4: %f",
output1, output2, output3, output4)
}
/* 在配备 10 核 CPU、32 核 GPU 和 16 核神经网络引擎的 M1 Max 芯片 Macbook Pro 上的输出
4 个 worker 串行执行时间:  1.133640541s
Output1: 8960000016.634367
Output2: 6480000011.637030
Output3: 95680000176.244049
Output4: 123580000205.352280
4 个 worker 并行执行时间:  756.305958ms
Output1: 8960000016.634367
Output2: 6480000011.637030
Output3: 95680000176.244049
Output4: 123580000205.352280
*/
清单 1-17
计算求和：有并发与无并发对比
```

每个 worker 函数都通过追加一个 `float64` 值来构建一个包含 1 亿个值的输出切片，同时计算切片内元素的和。

在 `main` 函数中，我们计算并输出了 worker 函数顺序执行时的计算时间。接着，我们使用协程并发执行这四个 worker 函数。通过对比有并发和无并发情况下的求和结果并输出，来比较计算时间并验证结果正确性。

并发执行四个 worker 函数的计算时间，是串行执行四个函数所需时间的 67%。不出所料，计算出的总和是相同的。

假设我们希望通过使用并发来加速对数字序列求和的运算。清单 1-18 演示了这一点。

```go
package main
import (
"fmt"
"time"
)
const (
NumbersToSum = 10_000_000
)
func sum(s []float64, c chan<- float64) {
// 一个将数据放入通道的生成器
sum := 0.0
for _, v := range s {
sum += float64(v)
}
c <- sum // 阻塞直到从通道中取出 c
}
func plainSum(s []float64) float64 {
sum := 0.0
for _, v := range s {
sum += float64(v)
}
return sum
}
func main() {
s := []float64{}
for i := 0; i < NumbersToSum; i++ {
s = append(s, 1.0)
}
c := make(chan float64)
start := time.Now()
go sum(s[:len(s) / 2], c)
go sum(s[len(s) / 2 :], c)
first, second := <-c, <-c // 从每个 c 接收数据
elapsed := time.Since(start)
fmt.Printf("first: %f  second: %f elapsed time: %v", first, second,
elapsed)
start = time.Now()
answer := plainSum(s)
elapsed = time.Since(start)
fmt.Printf("\nplain sum: %f elapsed time: %v", answer, elapsed)
}
/*
first: 5000000.000000  second: 5000000.000000 elapsed time: 5.864275ms
plain sum: 10000000.000000 elapsed time: 11.601511ms
*/
清单 1-18
使用并发加速求和运算
```

通过在两个协程中分别对一半的数字求和，执行时间得到了显著改善，这在程序输出中显而易见。

这两个协程在 for 循环中并发执行计算，并通过赋值给通道变量 `c` 来返回各自的值。在 `main` 函数中，将两个和值分别赋给 `first` 和 `second` 的操作会被阻塞，直到这两个值都已被赋值到通道中。

### 使用并发生成质数

接下来，我们探讨质数的生成。经典的质数生成算法是埃拉托斯特尼筛法。这是一个非常快速的非并发算法。

目标是找出直到指定数字（例如一千万）为止的所有质数。质数是只能被 1 和自身整除的整数。最初的几个质数是：2, 3, 5, 7, 11, 13, 17, 19, 23。除了 2 之外，所有其他质数都是奇数。



### 埃拉托斯特尼筛法算法

以下函数展示了埃拉托斯特尼筛法算法：

```go
func SieveOfEratosthenes(n int) []int {
    // 找出所有小于等于 n 的质数
    primes := make([]bool, n+1)
    for i := 2; i < n+1; i++ {
        primes[i] = true
    }
    // 筛除非质数索引的逻辑
    for p := 2; p*p <= n; p++ {
        if primes[p] == true {
            // 更新所有 p 的倍数
            for i := p * 2; i <= n; i += p {
                primes[i] = false
            }
        }
    }
    // 返回所有小于等于 n 的质数
    var primeNumbers []int
    for p := 2; p <= n; p++ {
        if primes[p] == true {
            primeNumbers = append(primeNumbers, p)
        }
    }
    return primeNumbers
}
```

初始化一个包含 `n + 1` 个**布尔**值的切片。切片中的每个元素都被初始化为 `true`，表示一开始所有数字都被视为质数。

在接下来的 for 循环中，索引变量 `p` 从 2 运行到 `n` 的平方根。如果 `primes[p]` 的值为 `true`（表示 `p` 是质数），那么 `primes` 切片中所有 `p` 的倍数索引都会被移除。例如，`n = 20`。当 `p` 为 2 时，`primes` 切片中以下索引被设置为 `false`：4, 6, 8, 10, 12, 14, 16, 18, 20。当 `p` 为 3 时，`primes` 切片中以下索引被设置为 `false`：6, 9, 12, 15, 18。当 `p` 为 4 时，`primes` 切片中以下索引被设置为 `false`：8, 12, 16, 20。由于 `p = 5` 的平方超过了 20，我们完成筛除。筛法已完成其工作。`primes` 值未被设置为 `false` 的索引是 2, 3, 5, 7, 11, 13, 17 和 19。

清单 1-19 展示了一个用于对该筛法性能进行基准测试的程序。

```go
// 埃拉托斯特尼筛法
package main

import (
    "fmt"
    "time"
)

const LargestPrime = 10_000_000

func SieveOfEratosthenes(n int) []int {
    // 找出所有小于等于 n 的质数
    primes := make([]bool, n+1)
    for i := 2; i < n+1; i++ {
        primes[i] = true
    }
    // 筛除逻辑
    for p := 2; p*p <= n; p++ {
        if primes[p] == true {
            // 更新所有 p 的倍数
            for i := p * 2; i <= n; i += p {
                primes[i] = false
            }
        }
    }
    // 返回所有小于等于 n 的质数
    var primeNumbers []int
    for p := 2; p <= n; p++ {
        if primes[p] == true {
            primeNumbers = append(primeNumbers, p)
        }
    }
    return primeNumbers
}

func main() {
    start := time.Now()
    sieve := SieveOfEratosthenes(LargestPrime)
    elapsed := time.Since(start)
    fmt.Println("\n 计算时间：", elapsed)
    fmt.Println("最大质数：", sieve[len(sieve)-1])
}

/* 输出
计算时间：28.881792ms
质数数量 = 664579
*/
清单 1-19
对埃拉托斯特尼筛法进行基准测试
```

如果你认为所有并发解决方案都更优秀，请考虑清单 1-20 中生成质数的并发解决方案。

该并发解决方案与清单 1-19 中展示的非并发埃拉托斯特尼筛法相比非常慢，以至于常量 `LargestPrime` 被降低了两个数量级，变为 100,000，而不是 1 千万。即便如此，该解决方案仍然更慢。

```go
// 一个并发的质数筛法
package main

import (
    "fmt"
    "time"
)

const LargestPrime = 100_000
var primes []int

// 将序列 3, 5, ... 发送到通道 'ch'。
func Generate(prime1 chan LargestPrime {
        break
    }
    primes = append(primes, prime)
    prime2 := make(chan int)
    go Filter(prime1, prime2, prime)
    prime1 = prime2
}
elapsed := time.Since(start)
fmt.Println("计算时间：", elapsed)
fmt.Println("质数数量 = ", len(primes))
}

/* 输出
计算时间：1.462818125s
质数数量 = 9591
*/
清单 1-20
一种生成质数的并发雏菊链解决方案
```

在 `Filter` 协程中使用取余运算符 `%` 会带来显著的性能损失。该协程从 `in` 通道接收信息，并将信息输出到 `out` 通道，正如函数签名中方向箭头所指示的那样。

我们能否通过使用另一种并发解决方案做得更好？

### 分段筛算法

作为回答这个问题的垫脚石，我们介绍了一种对清单 1-19 中实现的埃拉托斯特尼筛法算法的修改。清单 1-21 展示了一种分段筛算法，为稍后将要展示的并发解决方案提供了基础。

```go
package main

import (
    "fmt"
    "math"
    "time"
)

const LargestPrime = 10_000_000
var cores int

func SieveOfEratosthenes(n int) []int {
    // 找出所有小于等于 n 的质数
    primes := make([]bool, n+1)
    for i := 2; i = n {
            high = n
        }
        for {
            if low = n {
                high = n
            }
        } else {
            break
        }
    }
    return primeNumbers
}

func main() {
    start := time.Now()
    primeNumbers := SegmentedSieve(LargestPrime)
    elapsed := time.Since(start)
    fmt.Println("\n 计算时间：", elapsed)
    fmt.Println("质数数量 = ", len(primeNumbers))
}

/* 输出
计算时间：50.557584ms
质数数量 = 664579
*/
清单 1-21
分段筛算法
```

定义了一系列数组段，每个段的大小为 `n` 的平方根（其中 `n` 是待考虑的质数数组中的最大数字）。利用第一个这样的数组段（0 到 `sqrt(n)`）中的质数（通过 `SieveOfEratosthenes` 函数计算得出），我们使用 `primesBetween` 函数计算每个后续数组段中的质数。

让我们在 `n` 为 100 且每个数组段的大小为 10 时，逐步分析这个函数。具体来说，让我们检查在 10 到 20 这个段中质数的计算过程。

小于等于 10 的质数是 `2`, `3`, `7`。

变量 `low` 是 10，`high` 是 20。

定义了一个空切片 `result`，并创建了一个大小为 `limit + 1` 的 `segment` 布尔切片。这个 `segment` 切片被初始化为 `true`。

在一个从 0 到 3（`prime` 的长度）的 for 循环中，我们使用以下代码定义变量 `lowlimit`：

```go
int(math.Floor(float64(low)/float64(prime[i])) * float64(prime[i]))
```

计算结果为 `(10.0 / 2.0) * 2 = 10`。

在另一个从 `lowlimit` 到 `high`、步长为 2 的 for 循环中，我们将 `segment` 在索引 `10`, `12`, `14`, `16`, `18`, 和 `20` 处的值设置为 `false`。

我们将索引 `i` 从 0 推进到 1，并计算 `lowlimit` 为 `floor(10 / 3) * 3 = 9`。由于 `lowlimit` 现在小于 `low`，我们使用 `lowlimit += prime[1]` 将其设置为 12。

在索引为 `j` 的循环中，我们将 `segment` 切片在索引 `12`, `15`, 和 `18` 处设置为 `false`。

继续将 `i` 设为 2，`lowlimit` 为 `floor(10 / 7) * 7`，等于 7。由于该值小于 `low`，我们将其重新赋值为 14（`lowlimit += prime[2]`）。

在 `j` 循环中，我们将 `segment` 切片在索引 `14` 处设置为 `false`。

最后，我们捕获 `segment` 切片中仍为 `true` 的值：`11`, `13`, `17`, 和 `19`。

这种模式对于每个段切片都是相同的。计算次数与原始埃拉托斯特尼筛法函数相同。但现在数组切片小得多（大小是 10 而不是 100）。

从程序输出可以看出，分段筛解决方案比原始埃拉托斯特尼筛法解决方案（`28.881792ms`）慢了 1.75 倍（`50.557584ms`）。这是由于定义了十个 `segment` 切片以及对这些切片进行函数调用的开销，这些在原始筛法计算中是不需要的。

现在，为利用分段筛的并发解决方案做好了准备。



### 并发筛法解决方案

该并发解决方案见代码清单 1-22。

```
package main
import (
"fmt"
"math"
"runtime"
"sync"
"time"
)
const LargestPrime = 10_000_000
var cores int
var primeNumbers []int
var m sync.Mutex
var wg sync.WaitGroup
func SieveOfEratosthenes(n int) []int {
// 找出所有小于等于 n 的素数
primes := make([]bool, n+1)
for i := 2; i = n {
high = n
}
wg.Add(1)
go primesBetween(prime, low, high)
}
wg.Wait()
}
func main() {
cores = runtime.NumCPU()
start := time.Now()
SegmentedSieve(LargestPrime)
elapsed := time.Since(start)
fmt.Println("\nComputation time for concurrrent: ", elapsed)
fmt.Println("Number of primes = ", len(primeNumbers))
}
/* Output
Computation time for concurrrent: 19.783666ms
Number of primes = 664579
*/
清单 1-22 并发分段筛法
```

并发性在 `SegmentedSieve` 函数中实现，该函数启动一系列 goroutine (`primesBetween`)，并使用 `WaitGroup` 阻塞 `SegmentedSieve` 的完成，直到所有 goroutine 完成其工作。

为防止出现竞态条件，每个 goroutine 结束时使用互斥锁 `m`，通过赋值循环开始时的 `m.Lock()` 和赋值循环结束时的 `m.UnLock()`，确保对全局共享的 `primeNumbers` 切片的赋值得到控制。

获取 1000 万以内素数所需时间为 19.78366ms，小于埃拉托斯特尼筛法所需的 28.881792ms。分段大小通过 `runtime.NumCPU()` 返回的核心数计算得出。原则上，这应能使计算利用每个计算机核心，实现近似并行处理。使用互斥锁防止竞态条件会降低并发解决方案的效率，但鉴于所采用的方法，这是避免竞态条件的必要措施。

使用并发分段筛法生成的素数序列并非按顺序排列，但集合是完整的。这是因为 goroutine 以随机顺序异步运行。

## 1.5 本章小结

本章介绍并演示了泛型的使用。我们展示了使用泛型可以大大简化应用程序开发，避免每次更改代码所使用的底层类型时重复代码。我们为全书后续介绍的数据结构和算法中使用泛型奠定了基础。

我们还演示了使用并发性的潜在好处，并指出使用并发性并不能自动保证性能提升。我们研究了 goroutine 和通道作为 goroutine 间通信载体的使用，介绍了互斥锁作为避免竞态条件的结构，以及 `WaitGroup` 作为确保 goroutine 间一定同步的结构。

下一章，我们将进入算法设计的世界，讨论表征算法效率的方法，研究一些经典排序算法以及如何利用并发性实现更快的排序。

