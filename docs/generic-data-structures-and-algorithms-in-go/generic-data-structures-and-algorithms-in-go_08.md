# 3. 抽象数据类型：Go 中没有类的面向对象编程

在上一章中，我们讨论了算法复杂度，并给出了使用和不使用并发的示例。

在本章中，我们将展示如何在没有 `class` 构造的情况下进行面向对象编程。我们回顾了抽象数据类型的基本概念，并通过多个示例说明其用法。

## 3.1 使用类的抽象数据类型

1980 年，施乐帕克研究中心开发的 `Smalltalk` 语言正式发布。这门开创性的语言为一种新的编程范式——面向对象编程——奠定了基础。`Smalltalk` 及其后续更新的面向对象语言的核心是 `class` 构造。继 `Smalltalk` 之后的一些主要面向对象语言包括 Eiffel、C++（一种包含 `class` 构造的混合语言）、Java、Swift、Python 和 C#。这些语言都将 `class` 作为定义抽象数据类型和描述作为类实例的对象行为的核心构造。

`class` 构造实现了一些关于如何构建软件的、由来已久的成熟思想。具体来说，类实现了**抽象数据类型**。

一个**抽象数据类型**的特征在于可以对该底层类型执行的一组操作。考虑下面这个简单的例子。

我们定义 `Counter` 抽象数据类型如下：

| 属性 |
| `count int` – 每个 `Counter` 对象（实例）的内部数据 |
| 方法 |
| `Increment()` – 将属性 `count` 的当前值加一 |
| `Decrement()` – 仅当 `count > 0` 时，将属性 `count` 的当前值减一 |
| `Reset()` – 将 `count` 的值设置为零 |
| `GetCount()` – 返回 `count` 的当前值 |

如果 `myCounter` 被定义为 `Counter` 类型（某个 `Counter` 类的实例），则以下操作是合法的：

```
myCounter.Increment()
myCounter.Decrement()
myCounter.Reset()
countValue = myCounter.GetCount()
```

在前面的例子中，`myCounter` 被称为对象（`Counter` 类的实例），而通过点运算符连接到每个对象的方法调用就是可以对每个对象执行的合法操作，因此得名**面向对象编程**。

支持 `class` 构造的面向对象语言提供了一种机制，可以通过继承来扩展父类中定义的操作集。C++ 和 Eiffel 等语言允许子类从两个或多个父类继承操作，而 Java 和 C# 等语言只允许从一个父类继承。

关于面向对象语言中的继承已有大量文献。近年来，由于继承可能引入的复杂性以及在类层次结构中可能产生的依赖关系，它已逐渐失宠。

有两种近期出现的语言——Go 和 Rust——已经抛弃了继承，实际上也抛弃了类。但 Go 和 Rust 并未抛弃面向对象编程 (OOP)，而是改变了这种范式的使用方式。

在深入探讨 Go 中面向对象编程的细节之前，让我们先看看如何用 Python 实现 `Counter` 抽象数据类型。

```
class Counter:
def __init__(self):
self.count  = 0
def increment(self):
self.count += 1
def decrement(self):
self.count -= 1
def reset():
self,count = 0
def get_count(self) -> int:
return self.count
if __name__ == "__main__":
my_counter = Counter()
for index in range(10):
my_counter.increment()
my_counter.decrement()
current_count = my_counter.get_count()
print(current_count)
''' 输出

'''
```

关键字 `self` 被用作对 `Counter` 类任何实例的引用。定义在 `__init__` 方法中的属性 `count`，存储在每个 `Counter` 类的实例（对象）中。

`class` 构造的一个明显吸引力在于，所有对底层属性的操作都被封装在这个单一的构造中。

那么 Go 语言呢？在下一节中，我们将探讨如何在 Go 语言中不使用类来定义抽象数据类型。

## 3.2 Go 语言中的抽象数据类型

Go 语言不包含 `class` 构造。这与现代的面向对象语言有显著的不同。由于 Go 语言中没有 `class` 构造，因此也没有继承。



