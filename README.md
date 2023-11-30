# MySQL

官网地址：https://dev.mysql.com/doc/refman/8.0/en/preface.html

## 8  优化

此篇章基于 MySQL 8.0 版本

### 8.1 [优化概述](./optimization/index.md)

### 8.2  [优化 SQL 语句](./optimization/Optimizing%20SQL%20Statements/index.md)

#### 8.2.1 [优化 select 语句](./optimization/Optimizing%20SQL%20Statements/Optimizing%20SELECT%20Statement/index.md)

+ 8.2.1.1 [where 子句优化](./optimization/Optimizing%20SQL%20Statements/Optimizing%20SELECT%20Statement/where%20clause%20Optimization.md)

+ 8.2.1.2 [范围优化](./optimization/Optimizing%20SQL%20Statements/Optimizing%20SELECT%20Statement/Range%20Optimization.md)

### 8.8 了解查询执行计划

#### 8.8.1 使用  explain 优化查询

#### 8.8.2 [explain 输出格式](./optimization/Understanding%20the%20Query%20Execution%20Plan/EXPLAIN%20Output%20Format.md)

## 12 函数和运算符

### 12.10 [全文检索功能](./函数和运算符/12.10/index.md)

#### 12.10.4 [全文分词](./函数和运算符/12.10/12.10.4.全文分词.md)

## 15 InnoDB 存储引擎

基于 MySQL 5.7 的官方文档的 InnoDB 部分翻译，部分篇幅会有个人见解

### 15.1 InnoDB 简介

- 1.1 使用 InnoDB  表的好处
- 1.2 InnoDB 表的最佳实践
- 1.3 验证 InnoDB 是默认存储引擎
- 1.4 使用 InnoDB 进行测试和基准测试
- 1.5 关闭 InnoDB

### 15.2 InnoDB 和 ACID 模型

### 15.3. InnoDB 多版本控制 -- @[amlgithub](https://github.com/amlgithub)

### 15.4. InnoDB 架构

### 15.5. InnoDB 内存结构

- 5.1 缓冲池
- 5.2 更改缓冲区
- 5.3 自适应哈希索引
- 5.4 日志缓冲区

### 15.6. InnoDB 磁盘结构

#### 15.6.1 表

+ 创建 InnoDB 表  

+ 在外部创建表  

+ 导入 InnoDB 表  

+ 移动或复制 InnoDB 表  

+  [InnoDB 中的 auto_increment 处理](./InnoDB/6/6.1/6.1.InnoDB%E4%B8%ADauto_increment%E5%A4%84%E7%90%86%EF%BC%88%E5%AE%98%E7%BD%91%E7%89%88%EF%BC%89.md)

#### 15.6.2 索引

+ [聚集索引和二级索引](./InnoDB/6/6.2/6.2.聚集索引和二级索引.md)  

+ [InnoDB 索引的物理结构](./InnoDB/6/6.2/6.2.InnoDB索引的物理结构.md)  

+ 排序索引构建  

+ InnoDB 全文索引  

#### 15.6.3 [表空间](./InnoDB/6/6.3/6.3.表空间.md)

+ [系统表空间](./InnoDB/6/6.3/6.3.表空间.md)  

+ [每个表的文件表空间](./InnoDB/6/6.3/6.3.表空间.md)  

+ [通用表空间](./InnoDB/6/6.3/6.3.表空间.md)  

+ [撤销表空间](./InnoDB/6/6.3/6.3.表空间.md) 

+ [临时表空间](./InnoDB/6/6.3/6.3.表空间.md)  

#### 15.6.4 InnoDB 数据字典

#### 15.6.5 双写缓冲区

#### 15.6.6 [重做（redo）日志](./InnoDB/6/6.6/6.6.redo日志.md)

#### 15.6.7 [撤销（undo）日志](./InnoDB/6/6.7/6.7.undo日志.md)

### 15.7. [InnoDB 锁定和事务模型](./InnoDB/15.7/index.md)

#### 15.7.1 [InnoDB 锁定](./InnoDB/15.7/15.7.1.InnoDB锁定.md)

#### 15.7.2 [InnoDB 事务模型](./InnoDB/15.7/15.7.2.InnoDB事务模型/index.md)

