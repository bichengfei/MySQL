# 8.2.1.2 范围优化

范围访问使用单个索引来检索包含在一个或多个索引值间隔内的表数据的子集。它可作用于单部分或多部分索引。以下部分描述了优化器使用范围访问的条件。

- 单部分索引的范围访问方法

- 多部分索引的范围访问方法

- 多值比较的等式范围优化

- 跳过扫描范围访问方法

- 行构造函数表达式的范围优化

- 限制用于范围优化的内存使用

## 单部分索引的范围访问方法

对于单部分索引，索引值区间可以方便地用 where 子句中对应的条件来表示，表示为范围条件，而不是“区间”。

单部分索引的范围条件定义如下：

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
   
   ## 多部分索引多范围访问方法
   
   
