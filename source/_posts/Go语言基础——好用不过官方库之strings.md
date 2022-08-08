---
title: Go语言基础——好用不过官方库之strings
author: hypo
img: medias/featureimages/64.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-08-08 18:13:33
coverImg:
password:
summary: 好用不过官方库之strings
categories: Golang
tags:
- 笔记
- Golang
- 基础
---
# 好用不过官方库之strings

拒绝重复造轮子，看看那些好用的官方库的包，这里仅仅总结我自己觉得好用的库及应用点，具体包函数可查看[go官方库](https://studygolang.com/pkgdoc)

strings包实现了用于操作字符的简单函数。

### func EqualFold

func EqualFold(s, t string) bool

判断两个utf-8编码字符串（将unicode大写、小写、标题三种格式字符视为相同）是否相同。不论大小写，只判断字符。

```go
fmt.Println(strings.EqualFold("Go", "go")) //true
```

### func HasPrefix

```go
func HasPrefix(s, prefix string) bool {
    //字符串长度大于等于前缀且等于字符串前几位等于前缀
	return len(s) >= len(prefix) && s[0:len(prefix)] == prefix
}
```

判断s是否有前缀字符串prefix。

```go
fmt.Println(strings.HasPrefix("abcefg", "abc")) //true
fmt.Println(strings.HasPrefix("abcefg", "e")) //false
```

### func HasSuffix 

```go
func HasSuffix(s, suffix string) bool {
   //字符串长度大于等于后缀且等于字符串后几位等于后缀
   return len(s) >= len(suffix) && s[len(s)-len(suffix):] == suffix
}
```

判断s是否有后缀字符串suffix。

```go
fmt.Println(strings.HasPrefix("abcefg", "efg")) //true
fmt.Println(strings.HasPrefix("abcefg", "e")) //false
```

### func Index

```
func Index(s, sep string) int
```

子串sep在字符串s中第一次出现的位置，不存在则返回-1。

### func LastIndex

```go
func LastIndex(s, sep string) int
```

子串sep在字符串s中最后一次出现的位置，不存在则返回-1。

```go
fmt.Println(strings.Index("go gopher", "go")) //0
fmt.Println(strings.LastIndex("go gopher", "go")) //3
fmt.Println(strings.LastIndex("go gopher", "rodent")) //-1
```

### func Contains

```go
func Contains(s, substr string) bool {
    //即Index函数大于0则包含字串
	return Index(s, substr) >= 0
}
```


判断字符串s是否包含子串substr。

```go
fmt.Println(strings.Contains("seafood", "foo")) //true
```

