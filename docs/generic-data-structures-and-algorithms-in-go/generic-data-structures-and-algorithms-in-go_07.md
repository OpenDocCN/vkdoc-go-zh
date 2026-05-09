# 2. 算法效率：排序与搜索

前一章介绍了泛型并回顾了并发性。在本章及本书后续内容中，我们将继续使用这两项技术。

本书的主要目标是提供基于数据结构和算法的技术，使程序运行更快或占用更少空间（更高效）。我们在本章要解决的第一个问题是如何描述算法的效率。然后，我们将研究排序和搜索算法，并分析它们的效率。

## 2.1 描述算法的速度效率

确定算法效率的常规做法是以渐进方式估算其性能与问题规模的函数关系。也就是说，我们关注的是当问题规模变大时的计算速度。本节将介绍大 O 表示法及其在算法设计中的应用。

### 理解大 O

**大 O** 表示法描述了当问题规模变大时，算法执行时间随问题规模增长的特征。它基于对执行任务所需的基本操作数量（如赋值、交换和访问值）的分析。由于问题规模必须足够大这一要求，大 O 是一种渐进性能指标。

例如，假设我们能够将某个算法的运行时间描述为问题规模 `n` 的函数：`execution-time(n) = 12n² + 117n +25`。

当 `n` 较大时，上述表达式中的第一项占主导地位。我们忽略常数 12，重点关注 `n²`。这使我们将该算法描述为 `O(n²)`。对于较大的 `n`，如果将 `n` 的值加倍，执行时间将变为原来的四倍。如果 `n` 较小，这一关系则不成立。

大 O 提供的是一个渐进上界。对于较大的 `n`（问题规模），实际性能受限于 O 表示法内的函数。因此，`O(n)` 意味着对于较大的 `n`，算法的执行时间受限于 `n`，即与 `n` 呈线性关系。

具有 `O(2ⁿ)`（指数复杂度）的算法是难以处理的。同样，`O(n!)` 的算法也是难以处理的。随着问题规模 `n` 变大，计算时间将变得过大，无法在合理时间内完成。本书后续将探讨并处理计算上难以处理的问题。

现在我们回到计算上可行的问题，考虑以下示例：我们希望判断一个浮点数数组切片是否按从小到大的顺序排列。



### 判断数字切片是否已排序

解决这个问题的一种方法是使用`sort`包中的`Sort`函数，然后将结果数组切片与我们想要测试的数组切片进行比较。这是我见过许多经验不足的程序员采用的方法。

执行此操作的函数如下所示：

```
func isSorted1(data []float64) bool {
    var data1 []float64
    data1 = make([]float64, len(data)) // 创建一个长度为 len(data) 的切片
    copy(data1, data) // 将 data 复制到 data1
    sort.Float64s(data1)
    // 比较 data 和 data1
    for i := 0; i < size; i++ {
        if data[i] != data1[i] {
            return false
        }
    }
    return true
}
```

我们为`data1`切片分配存储空间，并将输入的`data`复制到`data1`中。我们使用`sort.Float64s(data1)`对`data1`进行排序。最后，我们将`data1`的每个元素与`data`进行比较，如果不匹配则返回`false`。

众所周知，`Sort`算法的渐近复杂度是`O(nlog2n)`。

现在考虑另一种方法，函数`isSorted2`，如下所示：

```
func isSorted2(data []float64) bool {
    for i := 1; i < len(data); i++ {
        if data[i] < data[i - 1] {
            return false
        }
    }
    return true
}
```

此函数比较`data`中的所有连续对。如果出现`data[i]`小于`data[i – 1]`的情况，函数立即返回`false`。如果没有返回`false`，则函数返回`true`。

此函数的渐近复杂度为`O(n)`。对于较大的`n`，`isSorted2`应该比`isSorted1`快得多。

清单 2-1 对这两个`isSorted`函数进行了基准比较。

