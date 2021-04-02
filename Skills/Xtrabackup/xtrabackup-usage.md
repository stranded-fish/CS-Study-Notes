# xtrabackup usage

Create a backup

备份：
xtrabackup --user=root --password=MyNewPass4! --compress --backup --target-dir=/data/backups/

执行 /usr/bin 路径 下
innobackupex --user=root --password=MyNewPass4! --stream=xbstream --compress /data/backups/ > /data/innobackupextest.xbstream

执行 绝对路径下编译的
/root/percona-xtrabackup-release-2.4.8/storage/innobase/xtrabackup/src/innobackupex --user=root --password=MyNewPass4! --stream=xbstream --compress /data/backups/ > /data/innobackupextest.xbstream



Preparing a backup

gdbserver :2333 xtrabackup --user=root --password=MyNewPass4! --backup --target-dir=/data/backups/




备份
innobackupex --defaults-file=/usr/local/mysql/my.cnf --user=root --stream=xbstream --compress /data/backup/ > /data/backup/full.xbstream
解压缩xbstream
xbstream -x < /data/innobackupextest.xbstream -C /data/backup_qp/
解压qp

innobackupex --decompress /data/backup_qp

绝对路径：
/root/percona-xtrabackup-release-2.4.8/storage/innobase/xtrabackup/src/innobackupex --decompress /data/backup_qp

删除qp文件
find /data/backup -name "*.qp" | xargs rm