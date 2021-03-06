---
title: "Introduction to NewSQL"
date: 2017-10-15T10:00:00+08:00
permalink: /slides/tidb
description: ""
tags: [TiDB, NewSQL]
categories: ["slide"]
---

class: center, middle, inverse

# Introduction to NewSQL


张彦功

2017-10-15

---

.right-column[
### contents
- Database History

- What's NewSQL

- Basic of TiDB

- Summary
]

---
class: center, middle, inverse

# Database History

---

.left-column[
  ## Database
  ### Problem
]

.right-column[
### 遇到的问题
- 期望每个微服务都是无状态的，但是很多时候状态都是不可避免的
- 通过存储层保存状态，业界最主要的还是各种数据库
- 数据一致性问题，更加重了对存储层的依赖
]

---
.left-column[
  ## Database
  ### Problem
  ### History
]

.right-column[
### SQL
- 1970s 由 IBM 提出
- 基于关系代数和元组关系演算，包括一个数据定义语言和数据操纵语言

### 产品
- Oracle, MySQL
- Microsoft SQL Server
- PostgreSQL
]

---
.left-column[
  ## Database
  ### Problem
  ### History
]

.right-column[
### NoSQL
- 2009 年提出
- 非关联型的

### 产品
- Key-value: redis
- Document: MongoDB
- Wide column: HBase
]

---
class: center, middle, inverse

# NewSQL

---
.left-column[
  ## NewSQL
  ### Feature
]

.right-column[
### NewSQL 特性
- 弹性伸缩：资源根据业务需求按需分配

- 高可用：实现数据一致性的数据复制方式来保证服务的高可用（Paxos/Raft）

- 易用透明：容量足够大，兼容性好，应用层迁移方便等

- 高吞吐：云数据库更多的是追求高吞吐，而不是低延迟

- 自动负载平衡：决定了云数据库系统性能的好坏
]

???
弹性伸缩
以 MySQL Sharding 为代表的数据分片方案，很多时候不得不提前对数据量进行规划，把扩容作为很重要的一个计划来做，从 DBA 到运维到测试到开发人员，很早之前就要做相关的准备工作，真正扩容的时候，为了保证数据安全，经常会选择停服务来保证没有新的数据写入，新的分片数据同步后还要做数据的一致性校验。

高吞吐
相对传统单机的数据库来说，在数据访问链路上至少也要多走一次网络，所以大部分并发量不大的小数据量请求，都会比单机延迟要高一些。
当并发大到一定规模，云数据库高吞吐特性就显现出来了，即使在很高的并发下，依然可以维持相当稳定的延迟

自动负载平衡
如果一个数据库节点的数据访问过热的话，就需要考虑把数据迁移到其他的数据库节点来分担负载，不然就很容易出现性能瓶颈



---
.left-column[
  ## NewSQL
  ### Feature
  ### Theory
]

.right-column[
- Spanner: Google’s Globally-Distributed Database, Google, Inc. 2012.10

- F1: A Distributed SQL Database That Scales, Google, Inc. 2013.8
]

???
Raft
斯坦福大学

---
.left-column[
  ## Theory
  ### Spanner
]

.right-column[
Spanner 项目源于 Google Adwords 系统在持久化方面的需要

该解决方案既要满足关系型与事务性，同时又要在全球范围内可伸缩部署

![spanner_server_organization](/images/tidb/spanner_server_organization.jpg)
]

???
Universe。一个Spanner部署实例称之为一个Universe。目前全世界有3个。一个开发，一个测试，一个线上。因为一个Universe就能覆盖全球，不需要多个。
Zones. 每个Zone相当于一个数据中心，一个Zone内部物理上必须在一起。而一个数据中心可能有多个Zone。可以在运行时添加移除Zone。一个Zone可以理解为一个BigTable部署实例
Universemaster: 监控这个universe里zone级别的状态信息
Placement driver：提供跨区数据迁移时管理功能
Zonemaster：相当于BigTable的Master。管理Spanserver上的数据。
Location proxy：存储数据的Location信息。客户端要先访问他才知道数据在那个Spanserver上。
Spanserver：相当于BigTable的ThunkServer。用于存储数据。

---
.left-column[
  ## Theory
  ### Spanner
]

.right-column[
核心特性：
- 高扩展性
- 多版本
- 世界级分布
- 同步复制

有趣的特性：
- 应用可以细粒度的指定数据分布的位置
- 读写的外部一致性，基于时间戳的全局的读一致

TrueTime API，全球时间同步机制，将数据中心之间的时间同步精确到 10ms 以内
]

