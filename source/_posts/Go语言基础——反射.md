---
title: Go语言基础——反射
author: 罗布派克
img: medias/featureimages/67.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-18 11:39:50
coverImg:
password:
summary: Go语言基础——反射
categories: Golang
tags:
- 转载
- Golang
- 基础
---
# 反射定律

罗布派克
2011 年 9 月 6 日

## 介绍

计算中的反射是程序检查其自身结构的能力，特别是通过类型。这是元编程的一种形式。这也是一个很大的混乱来源。

在本文中，我们试图通过解释反射在 Go 中的工作原理来澄清事情。每种语言的反射模型都不同（许多语言根本不支持），但这篇文章是关于 Go 的，所以对于本文的其余部分，“反射”这个词应该被理解为“Go 中的反射”。

2022 年 1 月添加的注释：这篇博文写于 2011 年，早于 Go 中的参数多态（又名泛型）。尽管由于语言的发展，文章中的任何重要内容都没有变得不正确，但它已经在一些地方进行了调整，以避免让熟悉现代 Go 的人感到困惑。

## 类型和接口

因为反射建立在类型系统之上，所以让我们从复习一下 Go 中的类型开始。

Go 是静态类型的。每个变量都有一个静态类型，也就是说，只有一种类型在编译时已知并固定： `int`、`float32`、`*MyType`、`[]byte`等等。如果我们声明

```
type MyInt int

var i int
var j MyInt
```

然后`i`有 type`int`并且`j`有 type `MyInt`。变量`i`和`j`具有不同的静态类型，尽管它们具有相同的基础类型，但它们不能在没有转换的情况下相互分配。