+ 15.7.2.1 [事务隔离级别](./InnoDB/15.7/15.7.2.InnoDB事务模型/15.7.2.1.事务隔离级别.md)

+ 15.7.2.2 [自动提交、提交和回滚](./InnoDB/15.7/15.7.2.InnoDB事务模型/15.7.2.2.自动提交、提交和回滚.md)

+ 15.7.2.3 [一致的非锁定读取](./InnoDB/15.7/15.7.2.InnoDB事务模型/15.7.2.3.一致的非锁定读取.md)

+ 15.7.2.4 [锁定读取](./InnoDB/15.7/15.7.2.InnoDB事务模型/15.7.2.4.锁定读取.md)

#### 15.7.3 [InnoDB 中不同 SQL 语句设置的锁](./InnoDB/15.7/15.7.3.InnoDB中不同SQL语句设置的锁.md)

#### 15.7.4 [幻影行](./InnoDB/15.7/15.7.4.幻行.md)

#### 15.7.5 [InnoDB 中的死锁](./InnoDB/15.7/15.7.5.InnoDB中的死锁/index.md)

+ 15.7.5.1 [InnoDB 死锁示例](./InnoDB/15.7/15.7.5.InnoDB中的死锁/15.7.5.1.InnoDB死锁示例.md)

+ 15.7.5.2 [死锁检测](./InnoDB/15.7/15.7.5.InnoDB中的死锁/15.7.5.2.死锁检测.md)

+ 15.7.5.3 [如何最小化和处理死锁](./InnoDB/15.7/15.7.5.InnoDB中的死锁/15.7.5.3.如何最小化和处理死锁.md)

#### 15.7.6 [事务调度](./InnoDB/15.7/15.7.6.事务调度.md)

### 15.8. InnoDB 配置

- 8.1 InnoDB 启动配置
- 8.2 为只读操作配置 InnoDB
- 8.3 InnoDB 缓冲池配置
- 8.4 为 InnoDB 配置内存分配器
- 8.5 为 InnoDB 配置线程并发
- 8.6 配置后台 InnoDB 的 I/O 线程数
- 8.7 在 Linux 上使用异步 I/O
- 8.8 配置 InnoDB 的 I/O 容量
- 8.9 配置自旋锁轮询
- 8.10 清除配置
- 8.11 为 InnoDB 配置优化器统计信息  
    &nbsp;&nbsp;&nbsp;&nbsp;配置持久优化器统计参数  
    &nbsp;&nbsp;&nbsp;&nbsp;配置非持久优化器统计参数  
    &nbsp;&nbsp;&nbsp;&nbsp;估计 InnoDB 表的 analyze table 复杂性  
- 8.12 配置索引页的合并阀值

### 15.9. InnoDB 表和页面压缩

- 9.1 InnoDB 表压缩  
    &nbsp;&nbsp;&nbsp;&nbsp;表压缩概述  
    &nbsp;&nbsp;&nbsp;&nbsp;创建压缩表  
    &nbsp;&nbsp;&nbsp;&nbsp;调整 InnoDB 表的压缩  
    &nbsp;&nbsp;&nbsp;&nbsp;在运行时监控 InnoDB 表的压缩  
    &nbsp;&nbsp;&nbsp;&nbsp;InnoDB 表的压缩如何工作  
    &nbsp;&nbsp;&nbsp;&nbsp;OLTP 工作负载的压缩  
    &nbsp;&nbsp;&nbsp;&nbsp;SQL 压缩语法警告与错误  
- 9.2 InnoDB 页面压缩

### 15.10. InnoDB 行格式

### 15.11. InnoDB 磁盘 I/O 和文件空间管理

- 12.1 InnoDB 磁盘 I/O  
- 12.2 文件空间管理  
- 12.3 InnoDB 检查点
- 12.4 对表进行碎片整理
- 12.5 使用 truncate table 回收磁盘空间

### 15.12. InnoDB 和在线 DDL

- 13.1 在线 DDL 操作
- 13.2 在线 DDL 性能和并发
- 13.3 在线 DDL 空间要求
- 13.4 使用在线 DDL 简化 DDL 语句
- 13.5 在线 DDL 失败条件
- 13.6 在线 DDL 限制

### 15.13. InnoDB 静态数据加密

