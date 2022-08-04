---
title: Go语言基础——好用不过官方库之os\signal
author: hypo
img: medias/featureimages/61.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-08-04 11:39:26
coverImg:
password:
summary: 好用不过官方库之os\signal
categories: Golang
tags:
- 笔记
- Golang
- 基础
---
# 好用不过官方库之os\signal

拒绝重复造轮子，看看那些好用的官方库的包，这里仅仅总结我自己觉得好用的库及应用点，具体包函数可查看[go官方库](https://studygolang.com/pkgdoc)

signal包实现了对输入信号的访问。

### func Notify

```go
func Notify(c chan<- os.Signal, sig ...os.Signal)
```

Notify函数让signal包将输入信号转发到c。如果没有列出要传递的信号，会将所有输入信号传递到c；否则只传递列出的输入信号。

signal包不会为了向c发送信息而阻塞（就是说如果发送时c阻塞了，signal包会直接放弃）：调用者应该保证c有足够的缓存空间可以跟上期望的信号频率。对使用单一信号用于通知的通道，缓存为1就足够了。

可以使用同一通道多次调用Notify：每一次都会扩展该通道接收的信号集。唯一从信号集去除信号的方法是调用Stop。可以使用同一信号和不同通道多次调用Notify：每一个通道都会独立接收到该信号的一个拷贝。

#### Example

##### func (*Server) Shutdown

```go
func (srv * Server ) Shutdown(ctx context . Context ) error
```

Shutdown 优雅地关闭服务器而不中断任何活动连接。关闭首先关闭所有打开的侦听器，然后关闭所有空闲连接，然后无限期地等待连接返回空闲状态，然后关闭。如果提供的上下文在关闭完成之前过期，则 Shutdown 返回上下文的错误，否则返回关闭服务器的底层侦听器返回的任何错误。

当调用 Shutdown 时，Serve、ListenAndServe 和 ListenAndServeTLS 立即返回 ErrServerClosed。确保程序不会退出，而是等待 Shutdown 返回。

Shutdown 不会尝试关闭或等待被劫持的连接，例如 WebSockets。Shutdown 的调用者应单独通知此类长期连接的关闭并等待它们关闭（如果需要）。有关注册关闭通知功能的方法，请参阅 RegisterOnShutdown。

一旦在服务器上调用了 Shutdown，它就不能被重用；将来对 Serve 等方法的调用将返回 ErrServerClosed。

```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
)

func main() {
    srv := &http.Server{
        Addr: "",
        Handler: Handler,
    }

	idleConnsClosed := make(chan struct{})
	go func() {
		sigint := make(chan os.Signal, 1)
		signal.Notify(sigint, os.Interrupt)
		<-sigint

		// We received an interrupt signal, shut down.
		if err := srv.Shutdown(context.Background()); err != nil {
			// Error from closing listeners, or context timeout:
			log.Printf("HTTP server Shutdown: %v", err)
		}
		close(idleConnsClosed)
	}()

	if err := srv.ListenAndServe(); err != http.ErrServerClosed {
		// Error starting or closing listener:
		log.Fatalf("HTTP server ListenAndServe: %v", err)
	}

	<-idleConnsClosed
}
```

更多例子也可看[李文周的博客-优雅地关机或重启](https://www.liwenzhou.com/posts/Go/graceful_shutdown/)

### func Stop

```go
func Stop(c chan<- os.Signal)
```

Stop函数让signal包停止向c转发信号。它会取消之前使用c调用的所有Notify的效果。当Stop返回后，会保证c不再接收到任何信号。

