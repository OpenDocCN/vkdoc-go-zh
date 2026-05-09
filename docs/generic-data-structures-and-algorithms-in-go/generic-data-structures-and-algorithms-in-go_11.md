# 6. 队列与列表

**队列**是另一种相对简单的数据类型。它在应用开发中有许多实际用途。

队列以先进先出（FIFO）的方式组织数据。由于 FIFO 的特性，其最明显的应用是模拟等待队列。这可以是等待服务的顾客队列、在打印队列中等待的打印作业、等待 CPU 访问权的并发进程，以及其他许多需要排队等待的应用。新项目被插入到队列的尾部，项目则从队列的头部移除。队列维护了项目插入的顺序。

在本章中，我们介绍**队列**的两种实现，并比较它们的效率。我们还介绍**队列**的几种应用。

**双端队列**比**队列**更通用。它允许在结构的前部以及后部进行插入和删除操作。我们介绍**双端队列**的一种实现以及使用**双端队列**的一个应用。

**优先队列**是**队列**的一个特殊类型。我们展示了**优先队列**的一种实现以及一个涉及航空公司乘客的应用。

**列表**是一种比**队列**更通用的数据类型。项目可以插入到列表的前部、尾部或中间任何位置。我们介绍了单向链表和双向链表的实现。

在下一节中，我们将定义队列抽象数据类型（ADT）。

## 6.1 队列 ADT

队列 ADT 由以下六种操作定义。

| `Insert(item)` – 向队列添加项目 |
| `Remove()` item – 移除并返回队列中第一个插入的项目 |
| `First()` item – 访问队列中第一个插入的项目，不修改队列 |
| `Size` int – 返回队列中的项目数量 |
| `Range()` – 返回一个迭代器 |
| `Empty()` – 返回一个布尔值，如果其应用的迭代器没有项目则返回 true |
| `Next()` – 返回迭代器中的下一个项目 |

我们介绍**队列**的两种实现：基于切片和基于节点。在下一节中，我们将重点介绍基于切片的队列实现。



## 6.2 切片队列的实现

代码清单 6-1 展示了 `slicequeue` 包中 `Queue` 的通用切片实现。

`Queue` 结构体包含一个字段 `items`，它是一个泛型类型 `T` 的切片。

```go
package slicequeue

type Queue[T any] struct {
    items []T
}

type Iterator[T any] struct {
    next  int // items 中的索引
    items []T
}

// Queue 方法
func (queue *Queue[T]) Insert(item T) {
    // 将 item 添加到切片的最右侧位置
    queue.items = append(queue.items, item)
}

func (queue *Queue[T]) Remove() T {
    returnValue := queue.items[0]
    queue.items = queue.items[1:]
    return returnValue
}

func (queue Queue[T]) First() T {
    return queue.items[0]
}

func (queue Queue[T]) Size() int {
    return len(queue.items)
}

func (queue *Queue[T]) Range() Iterator[T] {
    return Iterator[T]{0, queue.items}
}

// Iterator 方法
func (iterator *Iterator[T]) Empty() bool {
    return iterator.next == len(iterator.items)
}

func (iterator *Iterator[T]) Next() T {
    returnValue := iterator.items[iterator.next]
    iterator.next++
    return returnValue
}
代码清单 6-1
Queue 的通用切片实现
```

`Queue` 的 FIFO（先进先出）协议通过将新项插入到 `items` 切片的最右侧位置，并从 `items` 切片的最左侧位置（索引 0）移除项来实现。

### 迭代器

`Iterator` 类型是一个包含索引 `next` 和 `items` 切片的结构体。

如果迭代器字段 `next` 等于 `items` 切片的长度，则 `Iterator` 上的 `Empty` 方法返回 `true`。

`Iterator` 上的 `Next` 方法返回 `items` 切片中索引 `next` 处的值 `T`。

`Queue` 上的 `Range` 方法返回一个 `Iterator`。

代码清单 6-2 展示了一个简单的主驱动程序，它对一个泛型队列进行了操作。

```go
package main

import (
    "fmt"
    "example.com/slicequeue"
)

func main() {
    myQueue := slicequeue.Queue[int]{}
    myQueue.Insert(15)
    myQueue.Insert(20)
    myQueue.Insert(30)
    myQueue.Remove()
    fmt.Println(myQueue.First())

    queue := slicequeue.Queue[float64]{}
    for i := 0; i < 10; i++ {
        queue.Insert(float64(i))
    }
    iterator := queue.Range()
    for {
        if iterator.Empty() {
            break
        }
        fmt.Println(iterator.Next())
    }
    fmt.Println("queue.First() = ", queue.First())
}

/* 输出

queue.First() =  0
*/
代码清单 6-2
泛型队列的驱动程序
```

导入了 `slicequeue` 包。定义并操作了一个类型为 `int` 的队列和另一个类型为 `float64` 的队列。需要注意的是，当构造 `float64` 队列并使用迭代器显示其值时，队列的状态并未改变。

