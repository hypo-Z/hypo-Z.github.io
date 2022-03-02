---
title: Go语言进阶——数据结构之栈和队列
author: hypo
top: false
cover: false
toc: true
mathjax: false
date: 2022-03-02 12:08:31
img: medias/featureimages/52.jpeg
coverImg:
password:
summary: 由之前的文章可知，存储数据元素将数据放入或删除存储空间内，最基本的结构是数组和链表，而获取数据元素的方式有多种形式，但最简单的是取最前或最后。
categories: Galang
tags:
- Galang
- 进阶
---
# Go语言进阶——数据结构之栈和队列

## 一、简介

由之前的文章可知，存储数据元素将数据放入或删除存储空间内，最基本的结构是数组和链表，而获取数据元素的方式有多种形式，但最简单的是取最前或最后。

栈（stack）：先进后出，就像堆东西一样，需要将后面的取了才能取前面的。

队列（queue）：先进先出，排队。

我们可以用数据结构：`链表`（可连续或不连续的将数据与数据关联起来的结构），或 `数组`（连续的内存空间，按索引取值） 来实现 `栈（stack）` 和 `队列 (queue)`。



## 二、数组栈ArrayStack

数组形式的下压栈，后进先出：

使用切片来实现：

### 2.1 结构

```go
// 数组栈，后进先出
type ArrayStack struct {
    array []string   // 底层切片
    size  int        // 栈的元素数量
    lock  sync.Mutex // 为了并发安全使用的锁
}

```

### 2.2 入栈：

```go
// 入栈
func (stack *ArrayStack) Push(v string) {
    stack.lock.Lock()
    defer stack.lock.Unlock()

    // 放入切片中，后进的元素放在数组最后面
    stack.array = append(stack.array, v)

    // 栈中元素数量+1
    stack.size = stack.size + 1
}
```

将元素加到数组最后面，使用锁保证并发安全，切片容量会自动扩容。

### 2.3 出栈

```go
func (stack *ArrayStack) Pop() string {
    stack.lock.Lock()
    defer stack.lock.Unlock()

    // 栈中元素已空
    if stack.size == 0 {
        panic("empty")
    }

    // 栈顶元素
    v := stack.array[stack.size-1]

    // 切片收缩，但可能占用空间越来越大
    //stack.array = stack.array[0 : stack.size-1]

    // 创建新的切片，空间占用不会越来越大，但可能移动元素次数过多
    newArray := make([]string, stack.size-1, stack.size-1)
    for i := 0; i < stack.size-1; i++ {
        newArray[i] = stack.array[i]
    }
    stack.array = newArray

    // 栈中元素数量-1
    stack.size = stack.size - 1
    return v
}
```

返回栈顶元素，使用锁保证并发安全，创建新的切片将余下的元素复制到新切片，即储存空间降容

### 2.4 获取栈顶元素

```go
// 获取栈顶元素
func (stack *ArrayStack) Peek() string {
    // 栈中元素已空
    if stack.size == 0 {
        panic("empty")
    }

    // 栈顶元素值
    v := stack.array[stack.size-1]
    return v
}
```

### 2.5 获取栈大小和判断是否为空

```go
// 栈大小
func (stack *ArrayStack) Size() int {
    return stack.size
}

// 栈是否为空
func (stack *ArrayStack) IsEmpty() bool {
    return stack.size == 0
}
```

### 2.6 举个🌰

```go
func main() {
    arrayStack := new(ArrayStack)
    arrayStack.Push("cat")
    arrayStack.Push("dog")
    arrayStack.Push("hen")
    fmt.Println("size:", arrayStack.Size())
    fmt.Println("pop:", arrayStack.Pop())
    fmt.Println("pop:", arrayStack.Pop())
    fmt.Println("size:", arrayStack.Size())
    arrayStack.Push("drag")
    fmt.Println("pop:", arrayStack.Pop())
}
```

输出：

```go
size: 3
pop: hen
pop: dog
size: 1
pop: drag
```

## 三、数组队列ArrayQueue

与数组栈类似，但是元素是先进先出

### 3.1 结构：

```go
// 数组队列，先进先出
type ArrayQueue struct {
    array []string   // 底层切片
    size  int        // 队列的元素数量
    lock  sync.Mutex // 为了并发安全使用的锁
}
```

### 3.2 入队

```go
// 入队
func (queue *ArrayQueue) Add(v string) {
    queue.lock.Lock()
    defer queue.lock.Unlock()

    // 放入切片中，后进的元素放在数组最后面
    queue.array = append(queue.array, v)

    // 队中元素数量+1
    queue.size = queue.size + 1
}
```

和入栈一样，元素都是直接存放数组的最后方

### 3.3 出队

```go
// 出队
func (queue *ArrayQueue) Remove() string {
    queue.lock.Lock()
    defer queue.lock.Unlock()

    // 队中元素已空
    if queue.size == 0 {
        panic("empty")
    }

    // 队列最前面元素
    v := queue.array[0]

    /*    直接原位移动，但缩容后继的空间不会被释放
        for i := 1; i < queue.size; i++ {
            // 从第一位开始进行数据移动
            queue.array[i-1] = queue.array[i]
        }
        // 原数组缩容
        queue.array = queue.array[0 : queue.size-1]
    */

    // 创建新的数组，移动次数过多
    newArray := make([]string, queue.size-1, queue.size-1)
    for i := 1; i < queue.size; i++ {
        // 从老数组的第一位开始进行数据移动
        newArray[i-1] = queue.array[i]
    }
    queue.array = newArray

    // 队中元素数量-1
    queue.size = queue.size - 1
    return v
}
```