```
package main

import (
    "fmt"
    "math/rand"
    "time"
    "sort"
)

const size = 100_000_000

var data []float64

func isSorted1(data []float64) bool {
    var data1 []float64
    data1 = make([]float64, len(data))
    copy(data1, data) // 将 data 复制到 data1
    sort.Float64s(data1)
    // 比较 data 和 data1
    for i := 0; i < size; i++ {
        if data[i] != data1[i] {
            return false
        }
    }
    return true
}

func isSorted2(data []float64) bool {
    for i := 1; i < len(data); i++ {
        if data[i] < data[i - 1] {
            return false
        }
    }
    return true
}

func main() {
    data = make([]float64, size)
    for i := 0; i < size; i++ {
        data[i] = 100.0 * rand.Float64()
    }
    start := time.Now()
    result := isSorted1(data)
    elapsed := time.Since(start)
    fmt.Println("已排序: ", result)
    fmt.Println("使用 isSorted1 耗时:", elapsed)

    data2 := make([]float64, size)
    for i := 0; i < size; i++ {
        data2[i] = float64(2 * i)
    }
    start = time.Now()
    result = isSorted1(data2)
    elapsed = time.Since(start)
    fmt.Println("已排序: ", result)
    fmt.Println("使用 isSorted1 耗时:", elapsed)

    start = time.Now()
    result = isSorted2(data)
    elapsed = time.Since(start)
    fmt.Println("\n 已排序: ", result)
    fmt.Println("使用 isSorted2 耗时", elapsed)

    start = time.Now()
    result = isSorted2(data2)
    elapsed = time.Since(start)
    fmt.Println("已排序: ", result)
    fmt.Println("使用 isSorted2 耗时:", elapsed)
}

/* 输出
已排序:  false
使用 isSorted1 耗时: 20.554518978s
已排序:  true
使用 isSorted1 耗时: 7.328819941s

已排序:  false
使用 isSorted2 耗时 291ns
已排序:  true
使用 isSorted2 耗时: 76.644396ms
*/
清单 2-1
比较两个 isSorted 函数
```

每个函数都使用一个随机`float64`值的数组`data1`进行调用。接下来，每个函数都使用一个已排序的数组`data2`进行调用。

从输出中可以看出，`isSorted2`比`isSorted1`快 100 倍以上，证实了大 O 分析的结果。

### 使用并发

我们能通过使用并发做得更好吗？考虑函数`isSorted3`如下：

```
func isSegmentSorted(data []float64, a, b int, ch chan<- bool) {
    // 生成一个布尔值放入 ch
    for i := a + 1; i < b; i++ {
        if data[i] < data[i - 1] {
            ch <- false
        }
    }
    ch <- true
}

func isSorted3(data []float64) bool {
    ch := make(chan bool)
    numSegments := runtime.NumCPU()
    segmentSize := int(float64(len(data)) / float64(numSegments))
    // 启动 numSegments 个 goroutine
    for index := 0; index < numSegments; index++ {
        go isSegmentSorted(data, index * segmentSize,
            index * segmentSize + segmentSize, ch)
    }
    num := 0 // 已完成的 goroutine 数量
    for {
        select {
        case value := <- ch:  // 阻塞直到某个 goroutine 将布尔值放入通道
            if value == false {
                return false
            }
            num += 1
            if num == numSegments { // 所有 goroutine 已完成
                return true
            }
        }
    }
    return true
}
```

在函数`isSorted3`中，我们将数据细分为由计算机 CPU 数量决定的`numSegments`个段。在一个 for 循环中，我们启动`numSegments`个 goroutine，传入起始和结束索引`a`和`b`，以及一个`bool`类型的通道变量`ch`。

每个 goroutine 使用与函数`isSorted2`相同的逻辑，但在一个更小的区间内，并且与其他 goroutine 并发执行。每个 goroutine 将其结果赋给通道变量`ch`。

在函数`isSorted3`的一个 for 循环中，`select`语句从通道`ch`读取一个布尔值，并阻塞程序执行，直到另一个 goroutine 完成其工作。如果接收到`false`值，`isSorted3`立即返回`false`。否则，用于计数已完成任务 goroutine 的变量`num`加一。如果`num`达到`numSegments`，则`isSorted3`返回`true`，因为所有段都报告了`true`。

在清单 2-2 中，我们扩展了基准测试，包含了并发的`isSorted3`。

