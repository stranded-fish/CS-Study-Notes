# Percona XtraBackup 全量恢复过程源码解析

经过 Percona XtraBackup 全量压缩备份并转化为 xbstream 流格式后，会在指定路径生成 `.xbstream` 流压缩文件，利用该文件可实现数据库的全量恢复。

目录：

- [Percona XtraBackup 全量恢复过程源码解析](#percona-xtrabackup-全量恢复过程源码解析)
  - [Extract files from the stream](#extract-files-from-the-stream)
    - [提取命令](#提取命令)
  - [decompressing files](#decompressing-files)
    - [解压命令](#解压命令)
    - [解压过程](#解压过程)
    - [调用关系](#调用关系)
    - [流程解析](#流程解析)
  - [Preparing the backup](#preparing-the-backup)
    - [准备过程](#准备过程)
    - [调用关系](#调用关系-1)
    - [流程解析](#流程解析-1)
  - [Restoring the backup](#restoring-the-backup)

## Extract files from the stream

### 提取命令

利用 xbstream 工具进行流文件提取，命令示例如下：

```bash
xbstream -x < /data/backups/innobackupextest.xbstream -C /data/backup_qp/
```

**参数说明：**

* `-x < /data/backups/innobackupextest.xbstream` ：
  * `-x`，从标准输入流中提取文件内容并输出到当前磁盘路径中。
  * `< /data/backups/innobackupextest.xbstream`，通过输入重定向符将指定路径 xbstream 文件作为输入传递给 xbstream 命令。
* `-C /data/backup_qp/` ：改变提取出的文件的磁盘输出路径。

注：`-x`，除 `<` 输入重定向方式获取输入之外，还可以通过 `|` 重定向符，配置其他命令实现流文件输入，示例：

```bash
# 先通过 aes 算法对 xbstream 进行解密，再将解密内容重定向到 xbstream 命令
openssl enc -d -aes-256-cbc -salt -pass pass:123456 -in innobackupextest.xbstream.encrypt | xbstream -x -C /data/backup_qp
```

**命令作用：**

该命令提取 `/data/backups/innobackupextest.xbstream` 中的文件内容，并将其输出到 `/data/backup_qp/` 路径。

## decompressing files

### 解压命令

innobackupex 工具进行解压的命令示例如下：

```bash
innobackupex --decompress --parallel=4 /data/backup_qp
```

**参数说明：**

* `--decompress` ：解压所有以 `.qp` 为后缀的压缩文件。
* `--parallel=4` ：指定用于 data transfer 的 data copy thread 创建数量。
* `/data/backup_qp` ：待解压文件路径。

**命令作用：**

该命令将 `/data/backup_qp` 目录中的所有 `.qp` 压缩文件进行解压并输出到本地。

### 解压过程

TODO 时序图以及介绍

main()
  decrypt_decompress()

    my_setwd() // cd to backup directory
    ds_create(".", DS_TYPE_LOCAL); // copy the rest of tablespaces
    run_data_threads(it, decrypt_decompress_thread_func,xtrabackup_parallel ? xtrabackup_parallel : 1); // 创建线程并等待
      os_thread_create(func, data_threads + i, &data_threads[i].id);
        decrypt_decompress_thread_func 方法
          datadir_node_init(&node);
          datadir_iter_next(ctxt->it, &node)
          decrypt_decompress_file(node.filepath,ctxt->n_thread)
      os_thread_sleep(100000);

/root/percona-xtrabackup-release-2.4.8/storage/innobase/xtrabackup/src/innobackupex --decompress --parallel=7 --compress-threads=5 /data/backup_qp

pzstd -d -f -p 4 hello.c.zst -o hello.c

-d decompress
-f force
-p multi thread
-o store into a specific file

### 调用关系

TODO 函数调用关系图及介绍

### 流程解析

TODO 关键过程解析

## Preparing the backup

**参数说明：**

**命令作用：**

### 准备过程

### 调用关系

### 流程解析

## Restoring the backup

**参数说明：**

**命令作用：**
