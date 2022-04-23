# ParallelRaft 基础

PolarFS 有一个有着极低时延和高可用性的分布式文件系统，专为 PolarDB 数据库设计。

* PolarFS 在用户态运行，有着轻量级的网络和 I/O 栈，并充分利用 RDMA、NVMe 及 SPDK 等新兴技术，最终大幅降低了 PolarFS 的端到端时延。
* 为了保持 PolarFS 副本的一致性，并最大化 I/O 吞吐，PolarFS 还开发了 ParallelRaft，一种基于 Raft 的共识协议。ParallelRaft 利用数据库的容忍乱序 I/O 的能力，打破了 Raft 需要严格序列化的约束。ParallelRaft 不仅继承了 Raft 的可理解性和易实现性，还为 PolarFS 提供了更好的 I/O 可扩展性。

本文主要介绍 [PolarFS 论文](https://www.vldb.org/pvldb/vol11/p1849-cao.pdf) 中提出的 **ParallelRaft 共识协议**。

目录：

- [ParallelRaft 基础](#parallelraft-基础)
  - [PolarFS 架构](#polarfs-架构)
  - [一致性模型](#一致性模型)
    - [介绍背景（为什么要改 parallel raft）](#介绍背景为什么要改-parallel-raft)
    - [可行性分析与设计思想](#可行性分析与设计思想)
    - [详细设计思想](#详细设计思想)
      - [Out-of-Order Log Replication](#out-of-order-log-replication)
    - [正确性论证](#正确性论证)
    - [性能对比](#性能对比)
  - [金山云 binlog（部分 额外的...）](#金山云-binlog部分-额外的)
  - [参考资料](#参考资料)

## PolarFS 架构

1. 介绍基础的文件系统架构
2. raft 主要用到哪里？为什么需要 raft？

PolarFS 主要包括两层：

* 上层是文件系统层（File System Layer），负责管理文件系统元数据，提供文件系统 API。
  * 文件系统层支持卷内的文件管理，负责文件系统元数据并发访问的互斥和同步。对于数据库实例，PolarFS 将文件系统元数据存储在其卷中。
* 下层是存储层（Storage Layer）负责存储管理。
  * 存储层负责存储节点的所有磁盘资源，为每个数据库实例提供一个数据库卷。

![PolarFS 架构图](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204231627952.png)

上图展示了 PolarFS 集群中的主要组件。

* `libpfs` 是一个用户空间实现的文件系统库，带有一组类似 POSIX 的文件系统 API，链接到 PolarDB 进程；
* `PolarSwitch` 驻留在 **计算节点** 上，将 I/O 请求从应用程序重定向到 `ChunkServers`；
* `ChunkServer` 部署在 **存储节点** 上，为 I/O 请求提供服务；
* `PolarCtrl` 是控制面，它包括一组在微服务中实现的主节点，以及部署在所有计算和存储节点上的代理。`PolarCtrl` 使用 MySQL 实例作为元数据存储库。

## 一致性模型

### 介绍背景（为什么要改 parallel raft）

问题
缺点
等等

### 可行性分析与设计思想


### 详细设计思想

#### Out-of-Order Log Replication

原 raft 
parallel raft

### 正确性论证

### 性能对比

## 金山云 binlog（部分 额外的...）

1. 分析相同（均是应用层支持乱序提交）
1. 对比不同（应用层）

应用层的实现方案

0. 背景介绍（binlog）
1. 自己想的
2. 参照阿里的

## 参考资料

* [PolarFS: An Ultra-low Latency and Failure Resilient Distributed File System for Shared Storage Cloud Database](https://www.vldb.org/pvldb/vol11/p1849-cao.pdf)
