# CentOS 快速开始 - 编程环境配置

本文主要针对新建 **CentOS 7** 实例 **配置编程环境** 相关操作的总结。

**注：** 由于 CentOS 7 默认软件库版本较旧，所以本文主要针对官方最新版的安装。

目录：

- [CentOS 快速开始 - 编程环境配置](#centos-快速开始---编程环境配置)
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
  - [参考链接](#参考链接)

## C/C++

### GCC

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

### GDB

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

### make

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

### CMake

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

### 常用第三方库

#### Boost

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

#### gflags/glog

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

#### ProtoBuf

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

#### gRPC

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

## Java

### JDK

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

### Maven

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

## Go

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

## 参考链接

* [为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本](https://www.vpser.net/manage/centos-6-upgrade-gcc.html)
* [CentOS 7升级gcc版本](https://www.cnblogs.com/jixiaohua/p/11732225.html)
* [CentOS 7 安装 gdb 7.8.2](https://blog.csdn.net/qq_25231683/article/details/120018299)
* [linux centos7 升级 make 4.3](https://blog.csdn.net/CLinuxF/article/details/108705142)
* [centos升级和安装cmake](https://zhuanlan.zhihu.com/p/358146707)
* [linux下安装google protobuf（详细）](https://blog.csdn.net/xiexievv/article/details/47396725)
* [centos 7 安装golang](https://www.jianshu.com/p/35a161738d83)
* [CentOS 7 安装JDK 1.8 环境教程](https://www.timberkito.com/?p=12)
* [Linux|CentOS下配置Maven环境](https://cloud.tencent.com/developer/article/1640173)
* [gRPC C++ - Building from source](https://github.com/grpc/grpc/blob/master/BUILDING.md)