???
应用可以细粒度的指定数据分布的位置。精确的指定数据离用户有多远，可以有效的控制读延迟(读延迟取决于最近的拷贝)。指定数据拷贝之间有多远，可以控制写的延迟(写延迟取决于最远的拷贝)。还要数据的复制份数，可以控制数据的可靠性和读性能。(多写几份，可以抵御更大的事故)

Spanner还有两个一般分布式数据库不具备的特性：读写的外部一致性，基于时间戳的全局的读一致。这两个特性可以让Spanner支持一致的备份，一致的MapReduce，还有原子的Schema修改

用GPS和原子钟实现的时间API

---
.left-column[
  ## Theory
  ### Spanner
  ### F1
]

.right-column[
### F1
- 零容忍、全球分布式 OLTP 和 OLAP 数据库
- 结合了 NoSQL 的高可用、可扩展和传统 SQL 数据库的一致性、易用性
- 基于 Spanner 实现
- 分布式 SQL 查询引擎
]

---
.left-column[
  ## Theory
  ### Spanner
  ### F1
]

.right-column[
![F1_arch](/images/tidb/F1_arch.png)
]

---
.left-column[
  ## Theory
  ### Spanner
  ### F1
]

.right-column[
### 2012 年初，Google 把广告业务迁移到 F1
- 100+ 应用

- 1000+ 开发者

- 100TB+ 数据

- 三个 F1 实例：开发、测试、生成各一个

- 相比之前的 Sharding MySQL 方案，延迟没有增加
]

---
class: center, middle, inverse

# TiDB

---
.left-column[
  ## TiDB
  ### - 简介
]

.right-column[
TiDB 是 PingCAP 公司基于 Google Spanner/F1 论文实现的开源 NewSQL 数据库

TiDB 具备如下 NewSQL 核心特性：

- SQL 支持
- 水平线性弹性扩展
- 分布式事务
- 跨数据中心数据强一致性保证
- 故障自恢复的高可用
- TiDB 的设计目标是 100% 的 OLTP 场景和 80% 的 OLAP 场景
]

???
TiDB 对业务没有任何侵入性，能优雅的替换传统的数据库中间件、数据库分库分表等 Sharding 方案。同时它也让开发运维人员不用关注数据库 Scale 的细节问题，专注于业务开发，极大的提升研发的生产力

---
.left-column[
  ## TiDB
  ### - 简介
]

.right-column[
### 版本发布历史
- 开源，2015 年 9 月 6 日
- RC1，2016 年 12 月 23 日，总共发布了 4 个 RC 版本
- Pre-GA，2017 年 8 月 30 日
- GA 1.0，2017 年 10 月 16 日

从零开始到发布 1.0 版本，历时 2 年 6 个月
]

---
class: middle, center

.column[
![tidb-architecture](/images/tidb/tidb-architecture.png)
TiDB 整体架构
]

---

.left-column[
  ## TiDB
  ### - 简介
  ### - 架构
  ### - 三大组件
]

.right-column[
### TiDB Server 功能
- 接收 SQL 请求并处理 SQL 相关的逻辑
- 通过 PD 找到存储计算所需数据的 TiKV 地址
- 与 TiKV 交互获取数据，最终返回结果

### TiDB Server 特点
- 无状态的，其本身并不存储数据，只负责计算
- 无限水平扩展
- 通过负载均衡组件对外提供统一的接入地址
]

---

.left-column[
  ## TiDB
  ### - 简介
  ### - 架构
  ### - 三大组件

]

.right-column[
### PD Server

Placement Driver (简称 PD) 是整个集群的管理模块，主要工作：
- 存储集群的元信息（某个 Key 存储在哪个 TiKV 节点）
- 对 TiKV 集群进行调度和负载均衡
- 分配全局唯一且递增的事务 ID

PD 是一个集群，需要部署奇数个节点，一般线上推荐至少部署 3 个节点。
]

---

.left-column[
  ## TiDB
  ### - 简介
  ### - 架构
  ### - 三大组件

]

.right-column[
### TiKV Server

TiKV Server 负责存储数据，一个分布式的提供事务的 Key-Value 存储引擎。

存储数据的基本单位是 Region，每个 Region 负责存储一个 Key Range 的数据。

TiKV 使用 Raft 协议做复制，保持数据的一致性和容灾。

副本以 Region 为单位进行管理，不同节点上的多个 Region 构成一个 Raft Group。

数据在多个 TiKV 之间的负载均衡由 PD 调度，这里也是以 Region 为单位进行调度。
]

---

.left-column[
  ## 技术内幕
  ### - 存储
]

