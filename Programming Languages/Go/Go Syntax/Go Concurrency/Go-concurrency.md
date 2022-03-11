# Go 并发

目录：

- [Go 并发](#go-并发)
  - [goroutine](#goroutine)
    - [工具类](#工具类)
      - [sync.WaitGroup](#syncwaitgroup)
  - [基于共享资源的并发](#基于共享资源的并发)
    - [原子函数](#原子函数)
    - [互斥锁](#互斥锁)
  - [通道](#通道)
    - [基本操作](#基本操作)
    - [无缓冲的通道](#无缓冲的通道)
    - [有缓冲的通道](#有缓冲的通道)
  - [参考资料](#参考资料)

## goroutine

Go 语言中，每一个并发的执行单元叫作一个 `goroutine`。当一个程序启动时，其主函数即在一个单独的 `goroutine` 中运行，称为 `main goroutine`。

新的 `goroutine` 会用 `go` 语句来创建。在语法上，`go` 语句是一个普通的函数或方法调用前加上关键字 `go`。`go` 语句会使其语句中的函数在一个新创建的 `goroutine` 中运行。

```go
f()    // call f(); wait for it to return
go f() // create a new goroutine that calls f(); don't wait
```

操作系统会在物理处理器上调度线程来运行，Go 每个逻辑处理器都分别绑定到唯一的操作系统线程。Go 会在逻辑处理器上调度 `goroutine` 运行。

> 操作系统线程 **1 : 1** 逻辑处理器 **1 : n** `goroutine`。

如果希望让 `goroutine` 并行，必须使用多于一个逻辑处理器。当有多个逻辑处理器时，调度器会将 `goroutine` 平等分配到每个逻辑处理器，最终使 `goroutine` 在不同的线程上并行运行。

**修改逻辑处理器的数量：**

```go
import "runtime"

// 分配一个逻辑处理器
runtime.GOMAXPROCS(1)

// 给每个可用的核心分配一个逻辑处理器
runtime.GOMAXPROCS(runtime.NumCPU())
```

### 工具类

#### sync.WaitGroup

`WaitGroup` 是一个信号计数量，可以用来记录并维护运行的 `goroutine`。如果 `WaitGroup` 值大于 `0`，`Wait` 方法就会阻塞。

使用示例：

* `main` 协程：
  * 调用 `wg.Add(delta int)` 设置 `worker` 协程的个数；
  * 创建 `worker` 协程；
  * 调用 `wg.Wait()` 进入阻塞（`WaitGroup > 0`），直到所有 `worker` 协程全部执行结束后返回。
* `worker` 协程执行结束以后，调用 `wg.Done()`，通知 `main` 协程工作已经完成。
  * 为确保 `wg.Done()` 一定在协程完成其工作后被调用，可使用 `defer wg.Done()` 修改函数调用时机。

## 基于共享资源的并发

Go 语言提供了传统的同步 `goroutine` 的机制，即对共享资源加锁。

### 原子函数

原子函数以较底层的加锁机制来同步访问整型变量和指针。

```go
import "sync/atomic"

// T 表示数值类型 int32、int64、uint32、uint64 等
atomic.AddT(addr *T, delta T)  (new T)  // 原子加
atomic.LoadT(addr *T)          (val T)  // 原子读
atomic.StoreT(addr *T, val T)           // 原子写
```

### 互斥锁

互斥锁用于在代码上创建一个临界区，保证同一时间只有一个 `goroutine` 可以执行这个临界区代码。

```go
import "sync"

// 声明互斥锁
var mutex sync.Mutex

go func() {
	mutex.Lock()   // 获取互斥锁，进入临界区
	for i := 0; i < 1000000000; i++ {
		counter++
	}
	mutex.Unlock() // 释放互斥锁，退出临界区
}()
```

## 通道

Go 可以使用通道，通过发送和接收需要共享的资源，在 `goroutine` 之间做同步。这种并发同步模型来自一个叫做 `通信顺序进程（Communicating Sequential Processes, CSP）` 的范型。CSP 是一种消息传递模型，通过在 `goroutine` 之间传递数据来传递消息，而不是对数据进行加锁来实现同步访问。

### 基本操作

**创建通道：**

```go
// 无缓冲的整型通道
unbuffered := make(chan int)

// 有缓冲的字符串通道
buffered := make(chan string, 10)
```

**通道收发值：**

```go
buffered := make(chan string, 10)

buffered <- "Hello"  // 通过通道发送一个字符串
value := <- buffered // 从通道接收一个字符串
```

**关闭通道：**

```go
close(myChan)

value, ok := <- myChan
if !ok {
    // 通道被关闭......
}
```

### 无缓冲的通道

Go 语言中无缓冲的通道（unbuffered channel）是指在接收前没有能力保存任何值的通道。这种类型的通道要求发送 `goroutine` 和接收 `goroutine` 同时准备好，才能完成发送和接收操作。

如果两个 `goroutine` 没有同时准备好，通道会导致先执行发送或接收操作的 `goroutine` 阻塞等待。这种对通道进行发送和接收的交互行为本身就是同步的。其中任意一个操作都无法离开另一个操作单独存在。

### 有缓冲的通道

Go 语言中有缓冲的通道（buffered channel）是一种在被接收前能存储一个或者多个值的通道。这种类型的通道并不强制要求 `goroutine` 之间必须同时完成发送和接收。通道会阻塞发送和接收动作的条件也会不同:

* 只有在通道中没有要接收的值时，接收动作才会阻塞。
* 只有在通道没有可用缓冲区容纳被发送的值时，发送动作才会阻塞。

## 参考资料

* 《Go 语言实战》
* 《Go 语言圣经》
