`explain`语句提供有关 MySQl 如何执行语句的信息。`explain`适用于`select`,`delete`,`insert`,`update`,`replace`语句

`explain`为在`select`语句中使用的每一个表生成一行信息，按照`MySQL`在处理语句时读取它们的顺序有序的展示。`MySQL 5.7`使用`nested-loop`解析所有的 `join`，这意味着`MySQL`从第一个表中读取一行，然后在第二个表中找到匹配的行......第三个表......，处理完所有的表后，`MySQL`输出指定的列，并返回到第一张表，找到所有匹配到列。（*这一段描述主要在讲`nested-loop`，属于系统设计中的经典设计，不懂的另行查阅*）

`explain`输出包括分区信息，对于`select`语句，`explain`还会生成等同于`show warnings`的额外信息

```
Note:
在较久的 MySQL 中，分区和扩展信息通过 explain partitions 和 explain extended 生成，未来版本将删除
```

## `EXPLAIN`输出列解释

本节描述由 explain 生成的列，后面的章节会描述 type 和 extra 列的其他信息

explain 输出中的每一行信息都和一张表关联，每一行值和对应的详细描述都在下表中。下表中的第一列展示 explain 输出中的列名，第二列提供了 format=json 时的等效属性名称

| column        | json name     | 意义           |
| ------------- | ------------- | ------------ |
| id            | select_id     | 该 select 标识符 |
| select_type   | None          | 该 select 类型  |
| table         | table_name    | 本行解释的表       |
| partitions    | partitions    | 匹配的分区        |
| type          | access_type   | join 类型      |
| possible_keys | possible_keys | 可供选择的索引      |
| key           | key           | 实际选择的索引      |
| key_len       | key_length    | 选择索引的长度      |
| ref           | ref           | 与索引比较的列      |
| rows          | rows          | 要检索行数的预估值    |
| filtered      | filtered      | 按表条件过滤的行百分比  |
| extra         | none          | 额外信息         |

```
Note
JSON properties which are NULL are not displayed in JSON-formatted EXPLAIN output.
```

### id

select 标识符，是该 select 在本次查询中的序列号。当行数据指向 union 数据的时候，该值可能为 null。id 越大，优先级越高，id 相同，优先级相同，执行从上到下，具体看优化器

### select_type

select 类型，可以是下表中的任何一种。JSON 格式的 explain 将 select 类型座位一个 query_block 属性，除非他是 simple 或者 primary 类型。

| `select_type` Value  | JSON Name                  | 含义                                |
| -------------------- | -------------------------- | --------------------------------- |
| simple               | None                       | 简单的 select(不使用 union 或者子查询)       |
| primary              | None                       | 最外层的 select                       |
| union                | None                       | union 中的第二个或者更后面的 select          |
| dependent union      | dependent(true)            | union 中的第二个或者更后面的 select，依赖于外部查询  |
| union result         | union_result               | union 的结果                         |
| subquery             | None                       | select 或者 where 中的子查询，不依赖外层查询     |
| dependent subquery   | dependent(true)            | select 或者 where 中的子查询，依赖外层查询      |
| derived              | None                       | 派生表（例如 union 表）                   |
| materialized         | materialized from_subquery | 物化子查询                             |
| uncacheable subquery | cacheable(false)           | 无法缓存结果并且必须为外部查询的每一行重新评估的子查询       |
| uncacheable union    | cacheable(false)           | 无法换成结果并且是 union 中的第二个活更后面的 select |

派生表：当在`SELECT`语句的`FROM`子句中使用独立子查询时，我们将其称为派生表。mysql 5.7 需要设置 derived_merge=off，否则MySQL 可能会把派生表合并到外层查询

dependent 通常表示使用相关子查询，也就是子查询依赖于上层查询结果

dependent subquery 和 uncacheable subquery，区别在于对于外层查询到不同值，前者子查询仅评估一次，后者对每一行重新评估子查询

子查询的缓存和查询结果的缓存是不同的，子查询缓存发生在查询执行期间，而查询结果缓存发生在查询执行完成后

非 select 语句的 select_type 展示语句类型，例如：delete 语句的 select_type 为 delete

### table

本条执行计划影响的表的名称，也可能是以下之一：

+ <union<B>M</B>,<B>N</B>>
+ <derived<B>N</B>>
+ <subquery<B>N</B>>

### partitions

查询语句匹配到的数据所在的分区，非分区表该值为 null

### type

连接类型，下面详细说明

### possible_keys

possible_keys 列表明 MySQL 可以从 table 中选择哪些索引。请注意，该列完全独立于在 explain 中展示的 table 的顺序，这意味着 possible_keys 中的某些列在实际情况下，可能无法与生成的表顺序一起使用

如果此列为 null，则没有相关索引。在这种情况下，你可以通过检查 where 子句是否引用了适合构建索引的列，从而提升查询的性能。

查看表有哪些索引，使用`show index from table_name`

### key

