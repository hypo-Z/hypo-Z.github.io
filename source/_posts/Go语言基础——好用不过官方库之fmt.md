---
title: Go语言基础——好用不过官方库之fmt
author: hypo
img: medias/featureimages/58.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-08-01 11:18:32
coverImg:
password:
summary: 拒绝重复造轮子，看看那些好用的官方库的包，这里仅仅总结我自己觉得好用的库及应用点，具体包函数可查看[go官方库]
categories: Golang
tags:

- 笔记
- Golang
- 基础

---

# 好用不过官方库之fmt

拒绝重复造轮子，看看那些好用的官方库的包，这里仅仅总结我自己觉得好用的库及应用点，具体包函数可查看[go官方库](https://studygolang.com/pkgdoc)

## fmt

fmt包实现了类似C语言printf和scanf的格式化I/O。格式化动作（'verb'）源自C语言但更简单

### Printf

- 通用：

```go
//示例结构体
type Person struct {
Name    string
}
```
- 参数打印

|占位符| 说明| 示例| 输出 |
|------|------|-------|------|	
|%v    |值的默认格式表示 |Printf("%v",person )    | {hypo} |
|%+v    |类似%v，但输出结构体时会添加字段名  |Printf("%+v",person )|  {Name:hypo} |
|%#v    |值的Go语法表示 |Printf("#v",person ) | main.Person={hypo} |
|%T    |值的类型的Go语法表示 |Printf("%T",person )    | main.Person |
|%%    |百分号 |Printf("%%")    | %     |

- 可以打印精度和宽度，通过一个紧跟在百分号后面的十进制数指定，如果未指定宽度，则表示值时除必需之外不作填充。精度通过（可选的）宽度后跟点号后跟的十进制数指定。如果未指定精度，会使用默认精度；如果点号后没有跟数字，表示精度为0。举例如下：

```
%f:  //默认宽度，默认精度
%9f  //宽度9，默认精度
%.2f //默认宽度，精度2
%9.2f //宽度9，精度2
%9.f  //宽度9，精度0 
```

- 整数进制转换

|占位符    |说明    |示例    |输出|
|------|------|------|-----|
|%b    |二进制表示    |Printf("%b",5)|    101|
|%c    |该值对应的unicode码值    |Printf("%c",0x4E2d)|    中|
|%d    |十进制表示    |Printf("%d",0x12)|    18|
|%o    |八进制表示    |Printf("%o",10)|    12|
|%q    |单引号围绕的字符字面值，由Go语法安全的转译    |Printf("%q",0x4E2d)|    '中'|
|%x    |十六进制表示，字母形式为小写a-f    |Printf("%x",13)|    d|
|%X    |十六进制表示，字母形式为大写A-F    |Printf("%X",13)|    D|
|%U    |表示为Unicode格式：U+1234，等价于"U+%04X"    |Printf("%U",0x4E2d)|    U+4E2D|

### Fprintf

// 功能同上面三个函数，只不过将转换结果写入到 w 中。
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)

```go
func main() {
//创建文件
f, _ := os.Create("./test.txt")
defer f.Close()

//定义零值Buffer类型变量b
var b bytes.Buffer

//使用Write方法为写入字符串
b.Write([]byte("你好")) //写入缓存
f.Write([]byte("你好")) //写入文件

//这个是把一个字符串拼接到Buffer里
fmt.Fprint(&b, ",", "https://hypo.ltd")
//这是是把一个字符串拼接并写入文件中
fmt.Fprint(f, ",", "https://hypo.ltd")

//把Buffer里的内容打印到终端控制台
b.WriteTo(os.Stdout)
//读出文件内容并打印
content, err := ioutil.ReadFile("./test.txt")
if err != nil {
fmt.Println("read file failed, err:", err)
return
}
fmt.Println(string(content))

//官方示例，直接输出至控制台
const name, age = "Kim", 22
fmt.Fprint(os.Stdout, name, " is ", age, " years old.\n")
}
```

### Sprintf

// 功能同上面三个函数，只不过将转换结果以字符串形式返回。
func Sprint(a ...interface{}) string
func Sprintln(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string

```go
//可使用此函数标准化输出字符串string
func main() {
// go 中格式化字符串并赋值给新串，使用 fmt.Sprintf
// %s 表示字符串
var stockcode = "000987"
var enddate = "2020-12-31"
var url = "Code=%s&endDate=%s"
var target_url = fmt.Sprintf(url, stockcode, enddate)
fmt.Println(target_url)

// 另外一个实例，%d 表示整型
const name, age = "Kim", 22
s := fmt.Sprintf("%s is %d years old.\n", name, age)
io.WriteString(os.Stdout, s) // 简单起见，忽略一些错误
}
//输出
Code = 000987&endDate = 2020-12-31
Kim is 22 years old.
```

### Errorf

// 功能同 Sprintf，只不过结果字符串被包装成了 error 类型。
func Errorf(format string, a ...interface{}) error

### Scanf

// 从标准输入扫描文本，根据format 参数指定的格式将成功读取的空白分隔的值保存进成功传递给本函数的参数。返回成功扫描的条目个数和遇到的任何错误。

func Scanf(format string, a ...interface{}) (n int, err error)

```go
// 当程序执行到fmt.Scanl(&name),程序会停止这里,等待用户输入,多个参数使用空格,并回车
fmt.Scanf(&Name, &Age)
fmt.Printf("Name:%v,Age:%v \n", Name, Age)
```

### Fscanf

//Fscanf从r扫描文本，根据format 参数指定的格式将成功读取的空白分隔的值保存进成功传递给本函数的参数。返回成功扫描的条目个数和遇到的任何错误。

func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)

```go
func main(){
var name string
var age int
of, _ := os.Open("./test.txt") //文件里写hypo 18
defer of.Close()
//也可以根据官方示例学习
fmt.Fscanf(of, "%s %d", &name, &age)//可从文件中读入参数
fmt.Printf("name=%s,age=%d\n", name, age) //name=hypo,age=18   
}
```

### SScanf

//Sscanf从字符串str扫描文本，根据format 参数指定的格式将成功读取的空白分隔的值保存进成功传递给本函数的参数。返回成功扫描的条目个数和遇到的任何错误。

func Sscanf(str string, format string, a ...interface{}) (n int, err error)

```go
//官方示例
func main(){
var name string
var age int
n, err := fmt.Sscanf("Kim is 22 years old", "%s is %d years old", &name, &age)
if err != nil {
panic(err)
}
fmt.Printf("%d: %s, %d\n", n, name, age)
}
//输出
2: Kim, 22
```



