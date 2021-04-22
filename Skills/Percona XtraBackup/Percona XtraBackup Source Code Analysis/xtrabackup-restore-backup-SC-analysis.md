# Percona XtraBackup 全量恢复过程源码解析

利用 `innobackupex` 工具实现全量压缩备份并转化为 xbstream 流格式后，会在指定路径生成 `.xbstream` 流压缩文件，通过该文件可实现数据库的全量恢复。

通过 `.xbstream` 文件恢复数据库，有以下 4 个步骤：

1. XBSTREAM 流提取
2. 解压文件
3. 准备备份
4. 恢复备份

本文主要针对 **解压文件** 步骤的源码进行解析。

**注意：本文内容基于 percona-xtrabackup-2.4.8。**

目录：

- [Percona XtraBackup 全量恢复过程源码解析](#percona-xtrabackup-全量恢复过程源码解析)
  - [XBSTREAM 流提取](#xbstream-流提取)
  - [解压文件](#解压文件)
    - [解压过程](#解压过程)
    - [调用关系](#调用关系)
    - [关键流程解析](#关键流程解析)
      - [多线程模型](#多线程模型)
      - [执行解压](#执行解压)
  - [准备备份](#准备备份)
  - [恢复备份](#恢复备份)

## XBSTREAM 流提取

利用 `xbstream` 工具进行流文件提取，命令示例如下：

```bash
xbstream -x < /data/backups/innobackupextest.xbstream -C /data/backup_qp/
```

**参数说明：**

* `-x < /data/backups/innobackupextest.xbstream` ：
  * `-x`，从标准输入流中提取文件内容并输出到当前磁盘路径中。
  * `< /data/backups/innobackupextest.xbstream`，通过输入重定向符将指定路径 xbstream 文件作为输入传递给 xbstream 命令。
* `-C /data/backup_qp/` ：改变提取出的文件的磁盘输出路径。

**注意：** `-x`，除 `<` 输入重定向方式获取输入之外，还可以通过 `|` 重定向符，配置其他命令实现流文件输入，示例：

```bash
# 先通过 aes 算法对 xbstream 进行解密，再将解密内容重定向到 xbstream 命令
openssl enc -d -aes-256-cbc -salt -pass pass:123456 -in innobackupextest.xbstream.encrypt | xbstream -x -C /data/backup_qp
```

**命令作用：**

该命令提取 `/data/backups/innobackupextest.xbstream` 中的文件内容，并将其输出到 `/data/backup_qp/` 路径。

## 解压文件

innobackupex 工具进行解压的命令示例如下：

```bash
innobackupex --decompress --parallel=4 /data/backup_qp
```

**参数说明：**

* `--decompress` ：解压所有以 `.qp` 为后缀的压缩文件。
* `--parallel=4` ：指定用于执行解压任务的线程创建数量。
* `/data/backup_qp` ：待解压文件根目录。

**命令作用：**

该命令将解压 `/data/backup_qp` 目录中的所有 `.qp` 压缩文件并保存到本地。

**注意：** 该命令不会删除原压缩文件，如需解压后删除原压缩文件，可添加 `--remove-original` 参数。额外的，如果后续使用 `--copy-back` 或 `--move-back` 参数，将仅会复制解压后的文件，原压缩文件是否删除不会产生影响。

### 解压过程

TODO 时序图以及介绍

### 调用关系

![innobackupex restore backup 调用关系图](https://i.loli.net/2021/04/22/VWlAY85hyLod9Gq.png)

### 关键流程解析

#### 多线程模型

```cpp
ret = run_data_threads(it, decrypt_decompress_thread_func,
		xtrabackup_parallel ? xtrabackup_parallel : 1);
```

```cpp
for (i = 0; i < n; i++) {
    data_threads[i].it = it;
    data_threads[i].n_thread = i + 1;
    data_threads[i].count = &count;
    data_threads[i].count_mutex = &count_mutex;
    os_thread_create(func, data_threads + i, &data_threads[i].id);
}

/* Wait for threads to exit */
while (1) {
    os_thread_sleep(100000);
    mutex_enter(&count_mutex);
    if (count == 0) {
        mutex_exit(&count_mutex);
        break;
    }
    mutex_exit(&count_mutex);
}

mutex_free(&count_mutex);

ret = true;
for (i = 0; i < n; i++) {
    ret = data_threads[i].ret && ret;
    if (!data_threads[i].ret) {
        msg("Error: thread %u failed.\n", i);
    }
}

ut_free(data_threads);

return(ret);
```

#### 执行解压

```cpp
// Step 1. cat 命令读取 filepath 文件，并输出到 STDOUT（标准输出）
cmd << "cat " << filepath;

// 判断是否为以 .xbcrypt 结尾的加密文件，若是则执行解密过程
if (ends_with(filepath, ".xbcrypt") && opt_decrypt) {
    cmd << " | xbcrypt --decrypt --encrypt-algo="
        << xtrabackup_encrypt_algo_names[opt_decrypt_algo];
    if (xtrabackup_encrypt_key) {
        cmd << " --encrypt-key=" << xtrabackup_encrypt_key;
    } else {
        cmd << " --encrypt-key-file="
            << xtrabackup_encrypt_key_file;
    }
    dest_filepath[strlen(dest_filepath) - 8] = 0;
    message << "decrypting";
    needs_action = true;
}

// 判断是否为以 .qp 结尾的压缩文件，若是则执行解压过程
if (opt_decompress
    && (ends_with(filepath, ".qp")
    || (ends_with(filepath, ".qp.xbcrypt")
        && opt_decrypt))) {
    /* Step 2. 通过 | 重定向符将 cat 命令读取内容以标准输入方式传递给 qpress 程序，进行解压
    参数说明：
        -d 解压
        -i 从 STDIN（标准输入）中读取
        -o 输出到 STDOUT（标准输出）*/
    cmd << " | qpress -dio ";

    // 去掉 .qp 后缀
    dest_filepath[strlen(dest_filepath) - 3] = 0;
    if (needs_action) {
        message << " and ";
    }
    message << "decompressing";
    needs_action = true;
}

/* Step 3. 通过 > 输出重定向符，将解压后的内容输出到当前目录（即 .qp 文件原目录）
最终等价于：cat filepath | qpress -dio > dest_filepath */
cmd << " > " << dest_filepath;
```

## 准备备份

```bash
innobackupex --apply-log /data/backup_qp --redo-only
```

**参数说明：**

* `--apply-log`：通过应用同级目录下的 `xtrabackup_logfile` 事务日志文件来准备备份。
* `--redo-only`：如果该全量备份将作为后续增量备份的基础备份或者后续需要将某个增量备份应用到该全量备份，则需要该参数。该参数将强制 xtrabackup 跳过 “rollback” 阶段（回滚未提交事务），只执行 “redo” 阶段。

**命令作用：**

在数据文件准备好之前，它们在时间点上是不一致的，因为它们是在程序运行过程中的不同时间复制的，而且它们可能在此期间发生了更改。如果尝试用这些数据文件启动 InnoDB，InnoDB 会检测到损坏并崩溃。而该命令能够使文件恢复到一个一致性位点（即，备份过程中执行 FTWRL 的时间点），从而使 InnoDB 能够正常启动。

## 恢复备份

在数据文件准备好之后，停止数据库，并清空原数据库文件，将备份文件移动到 mysql 数据目录下，再修改文件权限，然后启动数据库即可恢复备份。