该列表明实际使用的索引。如果 MySQL 决定使用 possible_keys 中的某列来查找数据，则将该索引在 key 列中输出。

key 可能展示在 possible_keys 中不存在的列。如果 possible_keys 中的所有列都不适合查找数据，但查询的列都是某个索引的列，就会出现这种情况。也就是说，该索引覆盖了选定的列，虽然它不能确定要检索哪些行，但索引扫描比数据行扫描效率更高。

对于 InnoDB，二级索引可能覆盖 select 的列，即使 select 中包含主键，因为 InnoDB 的每个二级索引都有存储主键值。如果 key 值为空，表明 MySQL 找不让提升查询性能的索引。

如果要强制 MySQL 使用或者忽略 possible_keys 中的某些索引，可以在查询中使用 force index、use index 或者 ignore index。详见[Index Hints](https://dev.mysql.com/doc/refman/5.7/en/index-hints.html)

对于 MyISAM，详见 [analyze table statement](https://dev.mysql.com/doc/refman/5.7/en/analyze-table.html) 和 [MyISAM Table Maintenance and Crash Recovery](https://dev.mysql.com/doc/refman/5.7/en/myisam-table-maintenance.html)

### key_len

key_len 列是为了表明 MySQL 决定使用的 key 的长度，对于组合索引，可以帮助我们确定实际使用的列。如果 key 的值为 null，则 key_len 也一定会是 null。

因为存储格式的问题，null 的列比 not null 的列的索引长度会大 1 个字节。

### ref

ref 列展示 key 中的列将会和哪些列或者常量来比较，用以从表中查询数据

如果值为 fun，则使用的值是某个函数的结果。要查看是那个 func，在 explain 后面使用 show warnings 去查看额外的 explain 输出。该函数可能是一个运算符，例如算术运算符。

### rows

MySQL 认为执行这次查询，必须检查的行数

对于 InnoDB 表，这只是一个估计值，并不总是准确的

### filtered

按照 where 条件过滤后，剩下的数据所占 rows 的百分比。最大值为 100，意味着没有过滤行，从 100 开始递减。rows * filtered 显示与下表连接的行数。例如。如果 rows = 1000，filtered = 50，则要与下表连接的行数 = 1000 * 50% = 500

### extra

此列包含有关 MySQL 如何解析查询的附加信息。

## join 类型说明

explain 输出中的 type 表示如何连接表。以下描述了连接类型，性能按照从高到低排列：

### system

系统表，常驻内存，不需要磁盘 io

### const

该表最多有一个匹配行，在查询开始时被读取。因为只有一行，该行中的列值，可以被优化器的其他部分作为常量使用。const 表非常快，因为他们只被读取一次。

type 为 const 条件：

1. 条件列为 primary key 或 unique index 的全部列；
2. 比较的值为常量；

例如：

```sql
select * from tbl_name where primary_key = 1;

select * from tbl_name where primary_key_part1 = 1 and primary_key_part2 = 2;
```

### eq_ref

官网翻译：

对于先前表中的每个行组合，从该表中读取一行。这是除了 system 和 const 外，最好的连接类型。当连接使用索引的全部字段，并且索引是主键或唯一非空索引时使用它。

eq_ref 可用于通过 "=" 比较的索引列。比较值可以是常量或前面表中的列的表达式。在以下示例中，MySQL 可以使用 eq_ref 连接来处理 ref_table:

```sql
select * from ref_table, other_table where ref_table.key_column = other_table.column;

select * from ref_table, other_table where ref_table_key_column_part1 = other_table.column and ref_table.key_colunn_part2 = 1;
```

个人理解：

对于前面表（外循环）的每一行，当前表（内循环）都会根据某一列命中唯一索引。

注：前面表（外循环）、当前表（内循环），指 nested-loop 中的上一个循环表和当前循环表。哪张表属于外循环，才是 type 类型的关键

条件：

1. 连接查询，可以是 where、join

2. 等值连接

3. 连接的两个字段是否都是 primary key 或 unique not null index：
   
   如果是，则 sql 怎么写都可以，type 一定为 eq_ref；
   
   如果不是，并且只有一个字段是  primary key 或 unique not null index，那么只有是内循环的键是唯一的，才可以

例如：

```sql
mysql> select version();
+------------+
| version()  |
+------------+
| 5.7.11-log |
+------------+
1 row in set (0.00 sec)

mysql> CREATE TABLE `t_student`
    -> (
    ->     `id`   int(11)     NOT NULL,
    ->     `name` varchar(40) NOT NULL,
    ->     PRIMARY KEY (`id`)
    -> ) ENGINE = InnoDB;
Query OK, 0 rows affected (0.03 sec)

mysql> 
mysql> CREATE TABLE `t_grade`
    -> (
    ->     `id`         int(11) NOT NULL,
    ->     `student_id` int(11) DEFAULT NULL,
    ->     `course_id`  int(11) DEFAULT NULL,
    ->     `grade`      int(11) DEFAULT NULL,
    ->     PRIMARY KEY (`id`)
    -> ) ENGINE = InnoDB;
Query OK, 0 rows affected (0.02 sec)

mysql> INSERT INTO t_student (id, name) VALUES (1, '张三');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO t_student (id, name) VALUES (2, '王五');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO t_student (id, name) VALUES (3, '小李');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO t_student (id, name) VALUES (4, '王华');
Query OK, 1 row affected (0.06 sec)

mysql> INSERT INTO t_grade (id, student_id, course_id, grade) VALUES (1, 1, 1, 100);
Query OK, 1 row affected (0.01 sec)

mysql> explain select * from t_student s left join t_grade g on s.id = g.student_id;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | s     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    4 |   100.00 | NULL                                               |
|  1 | SIMPLE      | g     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.01 sec)

mysql> explain select * from t_grade g left join t_student s on s.id = g.student_id;
+----+-------------+-------+------------+--------+---------------+---------+---------+-------------------------+------+----------+-------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref                     | rows | filtered | Extra |
+----+-------------+-------+------------+--------+---------------+---------+---------+-------------------------+------+----------+-------+
|  1 | SIMPLE      | g     | NULL       | ALL    | NULL          | NULL    | NULL    | NULL                    |    1 |   100.00 | NULL  |
|  1 | SIMPLE      | s     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | spring_jpa.g.student_id |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+--------+---------------+---------+---------+-------------------------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)

mysql> INSERT INTO t_grade (id, student_id, course_id, grade) VALUES (2, 1, 2, 80);
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO t_grade (id, student_id, course_id, grade) VALUES (3, 1, 3, 10);
Query OK, 1 row affected (0.01 sec)

mysql> explain select * from t_grade g left join t_student s on s.id = g.student_id;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | g     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |   100.00 | NULL                                               |
|  1 | SIMPLE      | s     | NULL       | ALL  | PRIMARY       | NULL | NULL    | NULL |    4 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.01 sec)

mysql> explain select * from t_grade g left join t_student s on g.student_id = s.id;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | g     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |   100.00 | NULL                                               |
|  1 | SIMPLE      | s     | NULL       | ALL  | PRIMARY       | NULL | NULL    | NULL |    4 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.01 sec)

mysql> 
```

### ref

对于先前表中的每个行组合，从该表中读取具有匹配索引值的所有行。

条件：

1. 比较列为索引列
2. 仅使用最左前缀列，或者比较列索引类型不是主键或唯一索引

也就是，如果连接不能通过提供的值确定单行数据，则使用，如果使用的比较列只匹配几行，这是一个很好的连接类型。

ref 可用于通过 “=” 或 “<=>" 比较的索引列。在以下的示例中，MySQL 可以使用 ref 连接来处理 ref_table:

```sql
select * from ref_table where key_column = expr;

select * from ref_table, other_table where ref_table.key_column = other_table.column;

select * from ref_table, other_table where ref_table.key_column_part1 = other_table.column and ref_table.key_column_part2 = 1;
```

### fulltext

连接是使用`FULLTEXT` 索引执行的

### ref_or_null

类似于 ref，但 MySQL 会额外搜索包含 null 值的行。这种连接类型优化最常用于解析子查询。示例如下

```sql
select * from ref_table where key_column = expr or key_column is null;
```

### index_merge

此连接类型表明使用了索引合并优化。在这种情况下，key 列中包括所有使用到的索引，key_len 中包含所有使用到的索引的生效的长度。有关更多信息，参考[索引合并优化](https://dev.mysql.com/doc/refman/5.7/en/index-merge-optimization.html)

### unique_subquery

针对以下形式的子查询，此类型替换 eq_ref:

```sql
value in (select primary_key form single_table where some_expr)
```

unique_subquery 是一种完全替代子查询的索引查找功能，用以提升效率

### index_subquery

类似于 unique_subquery，但它适用于以下形式的子查询中的非唯一索引：

```sql
value in (select key_column from single_table where some_expr)
```

### range

使用索引检索给定范围内的行。key 列的输出表明使用哪个索引，key_len 表明实际使用的最长索引长度，此类型 ref 列为空。

该类型会出现在，索引列在与常量通过 <, <=, =, <=>, =>, >, <>, is null, between, like 或 in 运算符汇总的任何一个比较时

```sql
select * from tbl_name where key_column = 10;

select * from tbl_name where key_column between 10 and 20;

select * from tbl_name where key_column in (10, 20, 30);

select * from tbl_name where key_part1 = 10 and key_part2 in (10, 20, 30);
```

### index

该类型和 all 类似，区别在于扫描的是索引树。这有两种情况：

+ 索引是覆盖索引并且可以满足表中所需的所有数据，则仅扫描索引树。在这种情况下，extra 列显示 `Using index`。因为索引通常比整张表的空间更小，所以索引扫描通常比 all 要快

+ 

### all
