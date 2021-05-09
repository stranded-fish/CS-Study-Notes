# Linux 查看系统信息

本文主要对 Linux 系统的相关信息的查看操作进行了总结。

目录：

- [Linux 查看系统信息](#linux-查看系统信息)
  - [查看系统版本](#查看系统版本)
    - [查看 Linux 内核版本](#查看-linux-内核版本)
    - [查看 Linux 发行版本](#查看-linux-发行版本)
  - [查看硬件信息](#查看硬件信息)
    - [CPU](#cpu)
    - [内存](#内存)
    - [磁盘](#磁盘)
    - [网卡](#网卡)
    - [其他](#其他)
  - [查看开发环境](#查看开发环境)
  - [参考链接](#参考链接)

## 查看系统版本

系统版本分为 **内核版本** 和 **发行版本**。

**Linux 内核：** Linux 内核是计算机操作系统的核心，是为你运行的其他程序分配计算机资源的程序，其功能可以划分为系统、网络、存储、内存和计算。

**Linux 发行版：** Linux 发行版（Linux distribution）是由 Linux 内核（Linux kernal）和 软件包管理系统组合而成的操作系统，软件包管理系统中包括应用程序和实用软件（例如 GNU tools and libraries），针对不同的用户，包里装着不同的组件。常见发行版有 **基于 Redhat 的 CentOS** 和 **基于 Debian 的 Ubuntu**。

### 查看 Linux 内核版本

以下两种方法适用于所有发行版。

**eg 1.** `cat /proc/version`

**eg 2.** `uname -a`

### 查看 Linux 发行版本

**eg 1.** `cat /etc/issue` 适用于所有发行版。

**eg 2.** `cat /etc/redhat-release` 仅适用于 Redhat 系发行版。

## 查看硬件信息

### CPU

**1\. `lscpu` 查看 CPU 统计信息**

```bash
Architecture:          x86_64              # CPU 架构
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian       # 字节序
CPU(s):                72                  # 总逻辑 CPU 数
On-line CPU(s) list:   0-71     
Thread(s) per core:    2                   # 单核线程数
Core(s) per socket:    18                  # 单插槽核心数
Socket(s):             2                   # CPU 物理插槽数量
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Gold 6240 CPU @ 2.60GHz
Stepping:              7
CPU MHz:               2593.915
BogoMIPS:              5185.09
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              1024K
L3 cache:              25344K
NUMA node0 CPU(s):     0-17,36-53
NUMA node1 CPU(s):     18-35,54-71
```

对操作系统而言，其逻辑 CPU 数量：`CPU(s) = Socket(s) * Core(s) per socket * Thread(s) per core`。

以上信息表明，该系统共插有两个物理 CPU，每个 CPU 为 18 核 36 线程，共计 72 线程。

**2\. `cat /proc/cpuinfo` 查看每个 CPU 信息**

以下仅为输出的第一个 CPU 信息，该命令会将所有 CPU 信息全部打印出来。

```bash
processor	: 0
vendor_id	: AuthenticAMD
cpu family	: 23
model		: 49
model name	: AMD EPYC 7K62 48-Core Processor
stepping	: 0
microcode	: 0x1000065
cpu MHz		: 2595.124
cache size	: 512 KB
physical id	: 0
siblings	: 1
core id		: 0
cpu cores	: 1
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm art rep_good nopl extd_apicid eagerfpu pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext retpoline_amd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt sha_ni xsaveopt xsavec xgetbv1 arat
bogomips	: 5190.24
TLB size	: 1024 4K pages
clflush size	: 64
cache_alignment	: 64
address sizes	: 48 bits physical, 48 bits virtual
power management:
......
```

### 内存

**1\. `dmidecode -t memory` 查看内存硬件信息**

```bash
# dmidecode -t memory
# dmidecode 3.2
Getting SMBIOS data from sysfs.
SMBIOS 2.8 present.

Handle 0x1000, DMI type 16, 23 bytes
Physical Memory Array
	Location: Other
	Use: System Memory
	Error Correction Type: Multi-bit ECC
	Maximum Capacity: 2 GB
	Error Information Handle: Not Provided
	Number Of Devices: 1

Handle 0x1100, DMI type 17, 40 bytes
Memory Device
	Array Handle: 0x1000
	Error Information Handle: Not Provided
	Total Width: Unknown
	Data Width: Unknown
	Size: 2048 MB
	Form Factor: DIMM
	Set: None
	Locator: DIMM 0
	Bank Locator: Not Specified
	Type: RAM
	Type Detail: Other
	Speed: Unknown
	Manufacturer: Smdbmds
	Serial Number: Not Specified
	Asset Tag: Not Specified
	Part Number: Not Specified
	Rank: Unknown
	Configured Memory Speed: Unknown
	Minimum Voltage: Unknown
	Maximum Voltage: Unknown
	Configured Voltage: Unknown
```

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

### 磁盘

`fdisk -l` 查看磁盘硬件信息

```bash
# fdisk -l
Disk /dev/vda: 64.4 GB, 64424509440 bytes, 125829120 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0009ac89

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048   125829086    62913519+  83  Linux
```

### 网卡

`lspci | grep -i 'eth'` 查看网卡硬件信息

```bash
# lspci | grep -i 'eth'

08:00.0 Ethernet controller: Intel Corporation Device 37cc (rev 09)
58:00.0 Ethernet controller: Intel Corporation Ethernet Controller XXV710 for 25GbE SFP28 (rev 02)
58:00.1 Ethernet controller: Intel Corporation Ethernet Controller XXV710 for 25GbE SFP28 (rev 02)
```

### 其他

`lspci` 查看 PCI 信息，即主板所有硬件槽信息。

## 查看开发环境

## 参考链接

* [如何查看LINUX发行版的名称及其版本号](https://www.qiancheng.me/post/coding/show-linux-issue-version)
* [Linux发行版 vs Linux内核](https://www.jianshu.com/p/263a988ecf8b)
* [Linux 查看系统硬件信息](https://www.cnblogs.com/ggjucheng/archive/2013/01/14/2859613.html)
* [lscpu中的 socket、core、thread的意义](https://whosemario.github.io/2016/05/20/lscpu-cmd/)
