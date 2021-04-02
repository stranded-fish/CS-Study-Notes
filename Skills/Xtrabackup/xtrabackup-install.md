# xtrabackup install

## eg 1 下载 rpm 安装包安装

安装：

https://www.percona.com/doc/percona-xtrabackup/2.4/installation/yum_repo.html#standalone-rpm

如果想要安装不同版本的直接 修改 wget 中的链接即可

例如：

wget https://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.8/binary/redhat/7/x86_64/percona-xtrabackup-24-2.4.8-1.el7.x86_64.rpm

卸载

https://blog.csdn.net/anzhen0429/article/details/76302158

yum remove percona-xtrabackup-24

## eg 2 下载源码编译

安装：https://blog.csdn.net/anzhen0429/article/details/7630215

根目录执行：

cmake -DBUILD_CONFIG=xtrabackup_release -DWITH_MAN_PAGES=OFF -DWITH_BOOST=/usr/local/boost && make -j1 && make install


11
https://www.programmersought.com/article/25122068037/

https://blog.csdn.net/qq_29573053/article/details/69665996

bug 1 cmake/boost.cmake:81 解决
bug 2 internal compiler error: Killed (program cc1plus) 内存不够，编译器被 kill 了 解决

1. 下载指定版本源码 或者 下载整个 git 工程并切换至目标版本
2. 下载相关依赖
3. 项目根目录执行 cmake 命令
4. 可能会出现一些 bug 
5. 将可执行程序移动到 /usr/bin 目录




卸载：


## eg 3 内网离线安装



