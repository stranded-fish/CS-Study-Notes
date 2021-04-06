# xtrabackup 2.4.8 源码解析

**1\. innobackupex 和 xtrabackup**

2.3 之后 innobackupex 的功能已经全部集合到 xtrabackup 里面了，
为了兼容性，2.4.8 仍然保留了 innobackupex 可执行文件，innobackupex 可执行文件现在为软链接指向 xtrabackup

在 main func 中的 handle options 阶段，调用 handle_options() 方法，在该方法中通过 extern "C" const char* my_progname; 
my_progname 参数获取程序名。

判断调用程序名是否为 innobackupex，若是则模拟执行 innobackupex 脚本：

```c
if (strcmp(base_name(my_progname), INNOBACKUPEX_BIN_NAME) == 0 &&
    argc_client > 0) {
    /* emulate innobackupex script */
    innobackupex_mode = true;
    if (!ibx_handle_options(&argc_client, argv_client)) {
    exit(EXIT_FAILURE);
    }
}
```

ibx_handle_options() 方法将进一步调用 handle_options() 方法处理单个参数。处理完成后，设置 ibx_mode

```c
if (handle_options(argc, argv, ibx_long_options, ibx_get_one_option)) {
    return(false);
}

if (opt_ibx_apply_log) {
    ibx_mode = IBX_MODE_APPLY_LOG;
} else if (opt_ibx_copy_back) {
    ibx_mode = IBX_MODE_COPY_BACK;
} else if (opt_ibx_move_back) {
    ibx_mode = IBX_MODE_MOVE_BACK;
} else if (opt_ibx_decrypt || opt_ibx_decompress) {
    ibx_mode = IBX_MODE_DECRYPT_DECOMPRESS;
} else {
    ibx_mode = IBX_MODE_BACKUP;
}
```

main 方法中判断为 innobackupex 模式后，会调用其初始化方法，ibx_init() 函数，ibx_init() 函数会将之前 ibx_handle_options() 方法获取的参数信息传递给 xtrabackup 属性。

```c
switch (ibx_mode)
{
case IBX_MODE_APPLY_LOG:
    xtrabackup_prepare = TRUE;
    if (opt_ibx_redo_only)
    {
        xtrabackup_apply_log_only = TRUE;
    }
    xtrabackup_target_dir = ibx_position_arg;
    run = "apply-log";
    break;
case IBX_MODE_BACKUP:
    xtrabackup_backup = TRUE;
    xtrabackup_target_dir = ibx_backup_directory;
    if (opt_ibx_include != NULL)
    {
        xtrabackup_tables = opt_ibx_include;
    }
    run = "backup";
    break;
case IBX_MODE_COPY_BACK:
    xtrabackup_copy_back = TRUE;
    xtrabackup_target_dir = ibx_position_arg;
    run = "copy-back";
    break;
case IBX_MODE_MOVE_BACK:
    xtrabackup_move_back = TRUE;
    xtrabackup_target_dir = ibx_position_arg;
    run = "move-back";
    break;
case IBX_MODE_DECRYPT_DECOMPRESS:
    xtrabackup_decrypt_decompress = TRUE;
    xtrabackup_target_dir = ibx_position_arg;
    run = "decrypt and decompress";
    break;
default:
    ut_error;
}
```

最后 main 方法通过判断 xtrabackup 属性，并执行相应的 task

```c
/* --backup */
if (xtrabackup_backup)
    xtrabackup_backup_func();

/* --stats */
if (xtrabackup_stats)
    xtrabackup_stats_func();

/* --prepare */
if (xtrabackup_prepare)
    xtrabackup_prepare_func();

/* --copy-back || --move-back */
if (xtrabackup_copy_back || xtrabackup_move_back) {
    if (!check_if_param_set("datadir")) {
        msg("Error: datadir must be specified.\n");
        exit(EXIT_FAILURE);
    }
    if (!copy_back())
        exit(EXIT_FAILURE);
}

/* --decrypt || --decompress */
if (xtrabackup_decrypt_decompress && !decrypt_decompress()) {
    exit(EXIT_FAILURE);
}
```

**2\. backup - xtrabackup_backup_func();**

在 handle options 阶段，得到 xtrabackup_backup 为 true 后，main func 执行 xtrabackup_backup_func(); 方法，执行备份任务。

```c
/* --backup */
if (xtrabackup_backup) 
    xtrabackup_backup_func();
```

2.1 cd to datadir

cd 到数据库数据目录（默认：/var/lib/mysql 下），将其作为工作目录

2.2 initialize components

innodb_init_param()

2.3 init 相关内容

Initializes the synchronization primitives, memory system, and the thread
local storage.

srv_general_init();

Initializes the data structures used by ut_crc32*(). Does not do any
allocations, would not hurt if called twice, but would be pointless.

ut_crc32_init();

xb_keyring_init(xb_keyring_file_data);

xb_filters_init()

2.4 log_init()

log_init(); 相关

2.5 创建额外 LSN 如果之前不存在

2.6 创建目标 dir 如果不存在

**2.7 创建线程复制日志**

从最新的 checkpoint 点开始顺序拷贝 redo 日志；

并在这个阶段，初始化 datasink `xtrabackup_init_datasinks();`

**2.8 创建线程复制数据**

启动 ibd 数据拷贝线程，并 **等待** 创建的线程完成复制任务。

**2.9 停止复制日志线程**



