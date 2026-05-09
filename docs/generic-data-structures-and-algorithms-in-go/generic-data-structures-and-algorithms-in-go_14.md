# 双向链表

在双向链表中，每个节点既指向链表中的下一个节点，也指向上一个节点。由此，我们得到一个通用双向链表的数据结构如下：

```
type Ordered interface {
    ~string | ~int | ~float64
}
type Node[T Ordered] struct {
    Item T
    next *Node[T]
    prev *Node[T]
}
type List[T Ordered] struct {
    first       *Node[T]
    last        *Node[T]
    numberItems int
}
```

`List` 包含一个额外的字段 `last`，它指向链表的末尾。由于每个节点除了向前链接外还需向后链接，双向链表的方法更为复杂。这些细节展示在代码清单 6-14 中。

```
package doublylinkedlist

import (
    "fmt"
)

type Ordered interface {
    ~string | ~int | ~float64
}

type Node[T Ordered] struct {
    Item T
    next *Node[T]
    prev *Node[T]
}

type List[T Ordered] struct {
    first       *Node[T]
    last        *Node[T]
    numberItems int
}

// 方法

func (list *List[T]) Append(item T) {
    // 将元素添加到链表末尾的新节点中
    newNode := Node[T]{item, nil, nil}
    if list.first == nil {
        list.first = &newNode
        list.last = list.first
    } else {
        list.last.next = &newNode
        newNode.prev = list.last
        list.last = &newNode
    }
    list.numberItems += 1
}

func (list *List[T]) InsertAt(index int, item T) error {
    // 将元素添加到链表中指定索引位置的新节点
    if index < 0 || index > list.numberItems {
        return fmt.Errorf("Index out of bounds error")
    }
    newNode := Node[T]{item, nil, nil}
    if index == 0 {
        newNode.next = list.first
        if list.first != nil {
            list.first.prev = &newNode
        }
        list.first = &newNode
        list.numberItems += 1
        if list.numberItems == 1 {
            list.last = list.first
        }
        return nil // 无错误
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
    node.prev = &newNode
    newNode.prev = previous
    list.numberItems += 1
    return nil // 无错误
}

func (list *List[T]) RemoveAt(index int) (T, error) {
    // 从链表中移除指定索引位置的元素
    if index < 0 || index > list.numberItems {
        var zero T
        return zero, fmt.Errorf("Index out of bounds error")
    }
    node := list.first
    if index == 0 {
        toRemove := node
        list.first = toRemove.next
        list.numberItems -= 1
        if list.numberItems <= 1 {
            list.last = list.first
        }
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
    toRemove.next.prev = previous
    list.numberItems -= 1
    if list.numberItems <= 1 {
        list.last = list.first
    }
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
    // 扫描链表查找第一个出现的元素
    node := list.first
    for {
        if node == nil { // 未找到元素
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

func (list *List[T]) ItemBefore(item T) T {
    // 扫描链表查找第一个出现的元素
    node := list.first
    for {
        if node == nil { // 未找到元素
            var zero T
            return zero
        }
        if node.Item == item {
            break
        }
        node = node.next
    }
    return node.prev.Item
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

func (list *List[T]) ReverseItems() []T {
    result := []T{}
    node := list.last
    for {
        if node == nil {
            break
        }
        result = append(result, node.Item)
        node = node.prev
    }
    return result
}

func (list *List[T]) First() *Node[T] {
    return list.first
}

func (list *List[T]) Last() *Node[T] {
    return list.last
}

func (list *List[T]) Size() int {
    return list.numberItems
}
```

代码清单 6-14 – `doublylinkedlist` 包

如果将这个双向链表的实现细节与之前单向链表的细节进行比较，我们会看到多个赋值语句，这些语句将正在添加的节点链接到它前面的节点。当链表只包含一个节点时，`first` 和 `last` 指针是相同的。

我们详细考察 `InsertAt` 方法。这是所有方法中最复杂的一个。其他方法的工作方式类似。

```
func (list *List[T]) InsertAt(index int, item T) error {
    // 将元素添加到链表中指定索引位置的新节点
    if index < 0 || index > list.numberItems {
        return fmt.Errorf("Index out of bounds error")
    }
    newNode := Node[T]{item, nil, nil}
    if index == 0 {
        newNode.next = list.first
        if list.first != nil {
            list.first.prev = &newNode
        }
        list.first = &newNode
        list.numberItems += 1
        if list.numberItems == 1 {
            list.last = list.first
        }
        return nil // 无错误
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
    node.prev = &newNode
    newNode.prev = previous
    list.numberItems += 1
    return nil // 无错误
}
```

在验证传入的索引值未超出范围后，我们创建一个包含元素且前后指针均指向 nil 的 `newNode`。

我们首先考虑索引为 0 的情况。这种情况下，我们将 `newNode.next` 链接到现有的 `list.first`，即使它为 nil（空链表）。然后我们将 `list.first` 设置为 `newNode` 并递增 `list.numberItems`。如果元素数量为 1，我们将 `list.last` 赋值为 `list.first`，因为它们此时指向同一个节点。我们返回 nil 表示无错误。

当 `index` 不为 0 时，我们将局部变量 `node` 赋值为 `list.first`，局部变量 `count` 赋值为 0，局部变量 `previous` 赋值为 `node`。

在一个运行到 `count` 等于 `index` 的 for 循环中，我们将 `previous` 设置为 `node`，递增 `count`，并将 `node` 前进为 `node.next`。

在循环下方，找到了插入 `newNode` 的位置后，我们执行插入操作：将 `newNode.next` 设置为 `node`，将 `previous.next` 设置为 `&newNode`，将 `node.prev` 设置为 `&newNode`，将 `newNode.prev` 设置为 `previous`，并递增 `list.numberItems`。

### 双向链接的优势

你可能会想，考虑到构建和维护双向链表所需的额外复杂性，双向链接的优势到底是什么？

最重要的优势是能够反向遍历链表，即从尾部到头部。虽然使用单向链表也可以实现这一点，但计算代价会很高。

## 总结

在本章中，我们介绍了几种重要且实用的通用数据结构，包括 `Queue`、`Deque`、`PriorityQueue`，以及单向链表和双向链表。我们还展示了一些利用这些通用数据结构的应用。

在下一章中，我们将介绍哈希表结构以及一些使用该结构的应用。

