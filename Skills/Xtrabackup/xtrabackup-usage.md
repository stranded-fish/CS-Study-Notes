# Xtrabackup 使用教程

Xtrabackup 安装完成后，工具集中包含以下可执行文件：

bin/
├── innobackupex -> xtrabackup
├── xtrabackup
├── xbstream
├── xbcrypt
└── xbcloud

各可执行文件功能介绍如下：

![工具集介绍](https://i.loli.net/2021/04/05/5KbLOAuz2Z9npT6.png)

目录：

- [Xtrabackup 使用教程](#xtrabackup-使用教程)
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
/root/percona-xtrabackup-release-2.4.8/storage/innobase/xtrabackup/src/innobackupex --user=root --password=MyNewPass4! --stream=xbstream --compress /data/backups/ > /data/innobackupextest.xbstream



Preparing a backup

gdbserver :2333 xtrabackup --user=root --password=MyNewPass4! --backup --target-dir=/data/backups/


解压缩xbstream
xbstream -x < /data/innobackupextest.xbstream -C /data/backup_qp/
解压qp

innobackupex --decompress /data/backup_qp

绝对路径：
/root/percona-xtrabackup-release-2.4.8/storage/innobase/xtrabackup/src/innobackupex --decompress /data/backup_qp

删除qp文件
find /data/backup -name "*.qp" | xargs rm

## 参考链接
