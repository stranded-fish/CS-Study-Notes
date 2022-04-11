# CentOS 快速开始

本文主要针对新建 **CentOS 7** 实例 **配置编程环境** 与 **安装常用软件** 等相关操作的总结。

**注：** 由于 CentOS 7 默认软件库版本较老，所以本文主要针对官方最新版的安装。

目录：

- [CentOS 快速开始](#centos-快速开始)
  - [yum 更新](#yum-更新)
  - [编程环境](#编程环境)
    - [C/C++](#cc)
      - [GCC](#gcc)
      - [GDB](#gdb)
      - [make](#make)
      - [CMake](#cmake)
      - [常用第三方库](#常用第三方库)
        - [Boost](#boost)
        - [protobuf](#protobuf)
        - [grpc](#grpc)
        - [glog](#glog)
        - [leveldb](#leveldb)
        - [rocksdb](#rocksdb)
    - [Java](#java)
    - [Go](#go)
  - [基础软件](#基础软件)
    - [Git](#git)
  - [常用工具](#常用工具)
    - [lrzsz](#lrzsz)
    - [lsof](#lsof)
    - [tree](#tree)
    - [zip/unzip](#zipunzip)
    - [telnet](#telnet)
  - [界面美化](#界面美化)
    - [bash 美化](#bash-美化)
    - [Vim 美化](#vim-美化)
  - [参考链接](#参考链接)

## yum 更新

**Step 1.** 列出可更新软件

```bash
yum list updates # 列出所有可更新软件包
yum info updates # 列出所有可更新的软件包信息
```

**Step 2.** 更新软件

```bash
yum update <package_name> # 仅更新指定软件
yum update                # 更新所有软件
```

## 编程环境

### C/C++

#### GCC

CentOS 7.6 默认 gcc 版本为 4.8，可通过 devtoolset 工具安装多个不同高版本 gcc 备用。

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

**Step 4.** 检查版本

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

**Step 4.** 检查版本

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
./configure --prefix=/usr/local/cmake
```

make 安装路径被设置为 `/usr/local/cmake`。

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

**Step 6.** 检查版本

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

CMake 安装路径被设置为 `/usr/local/cmake`。

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

**Step 6.** 检查版本

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

Boost 安装路径被设置为 `/opt/boost`。

**Step 3.** 安装

```bash
./b2 install --prefix=/opt/boost --with=all
```

安装完成后，`/opt/boost` 将新增 `include` 与 `lib` 文件夹，分别用于保存头文件与库文件。

**Step 4.** 检验安装

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

##### protobuf

##### grpc

##### glog

##### leveldb

##### rocksdb

### Java

### Go

## 基础软件

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

**Step 5.** 检查版本

```bash
git --version
```

## 常用工具

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

## 界面美化

### bash 美化

### Vim 美化

## 参考链接

* [centos 7 yum安装、卸载、升级软件等命令](https://naibawu.com/1860.html)
* [为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本](https://www.vpser.net/manage/centos-6-upgrade-gcc.html)
* [CentOS 7升级gcc版本](https://www.cnblogs.com/jixiaohua/p/11732225.html)
* [CentOS 7 安装 gdb 7.8.2](https://blog.csdn.net/qq_25231683/article/details/120018299)
* [linux centos7 升级 make 4.3](https://blog.csdn.net/CLinuxF/article/details/108705142)
* [centos升级和安装cmake](https://zhuanlan.zhihu.com/p/358146707)
* [Build git from source code](https://gist.github.com/egorsmkv/30faa3e61c185a41e89cf849737d4d4b)
