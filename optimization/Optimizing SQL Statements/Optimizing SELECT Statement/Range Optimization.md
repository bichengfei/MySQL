# 8.2.1.2 范围优化

范围访问使用单个索引来检索包含在一个或多个索引值间隔内的表数据的子集。它可作用于单列或多列索引。以下部分描述了优化器使用范围访问的条件。

- 单列索引的范围访问方法

- 多列索引的范围访问方法

- 多值比较的等式范围优化

- 跳过扫描范围访问方法

- 行构造函数表达式的范围优化

- 限制用于范围优化的内存使用

## 单列索引的范围访问方法

对于单列索引，索引值区间可以方便地用 where 子句中对应的条件来表示，表示为范围条件，而不是“区间”。

单列索引的范围条件定义如下：

+ 对于 b-tree 和 hash 索引，在使用 =、<=>、in ()、is null 或 is not null 运算符时，将列与常量值比较的是范围条件

+ 此外，对于 b-tree 索引，当使用 >、<、>=、<=、between、!= 或者 <> 运算符时，将列与常量值比较的是范围条件，如果 like 的参数是不以通配符开头的常量字符串，则为 like 比较。

+ 对于任何索引类型，多个范围条件通过 and 或 or 合并成一个范围条件

上述描述中的“常量值”是指以下之一：

+ 来自查询中的字符串常量

+ 同一连接中 join 类型为 system 或 const 的表

+ 不相关子查询的结果

+ 任何完全由上述类型的子表达式组成的表达式

以下是 where 子句中具有范围条件的一些查询示例：

```sql
select * from t1 where key_col > 1 and key_col < 10;

select * from t1 where key_col = 1 or key_col in (15, 18, 20);

select * from t1 where key_col like 'ab%' or key_col between 'bar' and 'foo';
```

在优化器常量传播阶段，一些非常量值可能会转换为常量。

MySQL 尝试从 where 子句中为每个可能的索引提取范围条件。在提取过程中，丢弃不能用于构造范围条件的条件，合并产生重叠范围的条件，并去除产生空范围的条件

思考以下语句，其中 key1 是索引列，nonkey 不是索引列：

```sql
select * from t1 where
  (key1 < 'abc' and (key1 like 'abcde%' or key1 like '%b')) or
  (key1 < 'bar' and nonkey = 4) or
  (key1 < 'uux' and key1 > 'z')
```

对于 key1 的提取过程如下：

1. 从原始的 where 子句开始
   
   ```sql
   (key1 < 'abc' and (key1 like 'abcde%' or key1 like '%b')) or
   (key1 < 'bar' and nonkey = 4) or
   (key1 < 'uux' and key1 > 'z')
   ```

2. 删除`nonkey = 4`和`key1 like '%b`，因为他们不能用于范围扫描。删除它们的正确方法是用 true  来替换它们，这样我们在进行范围扫描时就不会错过任何匹配的行。下面是用 true 替换后的样子：
   
   ```sql
   (key1 < 'abc' and (key1 like 'abcde%' or true)) or
   (key1 < 'bar' and true) or
   (key1 < 'uux' and key1 > 'z')
   ```

3. 折叠总是为 tue 或 false 的条件
   
   + `(key1 like 'abcde%' or true)`永远为 true
   
   + `(key1 < 'uux' and key1 > 'z')`永远为 false
   
   用常量替换这些条件后：
   
   ```sql
   (key1 < 'abc' and true) or (key1 < 'bar' and true) or (false)
   ```
   
   删除不必要的 true 或 false 条件
   
   ```sql
   (key1 < 'abc') or (key1 < 'bar')
   ```

4. 将重叠的建个组合合并成一个用于范围扫描的最终条件
   
   ```sql
   (key1 < 'bar')
   ```
   
   通常（如前面的示例所示），用于范围扫描的条件没有 where 子句那么严格。MySQL 执行额外的检查以过滤调满足范围条件但不满足完整  where 子句的行。
   
   范围条件提取算法可以处理任意深度的嵌套 and/or 构造，其输出不依赖于条件在 where 子句中出现的顺序。
   
   对于空间索引的 join 类型为 range 的访问方法，MySQL 不支持合并多个范围。要解决此限制，你可以使用 union 合并相同的 select 语句，除了将每个空间谓词放在不同的 select.

## 多列索引的范围访问方法

   多列索引的范围条件是单列索引的范围条件的扩展。多列索引上的范围条件将索引行限制在一个或多个键元组区间内。键元组区间是在一组键元组上定义的，使用索引中的排序。

   例如，定义 (key_part1, key_part2, key_part3) 为多列索引，以及以下按照键顺序列出的键元组集：

```sql
key_part1  key_part2  key_part3
  NULL       1          'abc'
  NULL       1          'xyz'
  NULL       2          'foo'
   1         1          'abc'
   1         1          'xyz'
   1         2          'abc'
   2         1          'aaa'
```

   条件 key_part1 定义了这个间隔：

