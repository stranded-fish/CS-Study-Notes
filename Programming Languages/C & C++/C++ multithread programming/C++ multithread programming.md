# C++ 多线程编程

目录：

- [C++ 多线程编程](#c-多线程编程)
  - [线程基础操作](#线程基础操作)
    - [线程创建](#线程创建)
    - [线程监视](#线程监视)
    - [线程操作](#线程操作)
  - [参考链接](#参考链接)

## 线程基础操作

### 线程创建

```C++
// 默认构造函数，创建新的线程对象，但并未实际启动一个新线程
thread() noexcept;

// 移动构造函数，通过 other 构造新的执行线程，再此之后 other 失效
thread( thread&& other ) noexcept;

// 移动赋值运算符，同移动构造函数
thread& operator=( thread&& other ) noexcept;

// 创建新的线程，并执行所关联的 function
template< class Function, class... Args >
explicit thread( Function&& f, Args&&... args );

// 删除拷贝构造函数，线程不可被复制，不能同时有两个 std::thread 对象代表同一个执行线程
thread( const thread& ) = delete;
```

**注意：** 当线程启动失败，会抛出 `std::system_error` 异常。

### 线程监视

函数                   | 功能
-----------------------|-------------------------------------
`joinable`             | 判断线程是否可 join（可调用 join()），即正处于并行运行环境中
`get_id`               | 返回线程 id
`native_handle`        | 返回底层实现的 thread handle
`hardware_concurrency` | 返回系统支持的并发线程数

函数签名：

```C++
// 返回 true，如果线程对象标识了一个活动的执行线程（正在执行），否则 false
bool joinable() const noexcept;

// 返回与 *this 关联的线程 id，如果没有关联线程，则返回默认构造 id
std::thread::id get_id() const noexcept;

// 只有库函数支持该函数时，才能生效，如果有效，则获得与操作系统相关的原生线程句柄
native_handle_type native_handle();

// 返回系统支持的并发线程数
static unsigned int hardware_concurrency() noexcept;
```

### 线程操作

函数     | 功能
---------|------------------------------------------
`join`   | 阻塞当前 thread 对象所在的线程，直至 thread 对象表示的线程执行完毕
`detach` | 将当前线程从调用该函数的线程中分离出去，使彼此独立运行
`swap`   | 交换两个线程对象

函数签名：

```C++
// 阻塞当前线程，直到 *this 标识的线程完成执行
void join();

// 将执行线程与线程对象分离，允许执行独立继续。
// 一旦线程退出，任何分配的资源都将被释放。
void detach();

// 交换两个线程对象的句柄
void swap( std::thread& other ) noexcept;
```

## 参考链接

* [std::thread - cppreference.com](https://en.cppreference.com/w/cpp/thread/thread)
* [C++ 11 多线程编程详解](http://c.biancheng.net/view/8638.html)
* [C++ 多线程并发基础入门教程](https://zhuanlan.zhihu.com/p/194198073)