### ADT `Counter`

ADT `Counter` 必须限制可对 `Counter` 实例执行的操作。具体来说，我们不能为计数器随意赋值，也不能一次性将计数器的值改变超过 1。如果没有这些限制，定义这个抽象数据类型就失去了意义，我们完全可以改用简单的 `int` 类型。

清单 3-1 展示了 `Counter` 抽象数据类型 (ADT) 的首次实现。稍后我们会看到，这个实现存在缺陷。在将 `Counter` 定义为一个包含 `count` 字段的结构体后，我们定义了一系列方法，这些方法作用于 `Counter` 的实例 `c` 或指向 `Counter` 的指针上。

```
package main
import (
"fmt"
)
type Counter struct {
count int
}
// 方法
func (c *Counter) Increment() {
c.count++
}
func (c *Counter) Decrement() {
c.count--
}
func (c *Counter) Reset() {
c.count = 0
}
func (c Counter) GetCount() int {
return c.count
}
func main() {
myCounter := new(Counter)
// myCounter.count = 100 // 破坏了 Counter 的封装性
fmt.Println(myCounter.GetCount())
for i := 1; i <= 10; i++ {
myCounter.Increment()
}
myCounter.Decrement()
// myCounter.count -= 6 // 破坏了 Counter 的封装性
fmt.Println(myCounter.GetCount())
}
/*

*/
清单 3-1
Counter ADT 的首次实现
```

这个首次实现存在一个问题。如果取消注释下面两行代码：

```
myCounter.count = 100
myCounter.count -= 6
```

那么维护 ADT 完整性的封装就会被破坏。创建和实现 ADT 的全部意义在于强制实施抽象。在这个案例中，这意味着不允许一次性将 `count` 值改变超过 1，也不允许随意赋值。

我们修复了这个问题并强制实施了抽象，如清单 3-2 所示。我们定义了一个 `Counter` 接口（使用大写字母 C）来正式定义该 ADT。

```
// 创建 ADT Counter
package main
import (
"fmt"
)
// 该类型隐式地实现了 Counter ADT
type counter struct {
count int
}
// 接口用于暴露 counter 的公有特性
// 属性 count 是私有的
type Counter interface {
increment()
decrement()
reset()
getCount() int
}
func (c *counter) increment()  {
c.count += 1
}
func (c *counter) decrement()  {
if c.count > 0 {
c.count -= 1
}
}
func (c *counter) reset() {
c.count = 0
}
func (c counter) getCount() int {
return c.count
}
func main() {
myCounter := Counter(&counter{})
// 可以对我这个计数器执行的操作
// 仅限定于 Counter 接口中指定的那些
myCounter.increment()
myCounter.increment()
myCounter.reset()
myCounter.increment()
myCounter.increment()
myCounter.increment()
myCounter.increment()
myCounter.decrement()
countValue := myCounter.getCount()
fmt.Println(countValue)
}
// 3
清单 3-2
Counter ADT 的第二次实现
```

`Counter` 接口指定了可对 ADT `Counter` 执行的四个操作的签名。

方法 `increment()`、`decrement()`、`reset()` 和 `getCount()` 都定义在 `counter` 类型上，它们**隐式地**使得 `counter` 实现了 ADT `Counter`。

如果我们注释掉 `reset()` 方法，并注释掉 `main` 函数中的 `myCounter.reset()`，将会得到以下编译器错误信息：

`./counter2.go:41:26: cannot convert &counter{} (value of type *counter) to type Counter:`  
`counter does not implement Counter (missing reset method)`

如果 `counter` 类型上没有定义 `reset()` 方法，那么 `counter` 类型就不再被视为 `Counter` 类型，编译器会检测到这个错误。

在 Go 语言中，抽象数据类型总是通过定义一个接口来**隐式定义**，该接口指定了与 ADT 相关联的操作，然后在底层类型上实现与接口规范中签名完全一致的方法。