在下一节中，我们将介绍基于节点的 `Queue` 实现。

## 6.3 节点队列的实现

代码清单 6-3 展示了 `Queue` 的节点实现。

```go
package nodequeue

type Node[T any] struct {
    item T
    next *Node[T]
}

type Queue[T any] struct {
    first, last *Node[T]
    length      int
}

type Iterator[T any] struct {
    next *Node[T]
}

// 方法
func (queue *Queue[T]) Insert(item T) {
    newNode := &Node[T]{item, nil}
    if queue.first == nil {
        queue.first = newNode
        queue.last = queue.first
    } else {
        queue.last.next = newNode
        queue.last = newNode
    }
    queue.length += 1
}

func (queue *Queue[T]) Remove() T {
    returnValue := queue.first.item
    queue.first = queue.first.next
    if queue.first == nil {
        queue.last = nil
    }
    return returnValue
}

func (queue Queue[T]) First() T {
    return queue.first.item
}

func (queue Queue[T]) Size() int {
    return queue.length
}

func (queue *Queue[T]) Range() Iterator[T] {
    return Iterator[T]{queue.first}
}

func (iterator *Iterator[T]) Empty() bool {
    return iterator.next == nil
}

func (iterator *Iterator[T]) Next() T {
    returnValue := iterator.next.item
    if iterator.next != nil {
        iterator.next = iterator.next.next
    }
    return returnValue
}
代码清单 6-3
Queue 的通用节点实现
```

定义了一个泛型 `Node` 类型，其中包含一个类型为 `T` 的 `item` 字段和一个指向 `Node` 的指针 `next` 字段。这种递归结构与我们在 `nodestack` 中定义节点时所做的类似。

`Queue` 类型是一个包含两个指向 `Node` 的指针 `first` 和 `last` 的结构体。它们分别指向队列的开头和结尾。

`Insert` 方法会在队列为空时创建一个 `first` 值，并将 `last` 设置为等于 `first`。如果队列已有一个非空（非 nil）的 `first` 值，则会将当前的 `last` 值链接到新节点，并用指向这个新节点的指针替换 `last`。`first` 值不受影响。

`Remove` 方法返回 `first` 节点中的 `item`，并将 `first` 重置为它的 `first.next` 链接。如果 `first` 变为 nil，那么 `last` 字段也会被设置为 nil；否则，它不受影响。

`Iterator` 是一个包含 `next` 字段的结构体，该字段指向一个 `Node`。

`Range` 方法返回一个 `Iterator`，其包含的 `next` 字段指向队列中的第一个项（`first`）。

如果 `next` 字段指向 nil，则 `Iterator` 上的 `Empty` 方法返回 `true`；否则返回 `false`。

最后，`Iterator` 上的 `Next` 方法返回 `next` 节点中的值，并将字段 `iterator.next` 前进到 `iterator.next.next` 链接。

一个用于操作 `queuenode` 的主驱动程序和代码清单 6-2 中的程序相同，只是使用的包是 `example.com/nodequeue`。

在下一节中，我们将比较基于切片的 `Queue` 和基于节点的 `Queue` 的性能。

## 6.4 比较切片队列和节点队列的性能

代码清单 6-4 展示了一个程序，用于比较从 `slicequeue` 和 `nodequeue` 中插入和移除项的执行时间。

```go
// 我们比较 slicequeue 和 nodequeue 的性能
package main

import (
    "fmt"
    "example.com/nodequeue"
    "example.com/slicequeue"
    "time"
)

const size = 1_000_000

func main() {
    sliceQueue := slicequeue.Queue[int]{}
    nodeQueue := nodequeue.Queue[int]{}

    start := time.Now()
    for i := 0; i < size; i++ {
        sliceQueue.Insert(i)
    }
    elapsed := time.Since(start)
    fmt.Println("在 sliceQueue 中插入 100 万个整型数的时间为", elapsed)

    start = time.Now()
    for i := 0; i < size; i++ {
        nodeQueue.Insert(i)
    }
    elapsed = time.Since(start)
    fmt.Println("在 nodeQueue 中插入 100 万个整型数的时间为", elapsed)

    start = time.Now()
    for i := 0; i < size; i++ {
        sliceQueue.Remove()
    }
    elapsed = time.Since(start)
    fmt.Println("从 sliceQueue 中移除 100 万个整型数的时间为", elapsed)

    start = time.Now()
    for i := 0; i < size; i++ {
        nodeQueue.Remove()
    }
    elapsed = time.Since(start)
    fmt.Println("从 nodeQueue 中移除 100 万个整型数的时间为", elapsed)
}

/* 输出
在 sliceQueue 中插入 100 万个整型数的时间为 18.841914ms
在 nodeQueue 中插入 100 万个整型数的时间为 30.275662ms
从 sliceQueue 中移除 100 万个整型数的时间为 1.413447ms
从 nodeQueue 中移除 100 万个整型数的时间为 2.818313ms
*/
代码清单 6-4
对 slicequeue 和 nodequeue 进行性能基准测试
```