```
package main

import (
    "fmt"
    "math/rand"
    "time"
    "sort"
    "runtime"
)

const size = 1_000_000_000

var data []float64

// 代码省略

func isSorted3(data []float64) bool {
    ch := make(chan bool)
    numSegments := runtime.NumCPU()
    segmentSize := int(float64(len(data)) / float64(numSegments))
    // 启动 numSegments 个 goroutine
    for index := 0; index < numSegments; index++ {
        go isSegmentSorted(data, index * segmentSize,
            index * segmentSize + segmentSize, ch)
    }
    num := 0 // 已完成的 goroutine 数量
    for {
        select {
        case value := <- ch:
            if value == false {
                return false
            }
            num += 1
            if num == numSegments { // 所有 goroutine 已完成
                return true
            }
        }
    }
    return true
}

func main() {
    data = make([]float64, size)
    for i := 0; i < size; i++ {
        data[i] = 100.0 * rand.Float64()
    }
    data2 := make([]float64, size)
    // 创建一个排序的数字序列
    for i := 0; i < size; i++ {
        data2[i] = float64(2 * i)
    }

    start := time.Now()
    result := isSorted2(data)
    elapsed := time.Since(start)
    fmt.Println("\n 已排序: ", result)
    fmt.Println("使用 isSorted2 耗时", elapsed)

    start = time.Now()
    result = isSorted2(data2)
    elapsed = time.Since(start)
    fmt.Println("已排序: ", result)
    fmt.Println("使用 isSorted2 耗时:", elapsed)

    start = time.Now()
    result = isSorted3(data)
    elapsed = time.Since(start)
    fmt.Println("\n 已排序: ", result)
    fmt.Println("使用并发的 isSorted3 耗时", elapsed)

    start = time.Now()
    result = isSorted3(data2)
    elapsed = time.Since(start)
    fmt.Println("已排序: ", result)
    fmt.Println("使用并发的 isSorted3 耗时:", elapsed)
}

/* 输出
已排序:  false
使用 isSorted2 耗时 594ns
已排序:  true
使用 isSorted2 耗时: 845.586082ms

已排序:  false
使用并发的 isSorted3 耗时 61.863μs
已排序:  true
使用并发的 isSorted3 耗时: 132.375156ms
*/
清单 2-2
isSorted 的并发实现与计时
```

用于测试`isSorted`的数组大小已增加到十亿个浮点数。

结果非常显著。并发的`isSorted`解决方案比非并发的解决方案快六倍以上。两种解决方案都是`O(n)`的。通过一个常数因子提高性能并不会改变算法的大 O 复杂度。

在下一节中，我们将介绍几种经典的排序算法及其复杂度。


好的，作为一名高级文档工程师和翻译员，我将严格遵循您提供的注意事项和示例格式，将给定的英文文本翻译成中文。


## 2.2 排序算法

对数据集合（如 Go 语言中的切片）进行排序，一直是学习计算机科学的基础部分。在本节中，我们将介绍两种著名的排序算法，并使用大 O 分析法来检验它们的复杂度。

### 冒泡排序算法

代码清单 2-3 实现了一个通用的冒泡排序算法，该算法假设数据切片是有序的（其基础类型允许每个元素进行大于或小于的比较）。

```
package main
import(
"fmt"
)
type Ordered interface {
~float64 | ~int | ~string
}
func bubblesortT Ordered {
n := len(data)
for i:= 0; i < n - 1; i++ {
for j := 0; j < n - 1 - i; j++ {
if data[j] > data[j + 1] {
data[j], data[j + 1] = data[j + 1], data[j]
}
}
}
}
func main() {
numbers := []float64{3.5, -2.4, 12.8, 9.1}
names := []string{"Zachary", "John", "Moe", "Jim", "Robert"}
bubblesortfloat64
fmt.Println(numbers)
bubblesortstring
fmt.Println(names)
}
/* Output
[-2.4 3.5 9.1 12.8]
[Jim John Moe Robert Zachary]
*/
Listing 2-3
Generic bubble sort
```

`Ordered` 类型可以包含更多的基础类型。每个基础类型前面的波浪符号表示，任何使用了该给定基础类型的用户自定义类型都被视为`Ordered`。

冒泡排序因其相对简单而在计算机科学入门课程中颇受欢迎。元素会被顺序比较，如果顺序不对则进行交换。在外层循环的每次迭代中，最大的值会“冒泡”到切片的最右侧位置。由于条件

```
j < n – 1 – i.
```

这个位置不会在下一次内层循环中被考虑。

用粗体显示的嵌套 for 循环使得这个算法成为 `O(n²)` 算法。通常，k 个嵌套循环会产生一个 `O(n^k)` 的算法。

当被排序的切片已经有序时，冒泡排序效率最高；而当切片完全逆序时，效率最低。

接下来，我们将介绍使用最广泛的排序算法之一，经典的快速排序。

### 快速排序算法

顾名思义，该算法以其极快的排序性能而著称。

代码清单 2-4 展示了通用快速排序的实现。

