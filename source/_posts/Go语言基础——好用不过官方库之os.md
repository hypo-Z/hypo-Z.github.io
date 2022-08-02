---
title: Go语言基础——好用不过官方库之os
author: hypo
img: medias/featureimages/59.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-08-02 17:45:53
coverImg:
password:
summary: 好用不过官方库之os
categories: Golang
tags:
- 笔记
- Golang
- 基础
---
# 好用不过官方库之os

拒绝重复造轮子，看看那些好用的官方库的包，这里仅仅总结我自己觉得好用的库及应用点，具体包函数可查看[go官方库](https://studygolang.com/pkgdoc)

## os包

os包提供了操作系统函数的不依赖平台的接口。设计为Unix风格的，虽然错误处理是go风格的；失败的调用会返回错误值而非错误码。通常错误值里包含更多信息。

### 常数

```
const (
    O_RDONLY int = syscall.O_RDONLY // 只读模式打开文件
    O_WRONLY int = syscall.O_WRONLY // 只写模式打开文件
    O_RDWR   int = syscall.O_RDWR   // 读写模式打开文件
    O_APPEND int = syscall.O_APPEND // 写操作时将数据附加到文件尾部
    O_CREATE int = syscall.O_CREAT  // 如果不存在将创建一个新文件
    O_EXCL   int = syscall.O_EXCL   // 和O_CREATE配合使用，文件必须不存在
    O_SYNC   int = syscall.O_SYNC   // 打开文件用于同步I/O
    O_TRUNC  int = syscall.O_TRUNC  // 如果可能，打开时清空文件
)
```

用于包装底层系统的参数用于Open函数，不是所有的flag都能在特定系统里使用的。

```
const (
    SEEK_SET int = 0 // 相对于文件起始位置seek
    SEEK_CUR int = 1 // 相对于文件当前位置seek
    SEEK_END int = 2 // 相对于文件结尾位置seek
)
```

指定Seek函数从何处开始搜索（即相对位置）

```
const (
    PathSeparator     = '/' // 操作系统指定的路径分隔符
    PathListSeparator = ':' // 操作系统指定的表分隔符
)
const DevNull = "/dev/null"
```

DevNull是操作系统空设备的名字。在类似Unix的操作系统中，是"/dev/null"；在Windows中，为"NUL"。

### 变量

```
var (
    ErrInvalid    = errors.New("invalid argument")
    ErrPermission = errors.New("permission denied")
    ErrExist      = errors.New("file already exists")
    ErrNotExist   = errors.New("file does not exist")
)
```

一些可移植的、共有的系统调用错误。

```
var (
    Stdin  = NewFile(uintptr(syscall.Stdin), "/dev/stdin")
    Stdout = NewFile(uintptr(syscall.Stdout), "/dev/stdout")
    Stderr = NewFile(uintptr(syscall.Stderr), "/dev/stderr")
)
```

Stdin、Stdout和Stderr是指向标准输入、标准输出、标准错误输出的文件描述符。

```
var Args []string
```

### os.Args

Args保管了命令行参数，第一个元素是程序名。如果你只是简单的想要获取命令行参数，可使用以下函数：

```go
//os.Args demo
func main() {
	//os.Args是一个[]string
	if len(os.Args) > 0 {
		for index, arg := range os.Args {
			fmt.Printf("args[%d]=%v\n", index, arg)
		}
	}
}
```

将上面的代码执行`go build -o "args_demo"`编译之后，执行：

```bash
$ ./args_demo a b c d
args[0]=./args_demo
args[1]=a
args[2]=b
args[3]=c
args[4]=d
```

### 文件操作

```go
//文件信息接口
type FileInfo interface {
    Name() string
    Size() int64
    Mode() FileMode
    ModTime() time.Time
    IsDir() bool
    Sys() any
}
```

```go
func main() {
    //func Lstat(name string) (fi FileInfo, err error)
	//Lstat返回描述命名文件的FileInfo，如果指定的文件对象是一个符号链接，返回的FileInfo描述该符号链接的信息，本函数不会试图跳转该链接。若是stat则会跳转。
	fi, err := os.Lstat("some-file")
	if err != nil {
		log.Fatal(err)
	}
    //fi.Mode()是一个接口
	fmt.Printf("permissions: %#o\n", fi.Mode().Perm()) // 0400, 0777, etc.
	switch mode := fi.Mode(); {
	case mode.IsRegular():
		fmt.Println("regular file")
	case mode.IsDir():
		fmt.Println("directory")
	case mode&fs.ModeSymlink != 0:
		fmt.Println("symbolic link")
	case mode&fs.ModeNamedPipe != 0:
		fmt.Println("named pipe")
	}
	fmt.Println(fi.Name(), fi.ModTime(), fi.Size())
}
```

```go
//Getwd返回一个对应当前工作目录的根路径。如果当前目录可以经过多条路径抵达（因为硬链接），Getwd会返回其中一个。
dir,err:=os.Getwd()
//Mkdir使用指定的权限和名称创建一个目录。如果出错，会返回*PathError底层类型的错误。
err:=os.Mkdir(name,perm)
//MkdirAll使用指定的权限和名称创建一个目录，包括任何必要的上级目录，并返回nil，否则返回错误。权限位perm会应用在每一个被本函数创建的目录上。如果path指定了一个已经存在的目录，MkdirAll不做任何操作并返回nil。
err:=os.MkdirAll(path,perm)
//Rename修改一个文件的名字，移动一个文件。可能会有一些个操作系统特定的限制。
err:=os.Rename(oldpath,newpath)
//Remove删除name指定的文件或目录。如果出错，会返回*PathError底层类型的错误。
err:=os.Remove(name)
//RemoveAll删除path指定的文件，或目录及它包含的任何下级对象。它会尝试删除所有东西，除非遇到错误并返回。如果path指定的对象不存在，RemoveAll会返回nil而不返回错误。
err:=os.RemoveAll(path)
```

```
type File struct {
    // 内含隐藏或非导出字段
}
```

File代表一个打开的文件对象。

```
func Create(name string) (file *File, err error)
```

Create采用模式0666（任何人都可读写，不可执行）创建一个名为name的文件，如果文件已存在会截断它（为空文件）。如果成功，返回的文件对象可用于I/O；对应的文件描述符具有O_RDWR模式。如果出错，错误底层类型是*PathError。

```
func Open(name string) (file *File, err error)
```

Open打开一个文件用于读取。如果操作成功，返回的文件对象的方法可用于读取数据；对应的文件描述符具有O_RDONLY模式。如果出错，错误底层类型是*PathError。

```
func OpenFile(name string, flag int, perm FileMode) (file *File, err error)
```

OpenFile是一个更一般性的文件打开函数，大多数调用者都应用Open或Create代替本函数。它会使用指定的选项（如O_RDONLY等）、指定的模式（如0666等）打开指定名称的文件。如果操作成功，返回的文件对象可用于I/O。如果出错，错误底层类型是*PathError。

更多请看[Go语言基础——文件操作](https://hypo.ltd/2021/11/24/go-yu-yan-ji-chu-wen-jian-cao-zuo/)

