`缩容`即将原切片第一位后面的元素复制到新切片

## 四、链表栈LinkStack

### 4.1结构

```go
// 链表栈，后进先出
type LinkStack struct {
    root *LinkNode  // 链表起点
    size int        // 栈的元素数量
    lock sync.Mutex // 为了并发安全使用的锁
}

// 链表节点
type LinkNode struct {
    Next  *LinkNode
    Value string
}
```

这使用最简单的列表结构

### 4.2 入栈

```go
// 入栈
func (stack *LinkStack) Push(v string) {
    stack.lock.Lock()
    defer stack.lock.Unlock()

    // 如果栈顶为空，那么增加节点
    if stack.root == nil {
        stack.root = new(LinkNode)
        stack.root.Value = v
    } else {
        // 否则新元素插入链表的头部
        // 原来的链表
        preNode := stack.root

        // 新节点
        newNode := new(LinkNode)
        newNode.Value = v

        // 原来的链表链接到新元素后面
        newNode.Next = preNode

        // 将新节点放在头部
        stack.root = newNode
    }

    // 栈中元素数量+1
    stack.size = stack.size + 1
}
```

将新元素放在原链表的头部即满足了栈的后进先出

### 4.3 出栈

```go
// 出栈
func (stack *LinkStack) Pop() string {
    stack.lock.Lock()
    defer stack.lock.Unlock()

    // 栈中元素已空
    if stack.size == 0 {
        panic("empty")
    }

    // 顶部元素要出栈
    topNode := stack.root
    v := topNode.Value

    // 将顶部元素的后继链接链上
    stack.root = topNode.Next

    // 栈中元素数量-1
    stack.size = stack.size - 1

    return v
}
```

### 4.4 获取栈顶元素

```go
// 获取栈顶元素
func (stack *LinkStack) Peek() string {
    // 栈中元素已空
    if stack.size == 0 {
        panic("empty")
    }

    // 顶部元素值
    v := stack.root.Value
    return v
}
```

### 4.5 获取栈大小和判断是否为空

```go
// 栈大小
func (stack *LinkStack) Size() int {
    return stack.size
}

// 栈是否为空
func (stack *LinkStack) IsEmpty() bool {
    return stack.size == 0
}
```

### 4.6 举个🌰

```go
func main() {
    linkStack := new(LinkStack)
    linkStack.Push("cat")
    linkStack.Push("dog")
    linkStack.Push("hen")
    fmt.Println("size:", linkStack.Size())
    fmt.Println("pop:", linkStack.Pop())
    fmt.Println("pop:", linkStack.Pop())
    fmt.Println("size:", linkStack.Size())
    linkStack.Push("drag")
    fmt.Println("pop:", linkStack.Pop())
}
```

输出：

```go
size: 3
pop: hen
pop: dog
size: 1
pop: drag
```

## 五、链表队列LinkQueue

队列先进先出，和栈操作顺序相反，我们这里只实现入队，和出队操作，其他操作和栈一样。

### 5.1 结构

```go
// 链表队列，先进先出
type LinkQueue struct {
    root *LinkNode  // 链表起点
    size int        // 队列的元素数量
    lock sync.Mutex // 为了并发安全使用的锁
}

// 链表节点
type LinkNode struct {
    Next  *LinkNode
    Value string
}
```

### 5.2 入队

```go
// 入队
func (queue *LinkQueue) Add(v string) {
    queue.lock.Lock()
    defer queue.lock.Unlock()

    // 如果栈顶为空，那么增加节点
    if queue.root == nil {
        queue.root = new(LinkNode)
        queue.root.Value = v
    } else {
        // 否则新元素插入链表的末尾
        // 新节点
        newNode := new(LinkNode)
        newNode.Value = v

        // 一直遍历到链表尾部
        nowNode := queue.root
        for nowNode.Next != nil {
            nowNode = nowNode.Next
        }

        // 新节点放在链表尾部
        nowNode.Next = newNode
    }

    // 队中元素数量+1
    queue.size = queue.size + 1
}
```

### 5.3 出队

```go
// 出队
func (queue *LinkQueue) Remove() string {
    queue.lock.Lock()
    defer queue.lock.Unlock()

    // 队中元素已空
    if queue.size == 0 {
        panic("empty")
    }

    // 顶部元素要出队
    topNode := queue.root
    v := topNode.Value

    // 将顶部元素的后继链接链上
    queue.root = topNode.Next

    // 队中元素数量-1
    queue.size = queue.size - 1

    return v
}
```



## 总结

到此可以总结这几种存储结构的优缺：

数组实现栈和队列：能快速随机访问存储的元素，通过下标 `index` 访问，支持随机访问，查询速度快，但存在元素在数组空间中大量移动的操作，增删效率低。

链表实现栈和队列：只支持顺序访问，在某些遍历操作中查询速度慢，但增删元素快。

文章参考：
[数据结构和算法](https://goa.lenggirl.com/#/algorithm/link?id=_11%e5%88%9d%e5%a7%8b%e5%8c%96%e5%be%aa%e7%8e%af%e9%93%be%e8%a1%a8)
[菜鸟教程](https://www.runoob.com/go/go-arrays.html)