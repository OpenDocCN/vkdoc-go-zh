# 6.11 单向链表

一个泛型单向链表的数据结构定义如下：

```
package singlylinkedlist
import (
"fmt"
)
type Ordered interface {
~string | ~int | ~float64
}
type Node[T Ordered] struct {
Item T
next *Node[T]
}
type List[T Ordered] struct {
first       *Node[T]
numberItems int
}
```

类型`Node`包含一个`Item`（首字母大写，以便在包外访问）和一个指向列表中下一个节点的指针。

类型`List`包含一个指向列表头节点的指针`first`和一个整型`numberItems`。

现在我们来详细讨论一下可以在`List`上调用的两个方法。

方法`Append`创建一个新节点并将其添加到列表末尾，操作如下：

```
func (list *List[T]) Append(item T) {
// Adds item to a new node at the end of the list
newNode := Node[T]{item, nil}
if list.first == nil {
list.first = &newNode
} else {
last := list.first
for {
if last.next == nil {
break
}
last = last.next
}
last.next = &newNode
}
list.numberItems += 1
}
```

定义了一个泛型类型`T`的`newNode`，它包含`item`且其`next`指针指向`nil`。

如果调用该方法的列表为空，则将`list.first`赋值为`newNode`的地址。

否则，执行一个`for`循环，推进指针`last`，直到`last.next`为`nil`。然后将`last.next`赋值为`newNode`的地址。

方法`InsertAt`创建一个新节点并将其添加到指定的`index`位置，操作如下：

```
func (list *List[T]) InsertAt(index int, item T) error {
// Adds item to a new node at position index in the list
if index < 0 || index > list.numberItems {
return fmt.Errorf("Index out of bounds error")
}
newNode := Node[T]{item, nil}
if index == 0 {
newNode.next = list.first
list.first = &newNode
list.numberItems += 1
return nil // No error
}
node := list.first
count := 0
previous := node
for count < index {
previous = node
count++
node = node.next
}
newNode.next = node
previous.next = &newNode
list.numberItems += 1
return nil // no error
}
```

首先对`index`的值进行测试，如果`index`小于 0 或大于列表中现有元素的数量，则返回一个错误。

与之前一样，创建一个包含`item`且`next`指针指向`nil`的新节点。如果`index`为 0，则将新节点的`next`指针设置为指向`list.first`。然后将`list.first`赋值为新节点的地址。

如果`index`不为 0，则执行一个`for`循环，将指针`node`（初始赋值为`list.first`）向后移动`index – 1`次，同时有一个尾随指针`previous`跟随其后。

然后进行两次链接赋值：将新节点链接到`node`，并将`previous`节点的`next`指针赋值为新节点的地址。

代码清单 6-12 展示了整个`singlylinkedlist`包，而代码清单 6-13 展示了一个驱动程序，该程序使用了代码清单 6-12 中定义的所有方法。

```
package main
import (
"fmt"
"example.com/singlylinkedlist"
)
func main() {
cars := singlylinkedlist.List[string]{}
cars.Append("Honda")
cars.InsertAt(0, "Nissan")
cars.InsertAt(0, "Chevy")
cars.InsertAt(1, "Ford")
cars.InsertAt(1, "Tesla")
cars.InsertAt(0, "Audi")
cars.InsertAt(2, "Volkswagon")
cars.Append("Volvo")
fmt.Println(cars.Items())
fmt.Println("Index of Tesla: ", cars.IndexOf("Tesla"))
cars.RemoveAt(0)
car, _ := cars.RemoveAt(3)
fmt.Println("car removed is: ", car)
fmt.Println(cars.Items())
cars.RemoveAt(cars.Size() - 1)
fmt.Println(cars.Items())
cars.Append("Lexus")
fmt.Println(cars.Items())
fmt.Println("First car in the list is: ", cars.First().Item)
fmt.Println("Last car in the list is: ", cars.Items()[cars.Size() - 1])
}
/* Output
[Audi Chevy Volkswagon Tesla Ford Nissan Honda Volvo]
Index of Tesla:  3
car removed is:  Ford
[Chevy Volkswagon Tesla Nissan Honda Volvo]
[Chevy Volkswagon Tesla Nissan Honda]
[Chevy Volkswagon Tesla Nissan Honda Lexus]
First car in the list is:  Chevy
Last car in the list is:  Lexus
*/
代码清单 6-13
singlylinkedlist 的主驱动程序
```

```
package singlylinkedlist
import (
"fmt"
)
type Ordered interface {
~string | ~int | ~float64
}
type Node[T Ordered] struct {
Item T
next *Node[T]
}
type List[T Ordered] struct {
first       *Node[T]
numberItems int
}
// Methods
func (list *List[T]) Append(item T) {
// Adds item to a new node at the end of the list
newNode := Node[T]{item, nil}
if list.first == nil {
list.first = &newNode
} else {
last := list.first
for {
if last.next == nil {
break
}
last = last.next
}
last.next = &newNode
}
list.numberItems += 1
}
func (list *List[T]) InsertAt(index int, item T) error {
// Adds item to a new node at position index in the list
if index < 0 || index > list.numberItems {
return fmt.Errorf("Index out of bounds error")
}
newNode := Node[T]{item, nil}
if index == 0 {
newNode.next = list.first
list.first = &newNode
list.numberItems += 1
return nil // No error
}
node := list.first
count := 0
previous := node
for count < index {
previous = node
count++
node = node.next
}
newNode.next = node
previous.next = &newNode
list.numberItems += 1
return nil // no error
}
func (list *List[T]) RemoveAt(index int) (T, error) {
// Removes item at position index in the list
if index < 0 || index > list.numberItems {
var zero T
return zero, fmt.Errorf("Index out of bounds error")
}
node := list.first
if index == 0 {
toRemove := node
list.first = toRemove.next
list.numberItems -= 1
return toRemove.Item, nil
}
count := 0
previous := node
for count < index {
previous = node
count++
node = node.next
}
toRemove := node
previous.next = toRemove.next
list.numberItems -= 1
return toRemove.Item, nil
}
func (list *List[T]) IndexOf(item T) int {
node := list.first
count := 0
for {
if node.Item == item {
return count
}
if node.next == nil {
return -1
}
node = node.next
count += 1
}
}
func (list *List[T]) ItemAfter(item T) T {
// Scan list for the first occurence of item
node := list.first
for {
if node == nil { // item not found
var zero T
return zero
}
if node.Item == item {
break
}
node = node.next
}
return node.next.Item
}
func (list *List[T]) Items() []T {
result := []T{}
node := list.first
for i := 0; i < list.numberItems; i++ {
result = append(result, node.Item)
node = node.next
}
return result
}
func (list *List[T]) First() *Node[T] {
return list.first
}
func (list *List[T]) Size() int {
return list.numberItems
}
代码清单 6-12
singlylinkedlist 包
```

`singlylinkedlist`包中的其余方法留给读者自行研读和理解，并验证主驱动程序的输出是否正确。

为完整起见，下一节将介绍双向链表的实现细节。



