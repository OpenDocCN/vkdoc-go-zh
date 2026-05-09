# 20. TSP 的遗传算法

上一章介绍了模拟退火算法的实现，这是一种解决 TSP 问题的强大且有用的启发式算法。我们了解到，这种启发式算法通常能以相对较少的计算量获得问题的最优解。

本章将介绍另一种解决 TSP 问题的启发式方法——遗传算法。

在下一节中，我们将介绍这种启发式算法的基础。

## 20.1 遗传算法

遗传算法的灵感来源于生物学中的格言——**适者生存**。

随着物种的演化，与物种生存的生态系统相契合的特质会胜出。那些能提升在恶劣环境中生存能力的特质会成为主导，而导致弱势的特质则会随时间消失。这种进化过程假定，由繁殖引起的物种底层基因结构在不断变化。

我们将这种进化模型应用于 TSP 问题。

### 遗传算法的高级描述

首先生成一个初始种群，其中包含若干条路线，每条路线都访问每个城市一次并返回起始城市。每条路线的适应度是其成本的倒数。路线成本越低，适应度越高。

我们将**交配过程**定义为组合两条路线以产生两条子代路线，子代路线由父代路线的某种组合形成。

我们将**交配池**定义为将进行组合（交配）以产生下一代路线的父代路线的集合。

我们将**变异**定义为通过对现有路线进行微小的随机扰动而产生的新路线。

### 遗传算法的更详细描述

**初始世代：** 我们定义一个恒定的种群大小，例如`PopSize`。我们生成`PopSize`条随机路线，每条路线都从城市 0 出发并结束于城市 0。这些路线构成了我们的初始世代。

**对种群进行排名：** 我们根据每条路线的适应度对构成初始世代的路线进行排名。路线成本越低，适应度越高。我们按适应度对路线进行排序。

**交配池：** 我们使用一种锦标赛选择规则，其工作方式如下：从种群中随机选择指定的一组路线，并选择该组中适应度最高的路线作为第一个亲本。重复此过程以选择第二个亲本。我们继续这个过程，直到创建了`PopSize / 2`个交配对。

**交配：** 这是遗传算法中最具挑战性也是最重要的方面。我们需要组合两条路线以产生两条子代路线，每个子代保留其父代的一部分。我们将利用几种交叉算法来完成这种交配。

**变异：** 我们将使用简单交换路线中两个随机选择的城市来产生一条新路线。与发生变异的路线相比，这种新路线的适应度可能更差。这类似于模拟退火算法中的上坡移动。它促进了多样性，并有助于防止过早陷入解空间中的局部最小值。我们对给定世代中随机选择的很小一部分路线应用变异。

我们的遗传算法步骤如下：

1.  生成一个包含`ToursPerGeneration`条随机路线的初始种群。

2.  从现有世代中选择一小部分适应度最高的精英路线，将其移至下一代。

3.  对剩余的路线使用锦标赛选择来定义交配池。

4.  对交配池中的亲本进行交配，以形成新一代的路线。这个新世代包含来自上一代的精英群体以及通过交配形成的新子代。

5.  对新世代中随机选择的很小一部分路线执行变异。

6.  重复步骤 2 至 5，持续指定数量的世代。随着路线从一个世代进化到另一个世代，输出当前为止的最佳路线。

在下一节中，我们将遵循这些步骤来构建解决方案。

## 20.2 遗传算法的实现


好的，作为高级文档工程师和翻译员，我将严格遵循您提供的注意事项和示例，将给定的英文文本翻译成中文。


### 步骤 1 – 形成随机路径的初始种群

我们按如下方式形成路径的初始种群：

```go
var population [][]int
func CreateInitialPopulation() {
firstCities := make([]int, NUMCITIES - 1)
for i := 1; i < NUMCITIES; i++ {
firstCities[i - 1] = i
}
for row := 0; row < ToursPerGeneration; row++ {
rand.Shuffle(len(firstCities), func(i, j int) {
firstCities[i], firstCities[j] =
firstCities[j], firstCities[i]
})
population[row] = []int{0}
for col := 1; col < NUMCITIES; col++ {
population[row] = append(population[row],
firstCities[col - 1])
}
}
}
```

