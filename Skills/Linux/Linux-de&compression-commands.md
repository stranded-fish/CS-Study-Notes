# Linux 解压 & 压缩命令总结

该教程主要针对：**Linux 常用解压 & 压缩命令** 学习。

目录：

- [Linux 解压 & 压缩命令总结](#linux-解压--压缩命令总结)
  - [tar](#tar)
    - [语法](#语法)
    - [参数](#参数)
    - [实例](#实例)
  - [zip](#zip)
    - [语法](#语法-1)
    - [参数](#参数-1)
    - [实例](#实例-1)
  - [参考链接](#参考链接)

## tar

tar（全称：tape archive）命令用于将多个文件进行归档打包，同时还可以调用 gzip / bzip / compress 对归档进行压缩。

* 打包：打包指将一大堆文件或目录变成一个总的文件；
* 压缩：将一个大的文件通过一些压缩算法变成一个小文件。

### 语法

```bash
tar [参数...] [归档文件] [源文件...]
```

### 参数

参数                         | 说明
-----------------------------|------------------------------------
**-c, --create**             | 创建一个新归档。
**-x, --extract**            | 解压文件。
**-t, --list**               | 列出归档内容。
**-r, --append**             | 追加文件至归档结尾。
**-u, --update**             | 仅追加比归档中副本更新的文件。
-C, --directory=DIR          | 解压到指定目录
--remove-files               | 压缩后删除原文件
-f, --file                   | 指定归档文件。
-z, --gzip, --ungzip         | 通过 `gzip` 压缩或解压，最后以 `.tar.gz` 结尾。
-j, --bzip2                  | 通过 `bzip2` 压缩或解压，最后以 `.tar.bz2` 结尾。
-Z, --compress, --uncompress | 通过 `compress` 过滤归档
-v, --verbose                | 详细地列出处理的文件。
-p                           | 保留绝对路径，即允许备份数据中含有根目录。
-P                           | 保留数据原来权限及属性。
--exclude = FILE             | 压缩过程中，不要将 FILE 打包。

更多参数见：`man tar`

**注意：**

* `-c` `-x` `-t` `-r` `-u` 这 5 个参数只能同时使用其中一个。
* `-f` 是必须有的参数，其后接归档文件名字，且只能是最后一个参数。

### 实例

**eg 1. 归档 & 压缩**

```bash
tar -cvf archive.tar *.c      # 归档所有 .c 文件
tar –czvf archive.tar.gz *.c  # 归档所有 .c 文件，并调用 gzip 进行压缩
tar –cjvf archive.tar.bz2 *.c # 归档所有 .c 文件，并调用 bzip2 进行压缩
tar –cZvf archive.tar.Z *.c   # 归档所有 .c 文件，并调用 compress 进行压缩

# 归档所有 .c 文件，并调用 gzip 进行压缩，压缩完成后删除原文件
tar --remove-files -czvf archive.tar.gz *.c
# tar -czvf archive.tar.gz *.c --remove-files 保证 -f 参数紧跟归档文件名即可
```

**eg 2. 展开 & 解压**

```bash
tar –xvf file.tar      # 展开 tar
tar -xzvf file.tar.gz  # 解压 tar.gz
tar -xjvf file.tar.bz2 # 解压 tar.bz2
tar –xZvf file.tar.Z   # 解压 tar.Z

# 解压该压缩文件到指定目录
tar -C ./tmp -xzvf file.tar.gz
# tar -xzvf file.tar.gz -C ./tmp 保证 -f 参数紧跟归档文件名即可
```

**eg 3. 查看归档文件**

```bash
# 详细列举归档文件 archive.tar 中的所有文件。
tar -tvf archive.tar
```

## zip

zip 是个应用广泛的压缩程序，压缩后的文件后缀名为 `.zip`。

### 语法

```bash
zip | unzip [参数...] [压缩文件] [源文件...]
```

### 参数

**zip 压缩：**

参数     | 说明
---------|------------------------
-r       | 递归压缩目录。
-m       | 文件压缩后，删除原始文件。
-v       | 详细地列出处理的文件。
-q       | 压缩过程不显示命令的执行过程。
-n [1-9] | 压缩级别，数字 1~9，数字越大压缩效果越好。
-u       | 仅追加比归档中副本更新的文件。

**unzip 解压：**

参数        | 说明
------------|-------------------------
-d dir      | 将压缩文件解压到目录 dir 下。
-n          | 解压时并不覆盖已存在的文件。
-o          | 解压时覆盖已存在文件，且无需确认。
-v          | 查看压缩文件的详细信息。
-t          | 测试压缩文件是否损坏。
-x fileList | 解压文件，但不包含 fileList 中的文件。

更多参数见：`man zip | unzip`

### 实例

**eg 1. 压缩**

```bash
zip test.zip *.c     # 压缩所有 .c 文件
zip -r dir1.zip dir1 # 递归压缩 dir1 目录
zip -m test.zip *.c  # 压缩所有 .c 文件，并删除原文件
```

**eg 2. 解压**

```bash
unzip test.zip          # 解压文件到当前目录
unzip -d /tmp/ test.zip # 解压文件到指定目录
```

**eg 3. 查看压缩文件**

```bash
unzip -v archive.zip # 查看压缩文件详细信息
```

## 参考链接

* [Linux tar](https://wangchujiang.com/linux-command/c/tar.html)
* [【Linux】tar命令详解](https://segmentfault.com/a/1190000021112003)
* [Linux zip命令](https://www.runoob.com/linux/linux-comm-zip.html)
* [zip命令：压缩文件或目录&&unzip命令：解压zip文件](https://www.cnblogs.com/xinghen1216/p/11308006.html)
