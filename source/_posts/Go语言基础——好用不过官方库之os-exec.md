---
title: Go语言基础——好用不过官方库之os\exec
author: hypo
img: medias/featureimages/60.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-08-03 12:00:28
coverImg:
password:
summary: 好用不过官方库之os\exec
categories: Golang
tags:
- 笔记
- Golang
- 基础
---
# 好用不过官方库之os/exec

拒绝重复造轮子，看看那些好用的官方库的包，这里仅仅总结我自己觉得好用的库及应用点，具体包函数可查看[go官方库](https://studygolang.com/pkgdoc)

exec包执行外部命令。它包装了os.StartProcess函数以便更容易的修正输入和输出，使用管道连接I/O，以及作其它的一些调整。

### type Cmd

```go
type Cmd struct {
    // Path是将要执行的命令的路径。
    //
    // 该字段不能为空，如为相对路径会相对于Dir字段。
    Path string
    // Args保管命令的参数，包括命令名作为第一个参数；如果为空切片或者nil，相当于无参数命令。
    //
    // 典型用法下，Path和Args都应被Command函数设定。
    Args []string
    // Env指定进程的环境，如为nil，则是在当前进程的环境下执行。
    Env []string
    // Dir指定命令的工作目录。如为空字符串，会在调用者的进程当前目录下执行。
    Dir string
    // Stdin指定进程的标准输入，如为nil，进程会从空设备读取（os.DevNull）
    Stdin io.Reader
    // Stdout和Stderr指定进程的标准输出和标准错误输出。
    //
    // 如果任一个为nil，Run方法会将对应的文件描述符关联到空设备（os.DevNull）
    //
    // 如果两个字段相同，同一时间最多有一个线程可以写入。
    Stdout io.Writer
    Stderr io.Writer
    // ExtraFiles指定额外被新进程继承的已打开文件流，不包括标准输入、标准输出、标准错误输出。
    // 如果本字段非nil，entry i会变成文件描述符3+i。
    //
    // BUG: 在OS X 10.6系统中，子进程可能会继承不期望的文件描述符。
    // http://golang.org/issue/2603
    ExtraFiles []*os.File
    // SysProcAttr保管可选的、各操作系统特定的sys执行属性。
    // Run方法会将它作为os.ProcAttr的Sys字段传递给os.StartProcess函数。
    SysProcAttr *syscall.SysProcAttr
    // Process是底层的，只执行一次的进程。
    Process *os.Process
    // ProcessState包含一个已经存在的进程的信息，只有在调用Wait或Run后才可用。
    ProcessState *os.ProcessState
    // 内含隐藏或非导出字段
}
```

Cmd代表一个正在准备或者在执行中的外部命令。

### func Command

```go
func Command(name string, arg ...string) *Cmd
```

函数返回一个*Cmd，用于使用给出的参数执行name指定的程序。返回值只设定了Path和Args两个参数。

如果name不含路径分隔符，将使用LookPath获取完整路径；否则直接使用name。参数arg不应包含命令名。

```go
//重开一个新的端口运行命令
func main(){
    com := fmt.Sprintf("some-command")
	cmd := exec.Command("/bin/bash", "-c", com)
	cmd.Stdout = os.Stdout //结果输出至控制台
	cmd.Stderr = os.Stderr //错误输出至控制台
	err = cmd.Run()
	if err != nil {
		log.Fatalf("failed to call cmd.Run(): %v", err)
	} 
}
	
```

#### func (*Cmd) Run

```
func (c *Cmd) Run() error
```

Run执行c包含的命令，并阻塞直到完成。

如果命令成功执行，stdin、stdout、stderr的转交没有问题，并且返回状态码为0，方法的返回值为nil；如果命令没有执行或者执行失败，会返回*ExitError类型的错误；否则返回的error可能是表示I/O问题。

#### func (*Cmd) Start

```
func (c *Cmd) Start() error
```

Start开始执行c包含的命令，但并不会等待该命令完成即返回。Wait方法会返回命令的返回状态码并在命令返回后释放相关的资源。

#### func (*Cmd) Wait

```
func (c *Cmd) Wait() error
```

Wait会阻塞直到该命令执行完成，该命令必须是被Start方法开始执行的。

如果命令成功执行，stdin、stdout、stderr的转交没有问题，并且返回状态码为0，方法的返回值为nil；如果命令没有执行或者执行失败，会返回*ExitError类型的错误；否则返回的error可能是表示I/O问题。Wait方法会在命令返回后释放相关的资源。

#### func (*Cmd) Output

```
func (c *Cmd) Output() ([]byte, error)
```

执行命令并返回标准输出的切片。

Example

```
out, err := exec.Command("date").Output()
if err != nil {
    log.Fatal(err)
}
fmt.Printf("The date is %s\n", out)
```

#### func (*Cmd) CombinedOutput

```
func (c *Cmd) CombinedOutput() ([]byte, error)
```

执行命令并返回标准输出和错误输出合并的切片。

















