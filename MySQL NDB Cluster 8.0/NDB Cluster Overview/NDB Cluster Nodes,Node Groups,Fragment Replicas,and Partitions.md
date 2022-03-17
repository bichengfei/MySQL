# 23.2.2 NDB Cluster 节点、节点组、片段副本和分区

本节主要讨论 NDB 集群对于数据的切分和备份方式。

在接下来的几段中将讨论一些对理解该主题至关重要的概念。

### 数据节点

一个 ndbd 或 ndbmtd 进程，它存储一个或多个分片副本，分片副本是分配给该节点所属的组的分区的副本。

每个数据节点应位于单独的计算机上。虽然也可以在一台计算机上托管多个数据节点进程，但通常不推荐这种配置。

当指代 ndbd 或 ndbmtd 进程时，术语“节点”和“数据节点”通常可以互换使用；在本次讨论中，管理节点代指 ndb_mgmd 进程，SQL 节点代指 mysqld 进程。

### 节点组

节点组由一个或多个节点组成，并存储分区或片段副本集。

NDB Cluster 中的节点组数量不能直接配置，它是数据节点数量和片段副本数量（NoOfReplicas 配置参数）的函数，如下所示：

```sql
[# of node groups] = [# of data nodes] / NoOfReplicas
```

因此，对于有 4 个数据节点的 NDB 集群，我们可以在 config.ini 中配置 NoOfReplicas 参数：

+ NoOfReplicas = 1，则有 4 个节点组

+ NoOfReplicas = 2，则有 2 个节点组

+ NoOfReplicas = 4，则有 1 个节点组

本节后面将会对片段副本进行讨论，有关 NoOfReplicas 的更多信息，请参阅 [第 23.4.3.6 节，“定义 NDB Cluster 数据节点”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html "23.4.3.6 定义 NDB Cluster 数据节点")。

```textile
笔记
    NDB Cluster 中的所有节点组必须具有相同数量的数据节点
```

你可以在线将新节点组（以及新数据节点）添加到正在运行的 NDB Cluster；有关更多信息，请参阅 [第 23.6.7 节，“在线添加 NDB 集群数据节点”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node.html "23.6.7 在线添加 NDB Cluster 数据节点")。

### 分区（partiton）

这是集群存储的数据的一部分。对于分配给它的任何分区，每个节点负责保证至少一个副本可用于集群。

NDB 集群默认使用的分区数取决于数据节点的数量和数据节点使用的 LDM 线程数，如下所示：

```sql
[# of partitions] = [# of data nodes] * [# of LDM threads]
```

当数据节点使用 ndbd 进程时，只有一个 LDM 线程，这意味着集群分区与参与集群的节点一样多。当使用 ndbmtd 时，LDM 线程的数量由`MaxNoOfExecutionThreads`参数控制，当设置`MaxNoOfExecutionThreads`小于等于 3 时，集群分区也与参与集群的节点一样多。（您应该知道，LDM 线程的数量会随着此参数的值而增加，但不是以严格的线性方式增加，并且设置它有额外的限制；有关更多信息，请参阅 [`MaxNoOfExecutionThreads`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbd-definition.html#ndbparam-ndbmtd-maxnoofexecutionthreads)的描述。）

### NDB 和用户定义的分区

NDB 集群通常会对 NDB 的表自动进行分区，但是，用户也可以对 NDB 表进行自定义分区，这将受到以下限制：

1. 对于 NDB 表，在生产环境中支持 key 分区和 linear key 分区；

2. 对于任何 NDB 表，可以明确定义的最大分区数为 8 * [number of LDM threads] * [number of node groups]

有关更多信息，请参阅[第 23.5.3 节，“ndbmtd - NDB Cluster 数据节点守护程序（多线程）”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbmtd.html "23.5.3 ndbmtd — NDB 集群数据节点守护进程（多线程）")。

### 片段副本

这是集群分区的副本。节点组中的每个节点都存储一个片段副本，分片副本数等于每个节点组的节点数。

一个分片副本完全属于单个节点，一个节点可以存储多个片段副本。

下图是一个运行了 4 个 ndbd 节点的 NDB 集群，分成了 2 个节点组，node 1 和 node 2 属于节点组 0，node 3 和 node 4 属于节点组 1

```textile
笔记
    这里省略了 ndbd_mgmd 节点和 SQL 节点
```

![](/Users/bichengfei/opt/own/project/MySQL/MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/02.png)

集群存储的数据分成了 4 个分区，编号为0、1、2、3.每个分区以两个副本的形式存储在同一个节点组上。分区交叉存储在节点组上，如下所示：

+ 分区 0 存储在节点组 0 上，主分片副本存储在节点 1，备份分片副本存储在节点 2 上

+ 分区 1 存储在节点组 1 上，主分片副本存储在节点 3，备份分片副本存储在节点 4 上

+ 分区 2 存储在节点组 0 上，但是其两个分片副本的位置与分区 0 相反。主分片副本存储在节点 2，备份分片副本存储在节点 1

+ 分区 3 存储在节点组 1 上，但是其两个分片副本的位置与分区 1 相反

对于持续运行的 NDB 集群来说，只要每个节点组中至少有一个节点在运行，集群就拥有全部数据并保持可用。如下图所示：

![](/Users/bichengfei/opt/own/project/MySQL/MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/03.png)

在此示例中，集群由两个节点组组成，每个节点组由两个数据节点组成。每个数据节点都运行一个[**ndbd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html "23.5.1 ndbd — NDB 集群数据节点守护进程")实例。节点组 0 中的至少一个节点和节点组 1 中的至少一个节点的任意组合足以使集群保持“活动”。但是，如果单个节点组中的两个节点都发生故障，则由另一个节点组中剩余的两个节点组成的组合是不够的。在这种情况下，集群丢失了整个分区，因此无法再提供对所有 NDB Cluster 数据的完整集的访问。

单个 NDB Cluster 实例支持的最大节点组数为 48。