借助清单 3-2 中完成的工作，就无法再像清单 3-1 中那样违反 ADT 的封装性了。

### 创建 counter 包

保护 `count` 并维护 `counter` 抽象封装性的另一种方法是创建一个 `counter` 包，并导出 `Counter`，但不导出 `count`。

```
package counter
// 字段 count 因为是小写字母开头而被封装为私有
type Counter struct {
count int // 私有字段
}
func (c *Counter) Increment() {
c.count += 1
}
func (c *Counter) Decrement() {
if c.count > 0 {
c.count -= 1
}
}
func (c *Counter) Reset() {
c.count = 0
}
func (c Counter) GetCount() int {
return c.count
}
```

在 `counter` 包中，我们通过使用小写字符作为 `count` 的首字符，来保护 `Counter` 的 `count` 字段不被包外赋值。

### 创建包的机制

为了在你选择的子目录中定义一个包，我们必须遵循下面概述的一系列步骤。这里，我们希望将定义 `counter` 包的 `counter.go` 文件放在某个工作目录下的子目录 `counter` 中。一个主驱动程序 `main.go` 定义在另一个子目录 `maincounter` 中。

创建 `counter` 包所需的步骤如下：

1.  在你的工作目录中创建一个子目录 `counter`。
2.  将包含 `package counter` 的 `counter.go` 文件（参见上文）保存到此子目录中。
3.  在你的工作目录中创建一个子目录 `maincounter`。
4.  将 `maincounter.go` 文件保存到此子目录中。
5.  打开一个终端窗口，切换到 `counter` 目录。
6.  输入以下命令：`go mod init example.com/counter`
7.  输入以下命令：`go mod tidy`
8.  打开一个终端窗口，切换到 `maincounter` 目录。
9.  输入以下命令：`go mod init example.com/maincounter`
10. 编辑 `go.mod` 文件，使其内容如下：

    ```
    module example.com/maincounter
    go 1.18
    replace example.com/counter => ../counter
    ```

11. 输入以下命令：`go mod tidy`

清单 3-3 展示了 `Counter` ADT 的第三次实现。

```
// 在子目录 counter 中
package counter
type Counter struct {
count int
}
func (c *Counter) Increment()  {
c.count += 1
}
func (c *Counter) Decrement()  {
if c.count > 0 {
c.count -= 1
}
}
func (c *Counter) Reset() {
c.count = 0
}
func (c Counter) GetCount() int {
return c.count
}
// 在子目录 maincounter 中
package main
import (
"fmt"
"example.com/counter"
)
func main() {
myCounter := counter.Counter{}
myCounter.Increment()
myCounter.Increment()
myCounter.Reset()
myCounter.Increment()
myCounter.Increment()
myCounter.Increment()
myCounter.Increment()
myCounter.Decrement()
countValue := myCounter.GetCount()
fmt.Println(countValue)
}
// 3
清单 3-3
使用 counter 包的 Counter ADT 的第三次实现
```

导出的标识符（即可以在包外访问的标识符）必须以大写字母开头。这包括类型名称（如 `Counter`）和方法名称。`Counter` 结构体中的字段 `count` 特意使用了小写字母，这样它的值就不能在包外被访问。改变 `count` 的唯一方式是通过操作 `Counter` 的方法。

在清单 3-2 和 3-3 给出的两种方法中，实现 ADT `Counter` 应该使用哪一种？

如果一个 ADT 要被两个或更多应用程序使用，那么首选清单 3-3 中为 ADT 定义包的方案。如果一个 ADT 是一次性的，只在一个特定的应用程序中使用，那么使用接口类型的方案更简单。



### 实现抽象数据类型的另一个示例

我们来看一个更复杂的例子，以说明 Go 如何在**不使用类**的情况下实现抽象数据类型（ADT）。

考虑 ADT `Employee`（雇员）。

