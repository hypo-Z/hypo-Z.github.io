---
title: Go使用S3搭建Lambda无状态服务
date: 2021-12-04 19:15:55
author: 
top: false
hide: false
cover: false
img: medias/featureimages/35.png
password:
toc: false
mathjax: false
summary: Go使用S3搭建Lambda无状态服务
categories: Golang
tags:
- Golang
- AWS
- S3
- Lambda
---
# golang使用S3搭建Lambda无状态服务

由于项目需求我需要使用aws的Lambda服务来自动处理存入图片的缩略图的生成和存入功能。

## 需求

* 根据尺寸需求生成对应的缩略图
* 自动保存入S3对应位置
* 将位置信息告诉后端存入数据库
* 日志记录报错和运行信息

## 先决条件

#### 1.一个aws账号
#### 2.编辑好需求的程序,需要在本地测试好，确定没问题再转化成Lambda格式，虽然在aws控制台可以测试，但需要打包，很麻烦，先做测试，减少上线步骤。

官方的例子：
```go
package main

import (
        "fmt"
        "context"
        "github.com/aws/aws-lambda-go/lambda"
)

type MyEvent struct {
        Name string `json:"name"`
}

func HandleRequest(ctx context.Context, name MyEvent) (string, error) {
        return fmt.Sprintf("Hello %s!", name.Name ), nil
}

func main() {
        lambda.Start(HandleRequest)
}
```

下面列出了有效的处理程序签名。TIn 和 TOut 表示类型与 encoding/json 标准库兼容。有关更多信息，请参阅 func Unmarshal，以了解如何反序列化这些类型。
```go
func ()
func () error
func (TIn), error
func () (TOut, error)
func (context.Context) error
func (context.Context, TIn) error
func (context.Context) (TOut, error)
func (context.Context, TIn) (TOut, error)
```

#### 3.部署.zip文件存档

##### 在 macOS 和 Linux 上创建 .zip 文件

编译您的可执行文件。
```go
GOOS=linux go build main.go
```
将 GOOS 设置为 linux 可确保编译的可执行文件与 Go 运行时兼容（即使您在非 Linux 环境中编译它也是如此）。

（可选）如果您的 main 程序包包含多个文件，请使用以下 go build 命令来编译此程序包：
```go
GOOS=linux go build main
```
（可选）您可能需要使用 Linux 上的 CGO_ENABLED=0 编译程序包：
```go
GOOS=linux CGO_ENABLED=0 go build main.go
```
此命令为标准 C 库 (libc) 版本创建稳定的二进制程序包，这在 Lambda 和其他设备上可能有所不同。

Lambda 使用 POSIX 文件权限，因此在创建 .zip 文件存档之前，您可能需要为部署程序包文件夹设置权限。

通过将可执行文件打包为 .zip 文件来创建部署程序包。
```go
zip function.zip main
```

这里有个坑需要踩一下：
main包下，最好是将全部程序放在main.go里面，防止在编译时报错，这样打好zip包后到控制台测试运行，一定要做测试，这样才能确保你的程序能够成功运行，最好是查看图片与需求结果是否相同。

...补充...
