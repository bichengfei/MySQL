# 15.15 InnoDB 的 INFORMATION_SCHEMA 表

本节提供 InnoDB  `INFORMATION_SCHEMA` 系列表的信息和使用示例。

InnoDB  `information_schema` 系列表提供有关 InnoDB 存储引擎各个方面的元数据、状态信息和统计信息。你可以通过在 information_shema 库中使用 show tables 语句查看 InnoDB information_shcma 的表列表：

```sql
mysql> SHOW TABLES FROM INFORMATION_SCHEMA LIKE 'INNODB%';
```

有关表定义，请参阅 [第 26.4 节，“INFORMATION_SCHEMA InnoDB 表”](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-tables.html "26.4 INFORMATION_SCHEMA InnoDB 表")。有关`MySQL` `INFORMATION_SCHEMA`数据库的一般信息，请参阅 [第 26 章，*INFORMATION_SCHEMA 表*](https://dev.mysql.com/doc/refman/8.0/en/information-schema.html "第 26 章 INFORMATION_SCHEMA 表")。
