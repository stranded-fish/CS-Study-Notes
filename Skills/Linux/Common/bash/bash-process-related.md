# Bash Shell 进程相关操作总结

该文章主要针对 Linux 环境下 Bash Shell 对进程进行相关操作（如：查看占用指定 port 进程，杀死指定进程等）的场景与命令的总结。

目录：

- [Bash Shell 进程相关操作总结](#bash-shell-进程相关操作总结)
  - [查看端口占用情况](#查看端口占用情况)
  - [查看程序运行情况](#查看程序运行情况)
  - [进程挂起（后台运行、休眠、重启等）](#进程挂起后台运行休眠重启等)
  - [杀掉进程](#杀掉进程)
  - [参考链接](#参考链接)
  
## 查看端口占用情况

**eg 1. lsof**

```bash
lsof -i :port1,port2,port3...
```

lsof 命令详解：

lsof - List open files.

open files 可指普通文件，目录，特殊块（block）文件，特殊字符（character）文件，正在执行的文本引用（executing text reference），库（library），流（stream）或 **网络文件（network file：Internet socket， NFS file or UNIX domain socket）**。

> 在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，该文件描述符提供了大量关于这个应用程序本身的信息。

参数：

-i 列出 IPv[46] 文件。

该参数所需 Internet 地址的格式为（方括号内为可选项）：`[46][protocol][@hostname|hostaddr][:service|port]`

* `[46]` - 查询的 IP 版本。4 或 6，若无显示指定则同时显示所有 IP 版本信息。
* `[protocol]` - 协议名。TCP 或 UDP，若无显示指定则同时显示两种协议信息。
* `[@hostname]` - 网络主机名。除非指定了特定的 IP 版本，否则将选择与所有版本的主机名相关联的网络文件。
* `[@hostaddr]` - 地址。点分十进制 IPv4 地址，如果 UNIX 支持 IPv6，则可以使用冒号形式的 IPv6 数字地址（用括号括起来）。
* `[:service]` - /etc/services 名称。如，smtp 或其列表。
* `[:port]` - 端口号。也可是端口号的列表。

查询列表用 `,` 分割，如一次查询多个端口号：8080,8081,8082。

示例：

```bash
# 仅查询 IPv6
lsof -i 6

# 采用 TCP 协议  端口号 25
lsof -i TCP:25

# Internet IPv4 主机地址 1.2.3.4
lsof -i @1.2.3.4

# Internet IPv6 主机地址 3ffe:1ebc::1 端口号 1234
lsof  -i @[3ffe:1ebc::1]:1234

# 查询多个 端口号 8080 8081 8082
lsof -i :8080,8081,8082
```

查询输出的各列信息意义如下：

* COMMAND：进程的名称
* PID：进程标识符
* PPID：父进程标识符（需要指定-R参数）
* USER：进程所有者
* PGID：进程所属组
* FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等:
* TYPE：文件类型，如DIR、REG等，常见的文件类型:
* DEVICE：指定磁盘的名称
* SIZE：文件的大小
* NODE：索引节点（文件在磁盘上的标识）
* NAME：打开文件的确切名称

**eg 2. netstat**

```bash
netstat -tunlp | grep port
```

netstat 命令详解：

netstat - 打印网络连接、路由表、接口统计信息、伪连接（masquerade connections）和多播成员关系（multicast memberships）。

参数：

* -t | --tcp 列出 tcp 相关选项。
* -u | --udp 列出 udp 相关选项。
* -n | --numerical 显示数字地址，而非别名。
* -l | --listening 显示在 Listen（监听）的服务状态（该内容默认被省略）
* -p | --program 显示每个套接字所属程序的 PID 和名称。

示例：

```bash
# 查询当前所有 tcp 相关信息
netstat -tnlp

# 查询 80 端口的所有使用情况
netstat -tunlp | grep 80   //查看所有80端口使用情况
```

## 查看程序运行情况

如已知程序名 获取程序运行 端口号 和 pid

TODO

## 进程挂起（后台运行、休眠、重启等）

TODO

## 杀掉进程

在知道待杀掉进程的 PID 后，可通过 `kill` 命令杀掉该进程：

```bash
kill -9 PID
```

参数：

-9 | -1 杀掉你所能杀掉的所有进程

## 参考链接

* [Linux 查看端口占用情况](https://www.runoob.com/w3cnote/linux-check-port-usage.html)
* [lsof](https://ss64.com/bash/lsof.html)
* [lsof 一切皆文件](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/lsof.html)