| 属性 |
| --- |
| `LastName`（姓氏）字符串 |
| `FirstName`（名字）字符串 |
| `Role`（职位）字符串 |
| `Salary`（薪资）浮点数 |
| 方法 |
| `Get LastName`（获取姓氏，只读） |
| `Get FirstName`（获取名字，只读） |
| `Set/Get Role`（设置/获取职位） |
| `Set/Get Salary`（设置/获取薪资） |
| `String()` 字符串 – 将实例表示为字符串 |

我们指定了`LastName`和`FirstName`为只读属性。一旦它们的值被赋值，就不能被更改。

同时考虑 ADT `PartTimeEmployee`（兼职雇员），它是一个`Employee`，带有一个额外特性。

| 属性 |
| --- |
| `Employee` |
| `HourlyWage`（时薪）浮点数 |
| 方法 |
| `Set/Get HourlyWage`（设置/获取时薪） |
| `String()` 字符串 – 将实例表示为字符串 |

我们使用下面展示的结构体（struct）和接口（interface）定义来建立该 ADT：

```go
type employee struct {
    lastName string
    firstName string
    role string
    salary float64
}

type Employee interface {
    SetLastName(lName string)
    SetFirstName(fName string)
    SetRole(r string)
    GetRole() string
    SetSalary(s float64)
    GetSalary() float64
    String() string
}

type partTimeEmployee struct {
    employee
    hourlyWage float64
}

type PartTimeEmployee interface {
    Employee
    SetHourlyWage(hourly float64)
    GetHourlyWage() float64
}
```

### 使用组合（Composition）

每个结构体类型`employee`和`partTimeEmployee`都伴随有接口类型。这些接口定义了在其各自结构体类型上所需的操作，从而隐式地使结构体类型实现给定的接口。

我们在定义`partTimeEmployee`的第一个字段为`employee`时使用了嵌入（embedding）。

在软件设计中，这被称为**组合（composition）**。兼职雇员的抽象是由一个雇员和一个时薪组合而成的。

`PartTimeEmployee`接口也使用了嵌入，它首先包含了`Employee`接口。这要求`Employee`的所有方法以及两个新方法（设置/获取时薪）都必须被实现。

清单 3-4 充实了先前定义的 ADT，并展示了一个简短的 main 驱动程序。

```go
package main

import (
    "fmt"
)

type employee struct {
    lastName string
    firstName string
    role string
    salary float64
}

type Employee interface {
    SetLastName(lName string)
    SetFirstName(fName string)
    SetRole(r string)
    GetRole() string
    SetSalary(s float64)
    GetSalary() float64
    String() string
}

type partTimeEmployee struct {
    employee
    hourlyWage float64
}

type PartTimeEmployee interface {
    Employee
    SetHourlyWage(hourly float64)
    GetHourlyWage() float64
}

// 方法
func (person *employee) SetSalary(yearly float64) {
    person.salary = yearly
}

func (person employee) GetSalary() float64 {
    return person.salary
}

func (person *employee) SetFirstName(firstN string) {
    person.firstName = firstN
}

func (person employee) GetFirstName() string {
    return person.firstName
}

func (person *employee) SetLastName(lastN string) {
    person.lastName = lastN
}

func (person *employee) SetRole(r string) {
    person.role = r
}

func (person employee) GetRole() string {
    return person.role
}

func (person employee) String() string {
    result := "姓名：" + person.firstName + " " + person.lastName + "\n"
    result += "职位：" + person.role + "\n"
    result += "年薪：$" + fmt.Sprintf("%0.2f", person.salary) + "\n"
    return result
}

func (person partTimeEmployee) String() string {
    result := "姓名：" + person.firstName + " " + person.lastName + "\n"
    result += "职位：" + person.role + "\n"
    result += "时薪：$" + fmt.Sprintf("%0.2f", person.hourlyWage) + "\n"
    return result
}

func (person *partTimeEmployee) SetHourlyWage(amt float64) {
    person.hourlyWage = amt
}

func (person partTimeEmployee) GetHourlyWage() float64 {
    return person.hourlyWage
}

func main() {
    person := new(employee) // 返回一个 employee 的地址
    person.SetFirstName("Helen")
    person.SetLastName("Rose")
    person.SetRole("技术负责人")
    person.SetSalary(125_644.0)
    fmt.Println(person.String())

    hourlyWorker := new(partTimeEmployee) // 返回地址
    hourlyWorker.SetFirstName("Mark")
    hourlyWorker.SetLastName("Smith")
    hourlyWorker.SetRole("软件开发者")
    hourlyWorker.SetHourlyWage(85.00)
    fmt.Println(hourlyWorker.String())
}

/*
姓名：Helen Rose
职位：技术负责人
年薪：$125644.00

姓名：Mark Smith
职位：软件开发者
时薪：$85.00
*/
```