正如预期，切片队列明显比节点队列快，这是因为基于节点的队列中存在与指针访问相关的开销。

在下一节中，我们将介绍并实现 `Deque` 数据结构。



## 6.5 双端队列

`Deque`（双端队列）是一种允许在结构的前端或后端插入或删除元素的队列。

代码清单 6-5 展示了泛型 `Deque` 的一个切片实现。

```go
package deque
type Deque[T any] struct {
items []T
}
func (deque *Deque[T]) InsertFront(item T) {
deque.items = append(deque.items, item) // 扩展 deque.items
for i := len(deque.items) - 1; i > 0 ; i-- {
deque.items[i] = deque.items[i - 1]
}
deque.items[0] = item
}
func (deque *Deque[T]) InsertBack(item T) {
deque.items = append(deque.items, item)
}
func (deque *Deque[T]) First() T {
return deque.items[0]
}
func (deque *Deque[T]) RemoveFirst() T {
returnValue := deque.items[0]
deque.items = deque.items[1:]
return returnValue
}
func (deque *Deque[T]) Last() T {
return deque.items[len(deque.items) - 1]
}
func (deque *Deque[T]) RemoveLast() T {
length := len(deque.items)
returnValue := deque.items[length - 1]
deque.items = deque.items[:(length - 1)]
return returnValue
}
func (deque *Deque[T]) Empty() bool {
return len(deque.items) == 0
}
代码清单 6-5
Deque 的泛型切片实现
```

代码清单 6-6 展示了一个使用 `Deque` 的简单驱动程序。

```go
package main
import (
"fmt"
"example.com/deque"
)
func main() {
myDeque := deque.Deque[int]{}
myDeque.InsertFront(5)
myDeque.InsertBack(10)
myDeque.InsertFront(2)
myDeque.InsertBack(12) // 2 5 10 12
fmt.Println("myDeque.First() = ", myDeque.First())
fmt.Println("myDeque.Last() = ", myDeque.Last())
myDeque.RemoveLast()
myDeque.RemoveFirst()
fmt.Println("myDeque.First() = ", myDeque.First())
fmt.Println("myDeque.Last() = ", myDeque.Last())
}
/* 输出
myDeque.First() =  2
myDeque.Last() =  12
myDeque.First() =  5
myDeque.Last() =  10
*/
代码清单 6-6
操作 Deque
```

在下一节中，我们将介绍一个使用 `Deque` 的应用程序。

## 6.6 Deque 应用

给定一个数组和一个整数 `k`，找出每个大小为 `k` 的连续子数组中的最大值。

例如，考虑以下问题：

输入数组：`input := []int{9, 1, 1, 0, 0, 0, 1, 0, 6, 8}`，其中 `k = 3`

- `9, 1, 1` 的最大值是 `9`。
- `1, 1, 0` 的最大值是 `1`。
- `1, 0, 0` 的最大值是 `1`。
- `0, 0, 0` 的最大值是 `0`。
- `0, 0, 1` 的最大值是 `1`。
- …

因此输出为 `[9 1 1 0 1 1 6 8]`。

代码清单 6-7 展示了该问题的一个简单暴力求解方案。

```go
package main
import (
"fmt"
)
func MaxSubarray(input []int, k int) (output []int) {
for first := 0; first < len(input)-k+1; first++ {
max := input[first]
for second := 1; second < k; second++ {
if input[first+second] > max {
max = input[first+second]
}
}
output = append(output, max)
}
return output
}
func main() {
input := []int{3, 1, 6, 4, 2, 10, 5, 9}
output := MaxSubarray(input, 3)
fmt.Println("Output = ", output)
}
/* 输出
Output =  [6 6 6 10 10 10]
*/
代码清单 6-7
最大连续子数组问题的暴力求解方案
```

由于存在嵌套循环，该解决方案的计算复杂度为 `O(n * k)`，其中 `n` 是输入切片的长度。

我们能做得更好吗？当 `n` 和 `k` 很大时，这将会非常有用。利用 `Deque` 的服务，我们可以做得更好。

考虑如下所示的 `MaxSubarrayUsingDeque` 函数：

```go
func MaxSubarrayUsingDeque(input []int, k int) (output []int) {
deque := deque.Deque[int]{}
var index int
// 第一个窗口
for index = 0; index < k; index++ {
for {
if deque.Empty() || input[index] < input[deque.Last()] {
break
}
deque.RemoveLast()
}
deque.InsertBack(index)
}
output = append(output, input[deque.First()])
// 剩余窗口
for ; index < len(input); index++ {
// 移除窗口外的索引
for {
if deque.Empty() || deque.First() > index-k {
break
}
deque.RemoveFirst()
}
// 移除小于当前添加元素的值
for {
if deque.Empty() || input[index] < input[deque.Last()] {
break
}
deque.RemoveLast()
}
deque.InsertBack(index)
output = append(output, input[deque.First()])
}
return output
}
```

让我们以前面例子的一部分来逐步分析该函数。