```
package main
import(
"fmt"
)
type Ordered interface {
~float64 | ~int | ~string
}
func quicksortT Ordered {
if low < high {
pivot := partition(data, low, high)
quicksort(data, low, pivot - 1)
quicksort(data, pivot + 1, high)
}
}
func partitionT Ordered int {
pivot := data[low]
i := low
j := high + 1
for {
i++
j--
for data[i] < pivot && i < high {
i++
}
for data[j] > pivot && j > low {
j--
}
if i >= j {
break
}
data[i], data[j] = data[j], data[i]
}
data[low] = data[j]
data[j] = pivot
return j
}
func main() {
numbers := []float64{3.5, -2.4, 12.8, 9.1}
names := []string{"Zachary", "John", "Moe", "Jim", "Robert"}
quicksortfloat64 - 1)
fmt.Println(numbers)
quicksortstring - 1)
fmt.Println(names)
}
/* Output
[-2.4 3.5 9.1 12.8]
[Jim John Moe Robert Zachary]
*/
Listing 2-4
Generic quicksort
```

快速排序算法是分治算法的一个例子。我们将原始切片分成两个较小的切片，并通过继续将每个切片一分为二来对它们进行排序，直到最终得到包含两个元素的切片，然后对它们进行比较。

如果原始切片有 n 个元素，我们可以执行 `log[2]n` 次分治操作（即，将 n 除以 2 直到得到两个元素的次数）。

这假设我们每次都将切片对半分割。仔细检查 `partition` 函数会发现情况并非总是如此。

`partition` 函数使用其最左边的元素作为基准元素。然后，它在切片内移动数据，以确保基准元素左边的元素都更小，右边的元素都更大。

### 大 O 分析

我们通过一个例子来详细说明 `partition` 函数是如何工作的。

假设我们要分割的数组切片是

```
[6, 2, 3, 9, 8, 17, 4]
```

我们选择基准元素为 6（最左边的元素）。

我们递增索引 `i`，直到找到一个比基准元素 6 大的元素。即位置 3 上的元素 9。

现在，从位置 6（最右边位置）的索引 `j` 开始，我们递减 `j`，直到找到一个值小于基准元素的元素。即位置 6 上的元素 4。我们交换位置 3 和位置 6 上的元素，得到

```
[6, 2, 3, 4, 8, 17, 9]
```

从索引 3 开始，我们再次递增 `i`，直到找到一个大于 6 的元素，即位置 4 上的元素 8。我们递减 `j`（位于位置 6），直到找到一个小于基准元素 6 的元素。即位置 3 上的元素 4。因为 `i` 不小于 `j`，所以我们不交换位置 3 和位置 4 上的元素。

由于 `i` 不再小于 `j`，我们退出外层 for 循环。

最后，我们交换基准元素与位置 `j` 上的元素，得到

```
[4, 2, 3, 6, 8, 17, 9]
```

基准元素 6 左边的所有元素都小于 6，右边的所有元素都大于 6。

### 快速排序的最坏情况

如果原始数组已经有序，那么基准元素将是最小的，这将是最坏情况。我们必须在第一遍交换 n – 1 个元素，第二遍交换 n – 2 个元素，依此类推，从而产生一个 `O(n²)` 算法。这是快速排序的讽刺之处之一。数据越接近初始有序状态，快速排序的性能就越差。

在快速排序之前加入一个有用的过滤器是测试输入是否已经排好序。如果是，就退出，不执行任何排序。

由于 `partition` 函数是 `O(n)`，当被排序的数据尚未排序或接近排序时，快速排序算法是 `O(n log[2] n)`。

### 比较冒泡排序和快速排序

在代码清单 2-5 中，我们使用正弦波数据作为数组切片，比较了冒泡排序和快速排序。

```
package main
import(
"fmt"
"math"
"time"
)
const size = 100_000
type Ordered interface {
~float64 | ~int | ~string
}
// Snip – See Listings 2.3 and 2.4
func main() {
data := make([]float64, size)
for i := 0; i < size; i++ {
data[i] = math.Sin(float64(i * i))
}
start := time.Now()
quicksortfloat64 - 1)
elapsed := time.Since(start)
fmt.Println("Elapsed sort time for sine wave using quicksort: ", elapsed)
data = make([]float64, size)
for i := 0; i < size; i++ {
data[i] = math.Sin(float64(i * i))
}
start = time.Now()
bubblesortfloat64
elapsed = time.Since(start)
fmt.Println("Elapsed sort time for sine wave using bubblesort: ", elapsed)
}
/*Output
lapsed sort time for sine wave using quicksort:  7.808522ms
Elapsed sort time for sine wave using bubblesort:  12.26859692s
*/
Listing 2-5
Comparing bubblesort with quicksort
```

`O(n log[2] n)` 的快速排序比 `O(n²)` 的冒泡排序快了近 1600 倍。



### 并发快速排序

我们能否通过并发来提升`quicksort`的性能？

作为迈向并发解决方案的基石，我们首先考虑另一种`O(n²)`排序算法——`InsertSort`，其实现如下：

