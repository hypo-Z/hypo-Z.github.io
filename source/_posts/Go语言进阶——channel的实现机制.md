---
title: Go语言进阶——channel的实现机制
author: hypo
img: medias/featureimages/68.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-10-25 12:16:53
coverImg:
password:
summary: 本章从源码角度分析channel的实现机制
categories: Golang
tags:
- Golang
- 进阶
- 笔记
---
# channel的实现机制

> channel是Golang在语言层面提供的goroutine间的通信方式，比Unix管道更易用也更轻便。channel主要用于进程内各goroutine间通信，如果需要跨进程通信，建议使用分布式系统的方法来解决。

本章从源码角度分析channel的实现机制

## 数据结构

`src/runtime/chan.go:hchan`定义了channel的数据结构：

```go
type hchan struct {
    qcount   uint           // 当前队列中剩余元素个数
    dataqsiz uint           // 环形队列长度，即可以存放的元素个数
    buf      unsafe.Pointer // 环形队列指针
    elemsize uint16         // 每个元素的大小
    closed   uint32            // 标识关闭状态
    elemtype *_type         // 元素类型
    sendx    uint           // 队列下标，指示元素写入时存放到队列中的位置
    recvx    uint           // 队列下标，指示元素从队列的该位置读出
    recvq    waitq          // 等待读消息的goroutine队列
    sendq    waitq          // 等待写消息的goroutine队列
    lock mutex              // 互斥锁，chan不允许并发读写
}
```

## 环形队列

chan内部实现了一个环形队列作为其缓冲区，队列的长度是创建chan时指定的。

```go
ch:=make(chan int,6)
//队列可看成定长数组队列arrry:=[6]int
//dataqsiz指示了队列长度为7，即可缓存7个元素；
//buf指向队列的内存，buf:=&array ,队列中还剩余元素；
//qcount表示队列中还有元素个数len(array)；
//sendx指示后续写入的数据存储的位置，取值[0, 6]；
//recvx指示从该位置读取数据, 取值[0, 6]；
```

## 等待队列

从channel读数据，如果channel缓冲区为空或者没有缓冲区，当前goroutine会被阻塞。
向channel写数据，如果channel缓冲区已满或者没有缓冲区，当前goroutine会被阻塞。

被阻塞的goroutine将会挂在channel的等待队列中：

- 因读阻塞的goroutine会被向channel写入数据的goroutine唤醒；

- 因写阻塞的goroutine会被从channel读数据的goroutine唤醒；

注意，一般情况下recvq和sendq至少有一个为空。只有一个例外，那就是同一个goroutine使用select语句向channel一边写数据，一边读数据。

```go
package main

import (
	"fmt"
	"time"
)

//循环添加0-9数字至channel中，并睡眠1秒
func addNumberToChan(chanName chan int) {
	for {
		for i := 0; i < 10; i++ {
			chanName <- i
			time.Sleep(1 * time.Second)
		}
	}
}

func main() {
    //创建一个无缓存的channel
	var chan1 = make(chan int)
    //延时关闭channel，养成好习惯
    defer close(chan1)
    //一个协程负责添加数值
	go addNumberToChan(chan1)
	for {
		select {
            //若读channel不阻塞，打印结果
		case e := <-chan1:
			fmt.Printf("Get element from chan1: %d\n", e)
            //channel阻塞，提醒并睡眠1秒
		default:
			fmt.Printf("No element in chan1.\n")
			time.Sleep(1 * time.Second)
		}
	}
}
```

打印结果：

```go
No element in chan1.
Get element from chan1: 0
No element in chan1.
Get element from chan1: 1
No element in chan1.
Get element from chan1: 2
No element in chan1.
Get element from chan1: 3
No element in chan1.
Get element from chan1: 4
No element in chan1.
Get element from chan1: 5
No element in chan1.
Get element from chan1: 6
No element in chan1.
Get element from chan1: 7
No element in chan1.
Get element from chan1: 8
No element in chan1.
Get element from chan1: 9
No element in chan1.
...
```

运行稳定之后会顺序输出排列数字

## 锁lock mutex

一个channel同时仅允许被一个goroutine读写

## 关闭channel

panic出现的常见场景还有：

1. 关闭值为nil的channel
2. 关闭已经被关闭的channel
3. 向已经关闭的channel写数据

## range

通过range可以持续从channel中读出数据，好像在遍历一个数组一样，当channel中没有数据时会阻塞当前goroutine，与读channel时阻塞处理机制一样。

```go
package main

import (
	"fmt"
	"time"
)

func chanRange(chanName chan int) {
	for e := range chanName {
		fmt.Printf("Get element from chan: %d\n", e)
	}
}

func addNumberToChan1(chanName chan int) {
	for {
		for i := 0; i < 10; i++ {
			chanName <- i
		}
		time.Sleep(5 * time.Second)
	}
}

func main() {
	chan2 := make(chan int, 9)
	defer close(chan2)
	go addNumberToChan1(chan2)
	chanRange(chan2)
}
```

打印结果：

```go
Get element from chan: 0
Get element from chan: 1
Get element from chan: 2
Get element from chan: 3
Get element from chan: 4
Get element from chan: 5
Get element from chan: 6
Get element from chan: 7
Get element from chan: 8
Get element from chan: 9
//每隔5秒
Get element from chan: 0
Get element from chan: 1
Get element from chan: 2
Get element from chan: 3
Get element from chan: 4
Get element from chan: 5
Get element from chan: 6
Get element from chan: 7
Get element from chan: 8
Get element from chan: 9
...
```

参考文章：[Go专家编程](https://topgoer.cn/docs/gozhuanjia/gochan4)