### 15.14. InnoDB 启动选项和系统变量

### 15.15. [InnoDB 的 information_schema 表](./InnoDB/15.15/index.md)

#### 15.15.1 [InnoDB 的 information_schema 压缩表](./InnoDB/15.15/15.15.1/index.md)

+  [InnoDB_cmp 和 InnoDB_cmp_reset](./InnoDB/15.15/15.15.1/15.15.1.1.INNODB_CMP和INNODB_CMP_RESET.md)

+ [InnoDB_cmpmem 和 InnoDB_cmp_cmpmem_reset](./InnoDB/15.15/15.15.1/15.15.1.2.INNODB_CMPMEM和INNODB_CMPMEM_RESET.md)

+ [使用压缩信息模式表](./InnoDB/15.15/15.15.1/15.15.1.3.使用压缩信息模式表.md)

#### 15.15.2 [InnoDB information_schema 事务和锁定信息](./InnoDB/15.15/15.15.2/index.md)

+ [使用 InnoDB 事务和锁定信息](./InnoDB/15.15/15.15.2/15.15.2.1.使用InnoDB事务和锁定信息.md)

+ [InnoDB 锁定和锁定等待信息](./InnoDB/15.15/15.15.2/15.15.2.2.InnoDB锁定和锁定等待信息.md)

+ [InnoDB 事务和锁定信息的持久性和一致性](./InnoDB/15.15/15.15.2/15.15.2.3.InnoDB事务和锁定信息的持久性和一致性.md)

#### 15.15.3 InnoDB information_schema 系统表

#### 15.15.4 [InnoDB information_schema FullText 索引表](./InnoDB/15.15/15.15.4.InnoDB全文索引表.md)

#### 15.15.5 InnoDB information_schema 缓冲池表

#### 15.15.6 InnoDB information_schema 指标表

#### 15.15.7 InnoDB information_schema 临时表信息表

#### 15.15.8 从 information_schema.files 检索 InnoDB 表空间元数据

### 15.16. InnoDB 与 MySQL_Performance Schema 的集成

- 17.1 使用性能模式监控 InnoDB 表的 alter table 进度
- 17.2 使用性能监控模式监控 InnoDB Mutex 等待

### 15.17. InnoDB 监视器

- 18.1 InnoDB 监视器类型
- 18.2 启用 InnoDB 监视器
- 18.3 InnoDB 标准监视器和锁定监视器输出

### 15.18. InnoDB 备份和恢复

- 19.1 InnoDB 备份
- 19.2 InnoDB 恢复

### 15.19. InnoDB 和 MySQL 复制

### 15.20. InnoDB 内存缓存插件

- 21.1 InnoDB memcached 插件的好处
- 21.2 InnoDB 内存缓存架构
- 21.3 设置 InnoDB memcached 插件
- 21.4 InnoDB memcached 插件的安全注意事项
- 21.5 为 InnoDB memcached 插件编写应用程序   
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为 InnoDB memcached 插件调整现有的 MySQL 模式  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为 InnoDB 内存缓存插件调整内存缓存应用程序  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;调整 InnoDB memcached 插件性能  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;控制 InnoDB memcached 插件的事务行为  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使 DML 语句适应 memcached 操作  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在底层 InnoDB 表上执行 DML 和 DDL 语句  
- 21.6 InnoDB memcached 插件和复制
- 21.7 InnoDB memcached 插件内部
- 21.8 InnoDB memcached 插件故障排查

### 15.21. InnoDB 故障排查

- 22.1 对 InnoDB I/O 问题进行故障排除

- 22.2 强制 InnoDB 恢复

- 22.3 InnoDB 数据字典操作故障排除

- 22.4 InnoDB 错误处理
  
  ### 15.22. InnoDB Limits

### 15.24 InnoDB Restrictions and Limitations

### 23 [MySQL NDB Cluster 8.0](./MySQL%20NDB%20Cluster%208.0/index.md)

#### 23.1 [一般信息](./MySQL%20NDB%20Cluster%208.0/General%20Information.md)

#### 23.2 [NDB 集群概述](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/index.md)

##### 23.2.1 [NDB 集群核心概念](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/NDB%20Cluster%20Core%20Concepts.md)

