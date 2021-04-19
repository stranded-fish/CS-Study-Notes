# Linux 交换分区

当使用 Linux 虚拟机或者云服务器时，可能由于系统运行内存大小限制，导致运行某些程序失败（如：编译大型程序），这时可以考虑通过创建交换分区解决这一问题。

Linux 中 Swap（即：交换分区），类似于 Windows 的虚拟内存，就是当内存不足的时候，把一部分硬盘空间虚拟成内存使用，从而解决内存容量不足的问题。

目录：

- [Linux 交换分区](#linux-交换分区)
  - [查看 swap 空间](#查看-swap-空间)
  - [创建 swap 分区](#创建-swap-分区)
  - [删除 swap 分区](#删除-swap-分区)
  - [参考链接](#参考链接)

## 查看 swap 空间

显示包括文件和分区使用的详细信息：

```bash
swapon -s
```

```
Filename				Type		Size	Used	Priority
/swapfile                              	file	      4047996	  0	  -2
```

注：还可通过 `free -h` 命令查看整个系统内存的使用情况（包括物理内存、交换内存和内核缓冲区内存）

## 创建 swap 分区

**Step 1.** 创建分区文件，大小 2 G

```bash
dd if=/dev/zero of=/swapfile bs=1k count=2048000
```

参数说明：

* `bs=1k`：块大小
* `count=2048000`：块数量

文件大小 = bs * count

**Step 2.** 生成 swap 文件系统

```bash
mkswap /swapfile
```

**Step 3.** 启用 swap 文件

```bash
swapon /swapfile
```

**Step 4.** 设置重启系统自动加载

```bash
echo "/swapfile  swap  swap    defaults 0 0" >> /etc/fstab
```

## 删除 swap 分区

**Step 1.** 停止 swap 分区

```bash
swapoff /swapfile
```

**Step 2.** 删除 swap 分区文件

```bash
rm -rf /swapfile
```

**Step 3.** 取消重启自动加载

删除 `/etc/fstab` 文件，之前添加的 `/swapfile  swap  swap    defaults 0 0` 行。

## 参考链接

* [SWaP（Linux系统中的交换分区）](https://baike.baidu.com/item/Swap/2666174)
