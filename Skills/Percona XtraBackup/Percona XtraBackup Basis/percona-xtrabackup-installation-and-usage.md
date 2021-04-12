# Percona XtraBackup 安装与使用

Percona XtraBackup 是一个开源的 MySQL 热备份工具，利用该工具（2.4 版本）可以实现在 MySQL 5.5, 5.6 和 5.7 服务器上备份 InnoDB, XtraDB 和 MyISAM 表的数据，以及备份 Percona 服务器的 XtraDB MySQL。

Percona XtraBackup 特点：

* 快速可靠地完成备份。
* 备份期间不中断事务处理。
* 节省磁盘空间和网络带宽。
* 自动完成备份验证。
* 更快的恢复速度，以提升正常运行时间。

## Percona XtraBackup 安装

Percona XtraBackup 安装方式如下：

* 通过软件仓库安装。
* 通过二进制压缩文件安装。
* 通过 rpm 或 apt 安装包安装。
* 通过编译源码安装。

### 通过软件仓库安装

Percona 为 yum 和 apt 仓库提供了软件源，可通过 yum 或 apt 包管理器便捷的安装和更新 Percona XtraBackup 及其依赖。

Centos 7 系统安装命令如下：

### 通过二进制压缩文件安装

### 通过 rpm 或 apt 安装包安装

安装：

https://www.percona.com/doc/percona-xtrabackup/2.4/installation/yum_repo.html#standalone-rpm

如果想要安装不同版本的直接 修改 wget 中的链接即可

例如：

wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.8/binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.8-1.el7.x86_64.rpm

卸载

https://blog.csdn.net/anzhen0429/article/details/76302158

yum remove percona-xtrabackup-24

### 通过编译源码安装

安装：https://blog.csdn.net/anzhen0429/article/details/7630215

根目录执行：

cmake -DBUILD_CONFIG=xtrabackup_release -DWITH_MAN_PAGES=OFF -DWITH_BOOST=/usr/local/boost && make -j1 && make install


https://www.programmersought.com/article/25122068037/

https://blog.csdn.net/qq_29573053/article/details/69665996

bug 1 cmake/boost.cmake:81 解决
bug 2 internal compiler error: Killed (program cc1plus) 内存不够，编译器被 kill 了 解决

步骤：

1. 下载指定版本源码 或者 下载整个 git 工程并切换至目标版本
2. 下载相关依赖
3. 项目根目录执行 cmake 命令
4. 可能会出现一些 bug 
5. 将可执行程序移动到 /usr/bin 目录




卸载：



## Xtrabackup 使用

Xtrabackup 安装完成后，工具集包含以下可执行文件：

bin/
├── innobackupex -> xtrabackup
├── xtrabackup
├── xbstream
├── xbcrypt
└── xbcloud

各可执行文件功能介绍如下：

![工具集介绍](https://i.loli.net/2021/04/05/5KbLOAuz2Z9npT6.png)

目录：

- [Percona XtraBackup 安装与使用](#percona-xtrabackup-安装与使用)
  - [Percona XtraBackup 安装](#percona-xtrabackup-安装)
    - [通过软件仓库安装](#通过软件仓库安装)
    - [通过二进制压缩文件安装](#通过二进制压缩文件安装)
    - [通过 rpm 或 apt 安装包安装](#通过-rpm-或-apt-安装包安装)
    - [通过编译源码安装](#通过编译源码安装)
  - [Xtrabackup 使用](#xtrabackup-使用)
  - [xtrabackup 备份流程 - 全量备份](#xtrabackup-备份流程---全量备份)
  - [xtrabackup 备份流程 - 增量备份](#xtrabackup-备份流程---增量备份)
  - [innobackupex 备份流程 - 全量备份](#innobackupex-备份流程---全量备份)
  - [innobackupex 备份流程 - 增量备份](#innobackupex-备份流程---增量备份)
  - [参考链接](#参考链接)

## xtrabackup 备份流程 - 全量备份



## xtrabackup 备份流程 - 增量备份

## innobackupex 备份流程 - 全量备份

## innobackupex 备份流程 - 增量备份


Create a backup

备份：
xtrabackup --user=root --password=MyNewPass4! --compress --backup --target-dir=/data/backups/

执行 /usr/bin 路径 下
innobackupex --user=root --password=MyNewPass4! --stream=xbstream --compress /data/backups/ > /data/innobackupextest.xbstream

执行 绝对路径下编译的
/root/percona-xtrabackup-release-2.4.8/storage/innobase/xtrabackup/src/innobackupex --user=root --password=MyNewPass4! --stream=xbstream --compress /data/backups/ > /data/backups/innobackupextest.xbstream

Preparing a backup

gdbserver :2333 xtrabackup --user=root --password=MyNewPass4! --backup --target-dir=/data/backups/


解压缩xbstream
xbstream -x < /data/backups/innobackupextest.xbstream -C /data/backup_qp/
解压qp

```
innobackupex --decompress /data/backup_qp

```
绝对路径：
/root/percona-xtrabackup-release-2.4.8/storage/innobase/xtrabackup/src/innobackupex --decompress /data/backup_qp

删除qp文件
find /data/backup -name "*.qp" | xargs rm

## 参考链接

* [Installing Percona XtraBackup 2.4](https://www.percona.com/doc/percona-xtrabackup/2.4/installation.html#installing-from-binaries)