一个泛型类型为 `int` 的 `deque` 被初始化为空。

由于 `deque` 为空，我们跳出内部 for 循环，将索引 0 插入 deque，然后将 `index` 从 0 推进到 1。

由于 `input[1]` 小于 `input[0]`，我们再次跳出内部 for 循环，将索引 1 插入 deque 的后端，因此 deque 包含 `[0 1]`。我们将索引推进到 2。

由于 `input[2]` 不小于 `input[1]`，我们从 deque 中移除最后一个元素 1，deque 变为 `[0]`。由于 `input[2]` 小于 `input[0]`，我们跳出内部循环，将索引 2 插入 deque 的后端，产生 `[0 2]`。外部 for 循环执行完毕。我们确保 deque 中的第一个元素是 deque 中的最大值。

在第二个外部 for 循环中，我们将 `input[deque.First()]` 追加到 `output` 中，即值 9。

第二个外部 for 循环的逻辑与第一个外部 for 循环相同。首先，清除 deque 中超出索引窗口范围的值，该窗口每次迭代后向右移动一位。然后，将接下来的 k 个值填充到 deque 中，并对这些值进行旋转，使得 deque 中的第一个值最大。

该算法的计算复杂度为 `O(n)`。

代码清单 6-8 比较了暴力算法与基于 deque 的算法的性能。

```go
package main
import (
"fmt"
"example.com/deque"
"time"
"math/rand"
)
const size = 1_000_000
func MaxSubarrayBruteForce(input []int, k int) (output []int) {
for first := 0; first < len(input)-k+1; first++ {
max := input[first]
for second := 1; second < k; second++ {
if input[first+second] > max {
max = input[first+second]
}
}
output = append(output, max)
}
return output
}
func MaxSubarrayUsingDeque(input []int, k int) (output []int) {
deque := deque.Deque[int]{}
var index int
// 第一个窗口
for index = 0; index < k; index++ {
for {
if deque.Empty() || input[index] < input[deque.Last()] {
break
}
deque.RemoveLast()
}
deque.InsertBack(index)
}
output = append(output, input[deque.First()])
// 剩余窗口
for ; index < len(input); index++ {
// 移除窗口外的索引
for {
if deque.Empty() || deque.First() > index-k {
break
}
deque.RemoveFirst()
}
// 移除小于当前添加元素的值
for {
if deque.Empty() || input[index] < input[deque.Last()] {
break
}
deque.RemoveLast()
}
deque.InsertBack(index)
output = append(output, input[deque.First()])
}
return output
}
func main() {
input := []int{9, 1, 1, 0, 0, 0, 1, 0, 6, 8}
output1 := MaxSubarrayBruteForce(input, 3)
fmt.Println("Output = ", output1)
output2 := MaxSubarrayUsingDeque(input, 3)
fmt.Println("Output = ", output2)
// 对两种算法进行性能基准测试
input = []int{}
for i := 0; i < size; i++ {
input = append(input, rand.Intn(1000))
}
start := time.Now()
MaxSubarrayUsingDeque(input, 10000)
elapsed := time.Since(start)
fmt.Println("使用 Deque: ", elapsed)
start = time.Now()
MaxSubarrayBruteForce(input, 10000)
elapsed = time.Since(start)
fmt.Println("使用暴力算法: ", elapsed)
}
/* 输出
Output = [9 1 1 0 1 1 6 8]
Output = [9 1 1 0 1 1 6 8]
使用 Deque: 21.873658ms
使用暴力算法: 6.042102028s
*/
代码清单 6-8
比较暴力算法与基于 deque 的算法的性能
```

结果对比显著：基于 deque 的算法耗时 `21.87ms`，而暴力算法耗时 `6.04` 秒。

在下一节中，我们将介绍并实现一个优先队列。



## 6.7 优先队列

优先队列在现实世界中广泛存在。例如，当乘客排队登机时，许多航空公司会根据乘客的优先级进行排序。这种优先级可能基于年龄（儿童享有高优先级）、票价（头等舱乘客获得高优先级）、忠诚度积分（常旅客）、身体状况或其他决定客户优先级的因素。在每个优先级分组内部，遵循通常的先进先出（FIFO）队列规则。

我们在此假设每个待插入队列的项目只能被分配有限数量的优先级。

我们将展示一种实现方式：使用一个切片（slice），其中每个元素都包含一个普通队列。

切片中的第一个队列存放最高优先级的项目，第二个队列存放第二高优先级的项目，以此类推。

当插入一个项目时，我们访问与其优先级对应的队列，并在该队列中执行插入操作。

通过为切片的每个元素使用基于节点的队列，我们定义了一个泛型 `PriorityQueue` 和创建优先队列的函数，具体如下：

```
type PriorityQueue[T any] struct {
	q []nodequeue.Queue[T] // 队列切片
	size int
}

func NewPriorityQueueT any (pq PriorityQueue[T]) {
	pq.q = make([]nodequeue.Queue[T], numberPriorities)
	return pq
}
```