我们通过生成一个从 1 到 `NUMCITIES` 的随机整数序列，来使用 `rand` 包中的 `Shuffle` 函数。我们用值 0 初始化全局 `population` 变量的每一行，然后将随机序列追加到这个初始值之后。

完成后，种群矩阵的每一行都包含一个城市序列，每个序列都以城市 0 开头。当我们计算每条路径的成本时，我们会加上从序列中最后一个城市返回到城市 0 的成本。

给定路径（种群矩阵的行）的 `Cost` 函数如下：

```go
func Cost(graph [][]float64, tour []int) float64 {
result := 0.0
for index := 0; index < len(tour) - 2; index++ {
result += graph[tour[index]][tour[index+1]]
}
result += graph[tour[NUMCITIES - 1]][tour[0]]
return result
}
```

### 步骤 2 – 形成最优路径精英组

`ChooseEliteGroup()` 函数返回一个矩阵，其中包含当前种群中 `ELITENU` 条最优路径。

该函数如下所示：

```go
func ChooseEliteGroup() (elite [][]int) {
// 在此函数被调用之前，种群已经排序完毕
// 初始化精英组
elite = make([][]int, EliteNumber)
for row := 0; row < EliteNumber; row++ {
elite[row] = make([]int, EliteNumber)
}
for row := 0; row < EliteNumber; row++ {
elite[row] = DeepCopy(population[row])
}
return elite
}
```

需要使用 `DeepCopy` 是因为我们希望复制排序后种群中每行的值，而不是该行的地址。

### 步骤 3 – 锦标赛选择

为了从当前种群（排除精英路径）中获取配对，我们随机从（排除精英路径后的）种群中抓取 `TournamentNumber` 条路径，对它们进行排序，并返回其中最优的（成本最低的）路径。该路径即为 `parent1`。我们重复相同的过程以产生 `parent2`。我们将这两个父本进行交配，并将子代添加到 `newpopulation` 矩阵中。

