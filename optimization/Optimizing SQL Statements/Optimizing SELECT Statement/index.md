# 8.2.1 优化 SELECT 语句

+ where 子句优化

+ 范围优化

+ 索引合并优化

+ 引擎条件下推优化

+ 索引条件下推优化

+ 嵌套循环连接算法

+ 嵌套连接优化

+ 外连接优化

+ 外连接简化

+ 多范围读取优化

+ 阻止嵌套循环和批量密钥访问连接

+ 条件过滤

+ is null 优化

+ order by 优化

+ group by 优化

+ distinct 优化

+ limit 查询优化

+ 函数调用优化

+ 行构造函数表达式优化

+ 避免全表扫描

查询，以 select 语句的形式，执行着数据库中的所有查找操作。无论是实现动态网页的亚秒级响应时间，还是缩短生成大量隔夜报告的时间，调整这些语句是重中之重。

除了 select 语句之外，查询技术也适用于设计语句，类似 `create table ... as select`、`insert into ... select`和带有 where 子句的 delete。因为他们结合了写操作和面向读读查询操作，所有这些语句有额外的性能考虑。

NDB Cluster 支持连接下推优化，其中合格的连接被完整地发送到 NDB Cluster 数据节点，在那里它可以分布在它们之间并并行执行。有关此优化的更多信息，请参阅 [NDB 下推连接的条件](https://dev.mysql.com/doc/refman/5.7/en/mysql-cluster-options-variables.html#ndb_join_pushdown-conditions "NDB 下推连接的条件")。

优化查询的主要考虑因素是：

1. 要让慢 select ... where 查询更快，首先要检查是否可以添加索引。在 where 语句使用的列上设置索引，以加快评估、过滤和最终检索结果的速度。为避免浪费磁盘空间，请请构建一小组索引来加速应用程序中使用的许多相关查询。
   
   索引对于使用连接和外键等功能的多表查询尤为重要。你可以用 explain 语句来确定哪些索引用于 select。具体请参考`使用 explain 优化查询`

2. 隔离和调整查询的任何部分，例如函数调用，这会花费过多时间。根据查询的结构，可能为结果集中的每一行调用一次函数，甚至为表中的每一行调用一次函数，这极大的放大了任何低效率。

3. 尽量减少查询中的全表扫描次数，尤其是对于大表。

4. 通过定期使用`analyze table`语句保持最新的表统计信息，因此优化器拥有所需要的信息用以构建更有效的查询计划。

5. 了解每个表特定的存储引擎的调优技术、索引技术和配置参数。InnoDB 和 MyISAM 都有一套指导方针，用于启用和维持高性能查询。有关详细信息，请参阅[第 8.5.6 节，“优化 InnoDB 查询”](https://dev.mysql.com/doc/refman/5.7/en/optimizing-innodb-queries.html "8.5.6 优化 InnoDB 查询")和 [第 8.6.1 节，“优化 MyISAM 查询”](https://dev.mysql.com/doc/refman/5.7/en/optimizing-queries-myisam.html "8.6.1 优化 MyISAM 查询")。

6. 你可以通过[优化 InnoDB 只读事务](https://dev.mysql.com/doc/refman/5.7/en/innodb-performance-ro-txn.html)来优化表的单查询事务。

7. 避免把查询写的难以理解，尤其是优化器会自动做相同效果的转换时。

8. 如果性能问题无法通过基本准则之一轻松解决，请通过阅读 explain 计划并调整索引、where 子句、连接子句等来调查特定查询的内部细节。

9. 调整 MySQL 用于缓存的内存区域的大小和属性。通过有效使用 InnoDB 缓冲池、MyISAM 键缓存和 MySQL 查询缓存，使重复查询更快，因为第二次和后续的查询都是从内存中检索结果。

10. 即使对于使用缓存内存区域快速运行的查询，你仍然可以进一步优化，以便他们需要更少的缓存内存，从而使你的应用程序更具可扩展性。可扩展性意味着你的应用程序可以处理更多的并发用户、更大的请求等，而不会出现性能大幅下降。

11. 处理锁定问题，你的查询速度可能会收到同时访问表的其他会话的影响。