```
func InsertSortT Ordered {
	i := 1
	for i < len(data) {
		h := data[i]
		j := i - 1
		for j >= 0 && h < data[j] {
			data[j + 1] = data[j]
			j -= 1
		}
		data[j + 1] = h
		i += 1
	}
}
```

让我们通过一个简单示例来理解这个排序算法的工作原理。

假设我们想要排序的切片`data`为`[5, 1, 12, 9]`。

变量`i`被初始化为 1。

在 for 循环中，`h`被设置为`data[1]`，即 1。变量`j`被设置为 0。在嵌套的 for 循环中，`data[j + 1]`被设置为`data[0]`，所以`data[1]`被设置为 5。内层循环终止。接着，`data[0]`被设置为 1。此时切片变为`[1, 5, 12, 9]`。

递增`i`后，我们再次执行外层循环。变量`h`被设置为 12，`j`被设置为 1。由于 12 不小于 1 或 5，内层循环终止。我们将`i`递增到 3。变量`h`被设置为 9。在内层循环中，由于 9 小于 12，切片变为`[1, 5, 9, 12]`，排序完成。

由于存在两个嵌套循环，对于较大的 n，`InsertSort`的时间复杂度为`O(n²)`。

在外层循环的每次迭代中，下一个尚未处理的元素会被插入到第一个比它大的元素的左侧。

在代码清单 2-6 中，我们展示了对 5000 万个数字进行并发快速排序的实现。

```
package main

import (
	"fmt"
	"time"
	"math/rand"
	"sync"
)

const size = 50_000_000
const threshold = 5000

type Ordered interface {
	~float64 | ~int | ~string
}

func InsertSortT Ordered {
	i := 1
	for i < len(data) {
		h := data[i]
		j := i - 1
		for j >= 0 && h < data[j] {
			data[j + 1] = data[j]
			j -= 1
		}
		data[j + 1] = h
		i += 1
	}
}

func ConcurrentQuicksortT Ordered {
	defer wg.Done()
	if len(data) <= 30 {
		InsertSort(data)
		return
	}

	mid := Partition(data)
	var portion[] T
	if mid < len(data)/2 {
		portion = data[mid + 1:]
	} else {
		portion = data[:mid]
	}

	if len(portion) > threshold {
		wg.Add(1)
		go func(data[] T) {
			defer wg.Done()
			ConcurrentQuicksort(data, wg)
		}(portion)
	} else {
		ConcurrentQuicksort(portion, wg)
	}
	InsertSort(data)
}

func QSortT Ordered {
	var wg sync.WaitGroup
	ConcurrentQuicksort(data, &wg)
	wg.Wait()
}

func partitionT Ordered int {
	var pivot = data[low]
	var i = low
	var j = high
	for i < j {
		for data[i] <= pivot && i < high {
			i++
		}
		for data[j] > pivot && j > low {
			j--
		}
		if i < j {
			data[i], data[j] = data[j], data[i]
		}
	}
	data[low] = data[j]
	data[j] = pivot
	return j
}

func quicksortT Ordered {
	if low < high {
		var pivot = partition(data, low, high)
		quicksort(data, low, pivot)
		quicksort(data, pivot + 1, high)
	}
}

func main() {
	data := make([]float64, size)
	for i := 0; i < size; i++ {
		data[i] = 100.0 * rand.Float64()
	}
	data2 := make([]float64, size)
	copy(data2, data)

	start := time.Now()
	QSortfloat64
	elapsed := time.Since(start)
	fmt.Println("Elapsed time for concurrent quicksort = ", elapsed)
	fmt.Println("Is sorted: ", IsSorted(data))

	start = time.Now()
	quicksort(data2, 0, len(data2) - 1)
	elapsed = time.Since(start)
	fmt.Println("Elapsed time for regular quicksort = ", elapsed)
	fmt.Println("Is sorted: ", IsSorted(data2))
}

/* Output
Elapsed time for concurrent quicksort = 710.431619ms
Is sorted:  true
Elapsed time for regular quicksort = 5.382400384s
Is sorted:  true
*/
Listing 2-6
并发快速排序的实现
```

结果再次令人惊叹。在对 5000 万个数字进行排序时，比较普通快速排序和并发快速排序的排序时间，我们发现普通快速排序比并发快速排序慢约 7.6 倍。

当切片长度小于 30 时，我们使用`InsertSort`来完成排序。当切片长度小于阈值 5000 时，我们使用普通快速排序来完成排序。常量 30 和 5000 是通过经验确定的。这样做的目的是避免为排序小尺寸切片而部署大量 goroutine 所带来的开销。

