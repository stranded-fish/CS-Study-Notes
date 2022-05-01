# CentOS 快速开始 - 常用软件

本文主要针对新建 **CentOS 7** 实例 **安装常用软件** 相关操作的总结。

目录：

- [CentOS 快速开始 - 常用软件](#centos-快速开始---常用软件)
  - [Git](#git)
  - [lrzsz](#lrzsz)
  - [lsof](#lsof)
  - [tree](#tree)
  - [zip/unzip](#zipunzip)
  - [telnet](#telnet)
  - [sysbench](#sysbench)
  - [参考链接](#参考链接)

## Git

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

## lrzsz

```bash
yum install lrzsz
```

## lsof

```bash
yum install lsof
```

## tree

```bash
yum install tree
```

## zip/unzip

```bash
yum install zip unzip
```

## telnet

```bash
# 安装 xinetd
yum install xinetd

# 安装 telnet
yum install telnet
yum install telnet-server
```

## sysbench

```bash
yum install sysbench
```

## 参考链接

* [Build git from source code](https://gist.github.com/egorsmkv/30faa3e61c185a41e89cf849737d4d4b)
