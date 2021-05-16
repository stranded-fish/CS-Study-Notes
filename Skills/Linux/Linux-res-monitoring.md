# Linux 查看系统资源

本文主要对 Linux 系统的 **资源使用信息** 的查看操作进行了总结。

目录：

- [Linux 查看系统资源](#linux-查看系统资源)
  - [top](#top)
    - [命令使用](#命令使用)
      - [参数使用](#参数使用)
      - [交互命令](#交互命令)
      - [常用操作](#常用操作)
    - [参数说明](#参数说明)
  - [vmstat](#vmstat)
    - [命令使用](#命令使用-1)
      - [参数使用](#参数使用-1)
      - [常用操作](#常用操作-1)
    - [参数说明](#参数说明-1)
  - [sar](#sar)
    - [命令使用](#命令使用-2)
      - [参数使用](#参数使用-2)
      - [常用操作](#常用操作-2)
    - [参数说明](#参数说明-2)
  - [参考链接](#参考链接)

## top

`top` 命令为 Linux 下常用的性能分析工具，能够实时显示系统整体状态以及各个进程的资源占用情况，类似于 Windows 任务管理器。

### 命令使用

#### 参数使用

```bash
top -hv|-bcHiOSs -d secs -n max -u|U user -p pid -o fld -w [cols]
```

参数说明：

* `-d`：指定输出刷新时间间隔
* `-p`：指定监控进程 PID
* `-s`：安全模式，避免交换命令带来的潜在危险
* `-i`：不显示任何闲置或者僵尸进程
* `-c`：显示整个命令行而不只是命令名

#### 交互命令

`top` 命令在执行过程中可以使用一些交互命令。注意：如果使用 `-s` 参数可能导致一些命令被屏蔽掉，如 `k` 命令。

* `h/?`：显示帮助界面
* `k`：终止一个进程。系统将提示用户输入需要终止的进程 PID，以及需要发送给该进程什么样的信号。一般的终止进程可以使用 15 信号；如果不能正常结束那就使用信号 9 强制结束该进程。默认值是信号 15。在安全模式中此命令被屏蔽
* `i`：忽略闲置和僵死进程。这是一个开关式命令
* `q`：退出程序
* `E`：切换内存显示单位
* `r`：重新安排一个进程的优先级别。系统提示用户输入需要改变的进程 PID 以及需要设置的进程优先级值。输入一个正值将使优先级降低，反之则可以使该进程拥有更高的优先权。默认值是10
* `S`：切换到累计模式
* `s`：改变两次刷新之间的延迟时间。系统将提示用户输入新的时间，单位为 s。如果有小数，就换算为 ms。默认值是 3s
* `f/F`：从当前显示中添加或者删除项目
* `o/O`：改变显示项目的顺序
* `l`：切换显示平均负载和启动时间信息
* `m`：切换显示内存信息的方式
* `t`：切换显示进程和 CPU 状态信息的方式
* `c`：切换显示命令名称和完整命令行
* `M`：根据驻留内存大小进行排序
* `P`：根据 CPU 使用百分比大小进行排序
* `T`：根据时间/累计时间进行排序
* `W`：将当前设置写入 `~/.toprc` 文件中。这是写 `top` 配置文件的推荐方法

#### 常用操作

```bash
# 每隔 2 秒刷新所有进程的资源占用情况
top -d 2

# 每隔 3 秒刷新进程的资源占用情况，并显示进程的命令行参数（默认只有进程名）
top -c

# 每隔 3 秒刷新 PID 为 12345 和 6789 的两个进程的资源占用情况
top -p 12345 -p 6789

# 每隔 2 秒刷新 PID 为 12345 的进程的资源使用情况，并显示该进程启动的命令行参数
top -d 2 -c -p 123456
```

### 参数说明

```bash
top - 15:18:06 up 18 days, 16:04,  1 user,  load average: 0.00, 0.01, 0.05
Tasks: 118 total,   2 running, 116 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.7 us,  1.0 sy,  0.0 ni, 97.0 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882012 total,   793192 free,   194280 used,   894540 buff/cache
KiB Swap:  4047996 total,  2641136 free,  1406860 used.  1506816 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                                                                                                                 
 1047 root      20   0   50088    592    552 R  0.7  0.0 295:10.49 rshim                                                                                                                                                                   
 2300 root      20   0  733948   7012   1764 S  0.7  0.4  70:56.94 barad_agent                                                                                                                                                             
  563 dbus      20   0   60560   1924   1560 S  0.3  0.1   1:32.35 dbus-daemon                                                                                                                                                             
 3724 root      20   0 1012228  23356   6916 S  0.3  1.2  66:17.49 YDService                                                                                                                                                               
 4550 root      20   0  728040      0      0 S  0.3  0.0   0:20.09 mono.linux-x86_               
...... 
```

**Part 1.** 前五行为系统整体统计信息。

* 第一行 `top` 系统信息：
  * `15:18:06`：当前系统时间
  * `up 18 days`：系统运行时间
  * `1 user`：当前登录用户数
  * `load average: 0.00, 0.01, 0.05`：系统负载，即任务队列的平均长度。三个数值分别为 1 分钟、5 分钟、15 分钟到现在的平均值
* 第二行 `Tasks` 进程信息：
  * `118 total`：进程总数
  * `2 running`：正在运行的进程数
  * `116 sleeping`：处于睡眠的进程数
  * `0 stopped`：停止的进程数
  * `0 zombie`：僵尸进程数
* 第三行 `%Cpu(s)` CPU 信息：
  * `1.7 us`：用户空间占用 CPU 百分比
  * `1.0 sy`：内核空间占用 CPU 百分比
  * `0.0 ni`：用户进程空间内改变过优先级的进程占用 CPU 百分比
  * `97.0 id`：空闲 CPU 百分比
  * `0.3 wa`：等待 IO 的 CPU 时间百分比
  * `0.0 hi`：硬件 CPU 中断占用百分比
  * `0.0 si`：软中断占用百分比
  * `0.0 st`：虚拟机占用百分比
* 第四行 `KiB Mem` 内存信息：
  * `1882012 total`：内存总量
  * `793192 free`：空闲内存总量
  * `194280 used`：使用内存总量
  * `894540 buff/cache`：缓冲/缓存内存总量
* 第五行 `KiB Swap` 交换区信息：
  * `4047996 total`：交换区总量
  * `2641136 free`：空闲交换区总量
  * `1406860 used`：使用交换区总量
  * `1506816 avail Mem`：缓冲的交换区总量,内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖，该数值即为这些内容已存在于内存中的交换区的大小,相应的内存再次被换出时可不必再对交换区写入。

**Part 2.** 进程信息统计区显示了各个进程的详细信息。

序号 | 列名    | 含义
-----|---------|-------------------------------------------
a    | PID     | 进程 ID
b    | PPID    | 父进程 ID
c    | RUSER   | Real user name
d    | UID     | 进程所有者的用户 ID
e    | USER    | 进程所有者的用户名
f    | GROUP   | 进程所有者的组名
g    | TTY     | 启动进程的终端名。不是从终端启动的进程则显示为 ?
h    | PR      | 优先级
i    | NI      | nice 值，负值表示高优先级，正值表示低优先级
j    | P       | 最后使用的 CPU，仅在多 CPU 环境下有意义
k    | %CPU    | 上次更新到现在的 CPU 时间占用百分比
l    | TIME    | 进程使用的 CPU 时间总计，单位秒
m    | TIME+   | 进程使用的 CPU 时间总计，单位1/100秒
n    | %MEM    | 进程使用的物理内存百分比
o    | VIRT    | 进程使用的虚拟内存总量，单位 KB，VIRT=SWAP+RES
p    | SWAP    | 进程使用的虚拟内存中，被换出的大小，单位 KB
q    | RES     | 进程使用的、未被换出的物理内存大小，单位 KB，RES=CODE+DATA
r    | CODE    | 可执行代码占用的物理内存大小，单位 KB
s    | DATA    | 可执行代码以外的部分（数据段+栈）占用的物理内存大小，单位 KB
t    | SHR     | 共享内存大小，单位 KB
u    | nFLT    | 页面错误次数
v    | nDRT    | 最后一次写入到现在，被修改过的页面数。
w    | S       | 进程状态（D=不可中断的睡眠状态，R=运行，S=睡眠，T=跟踪/停止，Z=僵尸进程）
x    | COMMAND | 命令名/命令行
y    | WCHAN   | 若该进程在睡眠，则显示睡眠中的系统函数名
z    | Flags   | 任务标志，参考 sched.h

`top` 命令默认仅显示部分信息，可通过以下方式更改显示内容：

在 `top` 命令运行阶段，通过 <kbd>f</kbd> 键进入设置状态，<kbd>Up</kbd>/<kbd>Dn</kbd> 上下移动, <kbd>Right</kbd> 选中当前属性并移动，<kbd>Left</kbd> 选中当前属性再通过 <kbd>d</kbd> 或 <kbd>space</kbd> 选中新增显示，最后 <kbd>q</kbd> 或 <kbd>Esc</kbd> 保存退出。

## vmstat

`vmstat` 命令主要用于监控整个系统的 CPU、内存和 IO 使用情况。

### 命令使用

#### 参数使用

```bash
vmstat [-V] [-n] [delay [count]]
```

参数说明：

`-V`：版本信息
`-n`：只显示一次头部信息
`-a`：活动、不活动的统计信息
`-d`：磁盘统计
`-D`：磁盘表信息
`-p`：磁盘分区统计信息
`-s`：内存表信息
`-m`：cache 信息
`-t`：输出信息，附带时间戳
`-S`：输出信息单位，k/K/m/M，默认为 KB
`delay`：每隔一定时间显示
`count`：显示次数
`unit size`：k:1000 K:1024 m:1000000 M:1048576 (default is K)

#### 常用操作

```bash
# 每隔 2 秒采集一次信息，共采集 10 次
vmstat 2 10

# 每隔 1 秒采集一次信息，持续采集，直到终止程序
vmstat 1
```

### 参数说明

```bash
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 371303520 295860 19426488    0    0    57    51    0    0  0  0 100  0  0	
 1  0      0 371303008 295864 19426484    0    0     0     8  566  819  0  0 100  0  0	
```

默认单位 KB。

* procs
  `r`：在运行队列中等待的进程数
  `b`：在等待 IO 的进程数
* memory
  `swpd`：虚拟内存已使用的大小，若大于 0，则表示机器物理内存已不足。
  `free`：物理空闲内存大小
  `buff`：缓冲区内存大小
  `cache`：缓存区内存大小
* swap
  `si`：从磁盘交换到虚拟内存的交换页数量，单位：KB/秒
  `so`：从虚拟内存交换到磁盘的交换页数量，单位：KB/秒
* io
  `bi`：块设备（磁盘）写入数据量，单位：块/秒
  `bo`：块设备（磁盘）发送数据量，单位：块/秒
* system
  `in`：CPU 每秒中断数，包括时钟中断
  `cs`：每秒的环境（上下文）切换次数
* cpu
  `us`：用户模式时间百分比
  `sy`：内核模式时间百分比
  `id`：CPU 空闲时间百分比
  `wa`：等待 IO 时间百分比
  `st`：虚拟机占用时间百分比

## sar

sar（System Activity Reporter，系统活动情况报告）是目前 Linux 上最为全面的系统性能分析工具之一，可以从多方面对系统的活动进行报告，包括：文件的读写情况、系统调用的使用情况、磁盘 I/O、CPU 效率、内存使用状况、进程活动及 IPC 有关的活动等。

### 命令使用

#### 参数使用

```bash
sar [ options ] [ <interval> [ <count> ] ]
```

参数说明：

TODO

#### 常用操作

### 参数说明

**4. `mpstat`**

**5. `iostat`**

**6. `dstat`**






**2\. `free` 查看内存使用概要情况**

参数说明：

* `-b`, --bytes         show output in bytes
* `-k`, --kilo          show output in kilobytes
* `-m`, --mega          show output in megabytes
* `-g`, --giga          show output in gigabytes
* `-h`, --human         show human-readable output

```bash
# free -h
              total        used        free      shared  buff/cache   available
Mem:           1.8G        338M        1.4G         96K        109M        1.3G
Swap:          3.9G         76M        3.8G
```

`Mem` 为内存的使用信息，`Swap` 为交换空间的使用信息。

* `total`：系统总可用物理内存大小
* `used`：已被使用的物理内存大小
* `free`：未分配的内存
* `shared`：被共享使用的内存
* `buffers/cache`：被 buffer 和 cache 使用的物理内存大小
* `available`：可用内存数

**注意：** `free` 为真正未被使用的物理内存大小，`available` 为应用程序认为可用的内存数量，`available = free + buffer + cache`（理论情况），Linux 为了提升读写性能，会消耗一部分内存资源缓存磁盘数据，对于内核来说，buffer 和 cache 都属于已经被使用的内存。但当应用程序申请内存时，如果 free 内存不够，内核就会回收 buffer 和 cache 的内存来满足应用程序的请求。

可以通过以下命令手动回收 `buff/cache`：

```bash
sysctl -w vm.drop_caches=1
sysctl -w vm.drop_caches=2
sysctl -w vm.drop_caches=3
```

**3\. `cat /proc/meminfo` 查看内存使用详细情况**

```bash
# cat /proc/meminfo
MemTotal:        1882012 kB
MemFree:         1388508 kB
MemAvailable:    1382660 kB
Buffers:            3956 kB
Cached:           118592 kB
SwapCached:         7932 kB
Active:           213092 kB
......
```




## 参考链接

* [linux的top命令参数详解](https://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316399.html)
* [Linux系统查看CPU使用率的几个命令](https://blog.csdn.net/AlbenXie/article/details/72885951)
* [Linux内存、性能诊断中vmstat命令的详解](https://www.sijitao.net/1925.html)
