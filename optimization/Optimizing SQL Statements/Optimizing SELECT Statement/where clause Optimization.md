# 8.2.1.1 WHERE 子句优化

本节讨论处理 where 子句时进行的优化。这些示例使用 select 语句，但是同样适用于 delete 和 update 中的 where 子句

*因为 MySQL 优化器还在不断的完善中，因此此处并未记录 MySQL 执行的所有优化*

你可能想要重写查询以使算术运算更快，同时牺牲可读性。因为 MySQL 会自动进行类似的优化，所以你通常可以避免这项工作，并将查询保留在更易于理解和维护的形式中。MySQL 执行的一些优化如下：

- 删除不必要的括号：

```sql
   ((a and b) and c or (((a and b) and (c and d))))
-> (a and b and c) or (a and b and c and d)
```

- 常数折叠

```sql
   (a < b and b = c) and a = 5
-> b > 5 and b = c and a = 5
```

- 常数条件移除

```sql
   (b >= 5 and b = 5) or (b = 6 and 5 = 5) or (b = 7 and 5 = 6)
-> b = 5 or b = 6
```

*在 MySQL 8.0.14 及更高版本中，这发生在准备阶段而不是优化阶段，这有助于简化连接。有关更多信息和示例，请参阅[外部连接优化](https://dev.mysql.com/doc/refman/8.0/en/outer-join-optimization.html)*

- 索引使用的常量表达式只计算一次

- 从 MySQL 8.0.16 开始，对数值类型对列与常量值对比较进行检查并折叠或删除无效或超出范围对值：

```sql
# create table t (c tinyint unsigned not null);
   select * from t where c << 256;
-> select * from t where 1;
```

*有关详细信息，请参考[常数折叠优化]([MySQL :: MySQL 8.0 Reference Manual :: 8.2.1.14 Constant-Folding Optimization](https://dev.mysql.com/doc/refman/8.0/en/constant-folding-optimization.html))*

- 对于使用 MyISAM 和 Memory 引擎的单表，在没有 where 条件时，count(*) 将直接从表信息中获取。当仅用于一个表时，对于任何 not null 表达式也同样适用。

- 提前检查无效的常量表达式。MySQL 快速检测到某些 select 语句是不可能的并且不返回任何行。

- having 将被合并到 where 如果你不使用 group by 或聚合函数(count()、min() 等)。

- 对于连接中的每个表，构造一个更简单的 where 来对每个表得到一个更快的 where 评估并尽可能的跳过行。

- 在查询中的其他表读取之前，先读取常量表。常量表有以下形式：
  
  - 空表或只有一行的表
  
  - 在 where 子句中使用一个表的主键或唯一索引，其中所有索引部分都与常量表达式比较，并且都被定义为 not null

以下所有表都用作常量表

```sql
select * from t where primary_key = 1;
select * from t1, t2 where t1.primary_key = 1 and t2.primary_key = t1.id;
```

- 通过尝试所有可能性来找到用于连接表的最佳连接方式。如果 order by 和 group by 子句中的所有字段来自于相同的表，则在加入时首先选择该表。

- 如果有一个 order by 子句和一个不同的 group by 子句，或者如果 order by 或 group by 包含来自连接队列中第一个表以外的表的列，则会创建一个临时表。

- 如果使用 `sql_small_result`修饰符，MySQL 使用内存中的临时表。

- 查询每个表索引，并使用最佳索引，除非优化器认为使用全表扫描更有效。在过去，根据最佳索引是否超过全表扫描的 30%，但固定百分比不再决定使用索引或者全表扫描之间的选择。优化器现在更加负责，它会基于其他因素，例如表大小、行数和 I/O 块大小。

- 在某些情况下，MySQL 甚至可以在不查阅数据文件的情况下从索引中读取行。如果索引中使用的所有列都是数字，则仅使用索引树来解析查询。（*字符串字段误使用数字进行查询，会导致隐式类型转换，无法命中索引*）

- 在每一行输出之前，那些不匹配 having 子句的被跳过

一些非常快的查询示例：

```sql
select count(*) from tbl_name;

select min(key_part1), max(key_part1) from tbl_name;

select max(key_part2) from tbl_name where key_part1=constant;

select ... from tbl_name order by key_part1,key_part2,... limit 10;

select ... from tbl_name order by key_part1 desc, key_part2 desc, ... limit 10;
```

MySQL 仅使用索引树解析以下查询，假设索引列是数字：

```sql
select key_part1, key_part2 from tbl_name where key_part1 = val;

select count(*) from tbl_name where key_part1 = val1 and key_part2 = val2;

select max(key_part2) from tbl_name group by key_part1;

/* 个人见解：但使用字符串，主要不要隐式类型转换，也会走覆盖索引 */
create table tbl_name (
    id int primary key ,
    col1 int,
    col2 int,
    col3 varchar(10),
    col4 varchar(10),
    index index_col1_col2(col1, col2),
    index index_col3_col4(col3, col4)
);

INSERT INTO tbl_name (id, col1, col2, col3, col4) VALUES (1, 1, 2, '3', '4');
INSERT INTO tbl_name (id, col1, col2, col3, col4) VALUES (2, 11, 12, '13', '14');
INSERT INTO tbl_name (id, col1, col2, col3, col4) VALUES (3, 21, 22, '23', '24');

explain select col1, col2 from tbl_name where col1 = 1;
explain select col3, col4 from tbl_name where col3 = '3';
explain select col3, col4 from tbl_name where col3 = 3;

explain select count(*) from tbl_name where col1 = 1 and col2 = 2;
explain select count(*) from tbl_name where col3 = '3' and col4 = '4';
explain select count(*) from tbl_name where col3 = 4 and col2 = 4;

explain select max(col2) from tbl_name where col1 = 1;
explain select max(col4) from tbl_name where col3 = '3';
```

以下查询使用索引以排序顺序检索行，而无需单独的排序传递：

```sql
select ... from tbl_name order by key_part1, key_part2,..;

select ... from tbl_name order by key_part1 desc, key_part2 desc, ...;
```