.right-column[
### TiKV
- Key-Value 模型，提供有序遍历方法

- Key 和 Value 都是原始的 Byte 数组

- Key 按照 Byte 数组总的原始二进制比特位比较顺序排列

- 数据落地由 RocksDB 负责
]

???
TiKV 没有选择直接向磁盘上写数据，而是把数据保存在 RocksDB 中，具体的数据落地由 RocksDB 负责。
RocksDB 是一个非常优秀的开源的单机存储引擎

---
.left-column[
  ## 技术内幕
  ### - 存储
]

.right-column[
### TiKV
高可用的数据复制方案 Raft
1. Leader 选举
2. 成员变更
3. 日志复制

![raft-rocksdb](/images/tidb/raft-rocksdb.png)
]

???
TiKV 利用 Raft 来做数据复制，每个数据变更都会落地为一条 Raft 日志，通过 Raft 的日志复制功能，将数据安全可靠地同步到 Group 的多数节点中。

数据的写入是通过 Raft 这一层的接口写入，而不是直接写 RocksDB。通过实现 Raft，我们拥有了一个分布式的 KV，现在再也不用担心某台机器挂掉了。

---
.left-column[
  ## 技术内幕
  ### - 存储
]

.right-column[
### Region
将整个 Key-Value 空间分成很多段，每一段是一系列连续的 Key

会尽量保持每个 Region 中保存的数据不超过一定的大小

### 两件重要的事情
- 以 Region 为单位，将数据分散在集群中所有的节点上，并且尽量保证每个节点上服务的 Region 数量差不多

- 以 Region 为单位做 Raft 的复制和成员管理
]

???
存储容量的水平扩展（增加新的结点后，会自动将其他节点上的 Region 调度过来）
负载均衡（不会出现某个节点有很多数据，其他节点上没什么数据的情况）

有一个组件记录 Region 在节点上面的分布情况，也就是通过任意一个 Key 就能查询到这个 Key 在哪个 Region 中，以及这个 Region 目前在哪个节点上。至于是哪个组件负责这两项工作，会在后续介绍

---
.left-column[
  ## 技术内幕
  ### - 存储
]

.right-column[
### Region
TiKV 是以 Region 为单位做数据的复制，一个 Region 的数据会保存多个副本
一个 Region 的多个 Replica 会保存在不同的节点上，构成一个 Raft Group
其中一个 Replica 会作为这个 Group 的 Leader，其他的 Replica 作为 Follower
![raft-region](/images/tidb/raft-region.png)
]

???
TiKV 是以 Region 为单位做数据的复制，也就是一个 Region 的数据会保存多个副本，我们将每一个副本叫做一个 Replica。

Repica 之间是通过 Raft 来保持数据的一致，一个 Region 的多个 Replica 会保存在不同的节点上，构成一个 Raft Group。

其中一个 Replica 会作为这个 Group 的 Leader，其他的 Replica 作为 Follower。

---
.left-column[
  ## 技术内幕
  ### - 存储
]

.right-column[
### MVCC
在 Key 后面添加 Version 来实现

对于同一个 Key 的多个版本，把版本号较大的放在前面

Key1-Version3 -> Value

Key1-Version2 -> Value

Key1-Version1 -> Value
]

---
.left-column[
  ## 技术内幕
  ### - 存储
]

.right-column[
### 事务
基于 Percolator 模型

TiKV 的事务采用乐观锁，事务的执行过程中，不会检测写写冲突，只有在提交过程中，才会做冲突检测，冲突的双方中比较早完成提交的会写入成功，另一方会尝试重新执行整个事务
]

???
略讲

---
.left-column[
  ## 技术内幕
  ### - 计算
]

.right-column[
### 关系模型到 Key-Value 模型的映射

对于一个 Table 来说，需要存储的数据包括三部分：
- 表的元信息
- Table 中的 Row
- 索引数据

Select 语句。
- 能够简单快速地读取一行数据，所以每个 Row 需要有一个 ID
- 读取连续多行数据
- 通过索引读取数据的需求，对索引的使用可能是点查或者是范围查询。
]

???
一个全局有序的分布式 Key-Value 引擎

- 对于 Insert 语句，需要将 Row 写入 KV，并且建立好索引数据。

- 对于 Update 语句，需要将 Row 更新的同时，更新索引数据（如果有必要）。

- 对于 Delete 语句，需要在删除 Row 的同时，将索引也删除。


---
.left-column[
  ## 技术内幕
  ### - 计算
]

