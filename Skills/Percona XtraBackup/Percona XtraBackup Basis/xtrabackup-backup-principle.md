# Percona XtraBackup 原理解析

Percona XtraBackup 是由 Percona 公司开发的一个开源的 MySQL 热备份工具，同时也是目前市面上唯一的开源热备份工具。利用该工具（2.4 版本）可以实现在 MySQL 5.5, 5.6 和 5.7 服务器上备份 InnoDB、XtraDB 和 MyISAM 表的数据，以及备份 Percona 服务器的 XtraDB MySQL。

项目的完整工程文件与更新日志保存在 [Github](https://github.com/percona/percona-xtrabackup) 上，截至目前（2021/4/28）Percona XtraBackup 已更新至 8.0 版本（该版本仅支持 MySQL 8.0 的热备份）。

**注意：本文内容基于 percona-xtrabackup-2.4.8。** 本文主要介绍该版本的工作流程与原理。

目录：

- [Percona XtraBackup 原理解析](#percona-xtrabackup-原理解析)
  - [工作流程](#工作流程)
    - [工具集](#工具集)
  - [原理解析](#原理解析)
    - [全量备份](#全量备份)
      - [备份过程](#备份过程)
      - [线程协作](#线程协作)
    - [全量恢复](#全量恢复)
    - [增量备份](#增量备份)
    - [增量恢复](#增量恢复)

## 工作流程

### 工具集

Percona XtraBackup 安装完成后，工具集包含以下可执行文件：

bin/
├── xtrabackup
├── innobackupex -> xtrabackup
├── xbstream
├── xbcrypt
└── xbcloud

* `xtrabackup`：用于非阻塞备份各种类型的数据库表（注意：发行时间较早的旧版 xtrabackup 只能用于备份 innoDB 数据库表，而较新版 xtrabackup 均可备份）。
* `innobackupex`：为指向 xtrabackup 的软链接，通过在程序运行中判断程序名，执行不同的参数处理操作，以兼容旧版 innobackupex 命令行。
* `xbstream`：以 XBSTREAM 格式序列化/反序列化文件。
* `xbcrypt`：以 XBCRYPT 格式加密或解密文件。
* `xbcloud`：在云服务上管理备份。

其中主要通过 `xtrabackup` / `innobackupex` 工具，进行备份与恢复。

并且通常完整的备份流程不止需要复制源数据库文件，因为原数据库文件数据量较大，要求主从复制以及加密操作，通过会一起使用 `xbstream`、`xbcrypt` 等工具。

TODO 原始数据库文件 压缩 序列化 加密 
解密 反序列化 解压 恢复

## 原理解析

### 全量备份

#### 备份过程

![innobackupex full backup 时序图](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/JZkwHeO2v1f567D.png)

1. 由于 2.4.8 版本，`innobackupex` 仅作为 `xtrabackup` 的一个软链接，在 `xtrabackup` 启动后，在参数处理阶段会根据程序名执行不同的操作，从而将 `innobackupex` 参数适配成 `xtrabackup` 形式以兼容旧版。最终等价于直接运行 `xtrabackup` 命令。
2. `xtrabackup` 首先将根据之前参数处理阶段获取到的参数，初始化对应的 datasink，由于选择了 `--compress` 压缩备份，故在此阶段会初始化 compress datasink，同时 **创建 `compress threads`（多线程）**。
3. 完成 datasink 初始化后，`xtrabackup` 将首先 **创建 `io watching thread`（单线程）** 负责 io 节流。然后再 **创建 `log copying thread`（单线程）** 负责拷贝 redo 日志，`log copying thread` 将从当前最新的 checkpoint 点开始复制。最后 **创建 `data copying threads`（多线程）并等待**，`data copying threads` 主要负责获取待压缩文件指针以及压缩内容的地址信息并将其传递给 `compress threads`，然后通知其开始压缩。
4. 待 InnoDB 数据备份完成，`xtrabackup` 将 **执行 `FLUSH TABLES WITH READ LOCK`（FTWRL）**，获取一致性位点，并强制 MyISAM 表缓存落盘，然后开始备份非 InnoDB 数据，这个过程与备份 InnoDB 数据类似，同样会经历压缩、序列化等步骤。
5. 当非 InnoDB 数据备份完成后，`log copying thread` 将停止拷贝 redo 日志。
6. 日志拷贝停止后，`xtrabackup` 将会对数据库进行解锁，**执行 `UNLOCK TABLES`**，然后销毁 datasink 并停止 `io watching thread`。
7. 最后 `xtrabackup` 将打印复制的事务日志 LSN 范围，并进行相关收尾工作。

三个 io watch redo copy 这些线程的作用，

为什么要上锁，为什么要记录 redo 日志，redo 日志如何运用的

#### 线程协作

![XtraBackup 压缩 - 线程协作模型](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/YqUdfTLQgpvC81J.png)

**①** `main thread` 根据参数 **`--parallel`** 创建若干 `data copy threads`，并等待。
**②** `data copy threads` 首先获取一个待压缩文件的指针，然后执行 `while` 循环，每次对一部分文件进行压缩（长度为 `len`），然后 `compress_write` 方法会 **尝试获取 `xtrabackup compress threads` 的 `ctrl mutex`**（注意：一旦该 copy thread 获取到了 compress thread 的第一把锁，则会阻塞其他所有 copy thread，后续的 compress 锁（ctrl mutex）只能由该 copy thread 获取），并将该部分文件内容，再次分块（长度不超过由参数 **`--compress-chunk-size`** 设置的 `COMPRESS_CHUNK_SIZE`），并按地址顺序将每一块分配给一个 `xtrabackup compress thread` 进行压缩。（即，同一时刻，所有的 `xtrabackup compress threads` 将只能服务于一个 `data copy thread`）
**③** `xtrabackup compress threads` 收到待压缩内容的起始地址、块大小以及解锁信号后，开始执行压缩任务，完成任务之后设置信号量、发送 `signal` 通知给他分配任务的 `data copy thread`。
**④** `data copy thread` 会通过 `for` 循环按照顺序（之前分配压缩任务时的顺序）等待 `xtrabackup compress threads` 完成任务（即，如果压缩线程 2、3... 已完成压缩任务，但压缩线程 1 未完成，则只能等待压缩线程 1 完成后，才能继续下一步，因为要保证按照原始文件顺序写入压缩内容），收到完成信号之后，将该线程压缩后的内容写入到下一管道（buffer）中，**并释放该线程的 ctrl mutex**。待整个 `for` 循环完成后（即，表示当前 `data copy thread` 分配给 `xtrabackup compress threads` 的任务已经全部完成并按序写入下一管道），则由下一个获取到 `ctrl mutex` 的 `data copy thread` 重复步骤 **② - ④**。

### 全量恢复

重点 perpare 阶段

checkpoint 阶段内容

### 增量备份

增量备份原理，如何判断的，之呢个来给你

### 增量恢复

为什么需要将 增量的 应用到 全量的？