清单 3-4 — `Employee`和`PartTimeEmployee` ADT 的实践

变量`person`和`hourlyWorker`的行为类似于传统面向对象编程（OOP）语言中的对象（类的实例）。方法在这些变量上被调用，就像在传统 OOP 语言中做的那样。

在下一节中，我们将讨论 Go 中的多态（polymorphism）。这是面向对象编程的另一个基本支柱。

## 3.3 多态

多态是面向对象编程的一个基本支柱。它允许在**运行时**对对象执行操作，而该操作基于**接收操作的对象类型**。

在如 C#、Java 和 Swift 等传统的强类型面向对象语言中，如果操作是在一个形式类型上声明的，而实际类型是某个派生类的实例，运行时系统会选择属于接收该消息的实际类型的方法。

这在 Go 中不可能发生，因为派生类（继承）在 Go 中并不存在。

### 使用接口实现多态

我们可以使用接口实现多态行为，如下一个示例（清单 3-5）所示。

```go
package main

import (
    "fmt"
)

type FixedPriceJob struct {
    description string
    fixedPrice float64
}

type HourlyJob struct {
    description string
    hourlyRate float64
    numberHours int
}

type JobInterface interface {
    Cost() float64
    GetDescription() string
}

// 隐式地将 FixedPriceJob 定义为实现了 JobInterface
func (job FixedPriceJob) Cost() float64 {
    return job.fixedPrice
}

func (job FixedPriceJob) GetDescription() string {
    return job.description
}

// 隐式地将 HourlyJob 定义为实现了 JobInterface
func (hourlyJob HourlyJob) Cost() float64 {
    return hourlyJob.hourlyRate * float64(hourlyJob.numberHours)
}

func (hourlyJob HourlyJob) GetDescription() string {
    return hourlyJob.description
}

func TotalJobCost(jobs []JobInterface) float64 {
    result := 0.0
    for _, job := range jobs {
        result += job.Cost()
    }
    return result
}

func main() {
    job1 := FixedPriceJob{"外墙抹灰", 34760.0}
    job2 := HourlyJob{"景观美化", 40.0, 50}
    jobs := []JobInterface{job1, job2}
    totalCost := TotalJobCost(jobs)
    fmt.Printf("总工程成本：$%0.2f", totalCost)
}

// 总工程成本：$36760.00
```

清单 3-5 — 多态的实践

任何定义了与`JobInterface`中签名相匹配的方法的类型都隐式地实现了该接口。这就是我们在清单 3-5 中所做的。我们在`FixedPriceJob`和`HourlyJob`上都定义了`Cost()`和`GetDescription()`方法。

在函数`TotalJobCost`中，我们输入了一个`JobInterface`类型的切片。我们遍历输入的工程范围，并通过在每个工程上调用`Cost()`方法来累加总成本。运行时系统根据接收此方法的工程类型（在此例中是`FixedPriceJob`还是`HourlyJob`）来绑定正确的`Cost()`方法。这就是多态在实践中的应用。

在下一节中，我们将介绍一个面向对象编程（OOP）应用程序。将开发一个简单的二十一点纸牌游戏。



## 3.4 面向对象应用：简化版二十一点游戏

