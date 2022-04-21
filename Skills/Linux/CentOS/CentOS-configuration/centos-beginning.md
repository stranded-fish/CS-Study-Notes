# CentOS 快速开始

本文主要针对新建 **CentOS 7** 实例 **配置编程环境** 与 **安装常用软件** 等相关操作的总结。

**注：** 由于 CentOS 7 默认软件库版本较老，所以本文主要针对官方最新版的安装。

目录：

- [CentOS 快速开始](#centos-快速开始)
  - [yum 更新](#yum-更新)
  - [配置环境变量](#配置环境变量)
  - [编程环境](#编程环境)
    - [C/C++](#cc)
      - [GCC](#gcc)
      - [GDB](#gdb)
      - [make](#make)
      - [CMake](#cmake)
      - [常用第三方库](#常用第三方库)
        - [Boost](#boost)
        - [gflags/glog](#gflagsglog)
        - [ProtoBuf](#protobuf)
        - [gRPC](#grpc)
    - [Java](#java)
      - [JDK](#jdk)
      - [Maven](#maven)
    - [Go](#go)
  - [数据库](#数据库)
    - [MySQL](#mysql)
    - [LevelDB](#leveldb)
    - [RocksDB](#rocksdb)
    - [Redis](#redis)
  - [常用软件](#常用软件)
    - [Git](#git)
    - [lrzsz](#lrzsz)
    - [lsof](#lsof)
    - [tree](#tree)
    - [zip/unzip](#zipunzip)
    - [telnet](#telnet)
    - [sysbench](#sysbench)
  - [界面美化](#界面美化)
    - [自定义 Shell 终端提示符](#自定义-shell-终端提示符)
    - [Vim 配置](#vim-配置)
  - [参考链接](#参考链接)

## yum 更新

**Step 1.** 列出可更新软件

```bash
yum list updates # 列出所有可更新软件包
yum info updates # 列出所有可更新的软件包信息
```

**Step 2.** 更新软件

```bash
yum update <package_name> # 更新指定软件
yum update                # 更新所有软件
```

## 配置环境变量

**读取环境变量：**

* `export` 命令显示当前系统定义的所有环境变量。
* `echo $VAL` 命令输出当前的 `VAL` 环境变量的值。

**编辑 `/etc/profile`，永久修改系统配置：**

```bash
# 末尾添加
export VAL=$VAL:/usr/local/newApp/bin
```

**注意：**

* 生效时间：新建终端生效，或 `source /etc/profile` 立即生效；
* 生效期限：永久有效；
* 生效范围：对所有用户有效；
* 新配置的环境变量需要加上原来的配置，即 `$VAL` 部分，以避免覆盖原来的配置；
* `export` 空格敏感，`=` 两侧不能添加空格。

更多可参见：[Linux环境变量配置全攻略](https://www.cnblogs.com/youyoui/p/10680329.html)

**C/C++ 常用环境变量：**

* 头文件搜索路径，等价于编译参数 `-I`：
  * `C_INCLUDE_PATH`：C 程序头文件搜索路径（gcc）
  * `CPLUS_INCLUDE_PATH`：C++ 程序头文件搜索路径（g++）
* 库文件搜索路径，等价于 `-l` 系统库 \\ `-L` 自定义库：
  * `LIBRARY_PATH`：静态库搜索路径
  * `LD_LIBRARY_PATH`动态库搜索路径

## 编程环境

### C/C++

#### GCC

CentOS 7.6 默认 gcc 版本为 4.8，可通过 devtoolset 工具安装多个不同的高版本 gcc 备用。

**Step 1.** 安装 centos-release-scl

```bash
yum install centos-release-scl
```

**Step 2.** 安装 devtoolset

```bash
yum install devtoolset-9-gcc*
```

* 如果想安装 `gcc 8.*` 版本，就改为 `devtoolset-8-gcc*`，以此类推，目前已支持 gcc 7 - 11 版本。
* devtoolset 默认安装目录为 `/opt/rh`。可同时安装多个版本 `/opt/rh/devtoolset-x` 并根据需要进行切换。

**Step 3.** 切换 gcc 版本

**a.** 临时生效（仅当前会话生效）

```bash
scl enable devtoolset-9 bash
```

**b.** 长期生效

```bash
echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
source /etc/profile
```

**Step 4.** 安装检验

```bash
gcc --version
g++ --version
```

#### GDB

**Step 1.** 下载最新版 GDB

* 通过官网 [Download GDB](https://www.sourceware.org/gdb/download/) 获取最新版 `gdb-x.x.tar.gz` 下载地址；
* 下载至服务器并解压；
* 进入到源代码根目录。

```bash
wget https://ftp.gnu.org/gnu/gdb/gdb-11.2.tar.gz
tar -zxvf gdb-11.2.tar.gz
cd gdb-11.2/
```

**Step 2.** 配置

```bash
./configure
```

**Step 3.** 编译并安装 GDB

```bash
make && make install
```

**Step 4.** 安装检验

```bash
gdb --version
```

#### make

**Step 1.** 下载最新版 make

* 通过官网 [Index of /gnu/make](https://ftp.gnu.org/gnu/make/) 获取最新版 `make-x.x.tar.gz` 下载地址；
* 下载至服务器并解压；
* 进入到源代码根目录。

```bash
wget https://ftp.gnu.org/gnu/make/make-4.3.tar.gz
tar -zxvf make-4.3.tar.gz
cd make-4.3/
```

**Step 2.** 安装基本编译工具并设置安装路径

```bash
./configure --prefix=/usr/local/make
```

设置 make 安装路径为 `/usr/local/make`。

**Step 3.** 编译并安装 make

```bash
make && make install
```

**Step 4.** 备份原版 make

```bash
mv /usr/bin/make /usr/bin/make.bak
```

**Step 5.** 创建新版软链接

```bash
ln -s /usr/local/make/bin/make /usr/bin/make
```

**Step 6.** 安装检验

```bash
make -v
```

#### CMake

**Step 1.** 下载最新版 CMake

* 通过官网 [Download | CMake](https://cmake.org/download/) 获取最新版 `cmake-x.xx.x.tar.gz` 下载地址；
* 下载至服务器并解压；
* 进入到源代码根目录。

```bash
wget https://github.com/Kitware/CMake/releases/download/v3.23.0/cmake-3.23.0.tar.gz
tar -zxvf cmake-3.23.0.tar.gz
cd cmake-3.23.0/
```

**Step 2.** 安装基本编译工具并设置安装路径

```bash
./configure --prefix=/usr/local/cmake
```

设置 CMake 安装路径为 `/usr/local/cmake`。

**注意：** 该命令执行过程中，可能会出现 `CMake Error：Could not find OpenSSL.` 报错，可通过以下命令解决：

```bash
yum install openssl-devel
```

**Step 3.** 编译并安装 CMake

```bash
make && make install
```

**Step 4.** 卸载原有 CMake

```bash
yum -y remove cmake
```

**Step 5.** 添加到系统环境变量

**a.** 创建新版软链接

```bash
ln -s /usr/local/cmake/bin/cmake /usr/bin/cmake
```

**b.** 配置环境变量

```bash
[root@localhost ~]# vim /etc/profile
export CMAKE_HOME=/usr/local/cmake
export PATH=$PATH:$CMAKE_HOME/bin

# 使配置生效
[root@localhost ~]# source /etc/profile
```

**Step 6.** 安装检验

```bash
cmake --version
```

#### 常用第三方库

##### Boost

**Step 1.** 下载最新版 Boost

* 通过官网 [Boost C++ Libraries](https://www.boost.org) 获取最新版 `boost_x_xx_x.tar.gz` 下载地址；
* 下载至服务器并解压；
* 进入到源代码根目录。

```bash
wget https://boostorg.jfrog.io/artifactory/main/release/1.78.0/source/boost_1_78_0.tar.gz
tar -zxvf boost_1_78_0.tar.gz
cd boost_1_78_0/
```

**Step 2.** 配置

```bash
./bootstrap.sh --prefix=/opt/boost
```

设置 Boost 安装路径为 `/opt/boost`。

**Step 3.** 安装

```bash
./b2 install --prefix=/opt/boost --with=all
```

安装完成后，`/opt/boost` 将新增 `include` 与 `lib` 文件夹，分别用于保存头文件与库文件。

**Step 4.** 安装检验

创建 `test.cc` 文件内容如下：

```C++
#include <iostream>
#include <boost/lexical_cast.hpp>

using namespace std;
using namespace boost;

int main() {
    int a = lexical_cast<int>("123456");
    double b = lexical_cast<double>("123.456");
    cout << a << endl;
    cout << b << endl;
    return 0;
}
```

通过 `-I /opt/boost/include/` 将 Boost 安装路径添加到头文件搜索路径，然后编译运行：

```bash
g++ test.cc -o test -I /opt/boost/include/
./test
```

##### gflags/glog

glog 依赖于 gflags，需先安装 gflags。

**安装 gflags：**

```bash
git clone https://github.com/gflags/gflags.git
cd gflags
mkdir build && cd build
cmake .. -DBUILD_SHARED_LIBS=ON
make && make install
```

**安装 glog**：参见官网 [Building glog with CMake](https://github.com/google/glog#building-glog-with-cmake)。

##### ProtoBuf

**Step 1.** 下载最新版 ProtoBuf

* 通过官网 [ProtoBuf Releases](https://github.com/protocolbuffers/protobuf/releases) 获取最新版 `protobuf-all-x.xx.x.tar.gz` 下载地址；
* 下载至服务器并解压；
* 进入到源代码根目录。

```bash
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.20.0/protobuf-all-3.20.0.tar.gz
tar -zxvf protobuf-all-3.20.0.tar.gz
cd protobuf-3.20.0/
```

**Step 2.** 配置

```bash
./configure                                # 默认路径
# ./configure --prefix=/usr/local/protobuf # 自定义路径
```

**Step 3.** 编译并安装

```bash
make && make install
```

如果自定义安装路径，则安装完成后还需配置环境变量，编辑 `/etc/profile`，在其末尾添加如下配置：

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/protobuf/lib/
export LIBRARY_PATH=$LIBRARY_PATH:/usr/local/protobuf/lib/
export C_INCLUDE_PATH=$C_INCLUDE_PATH:/usr/local/protobuf/include/
export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/usr/local/protobuf/include/
export PATH=$PATH:/usr/local/protobuf/bin/
export PKG_CONFIG_PATH=/usr/local/protobuf/lib/pkgconfig/
```

保存后，可通过 `source /etc/profile` 命令使配置立即生效。

**Step 4.** 安装检验

```bash
protoc --version
```

##### gRPC

**Step 1.** Clone the repository (including submodules)

可通过官网获取 [latest stable release tag](https://github.com/grpc/grpc/releases)。

```bash
git clone -b RELEASE_TAG_HERE https://github.com/grpc/grpc
cd grpc
git submodule update --init
```

**Step 2.** 编译并安装

```bash
mkdir -p cmake/build
cd cmake/build
cmake ../..
make && make install
```

**Step 3.** 安装检验

C++ 可通过 `cmake` 编译并运行 `grpc/examples/cpp/helloworld` 示例来实现检验。

### Java

#### JDK

**Step 1.** 查询当前 yum 库支持安装的 jdk 版本

```bash
yum search java | grep jdk
```

**Step 2.** 选择并安装需要的 jdk

```bash
# 安装最新版 jdk
yum install java-latest-openjdk
yum install java-latest-openjdk-devel
```

**Step 3.** 安装检验

```bash
java --version
javac --version
```

#### Maven

**Step 1.** 下载最新版 Maven

* 通过官网 [Maven Download](https://maven.apache.org/download.cgi) 获取最新版 `apache-maven-x.x.x-bin.tar.gz` 下载地址；
* 下载至服务器并解压至指定目录；

```bash
wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
tar -zxvf apache-maven-3.8.5-bin.tar.gz -C /usr/local
```

**Step 2.** 配置环境变量

编辑 `/etc/profile`，在其末尾添加如下配置：

```bash
export MAVEN_HOME=/usr/local/apache-maven-3.8.5
export PATH=$MAVEN_HOME/bin:$PATH
```

保存后，可通过 `source /etc/profile` 命令使配置立即生效。

**Step 3.** 安装检验

```bash
mvn -v
```

### Go

**Step 1.** 下载最新版 Go

* 通过官网 [Go Download](https://go.dev/dl/) 获取最新版 `gox.xx.linux-amd64.tar.gz` 下载地址；
* 下载至服务器并解压至指定目录；

```bash
wget https://dl.google.com/go/go1.18.linux-amd64.tar.gz
tar -zxvf go1.18.linux-amd64.tar.gz -C /usr/local
```

**Step 2.** 配置环境变量

编辑 `/etc/profile`，在其末尾添加如下配置：

```bash
export GO111MODULE=auto
export GOROOT=/usr/local/go
export GOPATH=/home/gopath
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

保存后，可通过 `source /etc/profile` 命令使配置立即生效。

**Step 3.** 安装检验

```bash
go version
```

## 数据库

### MySQL

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

### LevelDB

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

### RocksDB

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

### Redis

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

## 常用软件

### Git

**Step 1.** 下载最新版 Git

* 通过官网 [Download for Linux and Unix](https://git-scm.com/download/linux) 获取最新版 `git-x.xx.x.tar.gz` 下载地址；
* 下载至服务器并解压；
* 进入到源代码根目录。

```bash
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.35.1.tar.gz
tar -zxvf git-2.35.1.tar.gz
cd git-2.35.1/
```

**Step 2.** 下载依赖

```bash
yum groupinstall 'Development Tools'
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-CPAN perl-devel
```

**Step 3.** 配置

```bash
make configure
./configure --prefix=/usr/local
```

**Step 4.** 编译并安装 Git

```bash
make all && make install
```

**Step 5.** 安装检验

```bash
git --version
```

### lrzsz

```bash
yum install lrzsz
```

### lsof

```bash
yum install lsof
```

### tree

```bash
yum install tree
```

### zip/unzip

```bash
yum install zip unzip
```

### telnet

```bash
# 安装 xinetd
yum install xinetd

# 安装 telnet
yum install telnet
yum install telnet-server
```

### sysbench

```bash
yum install sysbench
```

## 界面美化

### 自定义 Shell 终端提示符

编辑 `~/.bashrc`，在其末尾添加如下配置：

```txt
PS1="\[\033[0;34m\]\W\[\033[0;36m\] >\[\033[00m\] "
```

该配置简化了终端提示符，仅保留并显示当前目录：

```bash
local > pwd
/usr/local
```

### Vim 配置

参见：[The Ultimate vimrc](https://github.com/amix/vimrc)

## 参考链接

* [centos 7 yum安装、卸载、升级软件等命令](https://naibawu.com/1860.html)
* [为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本](https://www.vpser.net/manage/centos-6-upgrade-gcc.html)
* [CentOS 7升级gcc版本](https://www.cnblogs.com/jixiaohua/p/11732225.html)
* [CentOS 7 安装 gdb 7.8.2](https://blog.csdn.net/qq_25231683/article/details/120018299)
* [linux centos7 升级 make 4.3](https://blog.csdn.net/CLinuxF/article/details/108705142)
* [centos升级和安装cmake](https://zhuanlan.zhihu.com/p/358146707)
* [Build git from source code](https://gist.github.com/egorsmkv/30faa3e61c185a41e89cf849737d4d4b)
* [linux下安装google protobuf（详细）](https://blog.csdn.net/xiexievv/article/details/47396725)
* [centos 7 安装golang](https://www.jianshu.com/p/35a161738d83)
* [CentOS 7 安装JDK 1.8 环境教程](https://www.timberkito.com/?p=12)
* [Linux|CentOS下配置Maven环境](https://cloud.tencent.com/developer/article/1640173)
* [#Linux学习笔记# 自定义shell终端提示符](https://www.cnblogs.com/lienhua34/p/5018119.html)
* [gRPC C++ - Building from source](https://github.com/grpc/grpc/blob/master/BUILDING.md)
* [CentOS7上安装MySQL 5.7.32（超详细）](https://blog.csdn.net/m0_51510236/article/details/113791490)
