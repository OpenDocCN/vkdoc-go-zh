# 哈希表

在上一章中，我们介绍了队列和列表。我们展示了多种特殊类型的队列及其应用。

映射是一种将某个 `key` 转换（映射）为 `key-value` 对中 `value` 的函数。键和值可以是任何类型。哈希表是键值对的无序集合，其中每个键都是唯一的（无重复键）。值不要求唯一，因此两个或多个键可以映射到同一个值。哈希表支持通过键非常快速地访问信息。快速表查找是通过计算键的哈希值并获取表中包含该值的位置来实现的。

在下一节中，我们将回顾 Go 语言的标准数据结构 `map`。



## 7.1 `Map`

Go 语言的**映射**类型声明方式如下：

`map[KeyType]ValueType`

代码清单 7-1 是一个简单的程序，展示了使用标准 `map` 的基本操作。

```
package main
import (
"fmt"
)
type map1 map[string]string
func main() {
nicknames := make(map1, 5)
nicknames["Charles"] = "Chuck"
nicknames["Robert"] = "Bob"
nicknames["Richard"] = "Rick"
nicknames["Teddy"] = "Ted"
nicknames["Mohammad"] = "Mo"
for key, value := range (nicknames) {
fmt.Printf("\nThe nickname of %s is %s", key, value)
}
// 检查 map 中是否存在 James
_, present := nicknames["James"]
fmt.Println("\nThe key James is present: ", present)
// 检查 map 中是否存在 Teddy
_, present = nicknames["Teddy"]
fmt.Println("The key Teddy is present: ", present)
delete(nicknames, "Robert")
// 检查 map 中是否存在 Robert
_, present = nicknames["Robert"]
fmt.Println("The key Robert is present: ", present)
// 修改 Charles 的昵称
nicknames["Charles"] = "Charlie”
}
/* 输出
The nickname of Robert is Bob
The nickname of Richard is Rick
The nickname of Teddy is Ted
The nickname of Mohammad is Mo
The nickname of Charles is Chuck
The key James is present:  false
The key Teddy is present:  true
The key Robert is present:  false
*/
代码清单 7-1
使用标准 map
```

在代码清单 7-1 中，`for` 循环产生的键值对序列在每次运行程序时都可能不同。`map` 是一种无序集合。

如果某个键在 `map` 中不存在，测试该键时返回的值是 `map` 中存储的值类型的零值。但在访问键时会返回两个值。第一个是与该键相关联的值，第二个是一个 `Boolean` 值，用于判断该键是否存在于 `map` 中。我们在代码清单 7-1 中的多个位置使用了这种方式，以判断特定键是否存在于 `map` 中。

### 哈希加密

已经生产出用于支持加密的哈希函数包。在代码清单 7-2 中，我们将探讨两个此类哈希加密包的使用。

```
package main
import (
"crypto/md5"
"crypto/sha256"
"fmt"
)
func main() {
name1 := "Richard"
name2 := "Richards"
md5hash := md5.Sum([]byte(name1))
sha256hash := sha256.Sum256([]byte(name1))
fmt.Println("   MD5: ", md5hash)
fmt.Println("SHA256: ", sha256hash)
md5hash = md5.Sum([]byte(name2))
sha256hash = sha256.Sum256([]byte(name2))
fmt.Println("   MD5: ", md5hash)
fmt.Println("SHA256: ", sha256hash)
}
/* 输出
MD5:[197 28 139 189 158 140 139 196 144 66 204 213 211 233 134 77]
SHA256:[29 235 10 59 134 117 13 14 74 76 33 220 150 1 115 105 84 174 92 202 198 84 197 127 61 69 86 58 31 89 89 152]
MD5:[166 13 63 148 118 202 25 29 165 242 21 183 0 101 165 76]
SHA256:[107 180 140 197 199 134 66 52 247 101 104 172 63 77 46 205 135 103 147 106 45 109 84 183 195 48 107 144 11 99 127 198]
*/
代码清单 7-2
哈希加密
```

在 `md5` 上调用了 `Sum` 方法，并将输入字符串转换为 `[]byte`。在 `sha256` 上调用了 `Sum256` 方法，并将输入字符串转换为 `[]byte`。

有趣的是，从 `name1` 到 `name2` 仅仅增加了一个字符，就显著改变了 `md5` 和 `sha256` 的哈希值。

