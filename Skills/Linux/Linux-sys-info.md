# Linux 查看系统信息

本文主要对 Linux 系统的 **版本信息** 和 **硬件信息** 的查看操作进行了总结。

目录：

- [Linux 查看系统信息](#linux-查看系统信息)
  - [查看系统版本](#查看系统版本)
    - [查看 Linux 内核版本](#查看-linux-内核版本)
    - [查看 Linux 发行版本](#查看-linux-发行版本)
  - [查看硬件信息](#查看硬件信息)
    - [CPU](#cpu)
    - [内存](#内存)
    - [硬盘](#硬盘)
    - [网卡](#网卡)
    - [其他](#其他)
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

**1. `lscpu` 查看 CPU 统计信息**

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

**2. `cat /proc/cpuinfo` 查看每个 CPU 信息**

该命令会将所有 CPU 信息全部打印出来，以下仅为输出的第一个 CPU 信息。

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

**`dmidecode -t memory` 查看内存硬件信息**

```bash
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

### 硬盘

**1. `lsblk` 查看硬盘和分区分布**

```bash
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 130.8M  0 rom  
vda    253:0    0    60G  0 disk 
└─vda1 253:1    0    60G  0 part /
```

* `NAME`：设备名
* `MAJ:MIN`：主:从 设备数量
* `RM`：是否可移动设备
* `SIZE`：设备大小
* `RO`：是否只读设备
* `TYPE`：设备类型
* `MOUNTPOINT`：设备挂载点

**2. `fdisk -l` 查看硬盘和分区详细信息**

```bash
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

**1. `lspci | grep -i 'eth'` 查看网卡硬件信息**

```bash
08:00.0 Ethernet controller: Intel Corporation Device 37cc (rev 09)
58:00.0 Ethernet controller: Intel Corporation Ethernet Controller XXV710 for 25GbE SFP28 (rev 02)
58:00.1 Ethernet controller: Intel Corporation Ethernet Controller XXV710 for 25GbE SFP28 (rev 02)
```

**2. 查看系统所有网络接口**

**eg 1.** `ifconfig -a`

```bash
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.8.10  netmask 255.255.252.0  broadcast 10.0.11.255
        inet6 fe80::5054:ff:fe8c:5ac0  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:8c:5a:c0  txqueuelen 1000  (Ethernet)
        RX packets 14759958  bytes 2237100423 (2.0 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16041393  bytes 4194853634 (3.9 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 6239413  bytes 989573591 (943.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6239413  bytes 989573591 (943.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

**eg 2.** `ip link show`

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8c:5a:c0 brd ff:ff:ff:ff:ff:ff
```

**3. `ethtool eth0` 查看指定网络接口的详细信息**

```bash
Settings for eth0:
	Supported ports: [ FIBRE ]
	Supported link modes:   10000baseT/Full 
	Supported pause frame use: Symmetric
	Supports auto-negotiation: Yes
	Supported FEC modes: Not reported
	Advertised link modes:  10000baseT/Full 
	Advertised pause frame use: No
	Advertised auto-negotiation: Yes
	Advertised FEC modes: Not reported
	Speed: 10000Mb/s
	Duplex: Full
	Port: FIBRE
	PHYAD: 0
	Transceiver: external
	Auto-negotiation: off
	Supports Wake-on: d
	Wake-on: d
	Current message level: 0x0000000f (15)
			       drv probe link timer
	Link detected: yes
```

### 其他

**1. `lspci` 查看 PCI 信息，即主板所有硬件槽信息**

```bash
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
00:01.1 IDE interface: Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
00:01.2 USB controller: Intel Corporation 82371SB PIIX3 USB [Natoma/Triton II] (rev 01)
00:01.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 03)
00:02.0 VGA compatible controller: Cirrus Logic GD 5446
00:03.0 PCI bridge: Red Hat, Inc. QEMU PCI-PCI bridge
00:04.0 PCI bridge: Red Hat, Inc. QEMU PCI-PCI bridge
00:05.0 Ethernet controller: Red Hat, Inc. Virtio network device
00:06.0 SCSI storage controller: Red Hat, Inc. Virtio block device
00:07.0 Unclassified device [00ff]: Red Hat, Inc. Virtio memory balloon
```

**2. `dmidecode` 查看机器 DMI（Desktop Management Interface）信息**

```bash
BIOS Information
	......

System Information
	......

Chassis Information
	......

Processor Information
	......

Physical Memory Array
	......

Memory Device
  ......
Memory Array Mapped Address
	......

System Boot Information
	......
```

若仅需单独查看某一设备信息，可通过加参数 `-t` + 设备名实现。如，查看系统 bios 信息，`dmidecode -t bios`。

## 参考链接

* [如何查看LINUX发行版的名称及其版本号](https://www.qiancheng.me/post/coding/show-linux-issue-version)
* [Linux发行版 vs Linux内核](https://www.jianshu.com/p/263a988ecf8b)
* [Linux 查看系统硬件信息](https://www.cnblogs.com/ggjucheng/archive/2013/01/14/2859613.html)
* [lscpu中的 socket、core、thread的意义](https://whosemario.github.io/2016/05/20/lscpu-cmd/)
