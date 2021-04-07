# Xtrabackup 压缩全过程源码解析

TODO Xtrabackup 压缩全过程源码解析

版本：2.4.8

目录：

- [Xtrabackup 压缩全过程源码解析](#xtrabackup-压缩全过程源码解析)
  - [概述](#概述)
  - [关键结构体定义](#关键结构体定义)
  - [流程解析](#流程解析)
  - [参考链接](#参考链接)

## 概述

TODO

压缩命令：innobackupex --user=root --password=MyNewPass4! --stream=xbstream --compress /data/backups/ > /data/innobackupextest.xbstream

参数说明：

innobackupex 需要与数据库交互，所以需要连接数据库，故需要登录数据库 user 、password。

--user=root ： 登录数据库 user
--password=MyNewPass4! ： 登录数据库 password
--stream=xbstream ： 将压缩后的 .qp 文件打包转换为 xbsream 流格式保存
--compress /data/backups/ > /data/innobackupextest.xbstream ： 选择压缩数据文件，并指定 xbsream 保存路径以及名称

作用：
将 innodb 表数据（.ibd...）和非 innodb 表数据（.frm...）备份，并压缩为 .qp 格式，并最终转化为 xbsream 流格式，保存到指定位置。

## 关键结构体定义

TODO

## 流程解析

**step 1. 获取参数**

xtrabackup.cc main func

```c
// 处理执行命令及参数
handle_options(argc, argv, &client_defaults, &server_defaults);

// 判断是否为 innobackupex 模式，若是则调动对应初始化方法
if (innobackupex_mode) {
    if (!ibx_init()) {
        exit(EXIT_FAILURE);
    }
}
```

```c
// 判断调用程序名是否为 innobackupex，若是则模拟执行 innobackupex 脚本
if (strcmp(base_name(my_progname), INNOBACKUPEX_BIN_NAME) == 0 &&
    argc_client > 0) {
    /* emulate innobackupex script */
    innobackupex_mode = true;
    if (!ibx_handle_options(&argc_client, argv_client)) {
        exit(EXIT_FAILURE);
    }
}
```

首先通过 handle_options() 方法获取命令运行参数，并根据运行程序名，判断调用程序名是否为 innobackupex。

如果为 innobackupex 则通过程序模拟执行 innobackupex 脚本。

同时根据输入参数 --compress，将 xtrabackup_backup 设置 TRUE，以调用相关备份方法。

**step 2. 启动线程**

**1\. 先启动 io_watching_thread 线程，开始 io 限流（单线程）；**

```c
os_thread_create(io_watching_thread, NULL,&io_watching_thread_id);
```

**2\. 再启动 redo 日志复制线程（单线程）；**

```c
os_thread_create(log_copying_thread, NULL, &log_copying_thread_id);
```

**3\. 最后启动数据复制线程（多线程）。**

```c
// 根据并行线程数，分配线程执行所需内存
	data_threads = (data_thread_ctxt_t *)
		ut_malloc_nokey(sizeof(data_thread_ctxt_t) *
                                xtrabackup_parallel);
	count = xtrabackup_parallel;

	// 创建互斥锁
	mutex_create(LATCH_ID_XTRA_COUNT_MUTEX, &count_mutex);

	// 创建并行数据复制线程
	for (i = 0; i < (uint) xtrabackup_parallel; i++) {
		data_threads[i].it = it; 	                // 设置文件迭代器
		data_threads[i].num = i+1; 	                // 设置线程序号
		data_threads[i].count = &count;             // 设置线程数
		data_threads[i].count_mutex = &count_mutex; // 设置互斥锁

		// 线程 3 数据复制线程
		os_thread_create(data_copy_thread_func, data_threads + i,
				 &data_threads[i].id);
				 
	}

	/* Wait for threads to exit */
	while (1) {
		os_thread_sleep(1000000);

		// 尝试获取锁，若已被占用，则等待
		mutex_enter(&count_mutex);

		// 当 count 为 0 时，表示所有线程 data_copy_thread_func 执行完毕，此时可退出循环
		if (count == 0) {

			// 释放锁，同时唤醒所有等待的线程，拿到锁的线程开始执行，其余线程继续等待
			mutex_exit(&count_mutex);
			break;
		}

		// 释放锁......
		mutex_exit(&count_mutex);
	}

	// 释放锁
	mutex_free(&count_mutex);
```

**step 3. data_copy_thread_func**

```c
while ((node = datafiles_iter_next(ctxt->it)) != NULL) {

	/* copy the datafile */
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

该方法将启动一个 while 循环，通过迭代器来进行遍历，直到指针为 NULL，每次循环将调用 xtrabackup_copy_datafile 方法执行 copy the datafile。

当遍历完成后，即表示该线程任务完成，该线程将尝试获取互斥锁，获取到互斥锁后，将 count 计数 - 1，当 count 计数为 0 时，表示所有线程完成任务。

**step 4. xtrabackup_copy_datafile**

```c
/* Setup the page write filter */
if (xtrabackup_incremental) {
	write_filter = &wf_incremental;
} else if (xtrabackup_compact) {
	write_filter = &wf_compact;
} else {
	// 走的这！！！！
	write_filter = &wf_write_through;
}


xb_write_filt_t wf_write_through = {
	&wf_wt_init,
	&wf_wt_process,
	NULL,
	NULL
};

```

因为输入参数为 --compress，在参数处理阶段，xtrabackup_compress 设置为了 TRUE，而其他模式均为初始化状态 FALSE，故这个地方将执行 `write_filter = &wf_write_through;`

```c
/* The main copy loop */
while ((res = xb_fil_cur_read(&cursor)) == XB_FIL_CUR_SUCCESS) {
	// TODO 这个地方调用了 compress_write 方法！！！！！！！！！！！！
	if (!write_filter->process(&write_filt_ctxt, dstfile)) {
		goto error;
	}
}
```

在 main copy loop 中，将不断调用 `write_filter->process` 方法，也即 `wf_write_through->process` 方法。

**step 5. wf_wt_process & ds_write**

```c
if (ds_write(dstfile, cursor->buf, cursor->buf_read)) {
	return(FALSE);
}

int
ds_write(ds_file_t *file, const void *buf, size_t len)
{
	return file->datasink->write(file, buf, len);
}
```

datasink 在之前已经初始化，此时调用的方法为 compress_write : ds_compress.c 方法。

**step 6. compress_write**

```c
// TODO 获取执行压缩任务的线程
threads = comp_ctxt->threads;
nthreads = comp_ctxt->nthreads;

ptr = (const char *) buf;
while (len > 0) {
uint max_thread;

/* Send data to worker threads for compression */
for (i = 0; i < nthreads; i++) {
	size_t chunk_len;

	thd = threads + i;

	pthread_mutex_lock(&thd->ctrl_mutex);

	chunk_len = (len > COMPRESS_CHUNK_SIZE) ?
		COMPRESS_CHUNK_SIZE : len;
	thd->from = ptr;
	thd->from_len = chunk_len;

	pthread_mutex_lock(&thd->data_mutex);
	
	// 解锁线程，准备执行 qlz_compress 压缩
	thd->data_avail = TRUE;

	pthread_cond_signal(&thd->data_cond);
	pthread_mutex_unlock(&thd->data_mutex);

	len -= chunk_len;
	if (len == 0) {
		break;
	}
	ptr += chunk_len;
}
```

在 compress_write 方法中，需要设置一些线程所需必要环境参数，（线程在 datasink init 阶段已初始化，并一直在 while 死循环，等待必要事件发生），然后执行 `thd->data_avail = TRUE;`，解锁线程开始执行压缩。

线程解锁后，compress_worker_thread_func 方法，将调用 qlz_compress 函数实现最终的压缩。

## 参考链接
