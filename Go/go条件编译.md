---
Go 条件编译
---

[toc]



# 编译标签（build tag)

在源码文件顶部添加注释，来决定文件是否参与编译

```go
// +build <tags>
```

**<tags>**说明

- 以空格分开表示AND
- 以逗号分开表示OR
- !表示NOT

标签可以指定为以下内容：

- 操作系统，环境变量中`GOOS`的值，如：`linux`、`darwin`、`windows`等等。
- 操作系统的架构，环境变量中`GOARCH`的值，如：`arch64`、`x86`、`i386`等等。
- 使用的编译器,`gc`或者`gccgo`。
- 是否开启CGO,`cgo`。
- golang版本号： 比如Go Version 1.1为`go1.1`,Go Version 1.12版本为`go1.12`，以此类推。
- 其它自定义标签，通过`go build -tags`指定的值

```go
###编译条件为（Linux AND 386） OR （darwin AND (NOT cgo))
// +build linux, 386 darwin, !cgo

###一个文件可以有多个编译约束，如：(linux OR darwin) AND amd64
// +build linux darwin
// +build amd64

##将一个文件从编译中排除，使用ignore标签
// +build ignore
```

> ==// +build的下一行必须是空行==



# 文件后缀

编译器根据文件后缀来自动选择编译文件

```go
$filename_$GOOS.go
$filename_$GOARCH.go
$filename_$GOOS_$GOARCH.go
```

- `$filename`: 源文件名称。
- `$GOOS`: 表示操作系统，从环境变量中获取。
- `$GOARCH`: 表示系统架构，从环境变量中获取

如在项目中有`tcp.go`和`tcp_linux_x86.go`两个文件，执行：

```go
GOOS=linux GOARCH=x86 go build
```

将选`tcp_linux_x86.go`进行编译,而执行

```go
GOOS=linux GOARCH=x86 go build
```

选择`tcp.go`进行编译。



# 利用ldflags在编译过程中为变量赋值

```go
go build -ldflags "-w -s -X main.Version=${VERSION} -X github.com/demo/version.BuildNo=${BUILD_NO}"
```

参数：

​	-w 删除DWARF信息： 编译出来的程序无法用gdb进行调试

​	-s 删除符号表： panic的stack trace 没有文件名/行号信息，等价于C/C++程序被strip。

​	-X 替换包中的变量的值。

> 加上-w -s 可以有效减少编译出来的程序的大小，但不利于进行调试和日志追踪