.right-column[

TiDB 对每个表分配一个 TableID，每一个索引都会分配一个 IndexID，每一行分配一个 RowID

TableID 在整个集群内唯一，IndexID/RowID 在表内唯一，这些 ID 都是 int64 类型


每行数据按照如下规则进行编码成 Key-Value pair：

```
Key： tablePrefix_rowPrefix_tableID_rowID
Value: [col1, col2, col3, col4]
```

对于 Unique Index 数据，会按照如下规则编码成 Key-Value pair：
```
Key: tablePrefix_idxPrefix_tableID_indexID_indexColumnsValue
Value: rowID
```
对于非 Unique Index 数据，会按照如下规则编码成 Key-Value pair：
```
Key: tablePrefix_idxPrefix_tableID_indexID_ColumnsValue_rowID
Value：null
```
]

???
如果表有整数型的 Primary Key，那么会用 Primary Key 的值当做 RowID

---
.left-column[
  ## 技术内幕
  ### - 计算
]

.right-column[
### TiDB Servers
- 节点都是无状态的节点，节点之间完全对等
- 工作是处理用户请求，执行 SQL 运算逻辑

### 分布式 SQL 运算
- 计算尽量靠近存储节点
- Filter 下推到存储节点进行计算
- 聚合函数、GroupBy 下推到存储节点，进行预聚合
]

---
.left-column[
  ## 技术内幕
  ### - 计算
]

.right-column[
### SQL 层
- tidb-server 会解析 MySQL Protocol Packet，获取请求内容，然后做语法解析、查询计划制定和优化、执行查询计划获取和处理数据

- 数据全部存储在 TiKV 集群中，所以在这个过程中 tidb-server 需要和 tikv-server 交互，获取数据

- tidb-server 需要将查询结果返回给用户
]

---
.left-column[
  ## 技术内幕
  ### - 调度
]

.right-column[
### 调度的需求

作为一个分布式高可用存储系统，必须满足的需求，包括四种：

- 副本数量不能多也不能少
- 副本需要分布在不同的机器上
- 新加节点后，可以将其他节点上的副本迁移过来
- 节点下线后，需要将该节点的数据迁移走
]

---
.left-column[
  ## 技术内幕
  ### - 调度
]

.right-column[
### 调度的需求

作为一个良好的分布式系统，需要优化的地方，包括：

- 维持整个集群的 Leader 分布均匀
- 维持每个节点的储存容量均匀
- 维持访问热点分布均匀
- 控制 Balance 的速度，避免影响在线服务
- 管理节点状态，包括手动上线/下线节点，以及自动下线失效节点
]

???
满足第一类需求后，整个系统将具备多副本容错、动态扩容/缩容、容忍节点掉线以及自动错误恢复的功能。满足第二类需求后，可以使得整体系统的负载更加均匀、且可以方便的管理。

---
.left-column[
  ## 技术内幕
  ### - 调度
]

.right-column[
### 调度的基本操作
- 增加一个 Replica
- 删除一个 Replica
- 将 Leader 角色在一个 Raft Group 的不同 Replica 之间 transfer

刚好 Raft 协议能够满足这三种需求，通过 AddReplica、RemoveReplica、TransferLeader 这三个命令，可以支撑上述三种基本操作。
]

---
.left-column[
  ## 技术内幕
  ### - 调度
]

.right-column[
### 信息收集
- 每个 TiKV 节点会定期向 PD 汇报节点的整体信息
- 每个 Raft Group 的 Leader 会定期向 PD 汇报信息

PD 不断的通过这两类心跳消息收集整个集群的信息，再以这些信息作为决策的依据

除此之外，PD 还可以通过管理接口接受额外的信息，用来做更准确的决策。
]

---
.left-column[
  ## 总结
]

.right-column[
### TiDB 的最佳适用场景
- 数据量大，单机保存不下
- 不希望做 Sharding
- 访问模式上没有明显的热点
- 需要事务、需要强一致、需要灾备
]

---

.left-column[
  ## 参考资料
]

.right-column[
- [1]: https://www.pingcap.com/blog-cloud-native-db-zh 云时代数据库的核心特点
- [2]: https://www.pingcap.com/docs-cn TiDB 简介与整体架构
- [3]: https://www.pingcap.com/blog-tidb-internal-1-zh 三篇文章了解 TiDB 技术内幕 - 说存储
- [4]: https://www.pingcap.com/blog-tidb-internal-2-zh 三篇文章了解 TiDB 技术内幕 - 说计算
- [5]: https://www.pingcap.com/blog-tidb-internal-3-zh 三篇文章了解 TiDB 技术内幕 - 谈调度
- [6]: Spanner: Google’s Globally-Distributed Database, Google, Inc. 2012.10
- [7]: F1: A Distributed SQL Database That Scales, Google, Inc. 2013.8
]

---
class: center, middle, inverse

# Q&A
