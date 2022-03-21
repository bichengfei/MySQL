# 23.3.1 在 Linux 上安装 NDB 集群

本节介绍 Linux 和其他类 Unix 操作系统上 NDB Cluster 的安装方法。虽然接下来的几节提到了 Linux 操作系统，但那里给出的说明和过程应该很容易适应其他受支持的类 Unix 平台。有关特定于 Windows 系统的手动安装和设置说明，请参阅 [第 23.3.2 节，“在 Windows 上安装 NDB Cluster”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows.html "23.3.2 在 Windows 上安装 NDB Cluster")。

每个 NDB Cluster 主机必须安装正确的可执行程序。运行 SQL 节点的主机必须在其上安装 MySQL 服务器二进制文件 ( [**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html "4.3.1 mysqld——MySQL 服务器") )。管理节点需要管理服务器守护进程（[**ndb_mgmd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html "23.5.4 ndb_mgmd — NDB 集群管理服务器守护进程")）；数据节点需要数据节点守护进程（[**ndbd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html "23.5.1 ndbd — NDB 集群数据节点守护进程")或[**ndbmtd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbmtd.html "23.5.3 ndbmtd — NDB 集群数据节点守护进程（多线程）")）。无需在管理节点主机和数据节点主机上安装 MySQL Server 二进制文件。建议您还在管理服务器主机上 安装管理客户端 ( [**ndb_mgm )。**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html "23.5.5 ndb_mgm — NDB 集群管理客户端")

在 Linux 上安装 NDB Cluster 可以使用 Oracle 的预编译二进制文件（下载为 .tar.gz 存档）、RPM 包（也可从 Oracle 获得）或源代码。所有这三种安装方法都将在下面的部分中进行描述。

无论使用哪种方法，在安装 NDB Cluster 二进制文件后，仍然需要为所有集群节点创建配置文件，然后才能启动集群。请参阅 [第 23.3.3 节，“NDB 集群的初始配置”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-configuration.html "23.3.3 NDB Cluster 的初始配置")。
