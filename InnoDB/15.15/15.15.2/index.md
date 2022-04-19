# 15.15.2 InnoDB INFORMATION_SCHEMA 事务和锁定信息

本节描述了由 performace_shcema 中的 data_locks 和 data_lock_waits 表提供的锁定信息，它们取代了 MySQL 8.0 中的 INFORMATION_SCHEMA 中的 INNODB_LOCKS 和 INNODB_LOCK_WAITS 表。有关根据旧`INFORMATION_SCHEMA`表编写的类似讨论，请参阅 [MySQL 5.7 参考手册](https://dev.mysql.com/doc/refman/5.7/en/)中的[InnoDB INFORMATION_SCHEMA Transaction and Locking Information](https://dev.mysql.com/doc/refman/5.7/en/innodb-information-schema-transactions.html)。

一张 information_schema 的表和两张 performace_schema 表，可以使你能够监控 InnoDB 事务并诊断潜在的锁定问题：

+ innodb_trx
  
  该 information_schema 表提供有关当前在 InnoDB 内部执行的每个事务的信息，包括事务状态（例如，它是正在运行或者等待锁定）、事务何时启动以及事务正在执行的特定 SQL 语句

+ data_locks
  
  该 performace_shema 表包含每个持有锁的行和每个被阻塞的锁的行，后者在等待锁被释放。
  
  + 无论持有锁的事务处于何种状态（innodb_trx.trx_state 是 running、lock wait、rolling back 或者 commiting），每个持有的锁都有一行。
  
  + InnoDB 中等待另一个事务释放锁的每个事务（innodb_trx.trx_state 是 lock wait）都恰好被一个阻塞锁请求阻塞。该阻塞锁请求是另一个事务以不兼容模式持有的行或表锁。锁定请求与阻塞请求，持有锁的模式总是不兼容的，例如读取与写入，共享和独占。
    
    在其它事务提交或回滚之前，阻塞的事务无法继续，也就无法释放请求的锁。对于阻塞的事务，data_locks 中对应的行，描述事务请求的每个锁以及它正在等待的锁。

+ data_lock_waits
  
  该 performace_shema 表指示那些事务正在等待给定的锁，或者给定的事务正在等待哪个锁。此表包含每个阻塞事务的一行或多行，指示它已请求的锁以及阻塞该请求的任何锁。REQUESTING_ENGINE_LOCK_ID值是指事务请求的锁，BLOCKING_ENGINE_LOCK_ID 值是指阻止第一个事务继续进行的锁（由另一个事务持有）。对于任何给定的阻塞事务，data_lock_waits 中的所有行中，REQUESTING_ENGINE_LOCK_ID 有相同的值，BLOCKING_ENGINE_LOCK_ID 有不同的值。

有关上述表的更多信息，请参阅 [第 26.4.28 节，“INFORMATION_SCHEMA INNODB_TRX 表”](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-trx-table.html "26.4.28 INFORMATION_SCHEMA INNODB_TRX 表")， [第 27.12.13.1 节，“data_locks 表”](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-locks-table.html "27.12.13.1 data_locks 表")和 [第 27.12.13.2 节，“data_lock_waits 表”](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-data-lock-waits-table.html "27.12.13.2 data_lock_waits 表")。
