# Percona XtraBackup 安装与使用

Percona XtraBackup 是一个开源的 MySQL 热备份工具，利用该工具（2.4 版本）可以实现在 MySQL 5.5, 5.6 和 5.7 服务器上备份 InnoDB、XtraDB 和 MyISAM 表的数据，以及备份 Percona 服务器的 XtraDB MySQL。

Percona XtraBackup 特点：

* 快速可靠地完成备份。
* 备份期间不中断事务处理。
* 节省磁盘空间和网络带宽。
* 自动完成备份验证。
* 更快的恢复速度，以提升正常运行时间。

**注意：本文内容基于 percona-xtrabackup-2.4.8**

目录：

- [Percona XtraBackup 安装与使用](#percona-xtrabackup-安装与使用)
  - [Percona XtraBackup 安装](#percona-xtrabackup-安装)
    - [通过软件仓库安装](#通过软件仓库安装)
    - [通过二进制压缩文件安装](#通过二进制压缩文件安装)
    - [通过 rpm 或 apt 安装包安装](#通过-rpm-或-apt-安装包安装)
    - [通过编译源码安装](#通过编译源码安装)
  - [Percona XtraBackup 使用](#percona-xtrabackup-使用)
    - [innobackupex 备份周期 - 全量备份](#innobackupex-备份周期---全量备份)
      - [Create a backup](#create-a-backup)
      - [Preparing a backup](#preparing-a-backup)
      - [Restoring a Backup](#restoring-a-backup)
    - [innobackupex 备份周期 - 增量备份](#innobackupex-备份周期---增量备份)
      - [Creating an Incremental Backup](#creating-an-incremental-backup)
      - [Preparing the Incremental Backups](#preparing-the-incremental-backups)
      - [Restoring a Incremental Backup](#restoring-a-incremental-backup)
  - [参考链接](#参考链接)

## Percona XtraBackup 安装

Percona XtraBackup 有如下安装方式：

* 通过软件仓库安装。
* 通过二进制压缩文件安装。
* 通过 rpm 或 apt 安装包安装。
* 通过编译源码安装。

**注意：** 由于 Percona XtraBackup 默认使用 qpress 压缩算法来完成备份过程中的解压缩，为了能够进行压缩备份，还需额外安装 qpress 工具。

```bash
yum install qpress
```

### 通过软件仓库安装

Percona 为 yum 和 apt 仓库提供了软件源，可通过 yum 或 apt 包管理器便捷的安装和更新 Percona XtraBackup 及其依赖。

**Step 1.** 安装 percona-release 配置工具

```bash
yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```

**Step 2.** 测试仓库

确保可以从仓库中获取到所需安装包

```bash
yum list | grep percona
```

应有如下测试结果：

```
...
percona-xtrabackup-20.x86_64               2.0.8-587.rhel5             percona-release-x86_64
percona-xtrabackup-20-debuginfo.x86_64     2.0.8-587.rhel5             percona-release-x86_64
percona-xtrabackup-20-test.x86_64          2.0.8-587.rhel5             percona-release-x86_64
percona-xtrabackup-21.x86_64               2.1.9-746.rhel5             percona-release-x86_64
percona-xtrabackup-21-debuginfo.x86_64     2.1.9-746.rhel5             percona-release-x86_64
percona-xtrabackup-22.x86_64               2.2.13-1.el5                percona-release-x86_64
percona-xtrabackup-22-debuginfo.x86_64     2.2.13-1.el5                percona-release-x86_64
percona-xtrabackup-debuginfo.x86_64        2.3.5-1.el5                 percona-release-x86_64
percona-xtrabackup-test.x86_64             2.3.5-1.el5                 percona-release-x86_64
percona-xtrabackup-test-21.x86_64          2.1.9-746.rhel5             percona-release-x86_64
percona-xtrabackup-test-22.x86_64          2.2.13-1.el5                percona-release-x86_64
...
```

**Step 3.** 启用存储库

```bash
percona-release Enable -only tools release
```

**Step 4.** 安装 Percona XtraBackup

```bash
yum install percona-xtrabackup-24
```

### 通过二进制压缩文件安装

下表列出了 Linux 系统中通用的二进制压缩文件类型（可以自行选择所需的 Percona XtraBackup 版本及类型）。

Type    | Name                                                                      | Description
--------|---------------------------------------------------------------------------|-----------------------------------------------------------------------------------
Full    | percona-xtrabackup-<version number>-Linux.x86_64.glibc2.12.tar.gz         | Contains binaries, libraries, test files, and debug symbols
Minimal | percona-xtrabackup-<version number>-Linux.x86_64.glibc2.12-minimal.tar.gz | Contains binaries, and libraries but does not include test files, or debug symbols

下载 2.4.21 版本示例如下：

```bash
wget https://downloads.percona.com/downloads/Percona-XtraBackup-2.4/Percona-XtraBackup-2.4.21/binary/tarball/percona-xtrabackup-2.4.21-Linux-x86_64.glibc2.12.tar.gz

tar xvf percona-xtrabackup-2.4.21-Linux-x86_64.glibc2.12.tar.gz
```

解压成功后，可执行文件将位于 `/percona-xtrabackup-2.4.21-Linux-x86_64.glibc2.12/bin` 目录。

### 通过 rpm 或 apt 安装包安装

从官网 [下载页面](https://www.percona.com/downloads/Percona-XtraBackup-LATEST/) 下载符合自身操作系统所需的安装包，然后通过 `yum localinstall` 命令安装。

CentOS 7 安装示例如下：

```bash
wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.8/binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.8-1.el7.x86_64.rpm

yum localinstall percona-xtrabackup-24-2.4.4-1.el7.x86_64.rpm
```

### 通过编译源码安装

源代码可以从 [Percona XtraBackup Github](https://github.com/percona/percona-xtrabackup) 项目中获得。通过 `git clone` 命令克隆。然后切换到想要安装的发布分支，如 2.4。

```bash
git clone https://github.com/percona/percona-xtrabackup.git
cd percona-xtrabackup
git checkout 2.4
```

**注意：** 由于整个 git 仓库较大，且国内 `git clone` 速度较慢，容易断连。故建议直接下载所需版本的 zip 压缩包。

**Step 1.** 安装必要依赖

Percona XtraBackup 需要 GCC 5.3 或更高版本。

安装 `cmake` 和其他依赖：

```bash
sudo yum install cmake openssl-devel libaio libaio-devel automake autoconf \
bison libtool ncurses-devel libgcrypt-devel libev-devel libcurl-devel zlib-devel \
vim-common
```

为了能够安装 manual pages，安装 `python3-sphinx` 包:

```bash
sudo yum install python3-sphinx
```

**Step 2.** 执行 cmake

根目录执行：

```bash
cmake -DBUILD_CONFIG=xtrabackup_release -DWITH_MAN_PAGES=OFF
```

**注意：** 在执行上述命令时，可能发生如下错误：

![cmake 报错](https://i.loli.net/2021/04/18/OuJyXdrav9mUitA.png)

**解决方法：**

可通过错误提示解决或通过以下方式解决：

```bash
mkdir -p /usr/local/boost && cd /usr/local/boost
wget http://www.sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
tar -xvzf boost_1_59_0.tar.gz
rm boost_1_59_0.tar.gz
```

**Step 3.** 执行 make

TODO cmake make 命令相关内容及作用

```bash
make -j1 && make install
```

**注意：** 如果编译机器运行内存较小，可能发生 `internal compiler error: Killed` 错误。

**解决方法：**

TODO 以下方式的具体意义

1\. 创建分区文件，大小 2 G；

```bash
dd if=/dev/zero of=/swapfile bs=1k count=2048000
```

2\. 生成 swap 文件系统；

```bash
mkswap /swapfile
```

3\. 激活 swap 文件；

```bash
swapon /swapfile
```

4\. 设置重启系统自动加载。

```bash
/swapfile  swap  swap    defaults 0 0
```

## Percona XtraBackup 使用

Percona XtraBackup 安装完成后，工具集包含以下可执行文件：

bin/
├── xtrabackup
├── innobackupex -> xtrabackup
├── xbstream
├── xbcrypt
└── xbcloud

* `xtrabackup` ：用于非阻塞备份各种类型的数据库表（注意：发行时间较早的旧版 xtrabackup 只能用于备份 innoDB 数据库表，而较新版 xtrabackup 均可备份）。
* `innobackupex` ：为指向 xtrabackup 的软链接，通过在程序运行中判断程序名，执行不同的参数处理操作，以兼容旧版 innobackupex 命令行。
* `xbstream` ：以 XBSTREAM 格式序列化/反序列化文件。
* `xbcrypt` ：以 XBCRYPT 格式加密或解密文件。
* `xbcloud` ：在云服务上管理备份。

### innobackupex 备份周期 - 全量备份

使用 innobackupex 工具，进行全量压缩流式备份并恢复。

#### Create a backup

创建备份：

```bash
innobackupex --user=root --password=MyNewPass4! --parallel=4 --stream=xbstream --compress --compress-thread=4 /data/backups/ > /data/backups/innobackupextest.xbstream
```

参数说明：

* `--user=root` | `--password=MyNewPass4!` ：待备份数据库的登录用户名与密码。
  由于 innobackupex 在备份过程中需要向 mysqld server 发送命令进行交互，如加 MDL 锁、加读锁（FTWRL）、获取位点（SHOW SLAVE STATUS）等，故需要数据库登录信息。
* `--parallel=4` ：指定用于 data transfer 的 data copy thread 创建数量。
* `--stream=xbstream` ：指定进行流备份的格式，一般选择 xbstream 格式。
* `--compress` ：进行压缩备份。
* `--compress-thread=4` ：指定用于执行压缩任务的 xtrabackup compress thread 创建数量。
* `/data/backups/ > /data/backups/innobackupextest.xbstream` ：指定保存目录路径以及流文件名称。

#### Preparing a backup

**Step 1.** 从 XBSTREAM 流提取文件

在准备备份之前，首先需要将文件以 XBSTREAM 格式反序列化还原。

```bash
xbstream -x < /data/backups/innobackupextest.xbstream -C /data/backup_qp/
```

参数说明：

* `-x < /data/backups/innobackupextest.xbstream` ：
  * `-x` ：从标准输入流中提取文件内容并输出到当前磁盘路径中。
  * `< /data/backups/innobackupextest.xbstream` ：通过输入重定向符将指定路径 xbstream 文件作为输入传递给 xbstream 命令。
* `-C /data/backup_qp/` ：改变提取出的文件的磁盘输出路径。

**注意：** `-x`，除 `<` 输入重定向方式获取输入之外，还可以通过 `|` 重定向符，配置其他命令实现流文件输入，示例：

```bash
# 先通过 aes 算法对 xbstream 进行解密，再将解密内容重定向到 xbstream 命令
openssl enc -d -aes-256-cbc -salt -pass pass:123456 -in innobackupextest.xbstream.encrypt | xbstream -x -C /data/backup_qp
```

**Step 2.** 解压文件

解压所有备份文件。

```bash
innobackupex --decompress --parallel=4 /data/backup_qp
```

参数说明：

* `--decompress` ：解压所有以 `.qp` 为后缀的压缩文件。
* `--parallel=4` ：指定用于 data transfer 的 data copy thread 创建数量。
* `/data/backup_qp` ：待解压文件路径。

**注意：** 以上解压操作不会自动删除原压缩文件，如需解压后删除原压缩文件，可添加 `--remove-original` 参数。额外的，如果后续使用 `--copy-back` 或 `--move-back` 参数，将仅会复制解压后的文件，原压缩文件是否删除不会产生影响。

**Step 3.** 准备备份

```bash
innobackupex --apply-log /data/backup_qp --redo-only
```

参数说明：

* `--apply-log` ：通过应用同级目录下的 `xtrabackup_logfile` 事务日志文件来准备备份。
* `--redo-only` ：在准备基本完全备份和合并除最后一个增量以外的所有增量时，应该使用此选项。

#### Restoring a Backup

**Step 1.** 关闭数据库

```bash
service mysqld stop
```

**Step 2.** 删除原数据库文件

```bash
rm -rf /var/lib/mysql/*
```

**注意：** 保险起见，删除前，可先进行备份 `cp -r /var/lib/mysql /var/lib/mysql.backup`

**Step 3.** 将备份数据复制到数据库 datadir（由配置文件决定）

```bash
innobackupex --copy-back /data/backup_qp
```

**Step 4.** 修改文件权限

```bash
chown -R mysql:mysql /var/lib/mysql
```

**Step 5.** 重启数据库

```bash
service mysqld start
```

### innobackupex 备份周期 - 增量备份

使用 innobackupex 工具，进行增量压缩流式备份并恢复。

#### Creating an Incremental Backup

TODO

#### Preparing the Incremental Backups

TODO

#### Restoring a Incremental Backup

TODO

## 参考链接

* [Installing Percona XtraBackup 2.4](https://www.percona.com/doc/percona-xtrabackup/2.4/installation.html)
* [MySQL install and uninstall backup Percona Xtrabackup](https://www.programmersought.com/article/25122068037/)
* [gcc 编译出现 internal compiler error: Killed](https://blog.csdn.net/qq_29573053/article/details/69665996)
* [CentOS7 下启动、关闭、重启、查看MySQL服务](https://bbs.huaweicloud.com/blogs/150610)
