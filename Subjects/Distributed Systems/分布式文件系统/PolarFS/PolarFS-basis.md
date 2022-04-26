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
    - [File System Layer](#file-system-layer)
    - [Storage Layer](#storage-layer)
  - [I/O 执行模型](#io-执行模型)
    - [写操作](#写操作)
    - [读操作](#读操作)
  - [一致性模型](#一致性模型)
  - [File System Layer 实现](#file-system-layer-实现)
    - [Metadata Organization](#metadata-organization)
    - [Coordination and Synchronization](#coordination-and-synchronization)
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

PolarFS 主要包括两层：

* 文件系统层（File System Layer），负责管理文件系统元数据，提供文件系统 API。
  * 文件系统层支持卷内的文件管理，负责文件系统元数据并发访问的互斥和同步。对于数据库实例，PolarFS 将文件系统元数据存储在其卷（volume）中。
* 存储层（Storage Layer）负责存储管理。
  * 存储层负责存储节点的所有磁盘资源，为文件系统层提供管理和访问磁盘卷的接口。

![PolarFS 架构图](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204231627952.png)

### File System Layer

**功能：** 文件系统层提供了一个共享和并行的文件系统，可以由多个数据库节点同时访问。

**组件：**

* libpfs
  * libpfs 是一个完全在用户空间中运行的轻量级文件系统实现。
  * libpfs 提供了一组类似 POSIX 的文件系统的 API。
  * libpfs 提供了 `pfs_mount_growfs` API 识别新分配的块并将它们标记为可用，以达到磁盘卷扩容目的。

### Storage Layer

**功能：** 存储层为文件系统层提供管理和访问磁盘卷的接口。

**存储组织：** PolarFS 的存储层次分为 volume（卷）、chunk（区）和 block（块）。

![存储组织](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204241054188.png)

数据库节点（计算节点） n:1 volume 1:n chunk 1:163840 block

**volume：**

针对数据库实例而言，存储层会为每一个实例分配一个 volume，它由一个 chunk 列表组成。volume 可以通过追加 chunk 来实现按需扩容（从 10 GB 到 100 TB）。

跟其他传统的块设备一样，卷上的读写 I/O 以 512B 大小对齐，对卷上同个 chunk 的修改操作是原子的。

跟传统文件系统不一样的是 PolarFS 是个共享文件系统即一个卷会被挂载到多个计算节点上，也就是说可能存在有多个客户端（挂载点）对文件系统进行读写和更新操作，所以 PolarFS 在卷上额外维护了一个 Paxos 文件。每个客户端在更新Journal 文件前，都需要使用 Paxos 文件执行 Disk Paxos 算法实现对 Journal 文件的互斥访问。

**chunk：**

同一个磁盘卷被分成多个 chunk ，这些 chunk 分布在多个 ChunkServer 中。chunk 是数据分布的最小单位。单个 chunk 不会跨 ChunkServer 上的多个磁盘，默认情况下，它的副本会复制到三个不同的 ChunkServer（始终位于不同的机架中）。 当存在热点时，可以在 ChunkServer 之间迁移 chunk。

chunk 的大小设置为 10 GB，这种设计的优缺点：

优点：

* 减少数据库中维护的元数据量，以及元数据管理；
* 使其能够全部缓存在 PolarSwitch 的主存中，避免额外 IO 开销。

缺点：

不能进一步拆分一个 chunk 上的热点，

解决方法： 但是由于 chunk 与服务器的高比例（现在大约是 1000:1），通常有大量的数据库实例（数千个或更多），以及跨服务器的 chunk 迁移能力，PolarFS 可以实现整个系统级的负载均衡。

**block：**

一个 chunk 在 ChunkServer 内部被进一步划分为多个 block，每个 block 设置为64KB。 block 是按需分配并映射到 chunk 以实现精简配置。 一个 10 GB 的 chunk 包含 163840 个 block。

chunk 的 LBA（Logical Block Address，线性地址范围从0到10GB，表示 block 在 chunk 内的偏移）到 block 物理地址的映射表，本地存储在 ChunkServer 的中，同时存储在 ChunkServer 中的还有每个磁盘上空闲块的位图。单个 chunk 的映射表占用 640KB 内存，这个空间相当小，可以缓存在 ChunkServer 的内存中。

**组件：**

* `PolarSwitch` 驻留在 **计算节点** 上（数据库节点），将 I/O 请求从应用程序重定向到 `ChunkServers`；
  * 在每个数据库进程中，libpfs 将 I/O 请求转发到 PolarSwitch 守护进程。每个请求都包含寻址信息，如卷标识符、偏移量和长度，从中可以识别相关的 chunk 。
  * 一个 I/O 请求可能跨越多个 chunk，在这种情况下，请求被进一步划分为多个子请求。最后一个原始的请求（elemental request）被发送到 chunk 的 leader 所在的 ChunkServer 。
  * PolarSwitch 通过查找本地元数据缓存，找出一个 chunk 的所有副本的位置，本地元数据缓存会与 PolarCtrl 同步。一个 chunk 的副本形成一个共识组，其中一个是 leader，其他是 follower。只有leader 可以应答 I/O 请求。共识组中的 leader 发生变化也同步和缓存到 PolarSwitch 的本地缓存中。如果发生响应超时，PolarSwitch 将继续以指数补偿的方式重试，同时检测是否发生了 leader 选举，如果发生了，则切换到新的 leader 并立即重传。
* `ChunkServer` 部署在 **存储节点** 上，为 I/O 请求提供服务；
  * 一个存储服务器上运行着多个 ChunkServer 进程，每个 ChunkServer 拥有一个独立的 NVMe SSD 盘，并绑定到一个专用的 CPU 核上，这样两个 ChunkServer 之间就不会发生资源争用。
  * ChunkServer 负责存储 chunk 并提供对 chunk 的随机访问。每个 chunk 都包含一个预写日志 (WAL)，在更新 chunk blocks 之前，对 chunk 的修改会追加到日志中，以确保原子性和持久性。
  * ChunkServer 使用 3D XPoint SSD 和普通 NVMe SSD 混合的 WAL 的写缓存，日志优先放置在 3D XPoint 缓冲区中。如果缓冲区已满，ChunkServer 尝试回收过期的日志。如果 3D XPoint 缓冲区中仍然没有足够的空间，则将日志写入 NVMe SSD。而 chunk blocks 始终写入 NVMe SSD。
  * ChunkServer 使用 ParallelRaft 协议相互复制 I/O 请求并形成共识组。一个 ChunkServer 可能因为各种原因离开其共识组，需要根据具体的原因进行处理。有时它是由偶然和临时故障引起的，例如网络暂时不可用，或服务器升级并重新启动。这种情况最好等待断开的 ChunkServer 重新上线，重新加入赶上其他人；另一些情况下，故障是永久性的或往往会持续很长时间，例如服务器损坏或脱机。这时候应该将对应的 ChunkServer 上的所有 chunk 迁移到其他 ChunkServer 上，以重新获得足够数量的副本。
* `PolarCtrl` 是控制面，它部署在一组专用机器（至少三台）上，以提供高可用服务。PolarCtrl 提供集群控制服务，例如 节点管理、卷管理、资源分配、元数据同步、监控等。
  * 作用
    * 跟踪集群中所有 ChunkServer 的成员资格和活跃度，如果 ChunkServer 过载或不可用的持续时间超过阈值，则开始将块副本从一台服务器迁移到其他服务器。
    * 在元数据存储的数据库中维护所有卷的状态和 chunk 位置信息。
    * 创建卷并将 chunk 分配给 ChunkServers。
    * 使用 push 和 pull 方法将元数据同步到 PolarSwitch。
    * 监控每个卷和 chunk 的延迟/IOPS 指标，沿 I/O 路径收集跟踪数据。
    * 定期调度副本内和副本间的 CRC 校验。
  * PolarCtrl 通过控制面命令定期将集群元数据（例如卷的 chunk 的位置）与 PolarSwitch 同步。PolarSwitch 将元数据保存在其本地缓存中。PolarSwitch 在接收到来自 libpfs 的 I/O 请求时，根据本地缓存将请求路由到相应的 ChunkServer。 有时，如果本地缓存落后于中心元数据存储库，PolarSwitch 会从 PolarCtrl 获取元数据。
  * 作为控制平面，PolarCtrl 不在关键 I/O 路径上，可以使用传统的高可用性技术保证其服务连续性。 即使在 PolarCtrl 崩溃和恢复之间的短暂间隔期间，PolarFS 中的 I/O 流也不太可能因为 PolarSwith 上缓存的元数据和 ChunkServer 的自管理受到影响。

## I/O 执行模型

### 写操作

当 POLARDB 访问数据时，它会通过 PFS 接口将文件 I/O 请求委托给 libpfs，通常通过 pfs_pread 或 pfs_pwrite 接口。 对于写请求，几乎不需要修改文件系统元数据，因为 block 是通过 pfs_fallocate 预先分配给文件的，从而避免了读写节点之间代价高昂的元数据同步。这是数据库系统的常见优化。

在大多数常见情况下，libpfs 只是根据挂载时已经构建的索引表将文件偏移量映射到块偏移量，并将文件 I/O 请求切割为一个或多个较小的固定大小的块 I/O 请求。 转换后，块 I/O 请求由 libpfs 通过它们之间的共享内存发送到 PolarSwitch。

共享内存的结构为多个环形缓冲区（ring buffer）。在共享内存的一端，libpfs 将块 I/O 请求放入一个环形缓冲区，环形缓冲区通过轮询的方式进行选择，然后等待其执行完成。在另一端，PolarSwitch 通过一个专用的线程不断轮询所有环形缓冲区。
一旦发现新请求，PolarSwitch 将请求从环形缓冲区取出，并将这些请求根据从 PolarCtrl 获得的路由信息转发到对应的 chunk 的 leader 节点。

ChunkServer 使用预写日志 (WAL) 技术来确保原子性和持久性，其中 I/O 请求在提交和应用之前先写入日志。日志被复制到其他副本集，并使用名为 ParallelRaft（下一节详述）的共识协议来保证副本之间的数据一致性。在 I/O 请求被持久化到大多数副本的日志中之前，它不会被识别为已提交。只有在此之后，该请求才能对客户端响应并应用于数据块。

![写操作 I/O 流](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202204241648253.png)

1. POLARDB 通过 PolarSwitch 和 libpfs 之间的环形缓冲区向 PolarSwitch 发送写 I/O 请求。
2. PolarSwitch 根据本地缓存的集群元数据，将请求传递给对应的 chunk 的 leader 节点。
3. 当新的写请求到达时，Leader 节点中的 RDMA 网卡会将写请求放入提前分配好的缓冲区，并据此构造一个请求对象加入请求队列中。I/O 循环线程不断轮询请求队列。一旦它看到一个新的请求到达，它就会立即开始处理。
4. 请求通过 SPDK 写入 chunk 对应的 WAL 日志块，通过 RDMA 传播到 follower 节点。这两个操作都是异步调用，所以数据传输是并发进行的。
5. 当请求到达一个 follower 节点时，follower 节点中的 RDMA 网卡也会将复制请求放入提前分配的缓冲区中，并加入到复制队列中。
6. 然后 follower 上的 I/O 循环线程被触发，将请求通过 SPDK 异步写入 WAL 日志块。
7. 当 follower 上的 write 成功后，回调函数通过 RDMA 向 leader 发回一个 acknowledge response。
8. 当成功收到大多数 follower 的响应后，leader 通过 SPDK 向数据块申请写请求。
9. 之后，leader 通过 RDMA 回复 PolarSwitch。
10. PolarSwitch 然后将请求标记为完成并通知客户端。

### 读操作

读取 I/O 请求（更简单）由 leader 直接处理。在 ChunkServer 中有一个名为 IoScheduler 的子模块，它负责仲裁并发的 I/O 请求，确定这些请求在 ChunkServer 上磁盘的 I/O 操作顺序。IoScheduler 保证读操作总能获到最新已提交的数据。

## 一致性模型

TODO

## File System Layer 实现

### Metadata Organization



### Coordination and Synchronization



## 参考资料

* [PolarFS: An Ultra-low Latency and Failure Resilient Distributed File System for Shared Storage Cloud Database](https://www.vldb.org/pvldb/vol11/p1849-cao.pdf)
* [阿里云PolarDB及其存储PolarFS技术实现分析](https://zhuanlan.zhihu.com/p/44874330)