```sql
(1, -inf, -inf) <= (key_part1, key_part2, key_part3) < (1, +inf, +inf)
```

   键元组区间所覆盖的前面数据集的第 4、5、6 元组，可以被范围访问使用。

   相比之下，条件 key_part3 = 'abc' 没有定义单个区间，并且不能由范围访问方法使用。

   以下描述更详细的说明了多列索引如何使用范围条件：

+ 对于 hash 索引，可以使用包含相同值的每个区间。这意味着只能针对以下形式的条件生成区间
  
  ```sql
      key_part1 cmp const1
  and key_part2 cmp const2
  and ...
  and key_partN cmp constN;
  ```
  
  const1, const2, ... 是常数，cmp 是 =, <=>, 或 is null 比较运算符之一，条件覆盖所有索引列（即，有 N 个条件，每个对应一个索引列）。
  
  例如，下面是三个 hash 索引列的范围条件：
  
  ```sql
  key_part1 = 1 and key_part2 is null and key_part3 = 'foo'
  ```
  
  有关常量的定义，请参考上面“单列索引的范围访问方法”

+ 对于 btree 索引，间隔可能用于与 and 结合的条件，其中每个条件通过 =, <=>, is null, >, <, >=, <=, !=, <>, between, 或者 like (不以通配符开头) 比较列与常量。只要可以确定单个列元组包含符合条件的所有行，就可以使用间隔。（如果使用 <> 或 != 则使用两个间隔）。
  
  只要运算符是 =, <=> 或者 is null，优化器就会尝试使用额外列来确定间隔。如果运算符是 >, <, >=, <=, !=, <>, between, like，优化器使用它但不再考虑后续的列。对于以下表达式，优化器使用 = 来做第一次比较，它还使用 >= 来做第二次比较，但不考虑其他的列，并且不使用第三个比较进行间隔构造：
  
  ```sql
  key_part1 = 'foo' and key_part2 >= 10 and key_part3 > 10
  ```
  
  区间为：
  
  ```
  ('foo', 10, -inf) < (key_part1, key_part2, key_part3) < ('foo', +inf, +inf)
  ```
  
  创建的间隔可能包含比初始条件更多的行。例如，前面的区间包含('foo', 11, 0)，但它不满足原始条件。

+ 如果覆盖区间内包含的行集的条件通过 or 组合，则它们形成一个包含它们的区间的并集的覆盖区间条件。如果条件通过 and 组合，则它们形成一个包含它们的区间的交集的覆盖区间条件。例如，对于以下条件查询（key_part1, key_part2）是组合索引：
  
  ```sql
  (key_part1 = 1 and key_part2 < 2) or (key_part1 > 5)
  ```
  
  区间是：
  
  ```sql
  (1, -inf) < (key_part1, key_part2) < (1, 2)
  (5, -inf) < (key_part1, key_part2)
  ```
  
  在此示例中，第一行的间隔使用一个列作为左边界，两列作为右边界。第二行的间隔只是用列一个列。Explain 中的 key_len 指示使用的键前缀的最大长度。
  
  在某些情况下，key_len 可能表明使用了哪个列部分，但这可能不是你所期望的。假设 key_part1 和 key_part2 可以是 null，然后对于以下条件，key_len 显示列两列的长度：
  
  ```sql
  key_part1 >= 1 and key_part2 < 2
  ```
  
  但实际上，条件转换为：
  
  ```sql
  key_part1 >= 1 and key_part2 is not null
  ```
  
  有关如何执行优化以组合或消除单列索引的范围条件间隔的描述，请参阅单列索引的范围访问方法。对多部分索引的范围条件执行类似的步骤  

## 多值比较的等式范围优化

考虑以下表达式，其中 col_name 是索引列：

```sql
col_name in (val1, ..., valN)
col_name = val1 or ... or col_name = valN
```

col_name 如果等于多个值中的任何一个，则每个表达式都为真。这些比较是相等范围比较（其中“范围”是单个值）。优化器评估读取符合条件的行以进行相等范围比较的成本如下：

+ 如果， col_name 上有唯一索引，则每个范围的行估计指为 1，因为最多一行可以具有给定值

+ 否则，col_name 上的索引不是唯一的，则优化器可以通过索引下探(index dives)或索引统计信息，来估计每个范围的行数

使用索引下探，优化器在范围（有序）的每一端都进行下探，并使用范围内的行数作为评估值。例如，表达式 col_name in (10, 20, 30)，具有三个相等范围，优化器对每个范围进行两次下探以生成行评估，每对下探都会产生给定值的行数的评估值。

索引下探提供准确的行评估，但随着表达式中比较值数量的增加，优化器需要更长的时间来生成行评估。索引统计的使用不如索引下探准确，但允许对大量值列表进行更快的行评估。

系统参数`eq_range_index_dive_limit`使你能够配置优化器从一种行评估策略切换到另一种策略的值的数量。
