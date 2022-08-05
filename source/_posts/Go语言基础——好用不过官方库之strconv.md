---
title: Go语言基础——好用不过官方库之strconv
author: hypo
img: medias/featureimages/62.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-08-05 14:33:23
coverImg:
password:
summary: 好用不过官方库之strconv
categories: Golang
tags:
- 笔记
- Golang
- 基础
---
# 好用不过官方库之strconv

拒绝重复造轮子，看看那些好用的官方库的包，这里仅仅总结我自己觉得好用的库及应用点，具体包函数可查看[go官方库](https://studygolang.com/pkgdoc)

strconv包实现了基本数据类型和其字符串表示的相互转换。底层源码也简单，下面进行分析

## String And Bool

### func ParseBool

```go
func ParseBool(str string) (bool, error) {
	switch str {
	case "1", "t", "T", "true", "TRUE", "True":
		return true, nil
	case "0", "f", "F", "false", "FALSE", "False":
		return false, nil
	}
	return false, syntaxError("ParseBool", str)
}
```

返回字符串表示的bool值。它接受1、0、t、f、T、F、true、false、True、False、TRUE、FALSE；否则返回错误。

### func FormatBool

```go
func FormatBool(b bool) string {
	if b {
		return "true"
	}
	return "false"
}
```

根据b的值返回"true"或"false"。

### func AppendBool

```go
func AppendBool(dst []byte, b bool) []byte {
	if b {
		return append(dst, "true"...)
	}
	return append(dst, "false"...)
}
```

等价于append(dst, FormatBool(b)...)

## String And Integer

大多数会使用到的两个string与整数转换函数和简写函数

### func ParseInt

```
func ParseInt(s string, base int, bitSize int) (i int64, err error)
```

string to "int"返回字符串表示的整数值，接受正负号。

base指定进制（2到36），如果base为0，则会从字符串前置判断，"0x"是16进制，"0"是8进制，否则是10进制；

bitSize指定结果必须能无溢出赋值的整数类型，0、8、16、32、64 分别代表 int、int8、int16、int32、int64；返回的err是*NumErr类型的，如果语法有误，err.Error = ErrSyntax；如果结果超出类型范围err.Error = ErrRange。

要理解ParseInt函数就要理解ParseUInt函数，实际上就是内部判断是否有符号，再根据无符号数进行转换

### func Atoi

```
func Atoi(s string) (i int, err error)
```

Atoi是ParseInt(s, 10, 0)的简写。

### func FormatInt

```
func FormatInt(i int64, base int) string
```

"int" to string返回i的base进制的字符串表示。base 必须在2到36之间，结果中会使用小写字母'a'到'z'表示大于10的数字。

### func Itoa

```
func Itoa(i int) string
```

Itoa是FormatInt(i, 10) 的简写。



### Example

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	b, _ := strconv.ParseBool("1")
	fmt.Println("ParseBool=", b)
	i, _ := strconv.ParseInt("-123", 10, 0)
	fmt.Println("ParseInt=", i)
	ui, _ := strconv.ParseInt("123", 10, 0)
	fmt.Println("ParseInt=", ui)
	f, _ := strconv.ParseFloat("0.002343241", 64)
	fmt.Println("ParseFloat=", f)
	is := strconv.FormatUint(1313, 10)
	fmt.Println("FormatUint=", is)
	ii, _ := strconv.Atoi("12313")
	fmt.Println("Atoi=", ii)
	ss := strconv.Itoa(12312)
	fmt.Println("Itoa=", ss)
}
```

```bash
//输出：
ParseBool= true
ParseInt= -123
ParseInt= 123
ParseFloat= 0.002343241
FormatUint= 1313
Atoi= 12313
Itoa= 12312
```