在传统的面向对象语言（如 Smalltalk、Java、C# 和 Swift）中，设计过程涉及将问题分解为类。 这在 Go 语言中是不可行的，因为 Go 语言中没有类。

我们演示了如何在 Go 中实现问题分解。 我们设计并实现了一个小型、简化的二十一点纸牌游戏。 该游戏基于控制台运行。

在二十一点中，从牌堆中向玩家和庄家各发两张牌。 目标是累积点数，但不得超过 21 点。 一张牌的点值是其牌面上的数字；如果牌是 J、Q 或 K，则点值为 10；如果牌是 A，则点值为 11。 如果手牌中有两张或更多 A，则从手牌总点数中减去 10。

玩家先行动，如果愿意，可以通过说“要牌”来获得额外的牌。 当玩家的分数接近 21 点时，玩家停止要牌。 如果玩家在“要牌”后分数超过 21，则游戏结束，庄家获胜。 如果没有，则轮到庄家。 在这里，我们简化了规则，假设如果庄家总分低于 17 点，庄家就会请求“要牌”。 庄家出牌结束后，分数最高且不超过 21 点的一方获胜。 可能出现平局。

在传统的面向对象语言中，我们会定义 `Card`、`Hand` 和 `Deck` 类，并为这些实体定义操作方法。

在 Go 中，我们使用结构体和方法来建模系统。

考虑以下代码：

```go
var ranks = []string {"2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"}
var suits = []rune {'\u2660', '\u2661', '\u2662', '\u2663'}
type Card struct {
    Rank string
    Suit string
}
type Hand struct {
    Cards []Card
}
type Deck struct {
    Cards []Card
}
```

变量 `ranks` 是一个切片，包含可用的牌，每个元素是一个字符串。 变量 `suits` 包含一个由四个 rune 值组成的切片，分别代表梅花、方块、红心和黑桃的符号。

类型 `Card` 是一个结构体，包含字段 `Rank` 和 `Suit`。

方法 `value` 在 `hand` 上操作如下：

```go
func (hand Hand) value() int {
    result := 0
    numberAces := 0
    for index := 0; index < len(hand.Cards); index++ {
        // ... (为简洁起见省略)
    }
    if result > 21 && numberAces > 1 {
        result -= 10 * numberAces
    }
    return result
}
```

其他辅助方法如下所示：

```go
func (hand *Hand) addCard(card Card) {
    hand.Cards = append(hand.Cards, card)
}

func (hand Hand) Display() {
    fmt.Println("\n")
    for _, card := range hand.Cards {
        fmt.Print(card.Rank + card.Suit + " ")
    }
}

func (deck *Deck) dealCard() Card {
    result := deck.Cards[0]
    deck.Cards = deck.Cards[1:]
    return result
}

func (deck *Deck) shuffle() {
    rand.Seed(time.Now().UnixNano())
    rand.Shuffle(len(deck.Cards), func(i, j int) {
        deck.Cards[i], deck.Cards[j] = deck.Cards[j], deck.Cards[i]
    })
}

func (deck *Deck) initializeDeck() Deck {
    for _, suit := range suits {
        for _, rank := range ranks {
            deck.Cards = append(deck.Cards, Card{rank, string(suit)})
        }
    }
    deck.shuffle()
    return *deck
}

func (deck Deck) display() {
    for _, card := range deck.Cards {
        fmt.Print(card.Rank + card.Suit + " ")
    }
}
```

方法 `shuffle` 使用了 `"math/rand"` 包中的 `Shuffle` 函数。

列表 3-6 给出了二十一点的完整 Go 应用程序。

