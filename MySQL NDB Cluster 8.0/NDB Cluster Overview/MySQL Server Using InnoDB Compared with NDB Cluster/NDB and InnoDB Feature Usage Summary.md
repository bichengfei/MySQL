# 23.2.6.3 NDB 和 InnoDB 功能使用总结

在将应用程序功能要求与 InnoDB 与 NDB 的功能进行比较时，有些情况下，一种有时候比另一种更兼容。

下面根据每个功能通常更适合的存储引擎列出了支持的应用程序功能。

## 首选 InnoDB

1. 外键（*NDB 8.0 集群支持外键*）

2. 全表扫描

3. 非常大的数据库、行或事务

4. Read_commited 以外的事务隔离级别

## 首选 NDB

1. 写入缩放

2. 99.999% 的正常运行时间（高可用）

3. 在线添加节点和在线模式操作（DDL）

4. 多个 SQL 和 NoSQL API（请参阅 [NDB Cluster API：概述和概念](https://dev.mysql.com/doc/ndbapi/en/mysql-cluster-api-overview.html)）

5. 实时性能

6. 限制使用 BLOB  列

7. 支持外键，尽管它们的使用可能会影响高吞吐量时的性能
