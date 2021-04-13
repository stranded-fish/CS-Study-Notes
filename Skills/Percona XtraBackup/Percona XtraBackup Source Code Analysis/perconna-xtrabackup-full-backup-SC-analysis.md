# Percona XtraBackup 全量备份过程源码解析

Xrabackup 工具集包含以下可执行文件：

bin/
├── innobackupex -> xtrabackup
├── xtrabackup
├── xbstream
├── xbcrypt
└── xbcloud

其中主要通过 innobackupex 工具，实现对 InnoDB 和非 InnoDB 表、文件的压缩备份。

本文主要介绍通过 innobackupex 命令，在设置参数 `--compress`、`--stream=xbsream` 情况下对数据进行压缩并转化为 xbsream 流格式备份的全过程源码解析。

**注：本文内容基于 percona-xtrabackup-2.4.8**

目录：

- [Percona XtraBackup 全量备份过程源码解析](#percona-xtrabackup-全量备份过程源码解析)
  - [备份命令](#备份命令)
  - [备份过程](#备份过程)
  - [调用关系](#调用关系)
  - [关键结构体定义](#关键结构体定义)
  - [核心设计思想](#核心设计思想)
  - [关键流程解析](#关键流程解析)
    - [输入参数 与 innobackupex 兼容性处理](#输入参数-与-innobackupex-兼容性处理)
    - [多线程模型](#多线程模型)
    - [数据落盘机制](#数据落盘机制)

## 备份命令

innobackupex 工具进行全量备份的命令示例如下：

```bash
innobackupex --user=root --password=MyNewPass4! --parallel=4 --stream=xbstream --compress --compress-thread=4 /data/backups/ > /data/innobackupextest.xbstream
```

**参数说明：**

* `--user=root` | `--password=MyNewPass4!` ：待备份数据库的登录用户名与密码。
  由于 innobackupex 在备份过程中需要向 mysqld server 发送命令进行交互，如加 MDL 锁、加读锁（FTWRL）、获取位点（SHOW SLAVE STATUS）等，故需要数据库登录信息。
* `--parallel=4` ：指定用于 data transfer 的 data copy thread 创建数量。
* `--stream=xbstream` ：指定进行流备份的格式，一般选择 xbstream 格式。
* `--compress` ：进行压缩备份。
* `--compress-thread=4` ：指定用于执行压缩任务的 xtrabackup compress thread 创建数量。
* `/data/backups/ > /data/innobackupextest.xbstream` ：指定保存目录路径以及流文件名称。

**命令作用：**

该命令将从数据库默认存储路径 `/var/lib/mysql` 中读取 innodb 表数据（`.ibd`...）和 非 innodb 表数据（`.frm`...），并调用 qpress 压缩算法将其压缩为 `.qp` 格式，然后合并为一个 xbsream 流格式文件，并保存到指定位置。

## 备份过程

![innobackupex full backup 时序图](https://i.loli.net/2021/04/10/JZkwHeO2v1f567D.png)

TODO 时许图介绍

## 调用关系

![innobackupex full backup 调用关系图](https://i.loli.net/2021/04/12/4GLpnjbJNh5mdfu.png)

## 关键结构体定义

TODO
1. 更为完整的结构体介绍
2. 加上 datasink 那个 init create write 那个结构体

```cpp
// 数据池环境 (ds_create)
typedef struct ds_ctxt {
	datasink_t	*datasink; 	    // 数据池类型，决定数据池执行的方法
	char 		*root;          // 数据池根地址
	void		*ptr;           // 数据池执行环境
	struct ds_ctxt	*pipe_ctxt; // 数据池管道，即当前数据池的输出目的地
} ds_ctxt_t;
```

数据池环境结构体定义，其中 `ptr` 用于指向该数据池具体执行环境，如 `compress_ds_data->ptr` 指向 `ds_compress_ctxt_t`，而 `ds_compress_ctxt_t` 包含执行具体压缩任务的线程变量，`pipe_ctxt` 指向下一数据池，如 `compress_ds_data->pipe_ctxt` 指向 `buffer_ds`。

```cpp
// 数据池文件环境 (ds_open)
typedef struct {
	void		*ptr;           // 执行文件，即具体的数据文件地址
	char		*path;          // 输出路径
	datasink_t	*datasink;      // 数据池类型，决定数据池执行的方法
} ds_file_t;
```

数据池文件环境定义，其中 `ptr` 用于指向具体文件类型，如 `ds_compress_file_t`：

```cpp
typedef struct {
	ds_file_t		*dest_file;
	ds_compress_ctxt_t	*comp_ctxt;
	size_t			bytes_processed;
} ds_compress_file_t;
```

## 核心设计思想

TODO 介绍那个 datasink 通过 init 方式 实现类似于 java 的多态

TODO 数据流再详细介绍一下，弄个图

**数据流传递方向：**

通过 ds_file_t->ptr 指针获取具体数据文件，再通过数据文件 dest_file 指针定位到下一个管道：

![数据流图](https://i.loli.net/2021/04/12/BbPwzFmi9qOMWCG.png)

## 关键流程解析

TODO 有待梳理更多关键的流程并总结

### 输入参数 与 innobackupex 兼容性处理

自版本 2.3 起，innobackupex 功能全部合并到了 xtrabackup 中，为了兼容之前版本，
最终工具集仍保留了名为 innobackupex 的可执行文件，但该文件实际为指向 xtrabackup 可执行文件的软链接。`file innobackupex > innobackupex: symbolic link to 'xtrabackup'`
故 xtrabackup \ innobackupex 命令入口均为 xtrabackup main 方法。

**main : xtrabackup.cc**

```cpp
// 处理执行命令及参数
handle_options(argc, argv, &client_defaults, &server_defaults);

// 判断是否为 innobackupex 模式，若是则调动对应初始化方法
if (innobackupex_mode) {
    if (!ibx_init()) {
        exit(EXIT_FAILURE);
    }
}
```

### 多线程模型

**Step 1\. xtrabackup_compress_threads 创建**

`xtrabackup_init_datasinks();` 初始化数据池，同时创建 `xtrabackup_compress_threads` 线程。`xtrabackup_compress_threads` 线程创建后将进入死循环，等待后续创建的 `data_copy_thread` 线程调用 `compress_write` 方法进行解锁。

`xtrabackup_compress_threads` 线程数默认为 1，用户可通过参数 `--compress-thread` 设置。

**Step 2\. data_copy_thread 创建**

`os_thread_create(data_copy_thread_func, data_threads + i, &data_threads[i].id);` 创建 `data_copy_thread`。

`data_copy_thread` 线程数量默认为 1，用户可通过参数 `--parallel` 设置。

**Step 3\. 线程协作**

InnoDB 数据压缩备份过程：

1. 首先根据用户 `--parallel` 参数，创建指定数量的 `data_copy_thread` 线程，并注册回调函数 `data_copy_thread_func`，然后主进程进入 wait 状态，等待所有的 `data_copy_thread` 线程执行完毕。
2. `data_copy_thread_func` 方法通过 `data_copy_thread` 线程迭代器，遍历所有数据，直到迭代器返回 `NULL`，此时该线程的任务完成，退出循环，并尝试修改 `count` 计数，当 `count == 0` 时，表示所有线程均完成任务，主线程退出 wait 状态。

`data_copy_thread` 线程执行具体 copy data 任务的方法 `xtrabackup_copy_datafile`：

1. `xtrabackup_copy_datafile` 最终会循环调用 `compress_write` 方法。
2. `compress_write` 方法会先获取执行压缩任务的 `xtrabackup_compress_threads` 线程，并尝试获取其 `ctrl_mutex` 锁。
3. 获取到 `ctrl_mutex` 锁之后，会设置 `xtrabackup_compress_threads` 线程压缩的数据起始地址，总长度信息，**然后解锁该线程，由 `xtrabackup_compress_threads` 执行实际压缩任务**，待 `xtrabackup_compress_threads` 线程执行完毕后，修改文件剩余总字节数 len、以及下一次压缩的起始地址 ptr。
4. 压缩完成后，`data_copy_thread` 调用 `buffer_write` 尝试将 stream 数据写入 buffer，如果 buffer 已满则触发一次 flush 将数据输出到 STDOUT，清空 buffer。

![线程模型](https://i.loli.net/2021/04/12/QnUq3dzLax8JsEF.png)

### 数据落盘机制

1. `data_copy_thread` 线程调用 `compress_write` 方法并等待压缩完成。
2. `xtrabackup_compress_threads` 线程完成压缩后，修改 `thd->data_avail` 值为 `FALSE`，通知 `data_copy_thread` 继续进行下一步工作。
3. `data_copy_thread` 调用 `buffer_write` 方法将压缩数据转化为 `xbstream` 格式并写入到 buffer 中。
4. 最后到 `xb_stream_write_data` 方法将数据写入到 buffer 中，若 buffer 已满（chunk buffer 大小由 `XB_STREAM_MIN_CHUNK_SIZE` 宏定义，默认为 10 MB）则执行 `xb_stream_flush` 方法将数据提前刷新到 STDOUT，最后调用 `xb_stream_write_chunk` 对数据进行处理并刷新到 STDOUT。
5. 最后由重定向符 > 将输出到 STDOUT 中的数据写入到磁盘中，完成数据落盘。
