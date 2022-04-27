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
    - [场景介绍](#场景介绍)
    - [朴素 Raft 缺点](#朴素-raft-缺点)
    - [乱序 Apply 前提](#乱序-apply-前提)
    - [ParallelRaft 模块](#parallelraft-模块)
      - [日志复制](#日志复制)
      - [Leader 选举](#leader-选举)
      - [Catch Up](#catch-up)
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

### 场景介绍

PolarFS 使用 ParallelRaft 协议相互复制 ChunkServer 的 I/O 请求并形成共识组。一个 ChunkServer 可能因为各种原因离开其共识组，需要根据具体的原因进行处理：

* 由偶然和临时故障引起的，例如网络暂时不可用，或服务器升级并重新启动。这种情况最好等待断开的 ChunkServer 重新上线，重新加入并赶上其他人；
* 故障是永久性的或往往会持续很长时间，例如服务器损坏或脱机。这时候应该将对应的 ChunkServer 上的所有 chunk 迁移到其他 ChunkServer 上，以重新获得足够数量的副本。

**补充：复制日志的实现**

**eg 1. 基于语句的复制**

主节点记录所执行的每个写请求（操作语句）并将该操作语句作为日志发送给从节点。对于关系数据库，这意味着每个 INSERT、UPDATE 或 DELETE 语句都会转发给从节点，然后每个从节点都会分析并执行这些 SQL 语句。

缺点，这种复制方式不适用以下场景：

* 任何调用非确定性函数的语句，如 `NOW()` 获取当前时间，或 `RAND()` 获取一个随机数等，可能会在不同的副本上产生不同的值。
* 如果语句中使用了自增列，或者依赖于数据库的现有数据（例如，`UPDATE ... WHERE <某些条件>`），则所有副本必须按照完全相同的顺序执行，否则可能会出现不同的结果。进而，如果有多个同时并发执行的事务时，会有很大的限制。
* 有副作用的语句（例如，触发器、存储过程、用户定义的函数等），可能会在每个副本上产生不同的副作用。

解决方法：

主节点可以在记录操作语句时将非确定性函数替换为执行之后的确定的结果，以保证所有节点直接使用相同的结果值。但该方法，需要考虑非常多的边界条件，因此基于语句的复制不是当前首选的复制实现方法。

> MySQL 基于 SQL 语句（binlog_format = STATEMENT）级别的 binlog 采用的就是基于语句的复制，该格式记录日志的逻辑 SQL 语句。

**eg 2. 基于预写日志（WAL）传输**

对数据库写入的字节序列都被记入日志。因此可以使用完全相同的日志在另一个节点上构建副本，即主节点除了将日志写入磁盘之外，还通过网络将其发送给从节点。

缺点：

预写日志（WAL）描述的数据非常底层：包含哪些磁盘块的哪些字节发生改变等。这使得复制方案和存储引擎紧密耦合，系统通常无法支持主从节点上运行不同版本的软件。

> MySQL InnoDB 存储引擎通过 redo、undo 日志实现 WAL。

**eg 3. 基于行的逻辑日志复制**

复制和存储引擎采用不同的日志格式，将复制与存储逻辑剥离。这种复制日志称为逻辑日志，以区分物理存储引擎的数据表示。

关系数据库的逻辑日志通常是指一系列记录来描述数据表行级别的写请求（包括：行插入、行删除、行更新），如果一条事务涉及多行的修改，则会产生多条这样的日志记录，并在后面跟着一条记录，指出该事务已经提交（MySQL binlog raw 形式使用该方式）。

优点：

* 逻辑日志与存储引擎解耦，因此可以更加容易地保持向后兼容，从而使主从节点能够运行不同版本的软件甚至是不同的存储引擎。
* 对于外部应用程序而言，逻辑日志格式将更容易解析。

> MySQL ROW 格式下（binlog_format = ROW）binlog 采用的就是基于行的逻辑日志复制，该格式记录表的行更改情况。

**eg 4. 基于触发器的复制**

`eg 1-3` 的复制方法均是由数据库系统实现的，不涉及任何应用程序代码。但是，在某些情况下，可能需要更高的灵活性，例如：

* 只想复制数据的一部分；
* 从一种数据库复制到另一种数据库；
* 需要订制、管理冲突解决逻辑，此时需要将复制控制权交给应用程序层。

实现方法：

* 部分工具，如 Oracle GoldenGate，可以通过读取数据库日志让应用程序获取数据变更；
* 大部分关系型数据库支持的功能：触发器和存储过程。
  * 触发器支持注册自己的应用层代码，使得当数据库系统发生数据更改（写事务）时自动执行上述自定义代码，触发器优缺点：
    * 优点：高度灵活性。
    * 缺点：通常比其他复制方式开销更大，也比数据库内置复制更容易出错，或者暴露一些限制。

TODO 阿里的选择（优缺点分析）、金山云的选择（优缺点分析）

### 朴素 Raft 缺点

Raft 被设计为高度序列化，以达到简单和可理解的目的。leader 和 follower 上的日志都不允许有空洞，这意味着 **日志项在 follower 确认（ack），leader 提交（commit）及应用（apply）到所有副本时都是按照顺序执行的。**

因此当 follower 收到乱序到达的日志时，在前面所有的请求（还未收到的日志项）都持久存储到磁盘并得到响应之前，队列尾部（提前到达的日志项）的那些请求无法提交和响应，这会 **增加平均延迟并降低吞吐量**。而对于高并发的环境，由于网络的问题出现乱序情况是很常见且无法避免的。

### 乱序 Apply 前提

MySQL、AliSQL 等数据库并不关心底层存储的 I/O 顺序。数据库的锁机制可以保证在任何时间点，只有一个线程可以在一个特定的 page 上工作。当不同的线程同时在不同的 page 上工作时，数据库只需要成功执行 I/O，它们的完成顺序无关紧要。

PolarFS 利用这一点放宽 Raft 中的严格顺序性限制，开发出更适合高 I/O 并发的 ParallelRaft。

> 页（page）是 MySQL InnoDB 存储引擎磁盘管理的最小单位，每个页默认 16 KB。

TODO 1. 金山的原因 binlog 逻辑日志设计

### ParallelRaft 模块

#### 日志复制

**Raft 需要在以下两个方面严格串行化：**

* follower 确认（ack）：leader 发送一条日志项到 follower 之后，follower 需要对此进行确认，表明这条日志项被收到和记录了，同时也显式地表明之前所有的日志项都已经收到和记录了。
* leader 提交（commit）：当 leader 提交一条日志项，并将此事件广播给所有的 follower ，也表明之前所有的日志项都已经提交了。

以上两方面的串行化同时也保证了应用层状态机 apply 时的串行化。

**ParallelRaft 打破了上述限制，使其可以乱序确认（ack），leader 提交（commit）及应用（apply）。**

因此，ParallelRaft 和 Raft 有些本质区别，如：

* ParallelRaft 中 follower 在收到日志项时，不必再根据 `preLogIndex` 判断是否乱序，同时 follower 确认该日志项，也不再代表已经收到和记录了之前的日志项。
* ParallelRaft 中 leader 在确认某一日志项获得超半数确认，可以提交时，并不意味着之前的所有项都已经成功提交。即 `last_committed_index` 失去了原有的语义。

**为了确保协议的正确性，需要保证：**

* 去掉严格序列化的限制后，所有的提交状态应该不违反传统关系型数据库中的存储语义。
* 所有已经提交的修改在任何极端场景下都不会丢失。

**乱序执行规则：**

ParallelRaft 的乱序日志执行遵循以下规则：如果日志项的写入范围彼此没有交集，则认为这些日志条目没有冲突，可以按任意顺序执行。否则，有冲突的项将严格按照到达顺序执行。这样，较新的数据永远不会被旧版本覆盖。

**冲突判断规则：**

ParallelRaft 可以很容易地知道冲突，因为它存储了所有未应用的日志项的 LBA 范围统计。

**执行步骤：**

以下部分描述了如何优化 ParallelRaft 的 Ack-Commit-Apply 步骤以及如何维持必要的一致性语义。

**Out-of-Order Acknowledge：**

* Raft follower 在收到 leader 复制的日志项后，要等到所有之前的日志条目都持久化存储后才会确认（ack），这会引入额外的等待时间，并且当有大量并发 I/O 写入执行时，平均延迟会显著增加。
* 但是，在 ParallelRaft 中，一旦日志条目写入成功，follower 就可以立即确认，这样就避免了额外的等待时间，从而优化了平均延迟。

**Out-of-Order Commit：**

* Raft leader 按顺序提交日志条目，一条日志项只有在它之前所有的日志项提交后，才能提交。
* 而在 ParallelRaft 中，可以在大多数副本确认后就立即提交（commit）日志项。

**Apply with Holes in the Log：**

* Raft 中，所有日志项都严格按照 Raft 中记录日志的顺序被应用（apply），因此数据文件在所有副本中都是一致的。
* 然而，通过乱序的日志复制和提交，ParallelRaft 允许日志中存在空洞。
  * 为确保在某些先前的日志项仍然缺失的情况下安全地应用某条日志项，ParallelRaft 在每个日志条目中，引入了一个名为 look behind buffer 的新数据结构来解决这个问题。look behind buffer 包含被之前的 N 个日志项修改过的 LBA，look behind buffer 在日志中可能存在的空洞的情况下，扮演了沟通桥梁的角色。
  * N 是这座桥的跨度，同时也是允许的最大空洞的数量。请注意，尽管日志中可能存在多个空洞，但所有日志项的 LBA 统计始终是完整的，除非某些空洞的大小大于 N。
  * 通过这个数据结构，follower 可以判断一个日志项是否有冲突，冲突意味着这个日志项修改的 LBAs 与一些它之前缺失的某些日志项之间有交集。
  * 可以安全地应用与任何其他条目不冲突的日志项，否则应该将这些日志项添加到待处理列表中，并在之前缺失的日志项被应用后再进行处理。
  * 根据我们使用 RDMA 网络的 PolarFS 的经验，N 设置为 2 足以满足其 I/O 并发性。

#### Leader 选举

**Raft：**

在进行新的 leader 选举时，ParallelRaft 和 Raft 一样，选择包含最新数据项、拥有的日志项最多的节点。

**ParallelRaft：**

说明 合并 阶段

#### Catch Up

**Raft：**

在进行新的 leader 选举时，ParallelRaft 和 Raft 一样，选择包含最新数据项、拥有的日志项最多的节点。

**ParallelRaft：**

catch up

## File System Layer 实现

### Metadata Organization



### Coordination and Synchronization



## 参考资料

* [PolarFS: An Ultra-low Latency and Failure Resilient Distributed File System for Shared Storage Cloud Database](https://www.vldb.org/pvldb/vol11/p1849-cao.pdf)
* [阿里云PolarDB及其存储PolarFS技术实现分析](https://zhuanlan.zhihu.com/p/44874330)
