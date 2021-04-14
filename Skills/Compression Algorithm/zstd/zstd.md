# zstd

Zstandard - 快速实时压缩算法。

目录：

- [zstd](#zstd)
  - [安装 zstd](#安装-zstd)
    - [yum 仓库安装](#yum-仓库安装)
    - [源码编译](#源码编译)
  - [pzstd 使用](#pzstd-使用)
  - [参考链接](#参考链接)

## 安装 zstd

### yum 仓库安装

Centos 7 安装：

```bash
yum install zstd
```

Centos 7 卸载：

```bash
yum remove zstd
```

pzstd 将会同步安装。

### 源码编译

zstd-dev 根目录执行 make 命令：

```bash
make install # create and install zstd cli, library and man pages
make check   # create and run zstd, tests its behavior on local platform
```

可执行程序将会默认安装到 /usr/local/bin/ 中

可在 /usr/bin 中执行 ln -s /usr/local/bin/zstd zstd，生成软链接，以全局使用

zstd-dev/contrib/pzstd 中执行 make 命令安装 pzstd：

```bash
make install
```

默认安装目录同上 zstd ，可通过设置软链解决全局访问。

## pzstd 使用

**压缩命令示例：**

```bash
pzstd -f -p 4 hello.c
```

参数说明：

* -f force
* -p multi thread

**解压命令示例：**

```bash
pzstd -d -f -p 4 hello.c.zst -o hello.c
```

参数说明：

* -d decompress
* -f force
* -p multi thread
* -o store into a specific file （不加该参数则自动命令为取消 .zst 后缀形式）

TODO 源码默认 install 状态为 release 状态，需要在 MakeFile 中添加

CXXFLAGS = -g
CFLAGS=-g

开启调试

## 参考链接