需要注意的是，`InsertSort`在尺寸小于 30 的切片上的性能并不受`O(n²)`支配，因为`O(n²)`是面向大 n 的渐近上界。

### 归并排序算法

接下来要研究的排序算法是经典的`mergesort`算法。与快速排序一样，它也是一种分治算法。我们将原始切片替换为两个切片，每个切片的大小为原始切片的一半。然后，这两个半切片再进一步分成四分之一切片，依此类推，直到得到大小为 1 的切片。通过递归，我们将这些切片合并起来，如代码清单 2-7 所示。

```
package main

import (
	"fmt"
	"math/rand"
	"time"
)

const size = 50_000_000

type Ordered interface {
	~float64 | ~int | ~string
}

func IsSortedT Ordered bool {
	for i := 1; i < len(data); i++ {
		if data[i] < data[i-1] {
			return false
		}
	}
	return true
}

func InsertSortT Ordered {
	i := 1
	for i < len(data) {
		h := data[i]
		j := i - 1
		for j >= 0 && h < data[j] {
			data[j + 1] = data[j]
			j -= 1
		}
		data[j + 1] = h
		i += 1
	}
}

func MergeT Ordered []T {
	result := make([]T, len(left) + len(right))
	i := 0
	for len(left) > 0 && len(right) > 0 {
		if left[0] < right[0] {
			result[i] = left[0]
			left = left[1:]
		} else {
			result[i] = right[0]
			right = right[1:]
		}
		i++
	}
	for j := 0; j < len(left); j++ {
		result[i] = left[j]
		i++
	}
	for j := 0; j < len(right); j++ {
		result[i] = right[j]
		i++
	}
	return result
}

func MergeSortT Ordered []T {
	if len(data) > 100 {
		middle := len(data) / 2
		left := data[:middle]
		right := data[middle:]
		data = Merge(MergeSort(right), MergeSort(left))
	} else {
		InsertSort(data)
	}
	return data
}

func main() {
	data := make([]float64, size)
	for i := 0; i < size; i++ {
		data[i] = 100.0 * rand.Float64()
	}
	/*
	data2 := make([]float64, size)
	copy(data2, data)
	*/
	start := time.Now()
	result := MergeSortfloat64
	elapsed := time.Since(start)
	fmt.Println("Elapsed time for MergeSort = ", elapsed)
	fmt.Println("Is sorted: ", IsSorted(result))
}

/* Output
Elapsed time for MergeSort = 6.18063849s
Is sorted: true
*/
Listing 2-7
归并排序算法
```

这个算法比快速排序更容易理解。函数`Merge`通过合并两个输入切片`left`和`right`中的元素，构建一个新的切片`result`，从而保证`result`有序。

递归函数`MergeSort`将输入数组划分为`left`和`right`两部分，并对递归调用`MergeSort`得到的结果调用`Merge`函数。

需要注意的是，与`quicksort`不同，`MergeSort`不是原地排序。与`quicksort`相比，这需要额外的内存分配。

由于`Merge`的时间复杂度为`O(n)`，且`MergeSort`中有`log₂n`次递归调用，因此`MergeSort`的复杂度为`O(n log₂n)`。



### 并发归并排序

我们能否通过并发来提升 `MergeSort` 的性能？答案是肯定的！

代码清单 2-8 展示了 `MergeSort` 的一个并发实现。