`NewPriorityQueue` 构造函数定义了一个包含 `numberPriorities` 个节点队列的切片。

列表 6-9 定义了 `Passenger` 类型，并展示了 `PriorityQueue` 的实现及 `main` 驱动函数。在该驱动函数中，定义了一个以 `Passenger` 为泛型类型的航空队列，并向其中插入了一组乘客。随后移除若干乘客，并输出了队列头部的元素。

```
package main

import (
	"example.com/nodequeue"
	"fmt"
)

type Passenger struct {
	name     string
	priority int
}

type PriorityQueue[T any] struct {
	q    []nodequeue.Queue[T]
	size int
}

func NewPriorityQueueT any (pq PriorityQueue[T]) {
	pq.q = make([]nodequeue.Queue[T], numberPriorities)
	return pq
}

// 优先队列的方法
func (pq *PriorityQueue[T]) Insert(item T, priority int) {
	pq.q[priority-1].Insert(item)
	pq.size++
}

func (pq *PriorityQueue[T]) Remove() T {
	pq.size--
	for i := 0; i < len(pq.q); i++ {
		if pq.q[i].Size() > 0 {
			return pq.q[i].Remove()
		}
	}
	var zero T
	return zero
}

func (pq *PriorityQueue[T]) First() T {
	for _, queue := range pq.q {
		if queue.Size() > 0 {
			return queue.First()
		}
	}
	var zero T
	return zero
}

func (pq *PriorityQueue[T]) IsEmpty() bool {
	result := true
	for _, queue := range pq.q {
		if queue.Size() > 0 {
			result = false
			break
		}
	}
	return result
}

func main() {
	airlineQueue := NewPriorityQueuePassenger
	passengers := []Passenger{
		{"Erika", 3}, {"Robert", 3}, {"Danielle", 3},
		{"Madison", 1}, {"Frederik", 1}, {"James", 2},
		{"Dante", 2}, {"Shelley", 3},
	}
	fmt.Println("乘客列表: ", passengers)
	for i := 0; i < len(passengers); i++ {
		airlineQueue.Insert(passengers[i], passengers[i].priority)
	}
	fmt.Println("队列头部乘客: ", airlineQueue.First())
	airlineQueue.Remove()
	airlineQueue.Remove()
	airlineQueue.Remove()
	fmt.Println("队列头部乘客: ", airlineQueue.First())
}

/* 输出
乘客列表: [{Erika 3} {Robert 3} {Danielle 3} {Madison 1} {Frederik 1} {James 2} {Dante 2} {Shelley 3}]
队列头部乘客: {Madison 1}
经过三次移除后的队列头部乘客: {Dante 2}*/
```

*列表 6-9*  
*优先队列的切片实现及驱动程序*

如果切片中所有队列均为空，`Remove` 方法将返回 `T`（此处为 `Passenger`）的零值。

前三次 `Remove` 调用移除了队列中全部两个优先级为 1 的乘客以及第一个优先级为 2 的乘客，使得“Dante”成为队列头部。

在列表 6-9 中，我们看到了抽象层次的分层，这是软件开发中的常见做法。我们也可以使用切片队列代替节点队列，只需改动导入语句中的一行代码，并将所有 `nodequeue.Queue` 替换为 `slicequeue.Queue` 即可。

下一节将介绍队列的一个重要应用——等待线的离散事件模拟。典型的等待线场景是客户排队竞争服务，等待服务器处理每位客户。例如超市的结账流程。

## 6.8 队列应用：等待线的离散事件模拟

假设我们在银行有一个服务等待线。客户按照**泊松**到达过程到达，并具有指定的平均到达率。客户获得服务的时间由在指定下限和上限之间均匀分布的随机服务时间决定。我们的目标是构建一个模拟程序，用于估算加入等待线的客户的平均等待时间（从到达等待线到服务完成的时间），以及八小时工作日内其他统计数据。

### 泊松过程

由泊松到达过程建模的事件需满足以下条件：

1. 事件相互独立。一个事件的发生不会影响另一个事件的发生时间。如果建模的事件是客户到达银行等待线，这很可能是一个合理的要求。

2. 事件的平均发生率保持恒定。这里我们以分钟为基本时间单位，因此平均发生率以每分钟事件数表示。

可以证明，事件之间的时间（一个随机变量）可通过以下函数生成：

```
func InterArrivalInterval(arrivalRate float64) float64 {
	// 模拟泊松过程并返回
	rn := rand.Float64() // 0.0 到 1 之间的随机浮点数
	return -math.Log(1.0 - rn) / arrivalRate
}
```

这对应于事件间等待时间大于某个 `t` 的概率为：

```
P(等待时间 > t) = e^-λ * t
```

其中 **λ** 是以事件/分钟为单位的平均到达率。这是一个指数分布。随着 `t` 增加，等待时间超过 `t` 的概率趋近于 0。当 `t` 等于 0 时，概率为 1。随着到达率 **λ** 增加（平均每分钟事件数更多），下一个事件的等待时间大于某个 `t` 的概率会降低。

