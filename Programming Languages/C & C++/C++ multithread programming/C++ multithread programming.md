# C++ 多线程编程

本文章主要针对 C++ 11 引入的 `<thread>` 多线程编程的学习与总结。

目录：

- [C++ 多线程编程](#c-多线程编程)
  - [线程基础操作](#线程基础操作)
    - [线程创建](#线程创建)
    - [线程监视](#线程监视)
    - [线程操作](#线程操作)
  - [互斥锁](#互斥锁)
    - [成员函数](#成员函数)
    - [使用示例](#使用示例)
      - [std::lock_guard](#stdlock_guard)
      - [std::unique_lock](#stdunique_lock)
  - [条件变量](#条件变量)
    - [成员函数](#成员函数-1)
      - [constructor](#constructor)
      - [Notification](#notification)
      - [Waiting](#waiting)
    - [使用示例](#使用示例-1)
  - [原子类型](#原子类型)
    - [使用示例](#使用示例-2)
  - [异步线程](#异步线程)
    - [成员函数](#成员函数-2)
      - [Getting the result](#getting-the-result)
      - [State](#state)
    - [使用示例](#使用示例-3)
  - [参考链接](#参考链接)

## 线程基础操作

`std::thread` 类定义于头文件 `<thread>`。

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

## 互斥锁

互斥锁 `std::mutex` 定义于头文件 `<mutex>`。

```C++
#include <mutex>
class mutex;
```

### 成员函数

```C++
// 对互斥锁加锁，必要时阻塞
// 如果互斥锁当前没有被任何线程锁定，则调用线程将其锁定（从此时开始，直到调用其成员解锁，线程拥有互斥锁）
// 如果互斥锁当前被另一个线程锁定，则调用线程的执行将被阻塞，直到被另一个线程解锁（其他未锁定的线程继续执行）
// 如果互斥锁当前被调用此函数的同一线程锁定，则会产生死锁（行为未定义）
void lock();

// 尝试在不阻塞的情况对互斥锁加锁
// 如果互斥锁当前没有被任何线程锁定，则调用线程将其锁定（从此时开始，直到调用其成员解锁，线程拥有互斥锁）
// 如果互斥锁当前被另一个线程锁定，则函数失败并返回 false，不会阻塞（调用线程继续执行）
// 如果互斥锁当前被调用此函数的同一线程锁定，则会产生死锁（行为未定义）
bool try_lock();

// 释放互斥锁
// 如果当前其他线程尝试锁定该互斥锁，则其中一个线程将获得对其的所有权并继续执行
void unlock();
```

### 使用示例

```C++
int cnt = 0;
std::mutex mx;

void cal(int val) {
    for (int i = 0; i < val; ++i) {
        mx.lock();
        cnt = cnt + 1;
        mx.unlock();
    }
}

int main() {
    thread t1(cal, 1000000);
    thread t2(cal, 1000000);
    t1.join();
    t2.join();
    cout << cnt << endl;
}
```

**注意：** 不建议直接使用 `lock()` 方法，因为可能会忘记执行 `unlock()` 或者程序出现异常无法 `unlock()`，最终导致锁无法释放，可以使用 RAII 对象思想来管理互斥锁（如：`lock_guard` 或 `unique_lock`）。

#### std::lock_guard

```C++
template< class Mutex >
class lock_guard;
```

* 类 `lock_guard` 是一个互斥体包装器，它提供了一种方便的 RAII 风格机制，用于在作用域块的持续时间内拥有互斥体。
* 当一个 `lock_guard` 对象被创建时，它会尝试获取给它的互斥锁的所有权。当控制离开创建 `lock_guard` 对象的范围时，`lock_guard` 被破坏并且互斥体被释放。
* `lock_guard` 类是不可复制的。

使用示例：

```C++
int cnt = 0;
std::mutex mx;

void safe_cal(int val) {
    for (int i = 0; i < val; ++i) {
        {
            lock_guard<mutex> lock(mx);
            cnt = cnt + 1;
        }
    }
}

int main() {
    thread t1(safe_cal, 1000000);
    thread t2(safe_cal, 1000000);
    t1.join();
    t2.join();
    cout << cnt << endl;
}
```

可以通过 `{}` 设定作用域，可以使得 `std::lock_guard` 在合适的地方被析构。

#### std::unique_lock

`std::unique_lock` 类似于 `lock_guard`，只是 `std::unique_lock` 用法更加丰富，同时支持 `std::lock_guard()` 的原有功能。

## 条件变量

条件变量 `std::condition_variable` 定义于头文件 `<condition_variable>`。

```C++
#include <condition_variable>
class condition_variable;
```

`condition_variable` 可用于同时阻塞一个线程或多个线程，直到另一个线程修改共享变量（条件）并通知 `condition_variable`。

**Notification**

函数       | 功能
-----------|------------
notify_one | 解除一个等待线程的阻塞
notify_all | 解除所有等待线程的阻塞

**Waiting**

函数       | 功能
-----------|----------------------------
wait       | 阻塞当前线程直到条件变量被唤醒
wait_for   | 阻塞当前线程，直到条件变量被唤醒或在指定的超时时间之后
wait_until | 阻塞当前线程，直到条件变量被唤醒或到达指定时间点

### 成员函数

#### constructor

```C++
condition_variable();
```

#### Notification

**1. `notify_one`：解除一个等待线程的阻塞**

```C++
void notify_one() noexcept;
```

**2. `notify_all`：解除所有等待线程的阻塞**

```C++
void notify_all() noexcept;
```

#### Waiting

**1. `wait`：阻塞当前线程直到条件变量被唤醒**

```C++
// (1)
void wait( std::unique_lock<std::mutex>& lock );

// (2)
template< class Predicate >
void wait( std::unique_lock<std::mutex>& lock, Predicate stop_waiting );
```

**2. `wait_for`：阻塞当前线程，直到条件变量被唤醒或在指定的超时时间之后**

```C++
// (1)
template< class Rep, class Period >
std::cv_status wait_for( std::unique_lock<std::mutex>& lock,
                         const std::chrono::duration<Rep, Period>& rel_time);

// (2)
template< class Rep, class Period, class Predicate >
bool wait_for( std::unique_lock<std::mutex>& lock,
               const std::chrono::duration<Rep, Period>& rel_time,
               Predicate stop_waiting);
```

**3. `wait_until`：阻塞当前线程，直到条件变量被唤醒或到达指定时间点**

```C++
// (1)
template< class Clock, class Duration >
std::cv_status
    wait_until( std::unique_lock<std::mutex>& lock,
                const std::chrono::time_point<Clock, Duration>& timeout_time );

// (2)
template< class Clock, class Duration, class Predicate >
bool wait_until( std::unique_lock<std::mutex>& lock,
                 const std::chrono::time_point<Clock, Duration>& timeout_time,
                 Predicate stop_waiting );
```

### 使用示例

**`std::condition_variable::notify_one` 示例 1：**

```C++
std::condition_variable cv;
std::mutex cv_m;
int i = 0;
bool done = false;

void waits() {
    std::unique_lock<std::mutex> lk(cv_m);
    std::cout << "Waiting... \n";
    cv.wait(lk, [] { std::cout << "check!" << std::endl; return i == 1; });
    std::cout << "...finished waiting; i == " << i << '\n';
    done = true;
}

void signals() {
    std::this_thread::sleep_for(200ms);
    std::cout << "Notifying falsely...\n";

    // 唤醒阻塞线程，此时 i == 0
    // 阻塞线程被唤醒，检查 i 发现 i == 1 为 false，继续 waiting
    cv.notify_one();

    std::unique_lock<std::mutex> lk(cv_m);
    i = 2; // 修改条件 i
    while (!done) {
        std::cout << "Notifying true change...\n";
        lk.unlock();
        cv.notify_one(); // 再次唤醒线程，此时 i == 1 为 true，阻塞线程开始执行
        std::this_thread::sleep_for(300ms);
        lk.lock();
    }
}

int main() {
    std::thread t1(waits), t2(signals);
    t1.join();
    t2.join();
}
```

**输出 1：**

```txt
Waiting... 
Notifying falsely...
Notifying true change...
...finished waiting; i == 1
```

**`std::condition_variable::wait_until` 示例 2：**

```C++
#include <iostream>
#include <atomic>
#include <condition_variable>
#include <thread>
#include <chrono>

using namespace std::chrono_literals;

std::condition_variable cv;
std::mutex cv_m;
std::atomic<int> i{0};

void waits(int idx) {
    std::unique_lock<std::mutex> lk(cv_m);
    auto now = std::chrono::system_clock::now();
    if (cv.wait_until(lk, now + idx * 100ms, []() { return i == 1; }))
        std::cerr << "Thread " << idx << " finished waiting. i == " << i << '\n';
    else
        std::cerr << "Thread " << idx << " timed out. i == " << i << '\n';
}

void signals() {
    std::this_thread::sleep_for(120ms);
    std::cerr << "Notifying...\n";
    cv.notify_all();
    std::this_thread::sleep_for(100ms);
    i = 1;
    std::cerr << "Notifying again...\n";
    cv.notify_all();
}

int main() {
    std::thread t1(waits, 1), t2(waits, 2), t3(waits, 3), t4(signals);
    t1.join(); t2.join(); t3.join(); t4.join();
}
```

**输出 2：**

```txt
Thread 1 timed out. i == 0
Notifying...
Thread 2 timed out. i == 0
Notifying again...
Thread 3 finished waiting. i == 1
```

## 原子类型

原子类型 `std::atomic` 定义于头文件 `<atomic>`

```C++
// (1)
template< class T >
struct atomic;

// (2)
template< class U >
struct atomic<U*>;
```

* `std::atomic` 模板的每个实例化和完全特化都定义了一个原子类型。如果一个线程写入一个原子对象，而另一个线程从它读取，则行为是明确定义的。
* `std::atomic` 既不可复制也不可移动。
* `std::atomic` 特化了以下操作以保证原子性：
  * `operator++`
  * `operator++(int)`
  * `operator--`
  * `operator--(int)`
  * `operator+=`
  * `operator-=`
  * `operator&=`
  * `operator|=`
  * `operator^=`

### 使用示例

```C++
atomic<long long> cnt;

void safe_inc(int val) {
    for (int i = 0; i < val; ++i) {
        ++cnt;
    }
}

int main() {
    thread t1(safe_inc, 1000000), t2(safe_inc, 1000000);
    t1.join(); t2.join();
    cout << cnt << endl; // 输出：2000000
}
```

## 异步线程

类模板 `std::future` 提供了一种异步执行的机制，异步操作对象 `std::future` 可以通过 `std::async`、`std::packaged_task`、`std::std::promise` 创建。

### 成员函数

#### Getting the result

**`get`：返回结果**

```C++
T get();    // (1)
T& get();   // (2)
void get(); // (3)
```

get 成员函数会一直等到 future 有一个有效的结果并（取决于使用的模板）检索它，它将调用 `wait()` 以等待结果。

#### State

**1. valid：检查 future 对象是否拥有共享的状态**

```C++
bool valid() const noexcept;
```

**2. wait：主线程等待结果可用**

```C++
void wait() const;
```

**3. wait_for：等待结果可用，直到超出规定时间则返回**

```C++
template< class Rep, class Period >
std::future_status wait_for( const std::chrono::duration<Rep,Period>& timeout_duration ) const;
```

**4. wait_until：等待结果，直到指定时间点可用才返回**

```C++
template< class Clock, class Duration >
std::future_status wait_until( const std::chrono::time_point<Clock,Duration>& timeout_time ) const;
```

### 使用示例

```C++
#include <iostream>
#include <future>
#include <thread>

int main() {
    // future from a packaged_task
    std::packaged_task<int()> task([] { return 7; }); // wrap the function
    std::future<int> f1 = task.get_future();  // get a future
    std::thread t(std::move(task)); // launch on a thread

    // future from an async()
    std::future<int> f2 = std::async(std::launch::async, [] { return 8; });

    // future from a promise
    std::promise<int> p;
    std::future<int> f3 = p.get_future();
    std::thread([&p] { p.set_value_at_thread_exit(9); }).detach();

    std::cout << "Waiting..." << std::flush;
    f1.wait();
    f2.wait();
    f3.wait();
    std::cout << "Done!\nResults are: "
              << f1.get() << ' ' << f2.get() << ' ' << f3.get() << '\n';
    t.join();
}

// 输出
// Waiting...Done!
// Results are: 7 8 9
```

## 参考链接

* [std::thread - cppreference.com](https://en.cppreference.com/w/cpp/thread/thread)
* [std::mutex - cppreference.com](https://en.cppreference.com/w/cpp/thread/mutex)
* [std::condition_variable - cppreference.com](https://en.cppreference.com/w/cpp/thread/condition_variable)
* [std::atomic - cppreference.com](https://en.cppreference.com/w/cpp/atomic/atomic)
* [std::future - cppreference.com](https://en.cppreference.com/w/cpp/thread/future)
