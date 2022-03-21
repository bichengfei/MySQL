# 23.2.1 NDB 集群核心概念

### 概念

NDBCLUSTER(也称为 NDB) 是一种内存存储引擎，提供高可用性和数据持久化功能。

NDB 存储引擎提供了一系列故障转移和负载均衡配置选项，但从集群级别但存储引擎开始是最简单的。NDB Cluster 的 NDB 存储引擎包含一整套数据，仅依赖于集群本身内的其它数据。

NDB Cluster 的 "Cluster" 部分独立于 MySQL 服务端进行配置。在 NDB Cluster 中，集群的每个部分都被是为一个节点（node）.

```textile
笔 记
    在许多情况下，术语“节点”用于表示计算机，但在讨论 NDB Cluster 时，它表示进程。
    可以在一台计算机上运行多个节点。
    对于运行一个或多个几集群节点的计算机，我们称之为集群主机。
```

集群节点有三种类型，在 NDB 集群的最小化配置中，至少有三个节点，每种节点对应一种类型：

+ 管理节点 (Management node)：此类节点的作用是管理 NDB Cluster 中的其它节点。执行诸如提供配置数据、启动和停止节点、执行备份等功能。因为这种节点类型管理其它节点的配置，所以应该首先启动这种类型的节点，然后再启动任何其它节点。使用`ndb_mgmd`启动管理节点。

+ 数据节点 (Data node)：这种类型的节点存储集群数据。数据节点的数量 = 分区数量 * 副本数量（请参阅[第 23.2.2 节，“NDB 集群节点、节点组、片段副本和分区”](./NDB%20Cluster%20Nodes,Node%20Groups,Fragment%20Replicas,and%20Partitions.md)。例如，对于有两个分区，两个副本的表，你需要四个数据节点。一个副本足以存储数据，但不提供冗余。因此，建议具有两个或更多的副本以提供冗余，从而提高可用性。使用命令 ndbd （请参阅[第 23.5.1 节，“ndbd - NDB Cluster 数据节点守护程序”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html "23.5.1 ndbd — NDB 集群数据节点守护进程")）或 ndbmtd （请参阅 [第 23.5.3 节，“ndbmtd - NDB Cluster 数据节点守护程序（多线程）”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbmtd.html "23.5.3 ndbmtd — NDB 集群数据节点守护进程（多线程）")）启动数据节点。
  
  NDB Cluster 表通常存储在内存中而不是磁盘上（这就是我们将 NDB Cluster 称为内存数据库的原因）。但是，一些 NDB Cluster 数据可以存储在磁盘上；有关更多信息，请参阅 [第 23.6.10 节，“NDB Cluster 磁盘数据表”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-disk-data.html "23.6.10 NDB Cluster 磁盘数据表")。

