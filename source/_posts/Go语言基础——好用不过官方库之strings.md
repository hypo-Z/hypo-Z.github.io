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

## strings 判断

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

```go
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

## strings 处理

### func ToLower

```go
func ToLower(s string) string
```

返回将所有字母都转为对应的小写版本的拷贝。

### func ToUpper

```go
func ToUpper(s string) string
```

返回将所有字母都转为对应的大写版本的拷贝。

### func Repeat

```go
func Repeat(s string, count int) string
```

返回count个s串联的字符串。

### func Replace

```go
func Replace(s, old, new string, n int) string
```

返回将s中前n个不重叠old子串都替换为new的新字符串，如果n<0会替换所有old子串。

```go
fmt.Println(strings.ToLower("Gopher")) //gopher
fmt.Println(strings.ToUpper("Gopher")) //GOPHER
fmt.Println("ba" + strings.Repeat("na", 2)) //banana
fmt.Println(strings.Replace("oink oink oink", "k", "ky", 2)) //oinky oinky oink
fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -1)) //moo moo moo
```

### func TrimPrefix

```go
func TrimPrefix(s, prefix string) string {
	if HasPrefix(s, prefix) {
		return s[len(prefix):]
	}
	return s
}
```

返回去除s可能的前缀prefix的字符串。

### func TrimSuffix

```go
func TrimSuffix(s, suffix string) string {
	if HasSuffix(s, suffix) {
		return s[:len(s)-len(suffix)]
	}
	return s
}
```

返回去除s可能的后缀suffix的字符串。

## strings 分割

### func Fields

```
func Fields(s string) []string
```

返回将字符串按照空白（unicode.IsSpace确定，可以是一到多个连续的空白字符）分割的多个字符串。如果字符串全部是空白或者是空字符串的话，会返回空切片。

###  func Split

```
func Split(s, sep string) []string
```

用去掉s中出现的sep的方式进行分割，会分割到结尾，并返回生成的所有片段组成的切片（每一个sep都会进行一次切割，即使两个sep相邻，也会进行两次切割）。如果sep为空字符，Split会将s切分成每一个unicode码值一个字符串。

```go
fmt.Printf("%q\n", strings.Fields("  foo bar  baz   ")) 				//["foo" "bar" "baz"]
fmt.Printf("%q\n", strings.Split("a,b,c", ",")) 						//["a" "b" "c"]
fmt.Printf("%q\n", strings.Split("a man a plan a canal panama", "a "))  //["" "man " "plan " "canal panama"]
fmt.Printf("%q\n", strings.Split(" xyz ", "")) 							//[" " "x" "y" "z" " "]
fmt.Printf("%q\n", strings.Split("", "Bernardo O'Higgins")) 			//[""]
```

## strings 合并

### func Join

```
func Join(a []string, sep string) string
```

将一系列字符串连接为一个字符串，之间用sep来分隔。

```go
s := []string{"foo", "bar", "baz"}
fmt.Println(strings.Join(s, ", "))
//foo, bar, baz
```