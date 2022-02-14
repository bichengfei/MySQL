# MySQL

基于 MySQL 5.7 的官方文档的 InnoDB 部分翻译，部分篇幅会有个人见解

官网地址：https://dev.mysql.com/doc/refman/5.7/en/preface.html

### InnoDB 存储引擎

### 1 InnoDB 简介

-  1.1 使用 InnoDB  表的好处
-  1.2 InnoDB 表的最佳实践
-  1.3 验证 InnoDB 是默认存储引擎
-  1.4 使用 InnoDB 进行测试和基准测试
-  1.5 关闭 InnoDB

### 2. InnoDB 和 ACID 模型

### 3. InnoDB 多版本控制 -- @[amlgithub](https://github.com/amlgithub)

### 4. InnoDB 架构

### 5. InnoDB 内存结构

- 5.1 缓冲池
- 5.2 更改缓冲区
- 5.3 自适应哈希索引
- 5.4 日志缓冲区

### 6. InnoDB 磁盘结构

- 6.1 表  
    &nbsp;&nbsp;&nbsp;&nbsp;创建 InnoDB 表  
    &nbsp;&nbsp;&nbsp;&nbsp;在外部创建表  
    &nbsp;&nbsp;&nbsp;&nbsp;导入 InnoDB 表  
    &nbsp;&nbsp;&nbsp;&nbsp;移动或复制 InnoDB 表  
    &nbsp;&nbsp;&nbsp;&nbsp;将表从 MyISAM 转换成 InnoDB  
    &nbsp;&nbsp;&nbsp;&nbsp;[InnoDB 中的 auto_increment 处理](./InnoDB/6/6.1/6.1.InnoDB中auto_increment处理（官网版）.md)  -- @[bichengfei](https://github.com/bichengfei)  
- 6.2 索引  -- @[bichengfei](https://github.com/bichengfei)  
    &nbsp;&nbsp;&nbsp;&nbsp;[聚集索引和二级索引](./InnoDB/6/6.2/6.2.聚集索引和二级索引.md)  
    &nbsp;&nbsp;&nbsp;&nbsp;[InnoDB 索引的物理结构](./InnoDB/6/6.2/6.2.InnoDB索引的物理结构.md)  
    &nbsp;&nbsp;&nbsp;&nbsp;排序索引构建  
    &nbsp;&nbsp;&nbsp;&nbsp;InnoDB 全文索引  
- 6.3 [表空间](./InnoDB/6/6.3/6.3.表空间.md)  
    &nbsp;&nbsp;&nbsp;&nbsp;[系统表空间](./InnoDB/6/6.3/6.3.表空间.md)  
    &nbsp;&nbsp;&nbsp;&nbsp;[每个表的文件表空间](./InnoDB/6/6.3/6.3.表空间.md)  
    &nbsp;&nbsp;&nbsp;&nbsp;[通用表空间](./InnoDB/6/6.3/6.3.表空间.md)  
    &nbsp;&nbsp;&nbsp;&nbsp;[撤销表空间](./InnoDB/6/6.3/6.3.表空间.md)  
    &nbsp;&nbsp;&nbsp;&nbsp;[临时表空间](./InnoDB/6/6.3/6.3.表空间.md)  
- 6.4 InnoDB 数据字典
- 6.5 双写缓冲区
- 6.6 [重做（redo）日志](./InnoDB/6/6.6/6.6.redo日志.md)
- 6.7 [撤销（undo）日志](./InnoDB/6/6.7/6.7.undo日志.md)
  
  ### 7. InnoDB 锁定和事务模型
  - 7.1 [InnoDB 锁定](./InnoDB/7/7.1.InnoDB锁定.md)
- 7.2 InnoDB 事务模型  
    &nbsp;&nbsp;&nbsp;&nbsp;事务隔离级别  
    &nbsp;&nbsp;&nbsp;&nbsp;自动提交、提交和回滚  
    &nbsp;&nbsp;&nbsp;&nbsp;一致的非锁定读取  
    &nbsp;&nbsp;&nbsp;&nbsp;锁定读取  
- 7.3 InnoDB 中不同 SQL 语句设置的锁
- 7.4 幻影行
- 7.5 InnoDB 中的死锁  
    &nbsp;&nbsp;&nbsp;&nbsp;InnoDB 死锁示例  
    &nbsp;&nbsp;&nbsp;&nbsp;死锁检测  
    &nbsp;&nbsp;&nbsp;&nbsp;如何最小化和处理死锁  

### 8. InnoDB 配置

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

### 9. InnoDB 表和页面压缩

- 9.1 InnoDB 表压缩  
    &nbsp;&nbsp;&nbsp;&nbsp;表压缩概述  
    &nbsp;&nbsp;&nbsp;&nbsp;创建压缩表  
    &nbsp;&nbsp;&nbsp;&nbsp;调整 InnoDB 表的压缩  
    &nbsp;&nbsp;&nbsp;&nbsp;在运行时监控 InnoDB 表的压缩  
    &nbsp;&nbsp;&nbsp;&nbsp;InnoDB 表的压缩如何工作  
    &nbsp;&nbsp;&nbsp;&nbsp;OLTP 工作负载的压缩  
    &nbsp;&nbsp;&nbsp;&nbsp;SQL 压缩语法警告与错误  
- 9.2 InnoDB 页面压缩

### 10. InnoDB 文件格式管理

- 10.1 启用文件格式
- 10.2 验证文件格式兼容性  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;InnoDB 启动时的兼容性检查  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开表时的兼容性检查
- 10.3 识别正在使用的文件格式
- 10.4 修改文件格式

### 11. InnoDB 行格式

### 12. InnoDB 磁盘 I/O 和文件空间管理

- 12.1 InnoDB 磁盘 I/O  
- 12.2 文件空间管理  
- 12.3 InnoDB 检查点
- 12.4 对表进行碎片整理
- 12.5 使用 truncate table 回收磁盘空间

### 13. InnoDB 和在线 DDL

- 13.1 在线 DDL 操作
- 13.2 在线 DDL 性能和并发
- 13.3 在线 DDL 空间要求
- 13.4 使用在线 DDL 简化 DDL 语句
- 13.5 在线 DDL 失败条件
- 13.6 在线 DDL 限制

### 14. InnoDB 静态数据加密

### 15. InnoDB 启动选项和系统变量

### 16. InnoDB 的 information_schema 表

- 16.1 InnoDB 的 information_schema 压缩表  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;InnoDB_cmp 和 InnoDB_cmp_reset  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;InnoDB_cmpmem 和 InnoDB_cmp_cmpmem_reset  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用压缩信息架构表  
- 16.2 InnoDB information_schema 事务和锁定信息  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用 InnoDB 事务和锁定信息  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;InnoDB 锁定和锁定等待信息  
     &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;InnoDB 事务和锁定信息的持久性和一致性  
- 16.3 InnoDB information_schema 系统表
- 16.4 InnoDB information_schema FullText 索引表
- 16.5 InnoDB information_schema 缓冲池表
- 16.6 InnoDB information_schema 指标表
- 16.7 InnoDB information_schema 临时表信息表
- 16.8 从 information_schema.files 检索 InnoDB 表空间元数据

### 17. InnoDB 与 MySQL_Performance Schema 的集成

- 17.1 使用性能模式监控 InnoDB 表的 alter table 进度
- 17.2 使用性能监控模式监控 InnoDB Mutex 等待

### 18. InnoDB 监视器

- 18.1 InnoDB 监视器类型
- 18.2 启用 InnoDB 监视器
- 18.3 InnoDB 标准监视器和锁定监视器输出

### 19. InnoDB 备份和恢复

- 19.1 InnoDB 备份
- 19.2 InnoDB 恢复

### 20. InnoDB 和 MySQL 复制

### 21. InnoDB 内存缓存插件

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

### 22. InnoDB 故障排查

- 22.1 对 InnoDB I/O 问题进行故障排除
- 22.2 强制 InnoDB 恢复
- 22.3 InnoDB 数据字典操作故障排除
- 22.4 InnoDB 错误处理
  
  ### 23. InnoDB Limits

### 24 InnoDB Restrictions and Limitations