一类重要的类型是接口类型，它表示固定的方法集。（在讨论反射时，我们可以忽略将接口定义用作多态代码中的约束。）接口变量可以存储任何具体（非接口）值，只要该值实现接口的方法即可。一对著名的例子是`io.Reader`and `io.Writer`，类型`Reader`and`Writer`来自[io 包](https://go.dev/pkg/io/)：

```
// Reader is the interface that wraps the basic Read method.
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Writer is the interface that wraps the basic Write method.
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

任何使用此签名实现`Read`(or `Write`) 方法的类型都被称为实现`io.Reader`(or `io.Writer`)。出于本讨论的目的，这意味着类型变量 `io.Reader`可以保存其类型具有`Read`方法的任何值：

```
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
// and so on
```

重要的是要清楚，无论具体值`r`可能是什么， `r`'s 的类型始终是`io.Reader`：Go 是静态类型的，而静态类型`r`是`io.Reader`。

接口类型的一个极其重要的例子是空接口：

```
interface{}
```

或其等效别名，

```
any
```

它代表空的方法集，任何值都可以满足它，因为每个值都有零个或多个方法。

有人说 Go 的接口是动态类型的，但这是一种误导。它们是静态类型的：接口类型的变量始终具有相同的静态类型，即使在运行时存储在接口变量中的值可能会改变类型，该值也将始终满足接口。

我们需要对所有这些都保持精确，因为反射和接口密切相关。

## 接口的表示

Russ Cox 写了一篇 关于 Go 中接口值表示的[详细博客文章。](https://research.swtch.com/2009/12/go-data-structures-interfaces.html)这里没有必要重复完整的故事，但一个简化的摘要是有序的。

接口类型的变量存储一对：分配给变量的具体值，以及该值的类型描述符。更准确地说，值是实现接口的底层具体数据项，类型描述了该项的完整类型。例如，之后

```
var r io.Reader
tty, err := os.OpenFile("/dev/tty", os.O_RDWR, 0)
if err != nil {
    return nil, err
}
r = tty
```

`r`示意性地包含 (value, type) 对 ( `tty`, `*os.File`)。请注意，该类型`*os.File`实现的方法不是`Read`; 即使接口值只提供对`Read`方法的访问，内部的值也包含有关该值的所有类型信息。这就是为什么我们可以这样做：

```
var w io.Writer
w = r.(io.Writer)
```

这个赋值中的表达式是一个类型断言；它断言的是里面的项目`r`也实现了`io.Writer`，所以我们可以将它分配给`w`. 赋值后，`w`将包含对 ( `tty`, `*os.File`)。那是同一对在举行`r`。接口的静态类型决定了可以使用接口变量调用哪些方法，即使内部的具体值可能具有更大的方法集。

继续，我们可以这样做：

```
var empty interface{}
empty = w
```

并且我们的空接口值`empty`将再次包含同一对 ( `tty`, `*os.File`)。这很方便：一个空接口可以保存任何值，并包含我们可能需要的关于该值的所有信息。

（我们在这里不需要类型断言，因为静态已知它 `w`满足空接口。在我们将值从 a 移动`Reader`到 a的示例中`Writer`，我们需要显式并使用类型断言，因为`Writer`' 的方法不是的子集`Reader`。）

一个重要的细节是接口变量内的对总是有形式（值，具体类型），不能有形式（值，接口类型）。接口不保存接口值。

现在我们准备好反思了。

## 第一反射定律

## 1. 反射从接口值到反射对象。

在基本层面上，反射只是一种检查存储在接口变量中的类型和值对的机制。首先，我们需要了解[反射包](https://go.dev/pkg/reflect/)中的两种类型： [类型](https://go.dev/pkg/reflect/#Type)和[值](https://go.dev/pkg/reflect/#Value)。这两种类型可以访问接口变量的内容，以及两个简单的函数，称为`reflect.TypeOf`and `reflect.ValueOf`，检索`reflect.Type`和`reflect.Value`分片接口值。（另外，从 a`reflect.Value`到对应的 很容易`reflect.Type`，但现在让`Value`和`Type`概念分开。）

让我们从`TypeOf`：

```
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    fmt.Println("type:", reflect.TypeOf(x))
}
```

该程序打印

```
type: float64
```

您可能想知道接口在哪里，因为程序看起来像是将`float64`变量`x`而不是接口值传递给`reflect.TypeOf`. 但它就在那里；作为[godoc 报告](https://go.dev/pkg/reflect/#TypeOf)， 的签名`reflect.TypeOf`包括一个空接口：

```
// TypeOf returns the reflection Type of the value in the interface{}.
func TypeOf(i interface{}) Type
```

当我们调用时`reflect.TypeOf(x)`，`x`首先存储在一个空接口中，然后作为参数传递； `reflect.TypeOf`解压缩该空接口以恢复类型信息。

当然，该`reflect.ValueOf`函数会恢复值（从这里开始，我们将省略样板文件并只关注可执行代码）：

```
var x float64 = 3.4
fmt.Println("value:", reflect.ValueOf(x).String())
```

印刷

```
value: <float64 Value>
```

（我们显式调用该`String`方法，因为默认情况下，`fmt`包会深入到 a`reflect.Value`中以显示内部的具体值。该`String`方法没有。）

两者`reflect.Type`都有`reflect.Value`很多方法可以让我们检查和操作它们。一个重要的例子是它`Value`有一个`Type`返回 `Type`a的方法`reflect.Value`。另一个是两者`Type`都有`Value`一个`Kind`方法，该方法返回一个常量，指示存储的项目类型： `Uint`、`Float64`、`Slice`等。还`Value`使用名称 like 的方法`Int`，`Float`让我们获取存储在其中的值（as`int64`和`float64`）：

```
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())
fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
fmt.Println("value:", v.Float())
```

印刷

```
type: float64
kind is float64: true
value: 3.4
```

也有类似的方法`SetInt`，`SetFloat`但要使用它们，我们需要了解可设置性，即反射第三定律的主题，将在下面讨论。

反射库有几个值得一提的属性。首先，为了保持 API 简单，“getter”和“setter”方法对`Value` 可以保存值的最大类型进行操作： `int64`例如，对于所有有符号整数。即 的`Int`方法`Value`返回一个`int64`，`SetInt` 值取一个`int64`; 可能需要转换为实际涉及的类型：

```
var x uint8 = 'x'
v := reflect.ValueOf(x)
fmt.Println("type:", v.Type())                            // uint8.
fmt.Println("kind is uint8: ", v.Kind() == reflect.Uint8) // true.
x = uint8(v.Uint())                                       // v.Uint returns a uint64.
```

第二个属性是`Kind`反射对象描述的是底层类型，而不是静态类型。如果反射对象包含用户定义的整数类型的值，如

```
type MyInt int
var x MyInt = 7
v := reflect.ValueOf(x)
Kind`of`v`仍然是，`reflect.Int`即使静态类型`x`是`MyInt`，不是`int`。换句话说，即使可以，`Kind`也无法区分 an`int`和 a 。`MyInt``Type
```

## 反射第二定律

## 2. 反射从反射对象到接口值。

像物理反射一样，Go 中的反射产生了它自己的逆。

给定 a`reflect.Value`我们可以使用该方法恢复接口值`Interface`；实际上，该方法将类型和值信息打包回接口表示并返回结果：

```
// Interface returns v's value as an interface{}.
func (v Value) Interface() interface{}
```

结果我们可以说

```
y := v.Interface().(float64) // y will have type float64.
fmt.Println(y)
```

打印`float64`反射对象表示的值`v`。

不过，我们可以做得更好。的参数`fmt.Println`等等 `fmt.Printf`都是作为空接口值传递的，然后`fmt`就像我们在前面的示例中所做的那样，它们在内部由包解包。因此，正确打印 a 的内容所需要做的`reflect.Value`就是将方法的结果传递`Interface`给格式化的打印例程：

```
fmt.Println(v.Interface())
```

（因为这篇文章是第一次写的，所以对`fmt` 包做了一个改变，让它自动解包这样的`reflect.Value`，所以我们可以说

```
fmt.Println(v)
```

得到相同的结果，但为了清楚起见，我们将在`.Interface()`此处保留调用。）

由于我们的值是 a `float64`，我们甚至可以根据需要使用浮点格式：

```
fmt.Printf("value is %7.1e\n", v.Interface())
```

在这种情况下得到

```
3.4e+00
```

同样，无需对 to 的结果进行类型`v.Interface()`断言`float64`；空接口值里面有具体值的类型信息，`Printf`会恢复。

简而言之，该`Interface`方法是函数的逆`ValueOf`函数，只是它的结果始终是静态类型`interface{}`。

重申：反射从接口值到反射对象再返回。

## 反射第三定律

## 3.要修改反射对象，其值必须是可设置的。

第三定律是最微妙和最令人困惑的，但是如果我们从第一原理开始就很容易理解。

这是一些不起作用的代码，但值得研究。

```
var x float64 = 3.4
v := reflect.ValueOf(x)
v.SetFloat(7.1) // Error: will panic.
```

如果你运行这段代码，它会因为神秘的消息而恐慌

```
panic: reflect.Value.SetFloat using unaddressable value
```

问题不在于该值`7.1`不可寻址；这`v`是不可设置的。可设置性是反射的属性`Value`，并非所有反射`Values`都有。

报告a的可设置性的`CanSet`方法；在我们的例子中，`Value``Value`

```
var x float64 = 3.4
v := reflect.ValueOf(x)
fmt.Println("settability of v:", v.CanSet())
```

印刷

```
settability of v: false
```

`Set`在 non-settable 上调用方法是错误的`Value`。但什么是可设置性？

可设置性有点像可寻址性，但更严格。这是反射对象可以修改用于创建反射对象的实际存储的属性。可设置性取决于反射对象是否持有原始项目。当我们说

```
var x float64 = 3.4
v := reflect.ValueOf(x)
```

我们传递`x`to的副本`reflect.ValueOf`，因此作为 to 参数创建的接口值`reflect.ValueOf`是 的副本`x`，而不是`x`自身。因此，如果语句

```
v.SetFloat(7.1)
```

被允许成功，它不会更新`x`，即使它`v`看起来像是从`x`. `x`相反，它会更新存储在反射值中的副本，并且`x`它本身不会受到影响。那会令人困惑和无用，因此它是非法的，而可设置性是用于避免此问题的属性。

如果这看起来很奇怪，其实不然。这实际上是一个穿着不寻常服装的熟悉情况。考虑传递`x`给一个函数：

```
f(x)
```

我们不希望`f`能够修改，因为我们传递了' 值`x`的副本，而不是它本身。如果我们想直接修改，我们必须将地址传递给我们的函数（即指向 的指针）：`x``x``f``x``x``x`

```
f(&x)
```

这是直截了当和熟悉的，反射也以同样的方式工作。如果我们想`x`通过反射进行修改，我们必须给反射库一个指向我们要修改的值的指针。

让我们这样做。首先我们像往常一样初始化`x`，然后创建一个指向它的反射值，称为`p`.

```
var x float64 = 3.4
p := reflect.ValueOf(&x) // Note: take the address of x.
fmt.Println("type of p:", p.Type())
fmt.Println("settability of p:", p.CanSet())
```

到目前为止的输出是

```
type of p: *float64
settability of p: false
```

反射对象`p`是不可设置的，但它不是`p`我们想要设置的，它是 (in effect) `*p`。为了得到指向什么`p`，我们调用 的`Elem`方法`Value`，该方法通过指针间接传递，并将结果保存在`Value`名为的反射中`v`：

```
v := p.Elem()
fmt.Println("settability of v:", v.CanSet())
```

现在`v`是一个可设置的反射对象，如输出所示，

```
settability of v: true
```

由于它代表`x`，我们终于可以用来`v.SetFloat`修改 的值`x`：

```
v.SetFloat(7.1)
fmt.Println(v.Interface())
fmt.Println(x)
```

正如预期的那样，输出是

```
7.1
7.1
```

反射可能很难理解，但它确实在做语言所做的事情，尽管是通过反射`Types`来`Values`掩盖正在发生的事情。请记住，反射值需要某些东西的地址才能修改它们所代表的内容。

## 结构

在我们之前的示例`v`中，它本身并不是一个指针，它只是从一个指针派生而来。出现这种情况的常见方法是使用反射来修改结构的字段。只要我们有结构的地址，我们就可以修改它的字段。

这是一个分析结构值的简单示例，`t`. 我们使用结构的地址创建反射对象，因为我们稍后会修改它。然后我们设置`typeOfT`它的类型并使用简单的方法调用迭代字段（有关详细信息，请参阅[包反映](https://go.dev/pkg/reflect/)）。请注意，我们从结构类型中提取字段的名称，但字段本身是常规`reflect.Value`对象。

```
type T struct {
    A int
    B string
}
t := T{23, "skidoo"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i := 0; i < s.NumField(); i++ {
    f := s.Field(i)
    fmt.Printf("%d: %s %s = %v\n", i,
        typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```

这个程序的输出是

```
0: A int = 23
1: B string = skidoo
```

这里顺便介绍了一点关于可设置性： 的字段名称`T`是大写的（导出的），因为只有结构的导出字段是可设置的。

因为`s`包含一个可设置的反射对象，所以我们可以修改结构体的字段。

```
s.Field(0).SetInt(77)
s.Field(1).SetString("Sunset Strip")
fmt.Println("t is now", t)
```

结果如下：

```
t is now {77 Sunset Strip}
```

如果我们修改程序以便`s`从`t`, not中创建，则对和`&t`的调用将失败，因为 的字段不可设置。`SetInt``SetString``t`

## 结论

这里又是反射定律：

- 反射从接口值到反射对象。
- 反射从反射对象到接口值。
- 要修改反射对象，该值必须是可设置的。

一旦你理解了这些定律，Go 中的反射就会变得更容易使用，尽管它仍然很微妙。这是一个强大的工具，除非绝对必要，否则应谨慎使用并避免使用。

还有很多我们没有涉及到的反射——在通道上发送和接收、分配内存、使用切片和映射、调用方法和函数——但是这篇文章已经足够长了。我们将在以后的文章中介绍其中的一些主题。

转载自：[https://go.dev/blog/laws-of-reflection](https://go.dev/blog/laws-of-reflection),转载请注