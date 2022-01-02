# xtrabackup 优化版部署

## 安装 xtrabackup

**Step 1.** 安装必要依赖

Percona XtraBackup 需要 GCC 5.3 或更高版本。

安装 `cmake` 和其他依赖：

```bash
sudo yum install cmake openssl-devel libaio libaio-devel automake autoconf \
bison libtool ncurses-devel libgcrypt-devel libev-devel libcurl-devel zlib-devel \
vim-common
```

为了能够安装 manual pages，安装 `python3-sphinx` 包:

```bash
sudo yum install python3-sphinx
```

**Step 2.** 解压 `percona-xtrabackup-release-2.4.8.tar.gz`

```bash
tar -zcvf percona-xtrabackup-release-2.4.8.tar.gz
```

**Step 3.** 执行 cmake

```bash
cd percona-xtrabackup-release-2.4.8
```

根目录执行：

```bash
cmake -DBUILD_CONFIG=xtrabackup_release -DWITH_MAN_PAGES=OFF
```

**注意：** 在执行上述命令时，可能发生如下错误：

![cmake 报错](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102144726.png)

**解决方法：**

可通过错误提示解决或通过以下方式解决：

```bash
mkdir -p /usr/local/boost && cd /usr/local/boost
wget http://www.sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
tar -xvzf boost_1_59_0.tar.gz
rm boost_1_59_0.tar.gz
```

然后再重新执行以下命令：

```bash
cmake -DBUILD_CONFIG=xtrabackup_release -DWITH_MAN_PAGES=OFF -DWITH_BOOST=/usr/local/boost/
```

**Step 3.** 执行 make

```bash
make -j1 && make install
```

**Step 4.** 验证 xtrabackup 安装

分别执行以下两条命令：

```bash
xtrabackup --version
innobackupex --version
```

输出应如下：

![xtrabackup 安装验证](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102144751.png)

## 安装 zstd & pzstd

**Step 1.** 解压 `zstd-dev.zip`

```bash
unzip zstd-dev.zip
```

**Step 2.** 安装 zstd

```bash
cd zstd-dev
```

```bash
make install
```

**Step 3.** 验证 zstd 安装

**验证 1：**

```bash
zstd --version
```

输出应如下：

![zstd 验证1](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102144811.png)

**验证 2：**

```bash
ll /usr/local/lib | grep libzstd.a
```

输出应如下：

![zstd 验证2](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102144829.png)

**验证 3：**

```bash
cat /usr/local/lib/pkgconfig/libzstd.pc
```

输出应如下：

![zstd 验证3](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102144840.png)

**Step 4.** 安装 pzstd

进入 zstd-dev/contrib/pzstd 目录：

```bash
cd contrib/pzstd/
```

```bash
make install
```

**Step 5.** 验证 pzstd 安装

```bash
pzstd --version
```

输出应如下：

![pzstd 验证](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/20220102144852.png)
