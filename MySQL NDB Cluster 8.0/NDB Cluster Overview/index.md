# 23.2 NDB 集群概述

NDB Cluster 是一种在无共享系统中启用内存数据库集群的技术。无共享架构使系统能够使用非常便宜的硬件，并且对硬件或软件对特定要求最低。

NDB Cluster 旨在没有任何单点故障。在无共享系统中，每个组件都应该有自己的内存和磁盘，不推荐也不支持使用网络共享、网络文件系统和 SAN 等共享存储机制。

NDB Cluster 将标准的 MySQL 服务端和称为 NDB（Network DataBase）的内存集群存储引擎集成在一起。在此文档中，术语 NDB 是指特定于存储引擎的设置部分，而 MySQL NDB  Cluster 是指一个或多个 MySQL 服务端与 NDB 存储引擎的组合。

NDB Cluster 由一组称为 hosts 的计算机组成，每台计算机都运行一个或多个进程。这些进程称之为节点，可能包括 MySQL 服务端（用于访问 NDB 数据）、数据节点（用于存储数据）、一个或多个管理服务端，以及可能的其他专用数据访问程序。NDB Cluster 中这些组件的关系如下所示：

![](./01.png)

所有这些程序一起工作形成 NDB Cluster（[第 23.5 节，“NDB Cluster 程序”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs.html "23.5 NDB 集群程序")）。

+ ndbd：数据节点，Data Nodes，当 NDB 存储引擎存储数据时，表和表数据存储在这里

+ mysqld：SQL 节点，SQL Nodes，SQL 服务端，数据节点中的任何表都可以直接通过 SQL 节点访问

因此，对于在 NDB Cluster 中存储工资单的应用程序，如果一个请求更新了员工的工资，那么所有对于该数据的其他查询都可以立即看到此更改。

尽管 NDB Cluster SQL 节点使用 mysqld 服务端守护进程，但它在许多关键部分与 MySQL 8.0 发行版提供的 mysqld 二进制文件不同，并且这两个版本的 mysqld 不可互换。

此外，未连接到 NDB Cluster 的 MySQL 服务端无法使用 NDB 存储引擎，也无法访问任何 NDB Cluster 数据。

存储在 NDB Cluster 的数据节点中的数据可以被镜像；集群可以处理单个数据节点的故障，除了少量事务由于丢失事务状态而中止之外没有其他影响。因为事务应用程序预计将会处理事务失败，所以这不应该是问题的根源。

单个节点可以停止和重新启动，然后可以重新加入集群。滚动重启（所有节点依次重启）可用于配置更改和软件升级（请参阅 [第 23.6.5 节，“执行 NDB 集群的滚动重启”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-rolling-restart.html "23.6.5 执行 NDB Cluster 的滚动重启")）。滚动重启也用作在线添加新数据节点的过程的一部分（请参阅[第 23.6.7 节，“在线添加 NDB Cluster 数据节点”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-online-add-node.html "23.6.7 在线添加 NDB Cluster 数据节点")）。有关数据节点，它们在 NDB Cluster 中的组织方式以及它们如何处理和存储 NDB Cluster 数据的更多信息，请参阅 [第 23.2.2 节，“NDB Cluster 节点、节点组、片段副本和分区”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-nodes-groups.html "23.2.2 NDB Cluster 节点、节点组、片段副本和分区")。

可以使用 NDB Cluster 管理客户端中的 -native 功能和 NDB Cluster 发行版中包含的 ndb_restore 程序来备份和恢复 NDB Cluster 数据。有关更多信息，请参阅 [第 23.6.8 节，“NDB 集群的在线备份”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-backup.html "23.6.8 NDB Cluster 在线备份")和 [第 23.5.23 节，“ndb_restore - 恢复 NDB 集群备份”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-restore.html "23.5.23 ndb_restore - 恢复 NDB Cluster 备份")。你还可以使用在 mysqldum 和 MySQL 服务端中为此目的提供的标准功能。有关详细信息，请参阅 [第 4.5.4 节，“mysqldump - 数据库备份程序”](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html "4.5.4 mysqldump——一个数据库备份程序")。

NDB Cluster 节点可以使用不同的传输机制进行节点间通信；在大多数实际部署中使用标准的 100 Mbps 或更快的以太网硬件上的 TCP/IP。
