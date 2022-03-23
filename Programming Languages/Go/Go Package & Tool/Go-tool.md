# Go 工具

目录：

- [Go 工具](#go-工具)
  - [帮助文档](#帮助文档)
  - [下载包](#下载包)
  - [构建包](#构建包)
    - [`go build` 命令](#go-build-命令)
    - [`go run` 命令](#go-run-命令)
    - [`go install` 命令](#go-install-命令)
  - [参考资料](#参考资料)

## 帮助文档

Go 语言的工具箱集合了一系列功能的命令集。可以通过 `go` 或 `go help` 命令查看内置的帮助文档：

```bash
[root@VM-8-14-centos ~]# go
Go is a tool for managing Go source code.

Usage:

	go <command> [arguments]

The commands are:

	bug         start a bug report
	build       compile packages and dependencies
	clean       remove object files and cached files
	doc         show documentation for package or symbol
	env         print Go environment information
	fix         update packages to use new APIs
	fmt         gofmt (reformat) package sources
	generate    generate Go files by processing source
	get         add dependencies to current module and install them
	install     compile and install packages and dependencies
	list        list packages or modules
	mod         module maintenance
	run         compile and run Go program
	test        test packages
	tool        run specified go tool
	version     print Go version
	vet         report likely mistakes in packages

Use "go help <command>" for more information about a command.
```

## 下载包

`go get` 可以下载一个单一的包或者用 `...` 下载整个子目录里面的每个包。`go get` 命令会同时计算并下载所依赖的每个包。

```bash
$ go get github.com/golang/lint/golint
```

`go get` 命令支持当前流行的托管网站 GitHub、Bitbucket 和 Launchpad，可以直接向它们的版本控制系统请求代码。同时 `go get` 命令获取的代码是真实的本地存储仓库，而不仅仅只是复制源文件，因此你依然可以使用版本管理工具比较本地代码的变更或者切换到其它的版本。

如果指定 `-u` 命令行标志参数，`go get` 命令将确保所有的包和依赖的包的版本都是最新的，然后重新编译和安装它们。如果不包含该标志参数的话，而且如果包已经在本地存在，那么代码将不会被自动更新。

## 构建包

### `go build` 命令

`go build` 命令编译命令行参数指定的每个包。

* 如果包是一个库，则忽略输出结果；这可以用于检测包是可以正确编译的。
* 如果包的名字是 `main`，`go build` 将调用链接器在当前目录创建一个可执行程序。

```bash
# 编译当前包目录所有 .go 文件
$ go build . 

# 最终生成 hello 可执行程序（根据第一个源文件名）
$ go build hello.go world.go 

# -o 参数，指定可执行程序名
$ go build -o myName hello.go
```

### `go run` 命令

对于一次性运行的程序，我们可以通过 `go run` 命令快速的构建并运行它。`go run` 命令实际上是结合了构建和运行的两个步骤：

```bash
# 编译并运行当前包目录所有 .go 文件
$ go run . 
```

### `go install` 命令

`go install` 命令和 `go build` 命令类似，但是它会保存每个包的编译成果，而不是将它们都丢弃。

* 被编译的包会被保存到 `$GOPATH/pkg` 目录下，目录路径和 `src` 目录路径对应；
* 可执行程序被保存到 `$GOPATH/bin` 目录。（很多用户会将 `$GOPATH/bin` 添加到可执行程序的搜索列表中。）

`go install` 命令和 `go build` 命令都不会重新编译没有发生变化的包，这可以使后续构建更快捷。为了方便编译依赖的包，`go build -i` 命令将安装每个目标所依赖的包。

## 参考资料

* 《Go 语言实战》
* 《Go 语言圣经》
* [go语言中包的使用](https://juejin.cn/post/6982045275007221773)
