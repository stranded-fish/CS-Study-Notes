# CentOS 快速开始 - 数据库

本文主要针对新建 **CentOS 7** 实例 **安装常用数据库** 相关操作的总结。

**注：** 由于 CentOS 7 默认软件库版本较旧，所以本文主要针对官方最新版的安装。

目录：

- [CentOS 快速开始 - 数据库](#centos-快速开始---数据库)
  - [MySQL](#mysql)
  - [LevelDB](#leveldb)
  - [RocksDB](#rocksdb)
  - [Redis](#redis)
  - [参考链接](#参考链接)

## MySQL

**Step 1.** 下载所需版本 MySQL

* 通过官网 [MySQL Product Archives](https://downloads.mysql.com/archives/community/) 获取所需版本 MySQL `rpm` 包，所需 `rpm` 包如下：

![CentOS 7 所需 rmp 包](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204202102202.png)

```bash
wget \
https://cdn.mysql.com/archives/mysql-5.7/mysql-community-server-5.7.36-1.el7.x86_64.rpm \
https://cdn.mysql.com/archives/mysql-5.7/mysql-community-client-5.7.36-1.el7.x86_64.rpm \
https://cdn.mysql.com/archives/mysql-5.7/mysql-community-common-5.7.36-1.el7.x86_64.rpm \
https://cdn.mysql.com/archives/mysql-5.7/mysql-community-libs-5.7.36-1.el7.x86_64.rpm \
https://cdn.mysql.com/archives/mysql-5.7/mysql-community-libs-compat-5.7.36-1.el7.x86_64.rpm
```

**Step 2.** 安装 MySQL

```bash
yum install -y mysql-community-*-5.7.36-1.el7.x86_64.rpm
```

最终显示如下内容表示安装成功：

![MySQL 安装成功输出](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204202111311.png)

**补充：**

如果之前曾安装过其他版本（不兼容的旧版），安装过程中可能出现如下错误：

```bash
Error: Package: 1:mariadb-devel-5.5.68-1.el7.x86_64 (@base)
           Requires: mariadb-libs(x86-64) = 1:5.5.68-1.el7
           Removing: 1:mariadb-libs-5.5.68-1.el7.x86_64 (@base)
               mariadb-libs(x86-64) = 1:5.5.68-1.el7
           Obsoleted By: mysql-community-libs-5.7.36-1.el7.x86_64 (/mysql-community-libs-5.7.36-1.el7.x86_64)
               Not found
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

解决方法：

```bash
yum remove mysql-libs
```

**Step 3.** 启动 MySQL server 并初始化密码

```bash
# 启动 MySQL server
systemctl start mysqld
# 查看默认生成的密码
cat /var/log/mysqld.log | grep password
# 使用默认生成密码登录 MySQL
mysql -uroot -p
```

登录成功后，通过以下命令修改默认密码：

```sql
# 设置密码等级（可选）
mysql> set global validate_password_length=4;
mysql> set global validate_password_policy=0;
# 修改默认密码（注意替换 '您的密码'）
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '您的密码';
```

修改完，退出即可。

**Step 4.** 修改部分默认字符集为 UTF-8

修改 `/etc/my.cnf` 文件，新增以下内容：

```txt
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
character-set-server=utf8
```

修改完成后，使用命令 `systemctl restart mysqld` 重启 MySQL server，并再次登录 MySQL 查看：

```sql
mysql> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

**Step 5.** 设置 root 用户远程登陆（可选）

```sql
mysql> use mysql;
# 注意替换 '您的密码'
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '您的密码' WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```

设置完成后，在确保防火墙开放 `3306` 端口的情况下，即可实现远程连接。

## LevelDB

**Step 1.** 下载源码

```bash
git clone --recurse-submodules https://github.com/google/leveldb.git
```

**Step 2.** 编译并安装

```bash
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=Release .. && cmake --build .
```

**Step 3.** 安装检验

参见：[搭建LevelDB环境及原理分析](https://blog.csdn.net/tuwenqi2013/article/details/88560600)

## RocksDB

**Step 1.** 下载最新版 RocksDB

* 通过官网 [RocksDB Releases](https://github.com/facebook/rocksdb/releases/) 获取最新版 `rocksdb-x.x.x.tar.gz` 下载地址；
* 下载至服务器并解压；
* 进入到源代码根目录。

```bash
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.20.0/protobuf-all-3.20.0.tar.gz
tar -zxvf rocksdb-7.0.4.tar.gz
cd rocksdb-7.0.4/
```

**Step 2.** 编译并安装

```bash
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/rocksdb ..
make && make install
```

**Step 3.** 安装检验

参见：[RocksDB 简单使用](https://www.jianshu.com/p/f233528c8303)

通过如下命令编译并运行：

```bash
g++ -std=c++17 rocksdbtest.cpp -o rocksdbtest -lpthread -lrocksdb
```

## Redis

**Step 1.** 下载所需版本 Redis

通过官网 [Redis Download](https://redis.io/download/) 获取 Redis 源码下载地址。

```bash
wget https://download.redis.io/releases/redis-6.2.6.tar.gz
tar -zxvf redis-6.2.6.tar.gz
mv redis-6.2.6/ /usr/local/
cd redis-6.2.6
```

选择将源码文件保存至 `/usr/local/` 以便于管理。

**Step 2.** 编译与安装

```bash
make && make PREFIX=/usr/local/redis-6.2.6/ install
```

指定可执行文件安装路径为：`PREFIX=/usr/local/redis-6.2.6/bin`。

**Step 3.** 配置环境变量

编辑 `/etc/profile`，在其末尾添加如下配置：

```bash
export REDIS_HOME=/usr/local/redis-6.2.6
export PATH=$PATH:$REDIS_HOME/bin
```

保存后，可通过 `source /etc/profile` 命令使配置立即生效。

**Step 4.** 安装检验

```bash
redis-server -v
redis-cls -v
```

## 参考链接

* [CentOS7上安装MySQL 5.7.32（超详细）](https://blog.csdn.net/m0_51510236/article/details/113791490)