这些加密函数被广泛用于保护密码。

在下一节中，我们将考察 `map` 的效率，并将其搜索时间与 `slice` 进行比较。

## 7.2 `Map` 有多快？

假设我们考察访问一个大型字典单词集合的速度。

具体来说，我们从文本文件中构建一个包含 466,551 个英文单词的 `slice`。接着，从同一个文本文件构建一个包含相同 466,551 个单词的 `map`。然后，我们比较在 `slice` 中和在 `map` 中查找所有单词所需的时间。

为了加快 `sliceCollection` 的搜索速度，我们对单词进行排序，以便使用二分查找。在对 `slice` 中的单词进行排序后，我们将 `map` 的搜索时间与 `slice` 的搜索时间进行比较。

代码清单 7-3 展示了此比较的详细信息。

```
// 我们比较使用 map 和 slice 进行字典查找
package main
import (
"bufio"
"fmt"
"log"
"os"
"sort"
"time"
)
var mapCollection map[string]string
var sliceCollection []string
func IsPresent(word string, sliceCollection []string) bool {
for i := 0; i < len(sliceCollection); i++ {
if sliceCollection[i] == word {
return true
}
}
return false
}
func IsPresentBinarySearch(word string, sliceCollection []string) bool {
// slice 集合已排序
low := 0
high := len(sliceCollection) - 1
for low <= high {
median := (low + high) / 2
if sliceCollection[median] < word {
low = median + 1
} else {
high = median - 1
}
}
if low == len(sliceCollection) || sliceCollection[low] != word {
return false
}
return true
}
func main() {
file, err := os.Open("words.txt")
defer file.Close()
if err != nil {
log.Fatalf("打开文件时出错: %s", err)
}
// 用单词填充 mapCollection 和 sliceCollection
scanner := bufio.NewScanner(file)
scanner.Split(bufio.ScanLines)
mapCollection = make(map[string]string)
sliceCollection = make([]string, 1)
var words []string
for scanner.Scan() {
word := scanner.Text()
words = append(words, word)
mapCollection[word] = word
sliceCollection = append(sliceCollection, word)
}
// 测试 mapCollection 中每个单词是否存在 的基准时间
start := time.Now()
for i := 0; i < len(words); i++ {
_, present := mapCollection[words[i]]
if !present {
fmt.Println("在 mapCollectio0n 中未找到单词")
}
}
elapsed := time.Since(start)
fmt.Println("mapCollection 中的单词数量: ", len(mapCollection))
fmt.Println("\n 在 mapCollection 中测试单词的时间: ", elapsed)
sort.Strings(sliceCollection)
// 测试 sliceCollection 中每个单词是否存在 的基准时间
start = time.Now()
for i := 0; i < len(sliceCollection); i++ {
if !IsPresent(sliceCollection[i], sliceCollection) {
fmt.Println("在 mapCollectio0n 中未找到单词")
}
}
elapsed = time.Since(start)
fmt.Println("在 sliceCollection 中测试单词的时间: ", elapsed)
// 测试已排序 sliceCollection 中每个单词是否存在 的基准时间
start = time.Now()
for i := 0; i < len(sliceCollection); i++ {
if !IsPresentBinarySearch(sliceCollection[i], sliceCollection) {
fmt.Println("在 mapCollectio0n 中未找到单词")
}
}
elapsed = time.Since(start)
fmt.Println("在已排序 sliceCollection 中测试单词的时间: ", elapsed)
}
/* 输出
mapCollection 中的单词数量: 466468
在 mapCollection 中测试单词的时间: 29.022542ms
在 sliceCollection 中测试单词的时间: 2m20.874580833s
在已排序 sliceCollection 中测试单词的时间: 51.836708ms
*/
代码清单 7-3
map 与 slice 的搜索时间
```

基于 `map` 的集合几乎比已排序的 `slice` 集合快两倍，这证实了我们之前关于 `map` 提供非常快速的信息访问的说法。未排序的 `slice` 集合效率低下，其速度比 `map` 集合慢很多倍。

在下一节中，我们将展示如何构建一个哈希表。



## 7.3 构建哈希表

商业级哈希表（映射）结构复杂，且如前文所述，效率极高。