```go
package main

import (
    "fmt"
    "time"
    "math/rand"
    "sync"
)

const size = 50_000_000
const max = 5000

type Ordered interface {
    ~float64 | ~int | ~string
}

func IsSortedT Ordered bool {
    for i := 1; i < len(data); i++ {
        if data[i] < data[i-1] {
            return false
        }
    }
    return true
}

func InsertSortT Ordered {
    for i := 1; i < len(data); i++ {
        value := data[i]
        j := i - 1
        for j >= 0 && data[j] > value {
            data[j+1] = data[j]
            j--
        }
        data[j+1] = value
    }
}

func MergeT Ordered []T {
    result := make([]T, len(left)+len(right))
    i, j, k := 0, 0, 0
    for i < len(left) && j < len(right) {
        if left[i] <= right[j] {
            result[k] = left[i]
            i++
        } else {
            result[k] = right[j]
            j++
        }
        k++
    }
    for i < len(left) {
        result[k] = left[i]
        i++
        k++
    }
    for j < len(right) {
        result[k] = right[j]
        j++
        k++
    }
    return result
}

func MergeSortT Ordered []T {
    if len(data) > 100 {
        middle := len(data) / 2
        left := data[:middle]
        right := data[middle:]
        data = Merge(MergeSort(right), MergeSort(left))
    } else {
        InsertSort(data)
    }
    return data
}

func ConcurrentMergeSortT Ordered []T {
    if len(data) > 1 {
        if len(data) <= max {
            return MergeSort(data)
        } else { // Concurrent
            middle := len(data) / 2
            left := data[:middle]
            right := data[middle:]
            var wg sync.WaitGroup
            wg.Add(2)
            var data1, data2 []T
            go func() {
                defer wg.Done()
                data1 = ConcurrentMergeSort(left)
            }()
            go func() {
                defer wg.Done()
                data2 = ConcurrentMergeSort(right)
            }()
            wg.Wait()
            return Merge(data1, data2)
        }
    }
    return nil
}

func main() {
    data := make([]float64, size)
    for i := 0; i < size; i++ {
        data[i] = 100.0 * rand.Float64()
    }
    start := time.Now()
    result := ConcurrentMergeSort(data)
    elapsed := time.Since(start)
    fmt.Println("Elapsed time for concurrent mergesort = ", elapsed)
    fmt.Println("Sorted: ", IsSorted(result))
}

/* Output
Elapsed time for concurrent mergesort =  1.275120179s
Sorted:  true
*/
代码清单 2-8
并发实现的归并排序
```

这两个用粗体标出的 goroutine 都并发地执行 `MergeSort`。`wg.Wait()` 会强制 `Merge` 等待这两个 goroutine 都完成，然后才对两个结果进行合并。

为了避免为小规模的 `data` 创建 goroutine 带来的开销，当 `data` 的大小小于 `max`（本例中为 5000）时，会使用普通的顺序 `MergeSort`。

`ConcurrentMergeSort` 在处理包含 5000 万个随机浮点数的切片时，性能略慢于 `ConcurrentQuickSort`，但快于顺序版本。

### 结论

快速排序的平均时间复杂度为 O(n log₂ n)，并且是原地排序（无需额外存储空间）。如果输入数据已经是排好序或接近排好序的，其复杂度会退化为 O(n²)。并发快速排序非常快。

归并排序的平均时间复杂度为 O(n log₂ n)。它不是原地排序，因此需要额外的存储空间。通常，它比快速排序慢。如果输入数据已经是排好序或接近排好序的，归并排序会非常快。并发归并排序也极其快，但仍慢于并发快速排序。

下一节，我们将探讨数组切片搜索的问题。

## 2.3 搜索数组切片

在本节中，我们将重点讨论如何高效地搜索数组切片。

在数据结构中搜索特定存储信息是我们执行的重要操作之一，而且通常也是我们创建该数据结构的原因。随着我们在本书后续部分逐步介绍各种数据结构，我们还将研究如何高效地搜索存储在数据结构中的信息。

### 线性搜索

用于搜索切片的最简单的搜索算法是线性搜索。我们按顺序遍历切片中的所有元素，直到找到匹配项或完成对所有元素的搜索。

代码清单 2-9 展示了在切片中进行线性搜索的代码。

```go
package main

import (
    "fmt"
    "time"
    "math/rand"
)

const size = 100_000_000

type Ordered interface {
    ~float64 | ~int | ~string
}

func linearSearchT Ordered bool {
    // 如果 T 在切片中则返回 true
    for i := 0; i < len(slice); i++ {
        if slice[i] == target {
            return true
        }
    }
    return false
}

func main() {
    data := make([]float64, size)
    for i := 0; i < size; i++ {
        data[i] = 100.0 * rand.Float64()
    }
    start := time.Now()
    result := linearSearchfloat64
    elapsed := time.Since(start)
    fmt.Println("使用 linearSearch 搜索包含 100_000_000 个浮点数的切片耗时 = ", elapsed)
    fmt.Println("搜索结果 = ", result)
    start = time.Now()
    result = linearSearchfloat64
    elapsed = time.Since(start)
    fmt.Println("使用 linearSearch 搜索包含 100_000_000 个浮点数的切片耗时 = ", elapsed)
    fmt.Println("搜索结果 = ", result)
}

/* Output
Time to search slice of 100_000_000 floats using linearSearch =  54.464458ms
Result of search is false
Time to search slice of 100_000_000 floats using linearSearch =  17.981833ms
Result of search is true
*/
代码清单 2-9
切片的线性搜索
```

上述基准测试是在一台配备 M1 Max 处理器和 32G 内存的 MacBook Pro 上完成的。

### 并发搜索

在代码清单 2-10 中，我们展示了数组切片并发搜索的细节。