我们将服务时长（处理一位客户所需的时间）建模为介于 `0.5 / 到达率` 和 `1.4 / 到达率` 之间的均匀分布。例如，如果到达率为 `0.25`（平均每 4 分钟一位客户），则服务时间被建模为在 2 分钟到 5.6 分钟之间均匀分布，平均为 3.8 分钟。由于平均服务时间小于平均到达间隔时间，这将形成一个稳定的队列。



### 模拟逻辑

让我们考察一个典型的事件序列，为构建模拟逻辑打下基础。`a`代表顾客到达时间。`d`代表顾客离开时间。队伍从左向右排列，因此最左边的顾客位于队首，将是下一个离开的顾客。

```
t1
0    a1                                       line: c1
t2
0    a1    a2                                 line: c1, c2
t3
0    a1    a2             d1                  line: c2
|  |
t4
0    a1    a2            d1   a3              line: c2, c3
t5
0    a1    a2            d1     a3 d2         line: c3
t6
0    a1    a2            d1     a3 d2    d3   line: empty
```

变量 `t`（时间）根据下一个事件——要么是到达，要么是离开——以离散的步长前进，因此得名离散事件模拟。

对于所示的事件序列，顾客 1、2、3 的等待时间如下：

- 顾客 1: (`d1 – a1`)
- 顾客 2: (`d2 – a2`)
- 顾客 3: (`d3 – a3`)

队列时间如下：
(`t2 – t1`) * 1 + (`t3 – t2`) * 2 + (`t4 – t3`) * 1 + (`t5 – t4`) * 2 + (`t6 – t5`) * 1

平均队列大小：`queueTime / t6`.

在每个事件（到达或离开）之后，队列时间通过计算新事件时间与上一事件时间的差值，并乘以当前队列大小来更新。如果下一个事件是离开，则移除队首顾客；将其 `departureTime – arrivalTime` 累加到等待时间中。如果下一个事件是到达，则将顾客插入队列。

下一个到达时间是前一个到达时间加上到达间隔。下一个离开时间计算为顾客成为队首的时刻加上该顾客的服务时间。

### 系统实现

基于上述观察，我们给出清单 6-10，它实现了这个系统。

```go
// 等待队列的离散事件模拟
package main

import (
    "math/rand"
    "math"
    "fmt"
    "time"
    "example.com/nodequeue"
)

const (
    arrivalRate           = 0.25 // 每分钟平均到达顾客数
    lowerBoundServiceTime = 0.5 / arrivalRate
    upperBoundServicetime = 2.0 / arrivalRate
    quitTime              = 480 // 8 小时工作日的分钟数
)

func InterArrivalInterval(arrivalRate float64) float64 {
    // 模拟泊松过程并返回
    rn := rand.Float64() // 0.0 到 1.0 之间的随机浮点数
    return -math.Log(1.0 - rn) / arrivalRate
}

func ServiceTime() float64 {
    // 均匀分布
    rn := rand.Float64() // rn 在 0.0 到 1.0 之间
    return lowerBoundServiceTime + (upperBoundServicetime - lowerBoundServiceTime) * rn
}

type Customer struct {
    arrivalTime     float64
    serviceDuration float64
}

// 统计信息抽象数据类型
type Statistics struct {
    waitTimes      []float64
    queueTime      float64 // 累计时间 * 队列大小
    longestQueue   int
    longestWaitTime float64
}

func (s *Statistics) AddWaitTime(wait float64) {
    s.waitTimes = append(s.waitTimes, wait)
    if wait > s.longestWaitTime {
        s.longestWaitTime = wait
    }
}

func (s *Statistics) AddQueueSizeTime(queueSize int, timeAtSize float64) {
    s.queueTime += float64(queueSize) * timeAtSize
}

func (s *Statistics) AddLength(length int) {
    if length > s.longestQueue {
        s.longestQueue = length
    }
}

var lastArrivalTime, departureTime, lastEventTime float64

func main() {
    rand.Seed(time.Now().UnixNano())
    lastEventTime := 0.0 // 一天开始
    line := nodequeue.Queue[Customer]{}
    statistics := Statistics{}

    // 开始模拟
    for {
        lastArrivalTime = lastArrivalTime + InterArrivalInterval(arrivalRate)
        if lastArrivalTime > quitTime {
            break
        }
        if line.Size() == 0 {
            lastEventTime = lastArrivalTime
            // fmt.Printf("\nline no longer empty at time: %0.2f. line size is 1", lastEventTime)
            serviceTime := ServiceTime()
            customer := Customer{lastArrivalTime, serviceTime}
            line.Insert(customer)
            statistics.AddLength(line.Size())
            departureTime = lastArrivalTime + serviceTime
        } else {
            if lastArrivalTime < departureTime {
                // 下一个事件是到达
                serviceTime := ServiceTime()
                customer := Customer{lastArrivalTime, serviceTime}
                line.Insert(customer)
                statistics.AddLength(line.Size())
                statistics.AddQueueSizeTime(line.Size()-1, lastArrivalTime-lastEventTime)
                lastEventTime = lastArrivalTime
            } else {
                // 下一个事件是离开
                // 从队列中移除第一个顾客
                removedCustomer := line.Remove()
                waitTime := departureTime - removedCustomer.arrivalTime
                statistics.AddWaitTime(waitTime)
                statistics.AddQueueSizeTime(line.Size()+1, departureTime-lastEventTime)
                lastEventTime = departureTime
                if line.Size() > 0 {
                    departureTime = lastEventTime + line.First().serviceDuration
                }
            }
        }
    }

    totalWaitTime := 0.0
    for i := 0; i < len(statistics.waitTimes); i++ {
        totalWaitTime += statistics.waitTimes[i]
    }
    averageWaitTime := totalWaitTime / float64(len(statistics.waitTimes))

    fmt.Printf("\n 从到达至离开的平均时间: %0.2f 分钟", averageWaitTime)
    fmt.Printf("\n 等待队列的平均大小: %0.2f", statistics.queueTime/lastEventTime)
    fmt.Printf("\n 一天中最长的队列: %d", statistics.longestQueue)
    fmt.Printf("\n 一天中最长的等待时间: %0.2f 分钟", statistics.longestWaitTime)
}

/* 输出示例
从到达至离开的平均时间: 16.19 分钟
等待队列的平均大小: 2.28
一天中最长的队列: 8
一天中最长的等待时间: 40.18 分钟
*/
```

