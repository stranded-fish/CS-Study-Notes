# Go 包

目录：

- [Go 包](#go-包)
  - [包简介](#包简介)
    - [包名惯例](#包名惯例)
    - [main 包](#main-包)
  - [工作区结构](#工作区结构)
    - [`GOPATH` 环境变量](#gopath-环境变量)
    - [`GOROOT` 环境变量](#goroot-环境变量)
    - [`go env` 命令查看环境变量](#go-env-命令查看环境变量)
  - [包导入](#包导入)
    - [远程导入](#远程导入)
    - [命名导入](#命名导入)
  - [导入方式](#导入方式)
    - [相对路径](#相对路径)
    - [设置 GOPATH](#设置-gopath)
    - [Go Modules](#go-modules)
  - [包的作用域](#包的作用域)
  - [init 函数](#init-函数)
  - [参考资料](#参考资料)

## 包简介

Go 语言的程序都会组织成若干组文件，每组文件被称为一个包。每个包的代码都可以作为很小的复用单元，被其他项目单独导入和使用。

* 所有 `.go` 文件，除了空行和注释，都应该在第一行声明自己所属的包；
* 同一个目录下的所有 `.go` 文件都必须声明同一个包名。即不能把多个包放到同一个目录中，也不能把同一个包的文件分拆到多个不同目录中。

### 包名惯例

* 包命名的惯例是使用包所在目录的名字；
* 给包及其目录命名时，应该使用简洁、清晰且全小写的名字；
* 尽可能让命名有描述性且无歧义；
* 包名一般采用单数的形式。

### main 包

Go 中，命名为 `main` 的包具有特殊的含义，Go 语言编译程序会试图把这种包编译为二进制可执行程序，**所有用 Go 编译的可执行程序都必须有一个名叫 main 的包。** 同时当编译器发现某个包的名字为 `main` 时，**也一定需要发现名为 `main()` 的函数，** 否则不会创建可执行文件。

当试图编译并运行在非 `main` 包的 `.go` 文件时，会报错：`go run: cannot run non-main package`。

## 工作区结构

### `GOPATH` 环境变量

用户可通过配置 `GOPATH` 环境变量，来指定当前的工作目录。当需要切换到不同工作区的时候，只要更新或添加 `GOPATH` 即可。

`GOPATH` 允许多个目录，设置多个目录时，需要使用环境变量分隔符来区分。Unix/Linux 的环境变量分隔符是冒号 `:`，windows 下是分号 `;`。

**Linux 设置多个 `GOPATH`：**

```bash
GOPATH="/home/www/gopath1:/home/www/gopath2"
```

`GOPATH` 对应的工作区目录需要有三个子目录：

* `src`：用于存储源代码，该目录下的每个子目录为一个包，可作为基地址，通过 `import` 直接导入。
* `pkg`：用于保存编译后的包的目标文件；
* `bin`：用于保存编译后的可执行程序。

### `GOROOT` 环境变量

环境变量 `GOROOT` 用来指定：

* `Go` 的安装目录；
* 还有它自带的标准库包的位置。

`GOROOT` 的目录结构和 `GOPATH` 类似（因此存放 `fmt` 包的源代码对应目录应该为 `$GOROOT/src/fmt`）。用户一般不需要设置 `GOROOT`，默认情况下 Go 语言安装工具会将其设置为安装的目录路径。

### `go env` 命令查看环境变量

`go env` 命令用于查看 Go 语言工具涉及的所有环境变量的值，包括未设置环境变量的默认值。

```go
$ go env
GOPATH="/root/go"
GOROOT="/usr/lib/golang"
GOOS="linux"
GOARCH="amd64"
...
```

其中：

* `GOOS` 环境变量用于指定目标操作系统，例如 android、linux、darwin 或 windows；
* `GOARCH` 环境变量用于指定处理器的类型，例如 amd64、386 或 arm 等。

## 包导入

Go 通过 `import` 语句导入包。

```go
import (
    "fmt"
    "strings"
)
```

* Go 会首先查找 Go 的安装目录 `GOROOT`（包含标准库中的包）；
* 然后再顺序查找 `GOPATH` 变量中列出的目录（包含开发者创建的包）。

一旦编译器找到一个满足 `import` 语句的包，就会停止进一步查找。

### 远程导入

如果导入路径中包含 `URL`，Go 工具链将从 `DVCS` 获取包，并把包的源代码保存到 `GOPATH` 指向的路径里与 `URL` 匹配的目录中。

```go
import "github.com/spf13/viper"
```

该过程由 `go get` 命令完成。`go get` 将获取任意指定的 `URL` 的包，或一个已经导入的包所依赖的其他包。即该命令会递归地获取到所有依赖包。

### 命名导入

如果要导入的包具有相同的名字，则可以通过 `命名导入` 来避免冲突。`命名导入` 指：在 `import` 语句给出的包路径左侧定义一个名字，将导入的包命名为新名字。

```go
import (
    "fmt"
    myfmt "mylib/fmt"
)

func main() {
    fmt.Println("Standard Library")
    myfmt.Println("mylib/fmt")
}
```

有时导入一个包，但不需要这个包的标识符，这种情况下，可以使用 `_` 空白标识符来重命名这个包,以避免编译器因为包未使用而报错。

## 导入方式

### 相对路径

Go 可以通过相对路径来导入特定包（不推荐）。

目录结构：

```bash
[root@VM-8-14-centos myProject]# pwd
/root/myProject
[root@VM-8-14-centos myProject]# tree
.
`-- src
    |-- main
    |   |-- main.go
    |   `-- world.go
    `-- mypkg
        |-- myTest1.go
        `-- myTest2.go

3 directories, 4 files

...

[root@VM-8-14-centos main]# pwd
/root/myProject/src/main
[root@VM-8-14-centos main]# go run .
myTest init
```

导入方式：

```go
package main

import "../mypkg"

func main() {
	// mypkg...
}
```

**注意：** 如果该工程路径已经配置到 `GOPATH` 中或该工程创建了 `go.mod` 文件并启用了模块支持，则不能使用相对路径导入包，此时 `go build` 会报错。Go 只有在 `GOPATH` 中找不到的包路径，同时没有启用 `Go Modules` 模块支持时，才允许使用相对路径导入包。

### 设置 GOPATH

将当前工程目录的基地址添加到 `GOPATH` 中：

```bash
GOPATH="/home/www/origin:/home/www/newProject"
```

**注意：** 需要保证该工程的目录结构符合 `GOPATH` 要求（根目录包含三个子目录 `src`、`pkg`、`bin`）

```bash
[root@VM-8-14-centos myProject]# tree
.
|-- bin
|-- pkg
`-- src
    |-- main
    |   |-- main.go
    |   `-- world.go
    `-- mypkg
        |-- myTest1.go
        `-- myTest2.go

5 directories, 4 files

# 添加 GOPATH 环境变量
[root@VM-8-14-centos myProject]# GOPATH="/root/go:/root/myProject"

[root@VM-8-14-centos main]# pwd
/root/myProject/src/main
[root@VM-8-14-centos main]# go run .
myTest init
```

```go
package main

// 以 GOPATH src 目录作为基地址，进行导入
import "mypkg"

func main() {
	// mypkg...
}
```

### Go Modules

Go 在 1.11 版本引入了 `Go Modules`，又称 Go 模块，它是 Go 官方提供的包管理工具，通过 `Go Modules`，我们可以不必将项目放在 `GOPATH` 上。

**通过环境变量 `GO111MODULE` 设置模块支持：**

```go
go env -w GO111MODULE=auto
```

它有三个可选值：`off`、`on`、`auto`，默认是 `auto`。

* `GO111MODULE=off`：无模块支持，Go 会从 `GOPATH` 寻找包；
* `GO111MODULE=on`：模块支持，根据 `go.mod` 文件下载依赖；
* `GO111MODULE=auto`：在 `$GOPATH/src` 外面且项目根目录有 `go.mod` 文件时，自动开启模块支持。

**在项目 `src` 根目录，通过以下命令初始化 `Go Module`：**

```bash
go mod init module_name
```

```bash
[root@VM-8-14-centos src]# go mod init myModule 
go: creating new go.mod: module myModule
go: to add module requirements and sums:
        go mod tidy

[root@VM-8-14-centos src]# tree
.
|-- go.mod
|-- main
|   |-- main.go
|   `-- world.go
`-- mypkg
    |-- myTest1.go
    `-- myTest2.go

2 directories, 5 files

[root@VM-8-14-centos src]# cat go.mod 
module myModule

go 1.16
```

```go
package main

import "myModule/mypkg"

func main() {
	// mypkg...
}
```

## 包的作用域

* 包内：在同一个包内定义的函数、变量、常量、结构体，可以被包内的所有其他代码任意访问，它们属于包内公开。

* 包外：如果函数、变量、常量、结构体位于不同的包下，且它们的首字母使用大写标识，表示它们是可以公开访问的。对于结构体字段。如果想要在包外进行访问，还要让结构体字段变量名使用首字母大写。

## init 函数

每个包可以包含任意多个 `init` 函数，这些函数都会在程序执行开始的时候被调用（即 `main` 函数之前）。

`init` 函数用于设置包、初始化变量或者其他要在程序运行前优先完成的引导工作。

```go
func init() {
	fmt.Println("First print")
}
```

## 参考资料

* 《Go 语言实战》
* 《Go 语言圣经》
* [go语言中包的使用](https://juejin.cn/post/6982045275007221773)