```go
package main

import (
    "fmt"
    "time"
    "math/rand"
    "runtime"
)

type Ordered interface {
    ~float64 | ~int | ~string
}

const size = 100_000_000

func searchSegmentT Ordered {
    // 生成布尔值并放入 ch
    for i := a; i < b; i++ {
        if slice[i] == target {
            ch <- true
        }
    }
    ch <- false
}

func concurrentSearchT Ordered bool {
    ch := make(chan bool)
    numSegments := runtime.NumCPU()
    segmentSize := int(float64(len(data)) / float64(numSegments))
    // 启动 numSegments 个 goroutine
    for index := 0; index < numSegments; index++ {
        go searchSegment(data, target, index*segmentSize, index*segmentSize+segmentSize, ch)
    }
    num := 0 // 已完成的 goroutine 数量
    for {
        select {
        case value := <-ch: // 阻塞，直到某个 goroutine 将一个布尔值放入通道
            if value == true {
                return true
            }
            num += 1
            if num == numSegments { // 所有 goroutine 都已完成
                return false
            }
        }
    }
    return false
}

func main() {
    data := make([]float64, size)
    for i := 0; i < size; i++ {
        data[i] = 100.0 * rand.Float64()
    }
    start := time.Now()
    result := concurrentSearchfloat64 // 应返回 false
    elapsed := time.Since(start)
    fmt.Println("使用 concurrentSearch 搜索包含 100_000_000 个浮点数的切片耗时 = ", elapsed)
    fmt.Println("搜索结果 = ", result)
    start = time.Now()
    result = concurrentSearchfloat64 // true
    elapsed = time.Since(start)
    fmt.Println("使用 concurrentSearch 搜索包含 100_000_000 个浮点数的切片耗时 = ", elapsed)
    fmt.Println("搜索结果 = ", result)
}

/*
Time to search slice of 100_000_000 floats using concurrentSearch =  9.666792ms
Result of search is false
Time to search slice of 100_000_000 floats using concurrentSearch =  5.311917ms
Result of search is true
*/
代码清单 2-10
切片的并发搜索
```

通过使用并发，最坏情况下的搜索时间实现了超过 5 倍的改进。线性搜索和并发搜索的复杂度均为 O(n)。



### 二分搜索

如果要搜索的切片数据已排序，可以使用二分搜索算法。由于每次迭代都会将搜索范围减半，该算法的时间复杂度为 `O(log[2]n)`。

列表 2-11 展示了在已排序数据上进行二分搜索的细节。

```
package main
import (
"fmt"
"time"
)
const size = 100_000_000
type Ordered interface {
~float64 | ~int | ~string
}
func binarySearchT Ordered bool {
low := 0
high := len(slice) - 1
for low <= high {
median := (low + high) / 2
if slice[median] < target {
low = median + 1
} else {
high = median - 1
}
}
if low == len(slice) || slice[low] != target {
return false
}
return true
}
func main() {
data := make([]float64, size)
for i := 0; i < size; i++ {
data[i] = float64(i) // 已排序
}
start := time.Now()
result := binarySearchfloat64
elapsed := time.Since(start)
fmt.Println("使用二分搜索在包含 100_000_000 个 float64 的切片中搜索耗时 = ", elapsed)
fmt.Println("搜索结果 = ", result)
start = time.Now()
result = binarySearchfloat64)
elapsed = time.Since(start)
fmt.Println("使用二分搜索在包含 100_000_000 个 float64 的切片中搜索耗时 = ", elapsed)
fmt.Println("搜索结果 = ", result)
}
/* 输出
使用二分搜索在包含 100_000_000 个 float64 的切片中搜索耗时 = 1.375μs
搜索结果 = false
使用二分搜索在包含 100_000_000 个 float64 的切片中搜索耗时 = 334ns
搜索结果 = true
*/
列表 2-11
对已排序数据进行二分搜索
```

这里的搜索时间明显小于之前对随机数据进行搜索的时间。这是因为数据是已排序的。

最快的排序算法复杂度为 `O(nlog[2]n)`。如果需要在切片内执行大量独立的搜索，那么先对数据进行排序，然后使用二分搜索进行多次搜索可能是值得的。

由于排序本身的开销，在仅对切片执行一次搜索之前对其进行排序是不划算的。

## 2.4 小结

大 O 表示法描述了算法的渐近效率。本章中我们研究了几种经典的排序和搜索算法，并通过它们的大 O 特性来表征它们。我们还为每种排序算法提供了并发解决方案，并观察到其性能得到了显著提升。

在下一章中，我们将讨论 Go 语言中不使用类的面向对象编程。