```go
package main

import (
    "strconv"
    "fmt"
    "math/rand"
    "time"
    "bufio"
    "os"
)

var ranks = []string {"2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"}
var suits = []rune {'\u2660', '\u2661', '\u2662', '\u2663'}

type Card struct {
    Rank string
    Suit string
}

type Hand struct {
    Cards []Card
}

type Deck struct {
    Cards []Card
}

func (hand Hand) value() int {
    result := 0
    numberAces := 0
    for index := 0; index < len(hand.Cards); index++ {
        // ... (为简洁起见省略)
    }
    if result > 21 && numberAces > 1 {
        result -= 10 * numberAces
    }
    return result
}

func (hand *Hand) addCard(card Card) {
    hand.Cards = append(hand.Cards, card)
}

func (hand Hand) Display() {
    fmt.Println("\n")
    for _, card := range hand.Cards {
        fmt.Print(card.Rank + card.Suit + " ")
    }
}

func (deck *Deck) dealCard() Card {
    result := deck.Cards[0]
    deck.Cards = deck.Cards[1:]
    return result
}

func (deck *Deck) shuffle() {
    rand.Seed(time.Now().UnixNano())
    rand.Shuffle(len(deck.Cards), func(i, j int) {
        deck.Cards[i], deck.Cards[j] = deck.Cards[j], deck.Cards[i]
    })
}

func (deck *Deck) initializeDeck() Deck {
    for _, suit := range suits {
        for _, rank := range ranks {
            deck.Cards = append(deck.Cards, Card{rank, string(suit)})
        }
    }
    deck.shuffle()
    return *deck
}

func (deck Deck) display() {
    for _, card := range deck.Cards {
        fmt.Print(card.Rank + card.Suit + " ")
    }
}

func main() {
    gameOver := false
    myDeck := Deck{}
    myDeck.initializeDeck()
    houseHand := Hand{}
    playerHand := Hand{}

    for i := 1; i <= 2; i++ {
        playerHand.addCard(myDeck.dealCard())
        houseHand.addCard(myDeck.dealCard())
    }

    // ... (主要游戏逻辑已省略，为求简洁)
}
```

列表 3-6: 二十一点

多次运行中的一次输出如下：

```
4♡ 3♣ 您想继续要牌吗 (y/n)?
y
4♡ 3♣ 6♠ 您想继续要牌吗 (y/n)?
y
4♡ 3♣ 6♠ 5♡ 您想继续要牌吗 (y/n)?
n
庄家分数超过玩家分数。 游戏结束。 庄家获胜！
```

在本章的最后一节，我们将介绍另一个面向对象应用。 该应用使用了 Go 中定义的标准 `map` 数据结构。

## 3.5 另一个面向对象应用：单词排列组

一个单词排列组包含一组单词，这些单词由相同的字母组成，并且都存在于同一个字典中。

例如，`"persist"` 的排列组包含一组由相同字母组成且都存在于同一字典中的单词。

`"persist"` 的排列组是 `['esprits', 'persist', 'priests', 'spriest', 'sprites', 'stirpes', 'stripes']`。

人们的第一反应可能是枚举该组字母的所有排列，并查看哪些子集在字典中。



### 使用标准 `map` 数据结构

我们将采用一种不同的方法。在扫描整个单词文件时，我们构建一个包含**键-值**对的 `map`，具体如下：

**键：** 按字母顺序排列的单词（将给定单词的所有字母按从小到大的顺序重新排列）。例如，`alphabetized("camp")` = `"acmp"`，`alphabetized("balloon")` = `"abllnoo"`。

**值：** 一个字典单词集合，这些单词都可以归约到同一个按字母顺序排列的单词。

在处理 `words.txt` 文件中的每个单词时，我们通过按字母顺序排列该单词来计算键，然后检查该键是否已存在于我们的 map 中。如果存在，我们将正在处理的单词添加到与该键关联的值集合中。如果不存在，我们创建一个新的集合，并将 `<alphabetized(word), new collection>` 键-值对添加到我们的 map 中。

map 构建完成后，我们通过计算指定单词的键，然后从 map 中获取与该键关联的集合，来找到该单词的排列组合。

我们首先定义一个全局变量 `dictionary`。

```
var dictionary map[string][]string
```

接下来，我们定义一个函数 `alphabetize`。

```
func alphabetize(word string) string {
    s := strings.Split(word, "")
    sort.Strings(s)
    return strings.Join(s, "")
}
```

