# Linux inode

inode (index node) 索引节点，是指在许多 “类 Unix 文件系统” 中的一种数据结构，用于描述文件系统对象（包括文件、目录、设备文件、socket、管道等）。每个 inode 保存了文件系统对象数据的属性和磁盘块 block 位置。文件系统对象属性包含了各种元数据（如：最后修改时间）、用户组（owner）和权限数据。

本文主要介绍了 **Linux 文件系统** 以及 **inode 相关操作**。

目录：

- [Linux inode](#linux-inode)
  - [Linux 文件系统](#linux-文件系统)
  - [inode 相关操作](#inode-相关操作)
  - [参考链接](#参考链接)

## Linux 文件系统

**① 文件系统创建（格式化）时，就把存储区域分为两大连续的存储区域：**

* 一个用来保存文件系统对象的元信息数据。这是由 inode 组成的表，每个 inode 默认是 256 字节或者 128 字节。
* 另一个用来保存文件系统对象的内容数据。这是由单个大小为 512 字节的扇区，共计 8 个扇区组成的 4K 字节的块构成。块是操作系统读写时的基本单位。

**② 一个文件系统的 inode 的总数一般情况下是固定的。** 这限制了该文件系统所能存储的文件系统对象的总数目。典型的实现下，所有 inode 占用了文件系统 1% 左右的存储容量。

**③ 文件系统中每个文件系统对象对应一个 “inode” 数据，并用一个整数值来辨识。** 这个整数常被称为 inode 号码（“i-number” 或 “inode number”）。由于文件系统的 inode 表的存储位置、总条目数量都是固定的，因此可以用 inode 号码去索引查找 inode 表。

**④ inode 存储了文件系统对象的一些元信息**，如所有者、访问权限（读、写、执行）、类型（是文件还是目录）、内容修改时间、inode 修改时间、上次访问时间、对应的文件系统存储块的地址，等等。知道了一个文件的 inode 号码，就可以在 inode 元数据中查出文件内容数据的存储地址。

**⑤ 文件名与目录名是文件系统对象便于使用的别名。** 一个文件系统对象可以有多个别名，但只能有一个 inode，并用这个 inode 来索引文件系统对象的存储位置。

* inode 不包含文件名或目录名的字符串，只包含文件或目录的 “元信息”。
* Unix 的文件系统的目录也是一种文件。打开目录，实际上就是读取 “目录文件”。目录文件的结构是一系列目录项（dirent）的列表。每个目录项，由两部分组成：所包含文件或目录的名字，以及该文件或目录名对应的 inode 号码。
* 文件系统中的一个文件是指存放在其所属目录的 “目录文件” 中的一个目录项，其所对应的 inode 的类别为 “文件”；文件系统中的一个目录是指存放在其 “父目录文件” 中的一个目录项，其所对应的 inode 的类别为 “目录”。可见，多个 “文件” 可以对应同一个 inode；多个 “目录” 可以对应同一个 inode。
* 文件系统中如果两个文件或者两个目录具有相同的 inode 号码，那么就称它们是 “硬链接” 关系。实际上都是这个 inode 的别名。换句话说，一个 inode 对应的所有文件（或目录）中的每一个，都对应着文件系统某个 “目录文件” 中唯一的一个目录项。
* 创建一个目录时，实际做了 3 件事：在其 “父目录文件” 中增加一个条目；分配一个 inode；再分配一个存储块，用来保存当前被创建目录包含的文件与子目录。被创建的 “目录文件” 中自动生成两个子目录的条目，名称分别是：“.” 和 “..”。前者与该目录具有相同的 inode 号码，因此是该目录的一个 “硬链接”。后者的 inode 号码就是该目录的父目录的 inode 号码。所以，任何一个目录的 “硬链接” 总数，总是等于它的子目录总数（含隐藏目录）加 2。即每个 “子目录文件” 中的 “..” 条目，加上它自身的 “目录文件” 中的 “.” 条目，再加上 “父目录文件” 中的对应该目录的条目。
* 通过文件名打开文件，实际上是分成三步实现：
  1. 操作系统找到这个文件名对应的 inode 号码；
  1. 通过 inode 号码，获取 inode 信息；
  1. 根据 inode 信息，找到文件数据所在的 block，读出数据。

## inode 相关操作

**eg 1. 显示指定文件 inode 信息**

通过 `stat` 命令显示指定文件或目录的 inode 信息：

```bash
[root@localhost ~]# stat file1
  File: ‘file1’
  Size: 24        	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 67620456    Links: 2
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2021-09-21 16:32:27.282578738 +0800
Modify: 2021-09-21 16:32:24.163578725 +0800
Change: 2021-09-21 16:32:24.167578725 +0800
 Birth: -

# 只列出文件 inode number
[root@localhost ~]# stat --format=%i file1
67620456
```

* Size：文件的字节数
* type：文件类型，普通文件，目录，管道等等
* Inode：inode number
* Links：链接数，链接到该 inode 的硬链接数
* Access：读、写、执行权限
* Uid：文件所有者 User ID
* Gid：所有者组 Group ID
* Access：时间戳，最近一次访问的时间
* Modify：时间戳，最近一次修改文件内容的时间
* Change：时间戳，最近一次更改文件的时间（如执行 chmod、chown 等命令）

**eg 2. 显示文件 inode number**

`ls` 命令用于列出目录的内容，配合 `-i` 参数可用于显示每个文件的 inode number：

```bash
[root@localhost ~]# ls -i
67620456 file1   67620455 file2   67620456 file3
```

**eg 3. 显示文件系统 inode 信息**

`df` 命令用于显示文件系统的磁盘空间使用情况，配合 `-i, --inodes` 参数可用于显示 inode 使用的相关信息：

```bash
[root@localhost ~]# df -i
Filesystem                Inodes IUsed    IFree IUse% Mounted on
/dev/mapper/centos-root 23066624 82588 22984036    1% /
devtmpfs                 4094353   415  4093938    1% /dev
tmpfs                    4097368     1  4097367    1% /dev/shm
tmpfs                    4097368   599  4096769    1% /run
tmpfs                    4097368    16  4097352    1% /sys/fs/cgroup
/dev/sda1                 524288   327   523961    1% /boot
tmpfs                    4097368    67  4097301    1% /run/user/0
```

## 参考链接

* [inode](https://zh.wikipedia.org/wiki/Inode)
* [理解 inode - 阮一峰](https://www.ruanyifeng.com/blog/2011/12/inode.html)
* [详解 Linux Inode](https://www.jianshu.com/p/0520d6b76318)