清单 6-10 – 等待队列的离散事件模拟

对模拟进行多次运行后，之前显示的输出统计信息会表现出较大的方差。

如果我们在代码中启用那三行被左对齐缩进的注释代码，我们将输出精确的事件序列及其发生时间。

在一次典型的模拟运行中，产生的部分输出如下所示：

```
line no longer empty at time: 0.81\. line size is 1
Departure event at 6.23 - line size is: 0:
line no longer empty at time: 7.82\. line size is 1
Departure event at 12.91 - line size is: 0:
line no longer empty at time: 28.79\. line size is 1
Arrival event at 29.87 - line size is: 2:
Departure event at 33.56 - line size is: 1:
Departure event at 41.07 - line size is: 0:
line no longer empty at time: 53.59\. line size is 1
Arrival event at 55.22 - line size is: 2:
Departure event at 58.17 - line size is: 1:
Departure event at 61.30 - line size is: 0:
line no longer empty at time: 62.79\. line size is 1
Arrival event at 67.35 - line size is: 2:
Departure event at 70.29 - line size is: 1:
Arrival event at 71.87 - line size is: 2:
Departure event at 77.07 - line size is: 1:
Departure event at 81.35 - line size is: 0:
line no longer empty at time: 85.97\. line size is 1
Arrival event at 89.44 - line size is: 2:
Departure event at 90.97 - line size is: 1:
Departure event at 93.27 - line size is: 0:
line no longer empty at time: 95.92\. line size is 1
Departure event at 99.66 - line size is: 0:
line no longer empty at time: 105.60\. line size is 1
Arrival event at 105.96 - line size is: 2:
Arrival event at 108.51 - line size is: 3:
Departure event at 109.23 - line size is: 2:
Arrival event at 114.78 - line size is: 3:
Arrival event at 115.19 - line size is: 4:
Arrival event at 115.93 - line size is: 5:
Departure event at 117.01 - line size is: 4:
Arrival event at 119.26 - line size is: 5:
Arrival event at 120.21 - line size is: 6:
Departure event at 124.32 - line size is: 5:
Departure event at 129.74 - line size is: 4:
Departure event at 133.90 - line size is: 3:
Departure event at 137.58 - line size is: 2:
Departure event at 140.13 - line size is: 1:
Departure event at 147.02 - line size is: 0:
line no longer empty at time: 170.46\. line size is 1
Arrival event at 170.87 - line size is: 2:
Arrival event at 173.63 - line size is: 3:
Departure event at 177.92 - line size is: 2:
Arrival event at 181.92 - line size is: 3:
Departure event at 184.98 - line size is: 2:
Departure event at 192.90 - line size is: 1:
Departure event at 195.10 - line size is: 0:
line no longer empty at time: 211.92\. line size is 1
Departure event at 215.48 - line size is: 0:
line no longer empty at time: 217.21\. line size is 1
```

这里展示的模拟有许多有趣且有用的变体。例如，我们可以研究在存在多个服务台（如银行出纳员）的情况下，是让一个等待队列为所有出纳员提供服务（当有出纳员空闲时，队列长度减少一）更高效，还是使用独立的等待队列更高效。我们将此类研究留给读者。

在下一节中，我们将介绍 `Queue` 的另一个应用：洗牌。



### 队列应用：洗牌

在 3.4 节的二十一点游戏中，我们引入了`Deck`（牌组）抽象，实现如下：