第一行创建一个字符数组。下一行对该数组进行原地排序。第三行将排序后的数组合并，形成最终的字符串。

函数 `buildDictionary` 创建一个 map，其中每个键代表一个按字母顺序排列的单词，每个值是能够按字母顺序排列为该键的单词切片。

该函数如下所示。

```
func buildDictionary() {
    dictionary = make(map[string][]string)
    file, err := os.Open("words.txt")
    if err != nil {
        log.Fatalf("failed opening file: %s", err)
    }
    scanner := bufio.NewScanner(file)
    scanner.Split(bufio.ScanLines)
    var txtwords []string
    for scanner.Scan() {
        txtwords = append(txtwords, scanner.Text())
    }
    file.Close()
    for _, word := range txtwords {
        alphabetized := alphabetize(word)
        var lst []string
        if len(dictionary) > 0 && len(dictionary[alphabetized]) > 0 {
            lst = dictionary[alphabetized]
        } else {
            lst = []string{}
        }
        lst = append(lst, word)
        dictionary[alphabetized] = lst
    }
}
```

`buildDictionary` 中处理文件的部分最为复杂。

```
file, err := os.Open("words.txt")
if err != nil {
    log.Fatalf("failed opening file: %s", err)
}
scanner := bufio.NewScanner(file)
scanner.Split(bufio.ScanLines)
var txtwords []string
for scanner.Scan() {
    txtwords = append(txtwords, scanner.Text())
}
file.Close()
```

使用导入的 `os` 包，打开一个包含单词的文本文件。通过在导入的 `bufio` 包上使用 `NewScanner` 来定义一个 `scanner`。使用 `scanner.Scan()`，生成包含在 `words.txt` 文件中的单词切片。

对该切片中的每个单词按字母顺序排列，然后将其添加到该键的现有 map 中，或者创建一个新键，并插入与该键关联的单词切片中的第一个值。

清单 3-7 展示了该应用程序的完整源代码。

```
package main

import (
    "fmt"
    "sort"
    "strings"
    "bufio"
    "log"
    "os"
)

func init() {
    buildDictionary()
}

var dictionary map[string][]string

func alphabetize(word string) string {
    s := strings.Split(word, "")
    sort.Strings(s)
    return strings.Join(s, "")
}

func buildDictionary() {
    dictionary = make(map[string][]string)
    file, err := os.Open("words.txt")
    if err != nil {
        log.Fatalf("failed opening file: %s", err)
    }
    scanner := bufio.NewScanner(file)
    scanner.Split(bufio.ScanLines)
    var txtwords []string
    for scanner.Scan() {
        txtwords = append(txtwords, scanner.Text())
    }
    file.Close()
    for _, word := range txtwords {
        alphabetized := alphabetize(word)
        var lst []string
        if len(dictionary) > 0 && len(dictionary[alphabetized]) > 0 {
            lst = dictionary[alphabetized]
        } else {
            lst = []string{}
        }
        lst = append(lst, word)
        dictionary[alphabetized] = lst
    }
}

func output(word string) {
    wd := alphabetize(word)
    fmt.Printf("Permutation group of %s is %s", word, dictionary[wd])
}

func main() {
    output("parties")
}

// Permutation group of parties is [parties pastier piaster piastre pirates // raspite spirate tapiser traipse]
清单 3-7
单词的排列组
```

## 3.6 总结

本章我们重点讨论了抽象数据类型的实现。展示了实现此目标的两种方法。第一种方法使用接口来定义 ADT 所需的操作。第二种方法使用包来公开 ADT 所需的公共特性，同时隐藏内部特性。我们介绍了多态性这一重要概念。这允许运行时系统确定将哪个特定方法绑定到接收该方法的对象上，前提是该对象是实现了接口的类型。我们给出了几个面向对象编程的示例。

在下一章中，我们将通过展示“生命游戏”的实现，给出一个更大的面向对象编程示例。我们将使用一个第三方图形用户界面（GUI）包。

