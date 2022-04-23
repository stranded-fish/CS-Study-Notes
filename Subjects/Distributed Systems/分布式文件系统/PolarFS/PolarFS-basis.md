# PolarFS 基础

> PolarFS 由阿里云数据库团队在 VLDB 2018 会议中提出。
>
> 论文地址：[PolarFS: An Ultra-low Latency and Failure Resilient Distributed File System for Shared Storage Cloud Database](https://www.vldb.org/pvldb/vol11/p1849-cao.pdf)

PolarFS 有一个有着极低时延和高可用性的分布式文件系统，专为 PolarDB 数据库设计。

* PolarFS 在用户态运行，有着轻量级的网络和 I/O 栈，并充分利用 RDMA、NVMe 及 SPDK 等新兴技术，最终大幅降低了 PolarFS 的端到端时延。
* 为了保持 PolarFS 副本的一致性，并最大化 I/O 吞吐，PolarFS 还开发了 ParallelRaft，一种基于 Raft 的共识协议。ParallelRaft 利用数据库的容忍乱序 I/O 的能力，打破了 Raft 需要严格序列化的约束。ParallelRaft 不仅继承了 Raft 的可理解性和易实现性，还为 PolarFS 提供了更好的 I/O 可扩展性。

目录：

- [PolarFS 基础](#polarfs-基础)
  - [背景与意义](#背景与意义)
    - [当前云存储技术的不足](#当前云存储技术的不足)
    - [PolarFS 特点](#polarfs-特点)
  - [PolarFS 架构](#polarfs-架构)
  - [I/O 执行模型](#io-执行模型)
  - [一致性模型](#一致性模型)
  - [参考资料](#参考资料)

## 背景与意义

### 当前云存储技术的不足

当前的云平台难以利用新兴的硬件标准与技术，如 RDMA、NVMe 等。例如，一些广泛使用的开源分布式文件系统，如 HDFS 和 Ceph，如果在这些存储系统上直接运行数据库（如：MySQL），性能将明显低于同配置下的本地 PCIe SSD。

为了解决这一个问题，各大云厂商均推出了实例存储。实例存储使用本地 SSD 和高 I/O 虚拟机实例来满足客户对高性能数据库的需求。但实例存储上运行云数据库仍有很多缺点：

* 单实例存储容量有限，不适合大型数据库服务。并且即使单实例容量上限满足要求，当其数据量过大的话，也会导致相应问题：
  * 单实例数据量过大，将导致备份时间过长，增加失败的可能。
  * 实例故障后重建时间较长，因为备份和恢复均需要较长时间。
* 无法应对潜在的磁盘故障。为了数据的可靠性，数据库需要自己维护数据副本。
* 实例存储使用通用文件系统，例如 ext4 或 XFS。当使用 RDMA 或 PCIe SSD 等低 I/O 延迟硬件时，内核空间和用户空间之间的消息传递开销会影响 I/O 吞吐量。
* 实例存储无法支持 shared-everything 数据库集群架构，而这是高级云数据库服务的一个关键特性。

### PolarFS 特点

PolarFS 通过以下机制实现了极低延迟、高吞吐量和高可用性。

* PolarFS 充分利用了 RDMA 和 NVMe SSD 等新兴硬件，在用户空间实现了轻量级的网络栈和 I/O 栈，避免陷入内核和处理内核锁
* PolarFS 提供了一个类似于 POSIX 的文件系统 API，其目的是编译到数据库进程中，替代操作系统提供的文件系统接口，使整个 I/O 路径可以保留在用户空间。
* PolarFS 的数据面的 I/O 模型设计为在关键数据路径上消除锁和避免上下文切换：去掉所有不必要的内存拷贝，同时大量使用 DMA 在主存和 RDMA 网卡/NVMe 磁盘之间传输数据。

> 通过以上特性极大的降低了 PolarFS 端到端的实延，使其非常接近 SSD 上的本地文件系统。

PolarFS 为了确保所有提交的修改在极端情况下不会丢失，选择使用 Raft 作为共识协议，来保持多个副本的一致性。而朴素 Raft 在使用超低延迟硬件（如 NVMe SSD 和 RDMA，其延迟在几十微秒量级）时严重阻碍了 PolarFS 的 I/O 可扩展性。

* 故 PolarFS 基于 Raft 开发了 ParallelRaft 使其允许乱序的日志确认（ack）、提交（commit）和应用（apply），同时符合传统的 I/O 语义。

> 通过该协议，PolarFS 的 I/O 并发性得到了显著提升。

## PolarFS 架构

TODO

## I/O 执行模型

TODO

## 一致性模型

TODO

## 参考资料

* [PolarFS: An Ultra-low Latency and Failure Resilient Distributed File System for Shared Storage Cloud Database](https://www.vldb.org/pvldb/vol11/p1849-cao.pdf)
