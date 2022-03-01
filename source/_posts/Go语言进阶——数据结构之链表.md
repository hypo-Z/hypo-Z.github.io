---
title: Go语言进阶——数据结构之链表
author: hypo
top: false
cover: false
toc: true
mathjax: false
date: 2022-03-01 13:51:52
img: medias/featureimages/51.jpeg
coverImg:
password:
summary: 链表由一个个数据节点组成的，它是一个递归结构，要么它是空的，要么它存在一个指向另外一个数据节点的引用。
categories: Golang
tags:
- Galang
- 进阶
---
# Go语言进阶——数据结构之链表

## 一、链表

> 链表由一个个数据节点组成的，它是一个递归结构，要么它是空的，要么它存在一个指向另外一个数据节点的引用。

链表最基础的数据结构

- 这是一个简单的单向链表结构：

```go
type LinkNode struct{
	Data     int64
    NextNode *LinkNode
}
```

```go
func singleRing() {
    // 新的节点
    node := new(LinkNode)
    node.Data = 2

    // 新的节点
    node1 := new(LinkNode)
    node1.Data = 3
    node.NextNode = node1 // node1 链接到 node 节点上

    // 新的节点
    node2 := new(LinkNode)
    node2.Data = 4
    node1.NextNode = node2 // node2 链接到 node1 节点上

    // 按顺序打印数据
    nowNode := node
    for {
        if nowNode != nil {
            // 打印节点值
            fmt.Println(nowNode.Data)
            // 获取下一个节点
            nowNode = nowNode.NextNode
            continue
        }

        // 如果下一个节点为空，表示链表结束了
        break
    }
}
```

运行打印：

```go
2
3
4
```

- 接下来看看双链表的实现：

在Go的官方标准库container/ring有实现：

双链表结构如下：

```go
// 循环链表
type Ring struct {
    next, prev *Ring       // 前驱和后驱节点
    Value      interface{} // 数据
}
```

```go
// 初始化空的循环链表，前驱和后驱都指向自己，因为是循环的
func (r *Ring) init() *Ring {
    r.next = r
    r.prev = r
    return r
}
```

```go
// 后驱指向下一个元素，所以值不能为空
func (r *Ring) Next() *Ring {
	if r.next == nil {
		return r.init()
	}
	return r.next
}

// 前驱指向上一个元素，所以值不能为空
func (r *Ring) Prev() *Ring {
	if r.next == nil {
		return r.init()
	}
	return r.prev
}
```

```go
// 创建一个元素为n的链表
func New(n int) *Ring {
	if n <= 0 {
		return nil
	}
	r := new(Ring)
	p := r
	for i := 1; i < n; i++ {
		p.next = &Ring{prev: p}
		p = p.next
	}
	p.next = r
	r.prev = p
	return r
}
```

```go
// 计算链表的长度及元素个数
// 它的执行时间与元素的数量成正比。
func (r *Ring) Len() int {
	n := 0
	if r != nil {
		n = 1
		for p := r.Next(); p != r; p = p.next {
			n++
		}
	}
	return n
}
```

```go
//遍历查找节点，当n小于零则向前n位，当n大于零则向后n位
func (r *Ring) Move(n int) *Ring {
	if r.next == nil {
		return r.init()
	}
	switch {
	case n < 0:
		for ; n < 0; n++ {
			r = r.prev
		}
	case n > 0:
		for ; n > 0; n-- {
			r = r.next
		}
	}
	return r
}
```

```go
// 往节点A，链接一个节点，并且返回之前节点A的后驱节点
func (r *Ring) Link(s *Ring) *Ring {
    n := r.Next()
    if s != nil {
        p := s.Prev()
        r.next = s
        s.prev = r
        n.prev = p
        p.next = n
    }
    return n
}
```

```go
// 删除节点后面的 n 个节点
func (r *Ring) Unlink(n int) *Ring {
    if n < 0 {
        return nil
    }
    return r.Link(r.Move(n + 1))
}
```



## 二、数组与链表

数组是编程语言作为一种基本类型提供出来的，相同数据类型的元素按一定顺序排列的集合。

它的作用只有一种：存放数据，让你很快能找到存的数据。如果你不去额外改进它，它就只是存放数据而已，它不会将一个数据节点和另外一个数据节点关联起来。

数组这一数据类型，是被编程语言高度抽象封装的结构，`下标` 会转换成 `虚拟内存地址`，然后操作系统会自动帮我们进行寻址，这个寻址过程是特别快的，所以往数组的某个下标取一个值和放一个值，时间复杂度都为 `O(1)`。

它是一种将 `虚拟内存地址` 和 `数据元素` 映射起来的内置语法结构，数据和数据之间是挨着，存放在一个连续的内存区域，每一个固定大小（8字节）的内存片段都有一个虚拟的地址编号。当然这个虚拟内存不是真正的内存，每个程序启动都会有一个虚拟内存空间来映射真正的内存

数组和链表是两个不同的概念。一个是编程语言提供的基本数据类型，表示一个连续的内存空间，可通过一个索引访问数据。另一个是我们定义的数据结构，通过一个数据节点，可以定位到另一个数据节点，不要求连续的内存空间。

数组的优点是占用空间小，查询快，直接使用索引就可以获取数据元素，缺点是移动和删除数据元素要大量移动空间。

链表的优点是移动和删除数据元素速度快，只要把相关的数据元素重新链接起来，但缺点是占用空间大，查找需要遍历。

很多其他的数据结构都由数组和链表配合实现的。

具体可在[Go语言基础——数组与切片](https://hypo.ltd/2021/11/24/go-yu-yan-ji-chu-shu-zu-yu-qie-pian/)了解详情

文章参考：
[数据结构和算法](https://goa.lenggirl.com/#/algorithm/link?id=_11%e5%88%9d%e5%a7%8b%e5%8c%96%e5%be%aa%e7%8e%af%e9%93%be%e8%a1%a8)
[菜鸟教程](https://www.runoob.com/go/go-arrays.html)