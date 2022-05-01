# CentOS 快速开始 - 基础配置

本文主要针对新建 **CentOS 7** 实例 **相关基础配置** 总结。

目录：

- [CentOS 快速开始 - 基础配置](#centos-快速开始---基础配置)
  - [yum 更新](#yum-更新)
  - [配置环境变量](#配置环境变量)
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

## 自定义 Shell 终端提示符

编辑 `~/.bashrc`，在其末尾添加如下配置：

```txt
PS1="\[\033[0;34m\]\W\[\033[0;36m\] >\[\033[00m\] "
```

该配置简化了终端提示符，仅保留并显示当前目录：

```bash
local > pwd
/usr/local
```

## Vim 配置

参见：[The Ultimate vimrc](https://github.com/amix/vimrc)

## 参考链接

* [centos 7 yum安装、卸载、升级软件等命令](https://naibawu.com/1860.html)
* [Linux环境变量配置全攻略](https://www.cnblogs.com/youyoui/p/10680329.html)
* [#Linux学习笔记# 自定义shell终端提示符](https://www.cnblogs.com/lienhua34/p/5018119.html)
