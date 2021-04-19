# sysbench 安装与使用

sysbench 是一款开源的多线程性能测试工具，可以执行 CPU/内存/线程/IO/数据库等方面的性能测试。

本文主要介绍 sysbench 的 **安装** 与 **使用**。

目录:

- [sysbench 安装与使用](#sysbench-安装与使用)
  - [安装](#安装)
  - [使用](#使用)
  - [参考链接](#参考链接)

## 安装

```bash
yum install sysbench
```

## 使用

sysbench 基本语法：

```bash
sysbench [options]... [testname] [command] 
```

* `testname`：可选参数。可以是一个内置的测试工具，一个绑定的 Lua 脚本名称，或者一个自定义 Lua 脚本路径。如果在命令行上没有指定测试名，它将被解析为 testname。如果测试名是一个破折号（“-”），表示 sysbench 希望 Lua 脚本在其标准输入上执行。
* `command`：可选参数。指定测试必须执行的操作。可用命令的列表取决于特定的测试。
  
  <br>以下是典型测试命令及其用途：
  <br>
  * `prepare`：为需要的测试执行准备操作，例如为 fileio 测试在磁盘上创建必要的文件，为数据库基准填充测试数据库。
  * `run`：运行带有 testname 参数的实际测试。所有测试均提供该命令。
  * `cleanup`：在创建临时数据的测试运行完毕后删除创建的临时数据。
  * `help`：显示 testname 帮助信息。例如：`sysbench /usr/share/sysbench/oltp_read_write.lua help` 将显示该测试脚本的使用说明。


`options`：可选参数。是一个以 `--` 开头的 0 个或多个命令行选项的列表。

测试时可以使用 sysbench 自带脚本，也可以使用自己开发的脚本。sysbench 1.0.20 通过 `yum install` 方式安装后的默认脚本存储路径为 `/usr/share/sysbench`。

注意：oltp 表示 On-Line Transaction Processing 联机事务处理，是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。参见 [OLTP](https://baike.baidu.com/item/OLTP)。

**使用实践：使用 sysbench 对 MySQL 开展基准测试**

**Step 1.** 创建测试用数据库

首先创建测试所需的数据库，如 `sbtest`：

```bash
mysql> create database sbtest;
```

**Step 2.** 准备测试数据

向测试数据库中填充数据。

```bash
sysbench /usr/share/sysbench/oltp_read_write.lua --tables=5 --table_size=100 --mysql-user=root --mysql-password=MyNewPass4! --mysql-host=localhost --mysql-port=3306 --mysql-db=sbtest prepare
```

关键参数说明：

`--tables`：指定生成表的数量。
`--table_size`：指定生成的每个表中数据行数。
`--mysql-db`：指定连接的数据库名。

**Step 3.** 选择脚本进行测试

如测试读写性能

```bash
 sysbench /usr/share/sysbench/oltp_read_write.lua --mysql-user=root --mysql-password=MyNewPass4! --mysql-host=localhost --mysql-port=3306 --mysql-db=sbtest --tables=5 --table_size=100 --threads=10 --time=30 --report-interval=3 run
```

测试结果：

![测试结果](https://i.loli.net/2021/04/19/dpy8N3acCiE5uqg.png)

关键信息：

* `transactions: 12054  (401.54 per sec.)`：事务总数及 tps
* `queries: 253549 (8446.07 per sec.)`：查询总数及 qps
* `95th percentile: 43.39`：前95%的请求的响应时间

**Step 4.** 清除测试用数据

```bash
sysbench /usr/share/sysbench/oltp_read_write.lua --tables=5 --table_size=100 --mysql-user=root --mysql-password=MyNewPass4! --mysql-host=localhost --mysql-port=3306 --mysql-db=sbtest cleanup
```

## 参考链接

* [akopytov/sysbench](https://github.com/akopytov/sysbench)
* [Sysbench测试神器：一条命令生成百万级测试数据](https://my.oschina.net/u/4579562/blog/4692263)