我们选择 `EliteNumber`，使得 `ToursPerGeneration` – `EliteNumber` 是偶数。我们需要交配的父本对的数量为 (`ToursPerGeneration` – `EliteNumber) / 2。

### 步骤 4 – 父本交配

我们用于交配两条父本路径的 `OrderedCrossover` 函数，会将父本之间的相对顺序信息传递给子代。

我们在父本中创建两个随机的交叉点，并将 `parent1` 中位于它们之间的片段复制给 `child1`。

从 `parent2` 中的第二个交叉点开始，我们将剩余的号码从第二个父本复制到第一个子代，不允许重复，并且在遇到 `parent2` 的末尾时进行回绕。

对第二个子代执行相同操作，交换两个父本的角色。

考虑以下示例。

`parent1`: **0, 1, 2, | 3, 4, 5, 6, | 7, 8, 9**

`parent2`: **0, 8, 7, | 4, 3, 2, 1, | 9, 6, 5**

这里，交叉索引为 3 和 6，用竖线标出。

让我们逐步了解生成 `child1` 的过程。

从 `parent1` 复制后，`child1` 如下所示：

x, x, x, 3, 4, 5, 6, x, x, x

从 `parent2` 中的 9 开始，向右操作，并在到达 `parent2` 和 `child1` 的末尾时回绕到起点，我们添加不在 `child1` 中的值，得到：

**7, 2, 1, 3, 4, 5, 6, 9, 0, 8**

交换 `parent1` 和 `parent2` 的角色，证明 `child2` 为：

**0, 5, 6, 4, 3, 2, 1, 7, 8, 9**

现在的挑战是编写实现上述逻辑的 `OrderedCrossover` 函数。

该逻辑并非微不足道。`OrderedCrossover` 函数如下：

```go
func OrderedCrossOver(parent1, parent2 []int) (child1,
child2 []int) {
var index1, index2 int
n := len(parent1)
for {
index1 = 1 + rand.Intn(len(parent1)-1)
index2 = 1 + rand.Intn(len(parent1)-1)
if index1 != index2 {
break // 两个索引不相等
}
}
if index1 > index2 {
index1, index2 = index2, index1
}
child1 = make([]int, len(parent1))
child2 = make([]int, len(parent1))
for i := 0; i < len(parent1); i++ {
// 因为 0 是一个合法值
child1[i] = -1
child2[i] = -1
}
// child1 的逻辑
for i := index1; i <= index2; i++ {
child1[i] = parent1[i]
}
k := index2 + 1 // child1 的索引
for i := index2 + 1; i < len(parent1); i++ {
found, _ := In(parent2[i], child1)
if !found {
child1[k%n] = parent2[i]
k += 1
}
}
for i := 0; i <= index2; i++ {
found, _ := In(parent2[i], child1)
if !found {
// j := (i + index2 + 1) % n
child1[k%n] = parent2[i]
k += 1
}
}
// child2 的逻辑
for i := index1; i <= index2; i++ {
child2[i] = parent2[i]
}
k = index2 + 1 // child2 的索引
for i := index2 + 1; i < len(parent2); i++ {
found, _ := In(parent1[i], child2)
if !found {
child2[k%n] = parent1[i]
k += 1
}
}
for i := 0; i <= index2; i++ {
found, _ := In(parent1[i], child2)
if !found {
// j := (i + index2 + 1) % n
child2[k%n] = parent1[i]
k += 1
}
}
// 形成 child11 和 child22
// 以使它们都从 0 开始
child11 := []int{}
child22 := []int{}
_, index0 := In(0, child1)
for i := index0; i < len(child1); i++ {
child11 = append(child11, child1[i])
}
for i := 0; i < index0; i++ {
child11 = append(child11, child1[i])
}
_, index0 = In(0, child2)
for i := index0; i < len(child2); i++ {
child22 = append(child22, child2[i])
}
for i := 0; i < index0; i++ {
child22 = append(child22, child2[i])
}
return child11, child22
}
```

我们通过从 `child1` 和 `child2` 创建 `child11` 和 `child22`，强制每个子代从城市 0 开始他们的路径，这样 `child11` 和 `child22` 都以城市 0 开头。`OrderedCrossover` 函数的最后部分实现了这一点。

辅助函数 `In` 在多个地方使用，如下所示：

```go
func In(value int, values []int) (bool, int) {
// 如果值存在于 values 中则返回 true
// 返回位置索引，如果未找到则返回 -1
for index := 0; index < len(values); index++ {
if values[index] == value {
return true, index
}
}
return false, -1
}
```



### 形成下一代

我们定义一个全局变量 `newpopulation`，它从全局变量 `population` 创建而来。

新种群包含来自 `population` 的精英路线、从已配对的父母中产生的子代，以及对 `newpopulation` 中每条路线按指定概率执行的变异。这些变异涉及交换路线中随机选择的两个城市。

实现此功能的函数如下所示。

```
func FormNextGeneration() {
elite := ChooseEliteGroup()
// 将精英路线移入 newpopulation
row := 0 // newpopulation 的索引
for ; row < EliteNumber; row++ {
newpopulation[row] = DeepCopy(elite[row])
}
// 从 population 中移除前 EliteNumber 行
population = population[EliteNumber:]
// 初始化 group1 和 group2
group1 := make([][]int, TournamentNumber)
for i := 0; i < TournamentNumber; i++ {
group1[i] = make([]int, NUMCITIES)
}
group2 := make([][]int, TournamentNumber)
for i := 0; i < TournamentNumber; i++ {
group2[i] = make([]int, NUMCITIES)
}
MatingPoolSize := (ToursPerGeneration -
EliteNumber) / 2
for index := 0; index < MatingPoolSize; index++ {
// 获取第一组
indicesChosen := []int{}
rowsChosen := 0;
for {
randomRow := rand.Intn(TournamentNumber)
found, _ := In(randomRow, indicesChosen)
if !found {
indicesChosen = append(indicesChosen,
randomRow)
group1[rowsChosen] =
DeepCopy(population[randomRow])
rowsChosen += 1
}
if rowsChosen == TournamentNumber {
break
}
}
// 获取第二组
indicesChosen = []int{}
rowsChosen = 0;
for {
randomRow := rand.Intn(TournamentNumber)
found, _ := In(randomRow, indicesChosen)
if !found {
indicesChosen = append(indicesChosen,
randomRow)
group2[rowsChosen] =
DeepCopy(population[randomRow])
rowsChosen += 1
}
if rowsChosen == TournamentNumber {
break
}
}
// 对 group1 和 group2 进行排序
sort.Slice(group1, func(i, j int) bool {
return Cost(group1[i]) < Cost(group1[j])
})
sort.Slice(group2, func(i, j int) bool {
return Cost(group2[i]) < Cost(group2[j])
})
parent1 := group1[0] // 组 1 中的最优路线
parent2 := group2[0] // 组 2 中的最优路线
child1, child2 := OrderedCrossOver(parent1,
parent2)
newpopulation[row] = child1
row += 1
newpopulation[row] = child2
row += 1
}
// 执行变异
for row := 0; row < ToursPerGeneration; row++ {
r := rand.Float64()
if r <= ProbMutation {
SwapMutation(newpopulation[row])
}
}
population = make([][]int, ToursPerGeneration)
for i := 0; i < NUMCITIES; i++ {
population[i] = make([]int, NUMCITIES)
}
// 将 newpopulation 复制到 population
for row := 0; row < ToursPerGeneration; row++ {
for col := 0; col < NUMCITIES; col++ {
population[row][col] =
newpopulation[row][col]
}
}
}
```

代码中包含大量注释，应该很容易理解。

排序是通过 `sort` 包中的 `Slice` 函数完成的。

```
sort.Slice(group1, func(i, j int) bool {
return Cost(group1[i]) < Cost(group1[j])
})
```

这里指定了以路线的成本作为排序依据，成本较低的路线排在成本较高的路线之前。

### 整合所有模块

在清单 20-1 中，我们展示了使用启发式遗传编程算法解决 TSP 问题的完整程序。我们包含了一个主驱动程序，该程序加载了第 19 章中（使用模拟退火方法解决）的同一个 29 城市问题。我们展示并比较了这两种获得 TSP 启发式解的方法的结果。


```go
package main

import (
	"fmt"
	"math"
	"math/rand"
	"sort"
	"time"
)

const (
	NUMCITIES          = 29
	EliteNumber        = 2
	ToursPerGeneration = 100
	NumberGenerations  = 50000
	TournamentNumber   = 4
	ProbMutation       = 0.25
)

type Point struct {
	x float64
	y float64
}

var population [][]int
var newpopulation [][]int
var graph [][]float64

func (pt Point) distance(other Point) float64 {
	dx := pt.x - other.x
	dy := pt.y - other.y
	return math.Sqrt(dx*dx + dy*dy)
}

func CreateGraph(numCities int, cities []Point, graph [][]float64) {
	for row := 0; row < numCities; row++ {
		for col := 0; col < numCities; col++ {
			graph[row][col] = cities[row].distance(cities[col])
		}
	}
}

func In(city int, tour []int) (bool, int) {
	for i, v := range tour {
		if v == city {
			return true, i
		}
	}
	return false, -1
}

func Cost(tour []int) float64 {
	cost := graph[tour[len(tour)-1]][tour[0]]
	for i := 0; i < len(tour)-1; i++ {
		cost += graph[tour[i]][tour[i+1]]
	}
	return cost
}

func CreateInitialPopulation() {
	for i := 0; i < ToursPerGeneration; i++ {
		population[i][0] = 0
		for j := 1; j < NUMCITIES; j++ {
			population[i][j] = j
		}
		for j := 1; j < NUMCITIES; j++ {
			index := 1 + rand.Intn(NUMCITIES-1)
			population[i][j], population[i][index] = population[i][index], population[i][j]
		}
	}
}

func Mutation(tour []int, probMutation float64) {
	n := len(tour)
	for i := 1; i < n; i++ {
		if rand.Float64() < probMutation {
			index := 1 + rand.Intn(n-1)
			tour[i], tour[index] = tour[index], tour[i]
		}
	}
}

func TournamentSelection(population [][]int, tournamentNumber int) []int {
	n := len(population)
	best := population[rand.Intn(n)]
	for i := 1; i < tournamentNumber; i++ {
		next := population[rand.Intn(n)]
		if Cost(next) < Cost(best) {
			best = next
		}
	}
	return best
}

func FormNextGeneration() {
	for i := 0; i < EliteNumber; i++ {
		newpopulation[i] = population[i]
	}
	for i := EliteNumber; i < ToursPerGeneration; i += 2 {
		parent1 := TournamentSelection(population, TournamentNumber)
		parent2 := TournamentSelection(population, TournamentNumber)
		child1, child2 := OrderedCrossOver(parent1, parent2)
		Mutation(child1, ProbMutation)
		Mutation(child2, ProbMutation)
		newpopulation[i] = child1
		if i+1 < ToursPerGeneration {
			newpopulation[i+1] = child2
		}
	}
	for i := 0; i < ToursPerGeneration; i++ {
		population[i] = newpopulation[i]
	}
}

func Permutation(index1, index2 int, tour []int) {
	if index1 > index2 {
		index1, index2 = index2, index1
	}
	tour[index1], tour[index2] = tour[index2], tour[index1]
}

func OrderedCrossOver(parent1, parent2 []int) (child1, child2 []int) {
	var index1, index2 int
	n := len(parent1)
	for {
		index1 = 1 + rand.Intn(len(parent1)-1)
		index2 = 1 + rand.Intn(len(parent1)-1)
		if index1 != index2 {
			break
		}
	}
	if index1 > index2 {
		index1, index2 = index2, index1
	}

	child1 = make([]int, len(parent1))
	child2 = make([]int, len(parent1))
	for i := 0; i < len(parent1); i++ {
		child1[i] = -1
		child2[i] = -1
	}

	for i := index1; i <= index2; i++ {
		child1[i] = parent1[i]
	}
	k := index2 + 1
	for i := index2 + 1; i < len(parent1); i++ {
		found, _ := In(parent2[i], child1)
		if !found {
			child1[k%n] = parent2[i]
			k++
		}
	}
	for i := 0; i <= index2; i++ {
		found, _ := In(parent2[i], child1)
		if !found {
			child1[k%n] = parent2[i]
			k++
		}
	}

	for i := index1; i <= index2; i++ {
		child2[i] = parent2[i]
	}
	k = index2 + 1
	for i := index2 + 1; i < len(parent2); i++ {
		found, _ := In(parent1[i], child2)
		if !found {
			child2[k%n] = parent1[i]
			k++
		}
	}
	for i := 0; i <= index2; i++ {
		found, _ := In(parent1[i], child2)
		if !found {
			child2[k%n] = parent1[i]
			k++
		}
	}

	child11 := []int{}
	child22 := []int{}
	_, index0 := In(0, child1)
	for i := index0; i < len(child1); i++ {
		child11 = append(child11, child1[i])
	}
	for i := 0; i < index0; i++ {
		child11 = append(child11, child1[i])
	}
	_, index0 = In(0, child2)
	for i := index0; i < len(child2); i++ {
		child22 = append(child22, child2[i])
	}
	for i := 0; i < index0; i++ {
		child22 = append(child22, child2[i])
	}
	return child11, child22
}

func GeneticAlgorithm() {
	generation := 0
	population = make([][]int, ToursPerGeneration)
	for i := 0; i < ToursPerGeneration; i++ {
		population[i] = make([]int, NUMCITIES)
	}
	newpopulation = make([][]int, ToursPerGeneration)
	for i := 0; i < ToursPerGeneration; i++ {
		newpopulation[i] = make([]int, NUMCITIES)
	}
	lowestCostTour := 1000000000.0
	CreateInitialPopulation()
	for {
		if generation == NumberGenerations {
			break
		}
		sort.Slice(population, func(i, j int) bool {
			return Cost(population[i]) < Cost(population[j])
		})
		bestCost := Cost(population[0])
		if bestCost < lowestCostTour {
			lowestCostTour = bestCost
			fmt.Printf("\nLowest cost tour at generation %d = %0.2f", generation, lowestCostTour)
		}
		FormNextGeneration()
		generation++
	}
}

func main() {
	rand.Seed(time.Now().UnixNano())
	cities := []Point{}
	pt1 := Point{1150.0, 1760.0}
	cities = append(cities, pt1)
	pt2 := Point{630.0, 1660.0}
	cities = append(cities, pt2)
	pt3 := Point{40.0, 2090.0}
	cities = append(cities, pt3)
	pt4 := Point{750.0, 1100.0}
	cities = append(cities, pt4)
	pt5 := Point{750.0, 2030.0}
	cities = append(cities, pt5)
	pt6 := Point{1030.0, 2070.0}
	cities = append(cities, pt6)
	pt7 := Point{1650.0, 650.0}
	cities = append(cities, pt7)
	pt8 := Point{1490.0, 1630.0}
	cities = append(cities, pt8)
	pt9 := Point{790.0, 2260.0}
	cities = append(cities, pt9)
	pt10 := Point{710.0, 1310.0}
	cities = append(cities, pt10)
	pt11 := Point{840.0, 550.0}
	cities = append(cities, pt11)
	pt12 := Point{1170.0, 2300.0}
	cities = append(cities, pt12)
	pt13 := Point{970.0, 1340.0}
	cities = append(cities, pt13)
	pt14 := Point{510.0, 700.0}
	cities = append(cities, pt14)
	pt15 := Point{750.0, 900.0}
	cities = append(cities, pt15)
	pt16 := Point{1280.0, 1200.0}
	cities = append(cities, pt16)
	pt17 := Point{230.0, 590.0}
	cities = append(cities, pt17)
	pt18 := Point{460.0, 860.0}
	cities = append(cities, pt18)
	pt19 := Point{1040.0, 950.0}
	cities = append(cities, pt19)
	pt20 := Point{590.0, 1390.0}
	cities = append(cities, pt20)
	pt21 := Point{830.0, 1770.0}
	cities = append(cities, pt21)
	pt22 := Point{490.0, 500.0}
	cities = append(cities, pt22)
	pt23 := Point{1840.0, 1240.0}
	cities = append(cities, pt23)
	pt24 := Point{1260.0, 1500.0}
	cities = append(cities, pt24)
	pt25 := Point{1280.0, 790.0}
	cities = append(cities, pt25)
	pt26 := Point{490.0, 2130.0}
	cities = append(cities, pt26)
	pt27 := Point{1460.0, 1420.0}
	cities = append(cities, pt27)
	pt28 := Point{1260.0, 1910.0}
	cities = append(cities, pt28)
	pt29 := Point{360.0, 1980.0}
	cities = append(cities, pt29)

	graph = make([][]float64, NUMCITIES)
	for i := 0; i < NUMCITIES; i++ {
		graph[i] = make([]float64, NUMCITIES)
	}
	CreateGraph(NUMCITIES, cities, graph)
	GeneticAlgorithm()
}
/* 输出
Lowest cost tour at generation 0 = 22019.11
Lowest cost tour at generation 1 = 20169.35
Lowest cost tour at generation 5 = 20017.31
Lowest cost tour at generation 6 = 19545.05
Lowest cost tour at generation 7 = 18447.20
Lowest cost tour at generation 12 = 18340.78
Lowest cost tour at generation 13 = 17953.87
Lowest cost tour at generation 15 = 17350.68
Lowest cost tour at generation 16 = 17095.07
Lowest cost tour at generation 18 = 16612.21
...
Lowest cost tour at generation 970 = 9362.68
Lowest cost tour at generation 1179 = 9285.43
*/
```

该程序终止耗时不到十秒。经过十次运行后，第 1179 代的最低成本路径为`9285`。这与已知最优解`9074`相比，误差为 2%。显然，这种解决旅行商问题（TSP）的方法十分有效。


## 20.3 本章小结

本章介绍了一种基于遗传建模和适者生存法则来解决旅行商问题（TSP）的方法。随着程序从一代演进到下一代，解也在不断进化，最佳路径最终会收敛到近似最优解。

遗传算法的每一次运行都是一次新的实验。其结果在很大程度上取决于所选的常数。

下一章，我们将把注意力转向机器学习和神经网络。