```
var ranks = []string {"2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"}
var suits = []rune {'\u2660', '\u2661', '\u2662', '\u2663'}
type Card struct {
Rank string
Suit string
}
type Deck struct {
Cards []Card
}
```

我们了解到，`math/rand`包包含一个`Shuffle()`方法，该方法可应用于切片，在此例中，即应用于`Card`类型的切片。

在本节中，我们将使用两个队列构建我们自己的`Shuffle`方法。

### 洗牌模型

一副扑克牌的洗牌过程可以建模如下：将 52 张牌的牌堆分成两叠，两叠牌的数量随机相差不超过五张。然后从两叠牌中交替取牌，直到两叠牌重新合并成一整副牌。当较短的那叠牌取完后，直接将较长那叠剩余的所有牌加入牌堆。

如果将每叠牌建模为一个队列，那么洗牌过程就变得简单明了。

代码清单 6-11 展示了前述的`Shuffle`方法。我们打印了原始牌堆和洗牌后的牌堆。

```
// 洗牌程序
package main
import (
"example.com/nodequeue"
"math/rand"
"time"
"fmt"
)
type Card struct {
Rank string
Suit string
}
type Deck struct {
Cards []Card
}
var ranks = []string {"2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"}
var suits = []rune {'\u2660', '\u2661', '\u2662', '\u2663'}
func NewDeck() (deck Deck) {
for _, suit := range(suits) {
for _, rank := range(ranks) {
deck.Cards = append(deck.Cards, Card{rank, string(suit)})
}
}
return deck
}
func (deck Deck) Shuffle() Deck {
q1 := nodequeue.Queue[Card]{}
q2 := nodequeue.Queue[Card]{}
// 切牌
mismatch := -5 + rand.Intn(11) // -5 到 5
var i int
for i = 0; i < 26 + mismatch; i++ {
q1.Insert(deck.Cards[i])
}
for ; i < 52; i++ {
q2.Insert(deck.Cards[i])
}
// 重建牌堆
deck = Deck{}
for {
if q1.Size() == 0 || q2.Size() == 0 {
break
}
card := q1.Remove()
deck.Cards = append(deck.Cards, card)
card = q2.Remove()
deck.Cards = append(deck.Cards, card)
}
if q2.Size() == 0 {
for {
if q1.Size() == 0 {
break
}
card := q1.Remove()
deck.Cards = append(deck.Cards, card)
}
}
if q1.Size() == 0 {
for {
if q2.Size() == 0 {
break
}
card := q2.Remove()
deck.Cards = append(deck.Cards, card)
}
}
return deck
}
func main() {
rand.Seed(time.Now().UnixNano())
deck := NewDeck()
fmt.Println("\n 原始牌堆: ", deck)
// 切牌 5 次
for index := 0; index < 5; index++ {
deck = deck.Shuffle()
}
fmt.Println("\n 洗牌后: ", deck)
}
代码清单 6-11
洗牌程序
```

经过五次洗牌后的典型输出如下所示：

原始牌堆: {[{2 ♠} {3 ♠} {4 ♠} {5 ♠} {6 ♠} {7 ♠} {8 ♠} {9 ♠} {10 ♠} {J ♠} {Q ♠} {K ♠} {A ♠} {2 ♡} {3 ♡} {4 ♡} {5 ♡} {6 ♡} {7 ♡} {8 ♡} {9 ♡} {10 ♡} {J ♡} {Q ♡} {K ♡} {A ♡} {2 ♢} {3 ♢} {4 ♢} {5 ♢} {6 ♢} {7 ♢} {8 ♢} {9 ♢} {10 ♢} {J ♢} {Q ♢} {K ♢} {A ♢} {2 ♣} {3 ♣} {4 ♣} {5 ♣} {6 ♣} {7 ♣} {8 ♣} {9 ♣} {10 ♣} {J ♣} {Q ♣} {K ♣} {A ♣}]}

洗牌后: {[{2 ♠} {10 ♡} {Q ♡} {7 ♡} {9 ♣} {4 ♣} {6 ♣} {A ♠} {2 ♡} {J ♢} {K ♢} {8 ♢} {9 ♠} {4 ♠} {6 ♠} {A ♡} {3 ♢} {J ♣} {K ♣} {8 ♣} {9 ♡} {4 ♡} {6 ♡} {2 ♣} {3 ♣} {J ♠} {K ♠} {8 ♠} {10 ♢} {5 ♢} {7 ♢} {2 ♢} {3 ♠} {J ♡} {K ♡} {8 ♡} {10 ♣} {5 ♣} {7 ♣} {9 ♢} {3 ♡} {Q ♢} {A ♢} {5 ♠} {10 ♠} {Q ♣} {7 ♠} {5 ♡} {4 ♢} {Q ♠} {A ♣} {6 ♢}]}

在`Shuffle`方法中，通用队列（此处来自`nodequeue`包，其通用参数为`Card`）的可用性极大地简化了我们的工作。

在下一节中，我们将介绍更通用的数据结构：链表。

