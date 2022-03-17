# 23.2.6 MySQ 服务器使用 InnoDB 与 NDB 集群比较

MySQL Server 在存储引擎中提供了多种选择。由于 NDB 和 InnoDB 都可以用作事务存储引擎，因此 MySQL 的用户有时会对 NDB 集群感兴趣。他们把 NDB 当作一种可能的选择，或者升级 MySQL 8.0 中默认的 InnoDB。NDB 和 InnoDB 有共同的特定，但在架构和实现上存在差异，因此一些现有的 MySQL 程序和使用场景可能适合 NDB，但不是全部。

在本节，对于 NDB 8.0 中的 NDB 和 MySQL 8.0 中的 InnoDB ，我们将讨论比较他们的一些特性。在许多情况下，必须根据具体情况决定何时何地使用 NDB Cluster，并考虑所有因素。虽然我们不可能为每一种可能的使用场景提供细节额外你到哪个，但我们还是尝试就某些常见类型，对 NDB 和 InnoDB 对相对使用性提供一些指导。

NDB 8.0 集群使用基于 MySQL 8.0 的 mysqld，包括了对 InnoDB 1.1 的支持。虽然剋在 NDB 集群中使用 InnoDB 表，但此类表不是集群的。NDB Cluster 8.0发行版中的程序或库也不能与MySQL Server 8.0一起使用，反之亦然。

虽然某些类型的通用业务程序可以在 NDB 集群和 MySQL (最有可能使用 InnoDB 引擎) 上运行，但是在体系结构和实现方面还是存在一些重要差异。[第 23.2.6.1 节，“NDB 和 InnoDB 存储引擎之间的差异”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-engines.html "23.2.6.1 NDB 和 InnoDB 存储引擎的区别")，提供了这些差异的摘要。由于差异，一些使用场景显然更适合一种引擎或另一种；请参阅 [第 23.2.6.2 节，“NDB 和 InnoDB 工作负载”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-workloads.html "23.2.6.2 NDB 和 InnoDB 工作负载")，这反过来又会影响到程序更适合使用 NDB 或 InnoDB；请参阅 [第 23.2.6.3 节，“NDB 和 InnoDB 功能使用摘要”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndb-innodb-usage.html "23.2.6.3 NDB 和 InnoDB 功能使用总结")，用于比较每种在常见类型的数据库应用程序中的相对适用性。

有关 存储引擎[`NDB`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html "第23章 MySQL NDB Cluster 8.0")和 [`MEMORY`](https://dev.mysql.com/doc/refman/8.0/en/memory-storage-engine.html "16.3 MEMORY存储引擎")存储引擎的相关特征的信息，请参阅 [何时使用 MEMORY 或 NDB Cluster](https://dev.mysql.com/doc/refman/8.0/en/memory-storage-engine.html#memory-storage-engine-compared-cluster "何时使用 MEMORY 或 NDB Cluster")。

有关 MySQL 存储引擎的更多信息， 请参阅[第 16 章，*替代存储引擎。*](https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html "第 16 章 替代存储引擎")