+ SQL 节点 (SQL node)：这是一个访问集群数据的节点。对于 NDB Cluster，SQL 节点是使用 NDB 存储引擎的 MySQL 服务端。SQL 节点是一个使用 `--ndbcluster`和`--ndb-connectstrin`选项启动的 mysqld 经常，本章其它地方将对此进行解释，可能还带有其它的 MySQL 服务端选项。
  
  SQL 节点实际上只是一种特殊类型的 API 节点，API 节点指定访问 NDB Cluster 数据的任何请求。API 节点的另一个应用实例是 nbb_restore 应用程序，用来恢复集群备份。可以用 NDB API 来编写此类请求。有关 NDB API 的基本信息，请参阅 [NDB API 入门](https://dev.mysql.com/doc/ndbapi/en/ndb-getting-started.html)

```textile
重要的
    期望在生产环境中采用三节点设置是不现实的，这种配置不提供冗余。
    要使用 NDB Cluster 的高可用功能，必须使用多个数据节点和 SQL 节点，还强烈建议使用多个管理节点。
```

集群的配置涉及配置集群中的每个节点，并在节点之间建立单独的通信链路。当前 NDB  Cluster 的设计目的是数据节点在处理器能力、内存空间和带宽方面是同质的。此外，为了提供单点配置，整个集群的所有配置数据都位于一个配置文件中。

管理服务器管理着集群的配置信息和集群日志。集群中的每个节点都从管理服务器获取配置信息，因此需要一种方法来确定管理服务器所在的位置。当数据节点中发生有趣当事件时，即诶单会将这些事件当相关信息传输到管理服务器，然后管理服务器将这些信息写入集群日志。

### 客户端

可以有任意数量到集群客户端进程或应用程序。其中包括标准 MySQL 客户端、特定的NDB API 程序、管理客户端。

#### 标准 MySQL 客户端

可以用 PHP、Perl、C、C++、Java、Python等编写 NDB Cluster 的客户端。此类客户端请求向 NDB 集群的 SQL 节点的 MySQL 服务器发送 SQL 语句并从其接收响应，其方式与它们和独立的 MySQL 服务器的交互非常相似。

使用 NDB 集群作为数据源的 MySQL 客户端，可以修改成连接多个 MySQL 服务器，以利用 NDB 集群实现负载均衡和高可用。例如，使用 Connector/J 5.0.6 以及更高版本的 Java 客户端可以使用`jdbc:mysql:loadbalance://URL`（在 Connector/J 5.1.7 中进行了改进）透明地实现负载均衡；有关将 Connector/J 与 NDB 集群一起使用的更多信息，请参阅 [将 Connector/J 与 NDB Cluster 一起使用](https://dev.mysql.com/doc/ndbapi/en/mccj-using-connectorj.html)。

#### NDB API 客户端程序

使用 NDB API，一个高级 C++ API，可以编写通过 NDB 存储引擎直接访问 NDB 集群数据的应用程序，不通过任何可以连接到集群的 MySQL 服务器。对于不需要数据的 SQL 接口的特殊场景，此类程序可能很有用。有关更多信息，请参阅 [NDB API](https://dev.mysql.com/doc/ndbapi/en/ndbapi.html)。

可以使用 NDB Cluster Connector for Java 为 NDB Cluster 编写特定的 Java 程序。此 NDB Cluster Connector 包括ClusterJ，这是一个高级数据库 API，类似于直接连接到的对象关系映射持久性框架（如 Hibernate 和 JPA `NDBCLUSTER`），因此不需要访问 MySQL 服务器。有关更多信息，请参阅 [Java 和 NDB Cluster](https://dev.mysql.com/doc/ndbapi/en/mccj-overview-java.html)以及 [ClusterJ API 和数据对象模型](https://dev.mysql.com/doc/ndbapi/en/mccj-overview-clusterj-object-models.html)。

NDB Cluster 还支持使用 Node.js 以 JavaScript 编写的应用程序。适用于 JavaScript 的 MySQL 连接器包括用于直接访问`NDB`存储引擎和 MySQL 服务器的适配器。使用此连接器的应用程序通常是事件驱动的，并使用在许多方面类似于 ClusterJ 所采用的领域对象模型。有关更多信息，请参阅[用于 JavaScript 的 MySQL NoSQL 连接器](https://dev.mysql.com/doc/ndbapi/en/ndb-nodejs.html)。  

#### 管理客户端

这些客户端连接到管理服务器，并提供用于正常启动、停止节点、启动和停止消息跟踪（仅限调试版本）、显示节点版本和状态、启动和停止备份等命令。此类程序等一个示例是 NDB 集群提供的 ndb_mgm 管理客户端 （请参阅[第 23.5.5 节，“ndb_mgm - NDB Cluster 管理客户端”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html "23.5.5 ndb_mgm — NDB 集群管理客户端")）。可以使用 MGM API编写此类应用程序，这是一种直接与一个或多个 NDB Cluster 管理服务器通信的 C 语言 API。有关更多信息，请参阅 [MGM API](https://dev.mysql.com/doc/ndbapi/en/mgm-api.html)。

Oracle 还提供 MySQL Cluster Manager，它提供了一个高级命令行界面，简化了许多复杂的 NDB Cluster 管理任务，例如重新启动具有大量节点的 NDB Cluster。MySQL Cluster Manager 客户端还支持获取和设置大多数节点配置参数的值以及与 NDB Cluster 相关的[**mysqld服务器选项和变量的命令。**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html "4.3.1 mysqld——MySQL 服务器")MySQL Cluster Manager 1.4.8 为 NDB 8.0 提供实验性支持。有关详细信息，请参阅[MySQL Cluster Manager 1.4.8 用户手册](https://dev.mysql.com/doc/mysql-cluster-manager/1.4/en/)。

### 事件日志（Event logs）

NDB 集群按类别（启动、关闭、错误、检查点等）、优先级、严重性记录事件。所有可报告事件的完整列表可以在 [第 23.6.3 节，“在 NDB Cluster 中生成的事件报告”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-event-reports.html "23.6.3 NDB Cluster 中生成的事件报告")中找到。事件日志是此处列出的两种类型：

+ 集群日志：记录整个集群的所有需要报告的事件

+ 节点日志：一个单独的日志，节点级别

```textile
笔记
    通常情况下，保留充足的集群日志并进行检查是必要的。节点日志，仅在程序开发和调试时才需要用到
```

### 检查点（Checkpoint）

一般来说，等数据被保存到磁盘时，就说已经到达了一个检查点。对于 NDB 集群来说，检查点是所有提交的事务都存储到磁盘上到时间点。对于 NDB 存储引擎，有两种类型的检查点协同工作，以确保集群数据视图的一致性。

+ 本地检查点（LCP）：Local Checkpoint，这是一个特定于单个节点的检查点，但是集群中的所有节点或多或少同时发生 LCP。LCP 通常每间隔几分钟发生一次，精确的事件间隔会有所不同，并且还取决于节点存储的数据量、集群活动的级别和其它因素。
  
  NDB 8.0 支持部分 LCP，在某些情况下可以显著提高性能。请参阅启用部分 LCP 并控制它们使用的存储量的参数`EnablePartialLcp`和`RecoverWork`的描述。

+ 全局检查点（GCP）：Global CheckPoint，当所有节点的事务同步并且重做日志刷新到磁盘时，每隔几秒就会发生一次 GCP。

有关本地检查点和全局检查点创建的文件和目录的更多信息，请参阅 [NDB Cluster 数据节点文件系统目录](https://dev.mysql.com/doc/ndb-internals/en/ndb-internals-ndbd-filesystemdir-files.html)。
