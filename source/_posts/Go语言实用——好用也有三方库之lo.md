---
title: Go语言实用——好用也有三方库之lo
author: hypo
img: medias/featureimages/77.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2024-01-10 14:18:41
coverImg:
password:
summary: 好用也有三方库之lo
categories: 笔记
tags:
- 笔记
- Golang
- 实用
---
# Go语言轮子-好用也有三方库之lo

在编程中，避免重复造轮子是一种普遍原则。我曾经写过一些关于学习官方标准库的笔记，而这篇文章将介绍一些我发现的，以及别人推荐的实用的第三方库。这也是我学习过程中的一些笔记。

库的地址是：https://github.com/samber/lo

原文的介绍是这样的：

> lo - 迭代切片、地图、通道...
>
> ✨samber/lo 是一个基于 Go 1.18+ 泛型的 Lodash 风格的 Go 库。
>
> 该项目最初是作为新泛型实施的实验而开始的。在某些方面，它可能看起来像 Lodash。我曾经使用过优秀的“go-funk”包进行编码，但是“go-funk”使用反射，因此不是类型安全的。
>
> 正如预期的那样，基准测试表明，泛型比基于“reflect”包的实现速度快得多。与纯 for 循环相比，基准测试也显示出类似的性能提升。见下文。

这个库完全基于官方标准库，没有依赖其他第三方库，因此在安全性上更有保障。在自己的项目中可以直接使用，它还提供了许多有价值的抽象。

下面我将提供一些库中的使用例子。

### 过滤

迭代集合并返回谓词函数返回 `true` 的所有元素的数组。

```go
even := lo.Filter([]int{1, 2, 3, 4}, func(x int, index int) bool {
    return x%2 == 0
})
// []int{2, 4}
```

### 类型映射

操作一种类型的切片并将其转换为另一种类型的切片：

```go
import "github.com/samber/lo"

lo.Map([]int64{1, 2, 3, 4}, func(x int64, index int) string {
    return strconv.FormatInt(x, 10)
})
// []string{"1", "2", "3", "4"}
```

### 去重

返回数组的无重复版本，其中只保留每个元素的第一次出现。结果值的顺序由它们在数组中的出现顺序决定。

```go
uniqValues := lo.Uniq([]int{1, 2, 2, 1})
// []int{1, 2}
```

### 条件去重

返回数组的无重复版本，其中只保留每个元素的第一次出现。结果值的顺序由它们在数组中的出现顺序决定。它接受 `iteratee` 并为数组中的每个元素调用它，以生成计算唯一性的标准。

```go
uniqValues := lo.UniqBy([]int{0, 1, 2, 3, 4, 5}, func(i int) int {
    return i%3
})
// []int{0, 1, 2}
```

### 通过...分组

返回一个对象，该对象由通过 iteratee 运行集合中每个元素的结果生成的键组成。

```go
groups := lo.GroupBy([]int{0, 1, 2, 3, 4, 5}, func(i int) int {
    return i%3
})
// map[int][]int{0: []int{0, 3}, 1: []int{1, 4}, 2: []int{2, 5}}
```

### 展平

返回一个单层深度的数组。

```go
flat := lo.Flatten([][]int{{0, 1}, {2, 3, 4, 5}})
// []int{0, 1, 2, 3, 4, 5}
```

### 打乱

返回打乱值的数组。使用 Fisher-Yates 洗牌算法。

```go
randomOrder := lo.Shuffle([]int{0, 1, 2, 3, 4, 5})
// []int{1, 4, 0, 3, 5, 2}
```

### 反转

反转数组，使第一个元素成为最后一个元素，第二个元素成为倒数第二个元素，依此类推。

⚠️这个助手是**可变的**。此行为可能会在`v2.0.0`. 参见[#160](https://github.com/samber/lo/issues/160)。

```
reverseOrder := lo.Reverse([]int{0, 1, 2, 3, 4, 5})
// []int{5, 4, 3, 2, 1, 0}
```

### 拒绝

与 Filter 相反，此方法返回谓词不返回 true 的集合元素。

```go
odd := lo.Reject([]int{1, 2, 3, 4}, func(x int, _ int) bool {
    return x%2 == 0
})
// []int{1, 3}
```

### 按键挑选

返回按给定键过滤的相同地图类型。

```go
m := lo.PickByKeys(map[string]int{"foo": 1, "bar": 2, "baz": 3}, []string{"foo", "baz"})
// map[string]int{"foo": 1, "baz": 3}
```

### 按值挑选

返回按给定值过滤的相同地图类型。

```go
m := lo.PickByValues(map[string]int{"foo": 1, "bar": 2, "baz": 3}, []int{1, 3})
// map[string]int{"foo": 1, "baz": 3}
```

### 范围/范围从/范围与步骤

创建从开始到结束（但不包括结束）的数字（正数和/或负数）数组。

```go
result := lo.Range(4)
// [0, 1, 2, 3]

result := lo.Range(-4)
// [0, -1, -2, -3]

result := lo.RangeFrom(1, 5)
// [1, 2, 3, 4, 5]

result := lo.RangeFrom[float64](1.0, 5)
// [1.0, 2.0, 3.0, 4.0, 5.0]

result := lo.RangeWithSteps(0, 20, 5)
// [0, 5, 10, 15]

result := lo.RangeWithSteps[float32](-1.0, -4.0, -1.0)
// [-1.0, -2.0, -3.0]

result := lo.RangeWithSteps(1, 4, -1)
// []

result := lo.Range(0)
// []
```

### 补集

返回不包括所有给定值的切片。

```go
subset := lo.Without([]int{0, 2, 10}, 2)
// []int{0, 10}

subset := lo.Without([]int{0, 2, 10}, 0, 1, 2, 3, 4, 5)
// []int{10}
```

更多的例子可以在库的地址下查看。

其中，函数的数组参数其实是一个泛型，实际上你可以是任何类型：结构体、map、string、flote64...

```go
func Filter[V any](collection []V, predicate func(item V, index int) bool) []V {
	result := make([]V, 0, len(collection))

	for i, item := range collection {
		if predicate(item, i) {
			result = append(result, item)
		}
	}

	return result
}
```

来看 filter 的源码：传入的参数 `collection []V` 的 `v` 是 `any` 类型即泛型，先是初始化的一个切片 `result := make([]V, 0, len(collection))`，然后在对输入参数进行遍历，在对对象进行条件筛选，返回新的切片。

实际上是很简单方法调用封装了一下。

对于数组、map、string等调用逻辑很频繁的可以尝试使用这个库，也顺便给一个 star，反正我是点了，更多源码可以在使用过程中慢慢发现，共学习共进步，good code！