##### 23.2.2 [NDB 集群节点、节点组、片段副本和分区](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/NDB%20Cluster%20Nodes,Node%20Groups,Fragment%20Replicas,and%20Partitions.md)

##### 23.2.3 NDB 集群硬件、软件和网络要求

##### 23.2.4 NDB 集群中的新功能

##### 23.2.5 NDB 8.0 中添加、弃用或删除的选项、变量和参数

##### 23.2.6 [MySQL 服务器使用 InnoDB 与 NDB 集群的比较](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/MySQL%20Server%20Using%20InnoDB%20Compared%20with%20NDB%20Cluster/index.md)

+ 23.2.6.1 [NDB 与 InnoDB 存储引擎之间的差异](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/MySQL%20Server%20Using%20InnoDB%20Compared%20with%20NDB%20Cluster/Differences%20Between%20the%20NDB%20and%20InnoDB%20Storage%20Engins.md)

+ 23.2.6.2 [NDB 和 InnoDB 工作负载](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/MySQL%20Server%20Using%20InnoDB%20Compared%20with%20NDB%20Cluster/NDB%20and%20InnoDB%20Workloads.md)

+ 23.2.6.3 [NDB 和 InnoDB 功能使用总结](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/MySQL%20Server%20Using%20InnoDB%20Compared%20with%20NDB%20Cluster/NDB%20and%20InnoDB%20Feature%20Usage%20Summary.md)

##### 23.2.7 NDB 集群已知的限制

+ 23.2.7.1 NDB 集群中不符合的 SQL 语法

+ 23.2.7.2 NDB 集群与标准 MySQL  限制的限制和差异

+ 23.2.7.3 NDB 集群中与事务处理相关的限制

+ 23.2.7.4 NDB 集群错误处理

+ 23.2.7.5 与 NDB 集群中的数据库对象相关的限制

+ 23.2.7.6 与 NDB 集群中不支持或缺少的功能

+ 23.2.7.7 与 NDB 集群中的性能相关的限制

+ 23.2.7.8 NDB 集群独有的问题

+ 23.2.7.9 与 NDB Cluster 磁盘数据存储相关的限制

+ 23.2.7.10 与多个 NDB 集群节点相关的限制

+ 23.2.7.11 NDB 8.0 集群中解决的以前的 NDB 集群的问题

#### 23.3 [NDB 集群安装](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/NDB%20Cluster%20Installation/index.md)

##### 23.3.1 [在 Linux 上安装 NDB 集群](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/NDB%20Cluster%20Installation/Installation%20of%20NDB%20Cluster%20on%20Linux/index.md)

+ [在 Linux 上安装 NDB 集群二进制版本](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/NDB%20Cluster%20Installation/Installation%20of%20NDB%20Cluster%20on%20Linux/Installing%20an%20NDB%20Cluster%20Binary%20Release%20on%20Linux.md)

+ 从 RPM 安装 NDB Cluster

+ 使用 .deb 文件安装 NDB Cluster

+ 在 Linux 上从源代码构建 NDB 集群

##### 23.3.2 在 Windows 上安装 NDB Cluster

##### 23.3.3 [NDB 集群的初始配置](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/NDB%20Cluster%20Installation/Installation%20of%20NDB%20Cluster%20on%20Linux/Initial%20Configuration%20of%20NDB%20Cluster.md)

##### 23.3.4 [NDB 集群的初始启动](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/NDB%20Cluster%20Installation/Installation%20of%20NDB%20Cluster%20on%20Linux/Initial%20Startup%20of%20NDB%20Cluster.md)

##### 23.3.5 [带有表和数据的 NDB 集群示例](./MySQL%20NDB%20Cluster%208.0/NDB%20Cluster%20Overview/NDB%20Cluster%20Installation/Installation%20of%20NDB%20Cluster%20on%20Linux/NDB%20Cluster%20Example%20with%20Tables%20and%20Data.md)

##### 23.3.6 NDB 集群的安全关闭和重启

##### 23.3.7 升级和降级 NDB 集群

##### 23.3.8 NDB 集群自动安装程序（不再支持）

#### 23.4 NDB 集群的配置

#### 23.5 NDB 集群程序

#### 23.6 NDB 集群的管理

#### 23.7 NDB 集群复制

#### 23.8 NDB 集群发行说明