关于 Go 语言 map 函数的具体构建细节，可参见 [`https://github.com/golang/go/blob/master/src/runtime/map.go`](https://github.com/golang/go/blob/master/src/runtime/map.go)。

在本节中，我们将从头构建一个效率相对较低的哈希表，以便简要探讨与哈希表构建相关的问题。

考虑以下代码片段：

```
package main
import (
"hash/fnv" // Fowler–Noll–Vo 算法
// 其他细节暂未展示
)
const tableSize = 100_000
func hash(s string) uint32 {
h := fnv.New32a() // Fowler-Noll-Vo 算法
h.Write([]byte(s))
return h.Sum32()
}
type WordType struct {
word string
list []string
}
// 每个索引位置包含一个单词和一个字符串切片
type HashTable [tableSize]WordType
```

`hash` 函数使用 Fowler-Noll-Vo 算法将字符串映射为一个无符号 32 位整数。感兴趣的读者可以自行研究该算法的细节。

我们定义了一个 `WordType` 结构体，包含字段 `word` 和 `list`。

接下来，考虑创建并返回一个 `Hashtable` 的 `NewTable` 函数。

### 创建一个空的哈希表

```
func NewTable() HashTable {
var table HashTable
for i := 0; i < tableSize; i++ {
table[i] = WordType{"", []string{}}
}
return table
}
```

在 `HashTable` 数组（固定大小为 `tableSize`）的每个位置上，都分配了一个 `WordType` 变量，其 `word` 为空，字符串切片 `list` 也为空，即 `[]string{}`。

### 向哈希表插入数据

现在来看 `Insert` 方法，如下所示：

```
func (table *HashTable) Insert(word string) {
index := hash(word) % tableSize // 范围在 0 到 tableSize - 1 之间
// 在 table[index] 中查找 word
if table[index].word == word {
return // 不允许重复
}
if len(table[index].list) > 0 {
for i := 0; i < len(table[index].list); i++ {
if table[index].list[i] == word {
return // 不允许重复
}
}
}
if table[index].word == "" {
table[index].word = word
} else {
table[index].list = append(table[index].list, word)
}
length += 1
}
```

我们使用前面展示的 `hash` 函数，将输入参数 `word`（一个字符串）“映射”到一个索引。

哈希表中不能包含重复的键，因此我们检查 `word` 是否已存在于 `index` 位置。如果存在，则直接返回，不修改表。

我们进一步通过扫描 `list`（如果非空）来检查是否存在重复条目，如果发现重复，同样直接返回，不修改表。

假设输入的单词不在表中，我们将 `word` 赋值给 `table` 的 `index` 位置。如果该索引位置已经有一个单词，则将输入单词追加到该位置的字符串切片中。

### 冲突与冲突解决

要使某个索引位置的字符串切片增长，必然发生了冲突。也就是说，输入单词被分配的索引与已存在于该位置的单词发生了冲突。这种冲突是不可避免的，因为我们通过以下方式将 `index` 压缩到表大小范围内：

```
index := hash(word) % tableSize // 范围在 0 到 tableSize - 1 之间
```

### 负载因子

表的**负载因子**是表中单词数量除以表大小。即使负载因子小于 1，冲突仍然可能发生，因为哈希函数并不能为所有输入单词生成唯一的值。一个表如果在多个索引位置有很长的字符串切片（冲突链），其访问时间就会更长。

### 判断键是否存在

接下来我们考虑 `IsPresent` 函数，如下所示：

```
func (table HashTable) IsPresent(word string) bool {
index := hash(word) % tableSize // 范围在 0 到 tableSize - 1 之间
// 在 table[index] 中查找 word
if table[index].word == word {
return true
}
if len(table[index].list) > 0 {
for i := 0; i < len(table[index].list); i++ {
if table[index].list[i] == word {
return true
}
}
}
return false
}
```

如果输入的单词位于 `index` 位置，或者位于以 `index` 为起点的非空列表中，该函数返回 true。

### 比较哈希表与标准 Map 的性能

代码清单 7-4 将前面讨论的各个部分组合在一起，并比较了哈希表与标准 map 的执行时间。

```
// 哈希表构建
package main
import (
"fmt"
"hash/fnv" // Fowler–Noll–Vo 算法
"strconv"
"time"
)
const tableSize = 100_000
var length int
func hash(s string) uint32 {
h := fnv.New32a()
h.Write([]byte(s))
return h.Sum32()
}
type WordType struct {
word string
list []string
}
// 每个索引位置包含一个字符串切片
type HashTable [tableSize]WordType
func NewTable() HashTable {
var table HashTable
for i := 0; i < tableSize; i++ {
table[i] = WordType{"", []string{}}
}
return table
}
func (table *HashTable) Insert(word string) {
index := hash(word) % tableSize
if table[index].word == word {
return
}
if len(table[index].list) > 0 {
for i := 0; i < len(table[index].list); i++ {
if table[index].list[i] == word {
return
}
}
}
if table[index].word == "" {
table[index].word = word
} else {
table[index].list = append(table[index].list, word)
}
length++
}
func (table HashTable) IsPresent(word string) bool {
index := hash(word) % tableSize
if table[index].word == word {
return true
}
if len(table[index].list) > 0 {
for i := 0; i < len(table[index].list); i++ {
if table[index].list[i] == word {
return true
}
}
}
return false
}
func main() {
myTable := NewTable()
mapCollection := make(map[string]string)
words := []string{}
for i := 0; i < 500_000; i++ {
word := strconv.Itoa(i)
words = append(words, word)
myTable.Insert(word)
mapCollection[word] = ""
}
fmt.Println("基准测试开始，测试单词数量：", length)
start := time.Now()
for i := 0; i < length; i++ {
if myTable.IsPresent(words[i]) == false {
fmt.Println("在 myTable 中未找到单词：", words[i])
}
}
elapsed := time.Since(start)
fmt.Println("在 myTable 中测试所有单词用时：", elapsed)
start = time.Now()
for i := 0; i < len(mapCollection); i++ {
_, present := mapCollection[words[i]]
if !present {
fmt.Println("在 mapCollection 中未找到单词：", words[i])
}
}
elapsed = time.Since(start)
fmt.Println("在 mapCollection 中测试单词用时：", elapsed)
}
/* 输出
基准测试开始，测试单词数量：500000
在 myTable 中测试所有单词用时：1m17.880336666s
在 mapCollection 中测试单词用时：24.405583ms
*/
代码清单 7-4
比较哈希表与 map
```

通过将循环中的整数索引转换为字符串，生成五十万个单词，并将这些字符串作为输入分别存入哈希表和 map。哈希表搜索所有已存入的单词耗时近 138 秒，而 map 的耗时不到 25 毫秒。差异非常显著！

下一节，我们将深入探讨字符串搜索这一重要的应用领域。我们将介绍一个经典的字符串搜索算法，该算法以哈希为基础。

## 7.4 哈希应用：字符串搜索

本节我们将探讨一个经典且重要的字符串搜索应用。**Rabin-Karp** 算法以哈希为特色，试图将搜索的复杂度从 `O(n*m)` 降低到 `O(n)`，其中 n 是被搜索字符串的长度，m 是我们正在搜索的模式字符串的长度。

下面给出了一个使用暴力法完成字符串搜索的函数：

```
func BruteForceSearch(txt, pattern string) (bool, int) {
patternLength := len(pattern)
for outer := 0; outer < len(txt)-patternLength; outer++ {
if txt[(outer):(outer+patternLength)] == pattern {
return true, outer
}
}
return false, -1
}
```

外层循环的范围是从 `0` 到 `len(txt) – patternLength`，我们将 `txt[(outer):(outer+patternLength)]` 限定的字符串与 `pattern` 进行比较。如果两个字符串相等，则返回 true 和外层位置。由于字符串比较的时间复杂度是 `O(patternLength)`，并且我们执行该操作 n 次，因此得到一个 `O(n * m)` 的算法，其中 m 是模式长度。

此刻，你可能会合理地问：“这跟哈希有什么关系？”

假设我们将执行了 `n – m` 次的字符串相等性测试，替换为对其哈希值的比较。也就是说，我们比较 `hash(txt[(outer):(outer+patternLength)])` 与 `hash(pattern)`，并执行此操作 `n – m` 次，如果它们的哈希值相同，则返回 true。



### 滚动哈希计算

当`外部`索引递增 1 时，能否从上一个哈希值计算出新的哈希值，从而避免从头开始重新计算哈希？这正是 Rabin-Karp 算法所做的。它使用一种“滚动”哈希函数，其中后续的哈希计算可以廉价地从前一次哈希计算中获得。

对于文本中从`i`到`i + m - 1`的部分（其中`m`是模式串的长度），哈希函数`H`的定义如下：

`H[i] = (c[i]R^(m-1) + c[i+1]R^(m-2) + ... + c[i+m-1]R⁰) mod Q`

这里的`c`是待搜索字符串中指定位置的整数字符值，`R`是一个基数，对应每个字符可能取值的数量。`Q`是一个大质数，用于防止计算出的哈希值溢出。

这个函数不能保证不同字符串的哈希值唯一，但如果`Q`和`n`（字符串长度）足够大，则可以最大程度地降低冲突概率。

位置`i + 1`处的哈希值可以在常数时间内从位置`i`处的哈希值计算得出，公式如下：

`H[i+1] = (H[i] - c[i]R^(m-1))R + c[i+m]`

假设我们将字符集限定为数字“0”到“9”。我们的字符串搜索尝试判断一个由数字字符串定义的模式是否包含在一个更大的数字字符串中。

哈希函数如下所示：

```go
const (
    Radix = uint64(10)
    Q     = uint64(10 ^ 9 + 9)
)
func Hash(s string, Length int) uint64 {
    // Horner 方法
    h := uint64(0)
    for i := 0; i < Length; i++ {
        h = (h*Radix + uint64(s[i])) % Q
    }
    return h
}
```

这是用于计算多项式的 Horner 方法。我们使用`uint64`类型来避免溢出。

### Rabin-Karp 算法

`Search`方法使用了前面概述的 Rabin-Karp 算法，如下所示：

```go
func Search(txt, pattern string) (bool, int) {
    strings.ToLower(txt)
    strings.ToLower(pattern)
    n := len(txt)
    m := len(pattern)
    patternHash := Hash(pattern, m)
    textHash := Hash(txt, m)
    if textHash == patternHash {
        return true, 0
    }
    PM := uint64(1)
    for i := 1; i <= m-1; i++ {
        PM = (Radix * PM) % Q
    }
    for i := m; i < n; i++ {
        textHash = (textHash + Q - PM*uint64(txt[i-m])%Q) % Q
        textHash = (textHash*Radix + uint64(txt[i])) % Q
        if (patternHash == textHash) && pattern == txt[(i-m+1):(i+1)] {
            return true, i - m + 1
        }
    }
    return false, -1
}
```

由于`patternHash`和`textHash`相等并不能保证已找到模式串，因此我们需要将模式串与文本段进行比对确认。

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

const (
    Radix = uint64(10)
    Q     = uint64(10 ^ 9 + 9)
)

func BruteForceSearch(txt, pattern string) (bool, int) {
    patternLength := len(pattern)
    for outer := 0; outer < len(txt)-patternLength; outer++ {
        if txt[(outer):(outer+patternLength)] == pattern {
            return true, outer
        }
    }
    return false, -1
}

func Hash(s string, Length int) uint64 {
    // Horner 方法
    h := uint64(0)
    for i := 0; i < Length; i++ {
        h = (h*Radix + uint64(s[i])) % Q
    }
    return h
}

func Search(txt, pattern string) (bool, int) {
    strings.ToLower(txt)
    strings.ToLower(pattern)
    n := len(txt)
    m := len(pattern)
    patternHash := Hash(pattern, m)
    textHash := Hash(txt, m)
    if textHash == patternHash {
        return true, 0
    }
    PM := uint64(1)
    for i := 1; i <= m-1; i++ {
        PM = (Radix * PM) % Q
    }
    for i := m; i < n; i++ {
        textHash = (textHash + Q - PM*uint64(txt[i-m])%Q) % Q
        textHash = (textHash*Radix + uint64(txt[i])) % Q
        if (patternHash == textHash) && pattern == txt[(i-m+1):(i+1)] {
            return true, i - m + 1
        }
    }
    return false, -1
}

func main() {
    text := "31415926535897932384626433832795028841971693993751058209749445923078164062862089986280348253421170679"
    pattern := "816406286208998628034825342"
    start := time.Now()
    _, _ = BruteForceSearch(text, pattern)
    elapsed := time.Since(start)
    fmt.Println("使用 BruteForceSearch 的计算时间:", elapsed)
    start = time.Now()
    _, _ = Search(text, pattern)
    elapsed = time.Since(start)
    fmt.Println("使用 Search 的计算时间:", elapsed)
    fmt.Println(BruteForceSearch(text, pattern))
    fmt.Println(Search(text, pattern))
}

/* 使用搭载 M1 Max 的 Macbook Pro 输出
使用 BruteForceSearch 的计算时间: 10.083μs
使用 Search 的计算时间:  1.375μs
true 67
true 67
使用搭载 3.2 GHz 8 核 Intel Xeon W 的 iMac 输出
使用 BruteForceSearch 的计算时间: 354ns
使用 Search 的计算时间: 1.161μs
*/
```

*清单 7-5 比较 Rabin-Karp 算法与暴力搜索算法*

该程序在两台计算机上运行，结果令人惊讶。在配备 M1 Max 处理器和 32G 统一内存的 MacBook Pro 上，Rabin-Karp 搜索在搜索圆周率的前 100 位时速度提高了七倍以上，这并不意外。而在配备 3.2 GHz 8 核 Xeon W 处理器的 iMac 上，结果则相反。暴力算法返回的时间比 Rabin-Karp 算法快三倍以上。

这些相互矛盾的基准测试再次凸显了一个事实：特定机器的硬件和指令集会极大地影响此类基准测试的结果。

下一节，我们将使用哈希来实现一个通用的`Set`集合。



## 7.5 泛型集合

Go 语言没有直接提供 `Set` 数据结构。本节我们将实现一个泛型 `Set` 数据结构。集合存储的是唯一值，且值在集合内是有序的。

`Set` 定义了以下操作：

| `Insert(item)` – 若 `item` 不存在，则将其添加到现有集合中 |
| `Delete(item)` – 若 `item` 存在，则将其从现有集合中移除 |
| `In(item)` – 若 `item` 存在于现有集合中则返回 `true`，否则返回 `false` |
| `Items()` – 返回现有集合中所有元素的一个切片 |
| `Size()` – 返回现有集合中的元素数量 |
| `Union(set2)` – 返回现有集合与 `set2` 中所有不重复的元素 |
| `Intersection(set2)` – 返回同时存在于现有集合与 `set2` 中的元素 |
| `Difference(set2)` – 返回存在于现有集合但不存在于 `set2` 中的元素 |
| `Subset(set2)` – 如果 `set2` 中的所有元素都在 `set1` 中，则返回 `true`，否则返回 `false` |

在我们的 `set` 包中，我们定义泛型类型 `Set` 如下：

```
package set
type Ordered interface {
~string | ~int | ~float64
}
type Set[T Ordered] struct {
items map[T]bool
}
```

在这里，我们将 `Set` 的 `items` 字段定义为一个 `map`，其泛型参数 `T` 的类型为 `Ordered`。

Go 语言中的 `map` 结构要求键值的类型是可比较的。

`Insert` 方法的实现如下：

```
// Add item to set
func (set *Set[T]) Insert(item T) {
if set.items == nil {
set.items = make(map[T]bool)
}
// Prevent duplicate entry
_, present := set.items[item]
if !present {
set.items[item] = true
}
}
```

我们首先判断 `set` 是否为空，如果为空，则 `set.items` 会是 `nil`。在这种情况下，我们使用 `make` 初始化 `map`。

为了阻止在集合中插入重复元素（这是不被允许的），我们检查 `item` 是否已经在 `set` 中。如果不存在，我们将其赋给 `set.items` 这个 `map`。

`Delete` 方法的实现如下：

```
// Remove item from set
func (set *Set[T]) Delete(item T) {
_, present := set.items[item]
if present {
delete(set.items, item)
}
}
```

如果 `item` 存在，我们将其从 `set.items` 这个 `map` 中删除。

`In` 方法的实现如下：

```
// Return true if item is in set, otherwise false
func (set *Set[T]) In(item T) bool {
_, present := set.items[item]
return present
}
```

如果 `item` 存在于 `set.items` 这个 `map` 中则返回 `true`，否则返回 `false`。

`Items` 方法的实现如下：

```
// Return a slice of all the items in set
func (set *Set[T]) Items() []T {
items := []T{}
for item := range set.items {
items = append(items, item)
}
return items
}
```

我们初始化了一个类型为 `T` 的空切片。然后我们遍历 `set.items` 这个 `map` 的范围，并将每个 `item` 追加到 `items` 中，最后返回 `items`。

完整的 `set` 包如代码清单 7-6 所示。其他方法也同样直截了当。

```
package set
type Ordered interface {
~string | ~int | ~float64
}
type Set[T Ordered] struct {
items map[T]bool
}
// Methods
// Add item to set
func (set *Set[T]) Insert(item T) {
if set.items == nil {
set.items = make(map[T]bool)
}
// Prevent duplicate entry
_, present := set.items[item]
if !present {
set.items[item] = true
}
}
// Remove item from set
func (set *Set[T]) Delete(item T) {
_, present := set.items[item]
if present {
delete(set.items, item)
}
}
// Return true if item is in set, otherwise false
func (set *Set[T]) In(item T) bool {
_, present := set.items[item]
return present
}
// Return a slice of all the items in set
func (set *Set[T]) Items() []T {
items := []T{}
for item := range set.items {
items = append(items, item)
}
return items
}
// Return the number of items in set
func (set *Set[T]) Size() int {
return len(set.items)
}
// Return a new set containing all the unique items of set and set2
func (set *Set[T]) Union(set2 Set[T]) *Set[T] {
result := Set[T]{}
result.items = make(map[T]bool)
for index := range set.items {
result.items[index] = true
}
for j := range set2.items {
_, present := result.items[j]
if !present {
result.items[j] = true
}
}
return &result
}
// Return a new set containing the items found in both set and set2
func (set *Set[T]) Intersection(set2 Set[T]) *Set[T] {
result := Set[T]{}
result.items = make(map[T]bool)
for i := range set2.items {
_, present := set.items[i]
if present {
result.items[i] = true
}
}
return &result
}
// Return a new set of items in set not found in set2
func (set *Set[T]) Difference(set2 Set[T]) *Set[T] {
result := Set[T]{}
result.items = make(map[T]bool)
for i := range set.items {
_, present := set2.items[i]
if !present {
result.items[i] = true
}
}
return &result
}
// Return true if all items of set2 are in set
func (set *Set[T]) Subset(set2 Set[T]) bool {
for i := range set.items {
_, present := set2.items[i]
if !present {
return false
}
}
return true
}
代码清单 7-6
包 set
```

代码清单 7-7 给出了一个主驱动测试程序，用于练习 `set` 包的方法。

```
package main
import (
"example.com/set"
"fmt"
)
func main() {
set1 := set.Set[int]{}
set1.Insert(3)
set1.Insert(5)
set1.Insert(7)
set1.Insert(9)
set2 := set.Set[int]{}
set2.Insert(3)
set2.Insert(6)
set2.Insert(8)
set2.Insert(9)
set2.Insert(11)
set2.Delete(11)
fmt.Println("Items in set2: ", set2.Items())
fmt.Println("5 in set1: ", set1.In(5))
fmt.Println("5 in set2: ", set2.In(5))
fmt.Println("Union of set1 and set2: ", set1.Union(set2).Items())
fmt.Println("Intersection of set1 and set2: ",
set1.Intersection(set2).Items())
fmt.Println("Difference of set2 with respect to set1: ",
set2.Difference(set1).Items())
fmt.Println("Size of this difference: ", set1.Intersection(set2).Size())
}
/* Output
Items in set2: [6 8 9 3]
5 in set1:  true
5 in set2:  false
Union of set1 and set2: [9 3 5 7 6 8]
Intersection of set1 and set2: [3 9]
Difference of set2 with respect to set1: [6 8]
Size of this difference:  2
*/
代码清单 7-7
练习 set 包的主驱动
```

## 7.6 小结

在本章中，我们研究了哈希函数和哈希表。我们看到，使用标准 `map` 数据结构的哈希表在搜索无序集合时非常高效。我们学习了经典的 Rabin-Karp 算法，用于高效地搜索字符串中的模式。该算法使用了滚动哈希函数。最后，我们使用哈希映射实现了一个 `Set`。

下一章，我们将注意力转向树数据结构。这是聚焦于二叉树的若干章节中的第一章。

