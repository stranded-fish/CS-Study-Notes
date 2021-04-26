# Percona XtraBackup 全量备份过程源码解析

本文主要介绍 `innobackupex` 命令，在设置参数 `--compress`、`--stream=xbstream` 情况下对数据进行压缩并转化为 xbstream 流格式备份的全过程源码解析。

**注意：本文内容基于 percona-xtrabackup-2.4.8。** 2.4.8 相较于旧版有较大改动：innobackupex 功能全部集成到了 xtrabackup 里，但考虑到兼容性，仍保留了 innobackupex 作为 xtrabackup 的一个软链接。

目录：

- [Percona XtraBackup 全量备份过程源码解析](#percona-xtrabackup-全量备份过程源码解析)
  - [备份命令](#备份命令)
  - [备份过程](#备份过程)
  - [调用关系](#调用关系)
  - [关键结构体定义](#关键结构体定义)
  - [关键流程解析](#关键流程解析)
    - [输入参数 与 innobackupex 兼容性处理](#输入参数-与-innobackupex-兼容性处理)
    - [多线程模型](#多线程模型)
    - [数据落盘机制](#数据落盘机制)

## 备份命令

`innobackupex` 工具进行全量备份的命令示例如下：

```bash
innobackupex --user=root --password=xxxxxxxx --parallel=4 --stream=xbstream --compress --compress-threads=4 /data/backups/ > /data/innobackupextest.xbstream
```

**参数说明：**

* `--user=root` | `--password=xxxxxxxx` ：待备份数据库的登录用户名与密码。
  由于 innobackupex 在备份过程中需要向 mysqld server 发送命令进行交互，如加 MDL 锁、加读锁（FTWRL）、获取位点（SHOW SLAVE STATUS）等，故需要数据库登录信息。
* `--parallel=4` ：指定用于 data transfer 的 data copy thread 创建数量。
* `--stream=xbstream` ：指定进行流备份的格式，一般选择 xbstream 格式。
* `--compress` ：指定进行压缩备份。
* `--compress-threads=4` ：指定用于执行压缩任务的 xtrabackup compress thread 创建数量。
* `/data/backups/ > /data/innobackupextest.xbstream` ：指定保存目录路径以及流文件名称。

**命令作用：**

该命令将从数据库默认存储路径 `/var/lib/mysql` 中读取 innodb 表数据（`.ibd`...）和 非 innodb 表数据（`.frm`...），并调用 qpress 压缩算法将其压缩为 `.qp` 格式，然后合并为一个 xbstream 流格式文件，并保存到 `/data/backups/` 目录。

## 备份过程

![innobackupex full backup 时序图](https://i.loli.net/2021/04/10/JZkwHeO2v1f567D.png)

1. 由于 2.4.8 版本，`innobackupex` 仅作为 `xtrabackup` 的一个软链接，在 `xtrabackup` 启动后，在参数处理阶段会根据程序名执行不同的操作，从而将 `innobackupex` 参数适配成 `xtrabackup` 形式以兼容旧版。最终等价于直接运行 `xtrabackup` 命令。
2. `xtrabackup` 首先将根据之前参数处理阶段获取到的参数，初始化对应的 datasink，由于选择了 `--compress` 压缩备份，故在此阶段会初始化 compress datasink，同时 **创建 `compress threads`（多线程）**。
3. 完成 datasink 初始化后，`xtrabackup` 将首先 **创建 `io watching thread`（单线程）** 负责 io 节流。然后再 **创建 `log copying thread`（单线程）** 负责拷贝 redo 日志，`log copying thread` 将从当前最新的 checkpoint 点开始复制。最后 **创建 `data copying threads`（多线程）并等待**，`data copying threads` 主要负责获取待压缩文件指针以及压缩内容的地址信息并将其传递给 `compress threads`，然后通知其开始压缩。
4. 待 InnoDB 数据备份完成，`xtrabackup` 将 **执行 `FLUSH TABLES WITH READ LOCK`（FTWRL）**，获取一致性位点，并强制 MyISAM 表缓存落盘，然后开始备份非 InnoDB 数据，这个过程与备份 InnoDB 数据类似，同样会经历压缩、序列化等步骤。
5. 当非 InnoDB 数据备份完成后，`log copying thread` 将停止拷贝 redo 日志。
6. 日志拷贝停止后，`xtrabackup` 将会对数据库进行解锁，**执行 `UNLOCK TABLES`**，然后销毁 datasink 并停止 `io watching thread`。
7. 最后 `xtrabackup` 将打印复制的事务日志 LSN 范围，并进行相关收尾工作。

## 调用关系

![innobackupex full backup 调用关系图](https://i.loli.net/2021/04/23/TO2IDQokrVR79zP.png)

## 关键结构体定义

**数据池类型：**

```cpp
struct datasink_struct {
    ds_ctxt_t *(*init)(const char *root); 			           // 初始化数据池
    ds_file_t *(*open)(ds_ctxt_t *ctxt, const char *path, MY_STAT *stat);  // 打开数据池文件
    int (*write)(ds_file_t *file, const void *buf, size_t len);            // 向数据池写入数据
    int (*close)(ds_file_t *file); 			                   // 关闭数据池，并将缓存中的数据输出到管道
    void (*deinit)(ds_ctxt_t *ctxt);                                       // 销毁数据池
};

/* 支持数据池类型 */
typedef enum {
    DS_TYPE_STDOUT,   // 标准输出数据池
    DS_TYPE_LOCAL,    // 本地数据池
    DS_TYPE_ARCHIVE,  // 归档数据池
    DS_TYPE_XBSTREAM, // xbstream 数据池
    DS_TYPE_COMPRESS, // 压缩数据池
    DS_TYPE_ENCRYPT,  //  加密数据池
    DS_TYPE_DECRYPT,  // 解密数据池
    DS_TYPE_TMPFILE,  // 临时文件数据池
    DS_TYPE_BUFFER    // 缓冲数据池
} ds_type_t;
```

数据池由 `ds_create()` 方法通过不同 `type` 参数设置并作为 `ds_ctxt_t` 结构体成员返回。通过给规格相同的函数指针赋值不同的地址，来实现类似于 Java 多态的效果。

**数据池环境：**

```cpp
typedef struct ds_ctxt {
    datasink_t	*datasink; 	    // 数据池类型，决定数据池执行的方法
    char 		*root;      // 数据池执行根地址
    void		*ptr;       // 数据池执行环境
    struct ds_ctxt	*pipe_ctxt; // 数据池管道，即当前数据池的输出目的地
} ds_ctxt_t;
```

数据池环境由 `ds_create()` 方法创建并返回。其中：

* `ptr` 用于指向该数据池具体执行环境，如 `compress_ds_data->ptr` 指向 `ds_compress_ctxt_t`，而 `ds_compress_ctxt_t` 包含执行具体压缩任务的线程变量。
* `pipe_ctxt` 指向下一数据池，如 `compress_ds_data->pipe_ctxt` 指向 `buffer_ds`，而 `buffer_ds` 数据池的 `pipe_ctxt` 又将指向 `xbstream_ds`。

**数据池文件环境：**

```cpp
typedef struct {
    void    *ptr;             // 执行文件，即具体的数据文件地址
    char    *path;            // 输出路径
    datasink_t  *datasink;    // 数据池类型，决定数据池执行的方法
} ds_file_t;
```

数据池文件环境由 `ds_open()` 方法创建并返回。其中：

* `ptr` 用于指向具体文件类型，如指向 `ds_compress_file_t`，而 `ds_compress_file_t` 又包含打开的目标数据池文件 `dest_file`、`ds_compress_ctxt_t` 以及该文件已处理字节数。

**数据流传递方向：**

通过 `ds_file_t->ptr` 指针获取具体数据文件，再通过数据文件 `dest_file` 指针定位到下一个管道：

![数据流图](https://i.loli.net/2021/04/12/BbPwzFmi9qOMWCG.png)

## 关键流程解析

### 输入参数 与 innobackupex 兼容性处理

自版本 2.3 起，`xtrabackup` \ `innobackupex` 命令入口均为 xtrabackup main 方法。

```cpp
/* main() : xtrabackup.cc */

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

**1\.** `xtrabackup_init_datasinks();` 初始化数据池的过程中调用 `ds_create(xtrabackup_target_dir, DS_TYPE_COMPRESS);` 方法；
**2\.** `ds_create()` 方法根据 `DS_TYPE_COMPRESS` 参数调用 `compress_init()` 方法创建 `xtrabackup_compress_threads` 线程；

```cpp
/* compress_init() : ds_compress.c */

// 创建多个 compress 线程，线程数量 xtrabackup_compress_threads 由 --compress-threads 参数指定
threads = create_worker_threads(xtrabackup_compress_threads);
if (threads == NULL) {
    msg("compress: failed to create worker threads.\n");
    return NULL;
}

ctxt = (ds_ctxt_t *) my_malloc(PSI_NOT_INSTRUMENTED,
                sizeof(ds_ctxt_t) +
                sizeof(ds_compress_ctxt_t),
                MYF(MY_FAE));

compress_ctxt = (ds_compress_ctxt_t *) (ctxt + 1);
compress_ctxt->threads = threads;                      // 线程环境变量
compress_ctxt->nthreads = xtrabackup_compress_threads; // 线程数量

ctxt->ptr = compress_ctxt;
ctxt->root = my_strdup(PSI_NOT_INSTRUMENTED, root, MYF(MY_FAE));

return ctxt;
```

**3\.** `xtrabackup_compress_threads` 线程创建后将进入死循环，等待后续创建的 `data_copy_thread` 线程调用 `compress_write` 方法进行解锁。

```cpp
/* compress_worker_thread_func() : ds_compress.c */

while (1) {
    /* 信号量设置为 FALSE，同时发送 signal 使 data_copy_thread 收到信号停止阻塞，
    并进行下一步将之前一次循环压缩完成的数据写入到下一管道 */
    thd->data_avail = FALSE;
    pthread_cond_signal(&thd->data_cond);

    // 等待执行 compress_write 方法设置必要参数，并将 thd->data_avail 设置为 TRUE 解锁
    while (!thd->data_avail && !thd->cancelled) {
        pthread_cond_wait(&thd->data_cond, &thd->data_mutex);
    }

    if (thd->cancelled)
        break;

    /* 调用 qpress API 对块进行压缩
        thd->from      待压缩数据起始地址
        thd->to        压缩数据写入地址
        thd->from_len  待压缩数据字节数
        &thd->state    状态 */
    thd->to_len = qlz_compress(thd->from, thd->to, thd->from_len, &thd->state);

    thd->adler = adler32(0x00000001, (uchar *) thd->to, thd->to_len);
}
```

`xtrabackup_compress_threads` 线程数默认为 1，用户可通过参数 `--compress-threads` 设置。

**Step 2\. data_copy_thread 创建**

创建 `data_copy_thread`。

```cpp
/* xtrabackup_backup_func() : xtrabackup.cc */

// 根据并行线程数，分配线程执行所需内存，线程数量 xtrabackup_parallel 由 --parallel 参数指定
data_threads = (data_thread_ctxt_t *) ut_malloc_nokey(sizeof(data_thread_ctxt_t) * xtrabackup_parallel);

count = xtrabackup_parallel;

// 创建互斥锁
mutex_create(LATCH_ID_XTRA_COUNT_MUTEX, &count_mutex);

// 创建并行数据复制线程
for (i = 0; i < (uint) xtrabackup_parallel; i++) {
    data_threads[i].it = it; 	                // 设置文件迭代器（多线程共享）
    data_threads[i].num = i+1; 	                // 设置线程序号
    data_threads[i].count = &count;             // 设置线程计数（多线程共享）
    data_threads[i].count_mutex = &count_mutex; // 设置互斥锁（多线程共享）

    // 线程 3 数据复制线程
    os_thread_create(data_copy_thread_func, data_threads + i, &data_threads[i].id);
}
```

`data_copy_thread` 线程数量默认为 1，用户可通过参数 `--parallel` 设置。

**Step 3\. 线程协作**

![线程协作模型](https://i.loli.net/2021/04/22/YqUdfTLQgpvC81J.png)

**①** `main thread` 根据参数 **`--parallel`** 创建若干 `data copy threads`，并等待。
**②** `data copy threads` 首先获取一个待压缩文件的指针，然后执行 `while` 循环，每次对一部分文件进行压缩（长度为 `len`），然后 `compress_write` 方法会 **尝试获取 `xtrabackup compress threads` 的 `ctrl mutex`**（注意：一旦该 copy thread 获取到了 compress thread 的第一把锁，则会阻塞其他所有 copy thread，后续的 compress 锁（ctrl mutex）只能由该 copy thread 获取），并将该部分文件内容，再次分块（长度不超过由参数 **`--compress-chunk-size`** 设置的 `COMPRESS_CHUNK_SIZE`），并按地址顺序将每一块分配给一个 `xtrabackup compress thread` 进行压缩。（即，同一时刻，所有的 `xtrabackup compress threads` 将只能服务于一个 `data copy thread`）
**③** `xtrabackup compress threads` 收到待压缩内容的起始地址、块大小以及解锁信号后，开始执行压缩任务，完成任务之后设置信号量、发送 `signal` 通知给他分配任务的 `data copy thread`。
**④** `data copy thread` 会通过 `for` 循环按照顺序（之前分配压缩任务时的顺序）等待 `xtrabackup compress threads` 完成任务（即，如果压缩线程 2、3... 已完成压缩任务，但压缩线程 1 未完成，则只能等待压缩线程 1 完成后，才能继续下一步，因为要保证按照原始文件顺序写入压缩内容），收到完成信号之后，将该线程压缩后的内容写入到下一管道（buffer）中，**并释放该线程的 ctrl mutex**。待整个 `for` 循环完成后（即，表示当前 `data copy thread` 分配给 `xtrabackup compress threads` 的任务已经全部完成并按序写入下一管道），则由下一个获取到 `ctrl mutex` 的 `data copy thread` 重复步骤 **② - ④**。

**源码实现：**

**1\.** 首先根据用户 `--parallel` 参数，创建指定数量的 `data_copy_thread` 线程，并注册回调函数 `data_copy_thread_func`，然后主进程进入 wait 状态，等待所有的 `data_copy_thread` 线程执行完毕。
**2\.** `data_copy_thread_func` 方法通过迭代器，遍历数据，并获取文件指针（即，处理单位为一个文件），直到迭代器返回 `NULL`，表示该线程的任务完成，退出循环，并尝试修改 `count` 计数，当 `count == 0` 时，表示所有线程均完成任务，主线程退出 wait 状态。

```cpp
/* data_copy_thread_func() : xtrabackup.cc */

/* 通过 datafile 迭代器获取文件指针
多个 data_copy_threads 共用一个迭代器，通过互斥锁确保获取到不同文件指针 */
while ((node = datafiles_iter_next(ctxt->it)) != NULL) {

    // copy the datafile
    if(xtrabackup_copy_datafile(node, num)) {
        msg("[%02u] xtrabackup: Error: "
            "failed to copy datafile.\n", num);
        exit(EXIT_FAILURE);
    }
}

// 线程任务完成，尝试修改 count 计数
mutex_enter(ctxt->count_mutex); // 尝试获取锁，若已被占用，则等待
(*ctxt->count)--;               // count 计数 - 1（当 count 为 0 时，表示所有线程任务完成，父线程停止循环等待）
mutex_exit(ctxt->count_mutex);  // 释放锁
```

**3\.** `data_copy_thread` 线程执行具体 copy data 任务的方法 `xtrabackup_copy_datafile`：

**3.1** `xtrabackup_copy_datafile` 首先会循环调用 `xb_fil_cur_read` 方法，读取并验证源文件中的下一个块。

```cpp
/* xtrabackup_copy_datafile() : xtrabackup.cc */

while ((res = xb_fil_cur_read(&cursor)) == XB_FIL_CUR_SUCCESS) {
    ......
}
```

```cpp
/* Pages 的读缓冲区大小 (640 pages = 10M，在 page 大小为 16 K 的情况下) */
#define XB_FIL_CUR_PAGES 640

......

// Step 1. 读取下一个 batch
cursor->read_filter->get_next_batch(&cursor->read_filter_ctxt,
					&offset, &to_read);

// to_read：读取字节数，若为 0 则表示已读到文件结尾
if (to_read == 0LL) {
    return(XB_FIL_CUR_EOF);
}

/* Step 2. 判断是否大于 buf_size，若大于，则设置为 buf_size
buf_size 大小在 xb_fil_cur_open 阶段确定
buf_size = XB_FIL_CUR_PAGES * page_size */
if (to_read > (ib_uint64_t) cursor->buf_size) {
    to_read = (ib_uint64_t) cursor->buf_size;
}

// 合法性验证
xb_a(to_read > 0 && to_read <= 0xFFFFFFFFLL);

// Step 3. 判断 to_read 是否是 page_size 的整数倍，如果不是则通过添加 offset 使其为整数倍
if (to_read % cursor->page_size != 0 &&
    offset + to_read == (ib_uint64_t) cursor->statinfo.st_size) {

    if (to_read < (ib_uint64_t) cursor->page_size) {
        msg("[%02u] xtrabackup: Warning: junk at the end of "
            "%s:\n", cursor->thread_n, cursor->abs_path);
        msg("[%02u] xtrabackup: Warning: offset = %llu, "
            "to_read = %llu\n",
            cursor->thread_n,
            (unsigned long long) offset,
            (unsigned long long) to_read);

        return(XB_FIL_CUR_EOF);
    }

    to_read = (ib_uint64_t) (((ulint) to_read) &
                ~(cursor->page_size - 1));
}

// 验证 to_read 是否为 page_size 整数倍
xb_a(to_read % cursor->page_size == 0);

// Step 4. 根据 to_read 确定需要读取的 page 数
npages = (ulint) (to_read >> cursor->page_size_shift);
```

**3.2** 在读取到源文件中的下一个块后，调用 `compress_write` 方法。

```cpp
/* xtrabackup_copy_datafile() : xtrabackup.cc */

// The main copy loop
while ((res = xb_fil_cur_read(&cursor)) == XB_FIL_CUR_SUCCESS) {

    // 执行 compress_write : ds_compress.c 方法，传递压缩信息，解锁压缩线程，开始执行压缩任务
    if (!write_filter->process(&write_filt_ctxt, dstfile)) {
        goto error;
    }
}
```

**3.3** `compress_write` 方法会先获取执行压缩任务的 `xtrabackup_compress_threads` 线程，并尝试获取其 `ctrl_mutex` 锁。
**3.4** 获取到 `ctrl_mutex` 锁之后，会设置 `xtrabackup_compress_threads` 线程压缩的数据起始地址，总长度信息，**然后解锁该线程，由 `xtrabackup_compress_threads` 执行实际压缩任务**，待 `xtrabackup_compress_threads` 线程执行完毕后，修改文件剩余总字节数 `len`、以及下一次压缩的起始地址 `ptr`。

```cpp
/* compress_write() : ds_compress.c */

// 将待压缩数据信息传递给 compress thread 进行压缩
for (i = 0; i < nthreads; i++) {
    size_t chunk_len;

    // 按顺序遍历下一个线程
    thd = threads + i;

    /* 尝试获取线程 ctrl 互斥锁（直到该线程压缩完成，并将压缩内容写入管道后才会解锁）
    注意：一旦当前 copy thread 获取到了 compress 的第一把锁，则其他所有 copy thread 
    均会阻塞，并且后续的 compress 锁（ctrl mutex）只能由该 copy thread 获得 */
    pthread_mutex_lock(&thd->ctrl_mutex);

    /* 待压缩数据长度（最大不可超过 COMPRESS_CHUNK_SIZE）
    COMPRESS_CHUNK_SIZE 由参数 --compress-chunk-size 设置 */
    chunk_len = (len > COMPRESS_CHUNK_SIZE) ?
    COMPRESS_CHUNK_SIZE : len;    
    thd->from = ptr;  	          // 待压缩数据起始地址
    thd->from_len = chunk_len;    // 待压缩数据长度

    // 尝试获取线程 data 互斥锁
    pthread_mutex_lock(&thd->data_mutex);

    // 解锁线程，准备执行 qlz_compress 压缩
    thd->data_avail = TRUE;

    // 发送信号给 xtrabackup_compress_threads 使其脱离阻塞状态
    pthread_cond_signal(&thd->data_cond);

    // 解锁线程 data 互斥锁
    pthread_mutex_unlock(&thd->data_mutex);

    // 总压缩字节数减去刚才压缩的长度
    len -= chunk_len;
    if (len == 0) {
        break;
    }

    // 待压缩数据起始地址向前移动 chunk_len 长度
    ptr += chunk_len;
}
```

**3.5** 压缩完成后，`data_copy_thread` 调用 `buffer_write` 尝试将 stream 数据写入 buffer，如果 buffer 已满则触发一次 flush。

### 数据落盘机制

**1\.** `data_copy_thread` 线程调用 `compress_write` 方法并等待压缩完成。
**2\.** `xtrabackup_compress_threads` 线程完成压缩后，修改 `thd->data_avail` 值为 `FALSE`，通知 `data_copy_thread` 继续进行下一步工作。
**3\.** `data_copy_thread` 调用 `buffer_write` 方法将压缩数据写入到 buffer 中。

```cpp
/* compress_write() : ds_compress.c */

// 获取压缩后的数据，并将其转化为 stream 
for (i = 0; i <= max_thread; i++) {
    thd = threads + i;

    pthread_mutex_lock(&thd->data_mutex);

    // 等待线程完成压缩
    while (thd->data_avail == TRUE) {
        pthread_cond_wait(&thd->data_cond,
            &thd->data_mutex);
    }

    xb_a(threads[i].to_len > 0);

    // 该压缩文件已处理字节数 += 最近一次压缩的字节数    
    comp_file->bytes_processed += threads[i].from_len;

    /* to: 压缩后的数据起始地址  to_len: 压缩后的数据长度
    将压缩后的数据,写入下一个管道 (即:buffer 管道) */
    if (ds_write(dest_file, threads[i].to, threads[i].to_len)) {
    msg("compress: write to the destination stream "
        "failed.\n");
    return 1;
    }

    pthread_mutex_unlock(&threads[i].data_mutex);

    // 完成压缩数据写入，释放该压缩线程的 ctrl_mutex
    pthread_mutex_unlock(&threads[i].ctrl_mutex);
}
```

**4\.** 最后到 `xb_stream_write_data` 方法将转化为 xbstream 格式的数据写入到 buffer 中，若 buffer 已满（chunk buffer 大小由 `XB_STREAM_MIN_CHUNK_SIZE` 宏定义，默认为 10 MB）则执行 `xb_stream_flush` 方法将数据提前刷新到 STDOUT，最后调用 `xb_stream_write_chunk` 对数据进行处理并刷新到 STDOUT。
**5\.** 最后由重定向符 `>` 将输出到 STDOUT 中的数据写入到磁盘中，完成数据落盘。
