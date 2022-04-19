# 12.10 全文搜索功能

match(col1, col2, ...) against (expr [search_modifier_])

```sql
search_modifier:
{
      IN NATURAL LANGUAGE MODE
    | IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION
    | IN BOOLEAN MODE
    | WITH QUERY EXPANSION
}
```

MySQL 支持全文索引和搜索：

+ MySQL 中的全文索引是类型为 FULLTEXT 的索引

+ 全文索引只能在 InnoDB 或 MyISAM 表中使用，并且只能为 char、varchar 或 text 列创建

+ MySQL 提供列一个内置的全文 ngram 解析器，它支持中文、日文和韩文（CJK），以及一个可安装的日文 MeCab 全文解析器插件。[第 12.10.8 节，“ngram 全文解析器”](https://dev.mysql.com/doc/refman/8.0/en/fulltext-search-ngram.html "12.10.8 ngram 全文解析器")和 [第 12.10.9 节，“MeCab 全文解析器插件”](https://dev.mysql.com/doc/refman/8.0/en/fulltext-search-mecab.html "12.10.9 MeCab 全文解析器插件")中概述了解析差异。

+ 可以在创建表时在语句中给出索引定义，或者稍后使用 alter table 或 create index 添加

+ 对于大型数据集，将数据加载到没有 FULLTEXT 索引的表中然后创建 FULLTEXT 索引，比将数据加载到具有现有 FULLTEXT 索引的表中要快得多。

使用`match() against()`语法执行全文搜索。`match()`接收一个以逗号分割的列表来命名要搜索的列；`against()`接收一个要搜索的字符串和一个可选的修饰符，指示要执行的搜索类型。搜索字符串必须是在查询评估期间保持不变的字符串值。例如，这排除了表列，因为每行可能不同

以前，MySQL 允许使用带有`match()`的汇总列，但使用此构造的查询性能不佳且结果不可靠。这是因为 MATCH() 不是作为其参数的函数实现的，而是作为基表底层扫描中当前行的行 ID 的函数。从 MySQL 8.0.28 开始，MySQL 不再允许此类查询，更具体地说，与此处列出的所有条件匹配的任何查询都会被拒绝，伴随着`ER_FULLTEXT_WITH_ROLLUP`:

+ MATCH() 出现在查询块的 select 列表、group by 子句、having 子句或者 order by 子句中

+ 查询块包含一个 `group by ... with rollup` 子句

+ 调用 match() 函数的参数是分组列之一

此处显示列此类查询的一些示例：

```sql
# MATCH() in SELECT list...
SELECT MATCH (a) AGAINST ('abc') FROM t GROUP BY a WITH ROLLUP;
SELECT 1 FROM t GROUP BY a, MATCH (a) AGAINST ('abc') WITH ROLLUP;

# ...in HAVING clause...
SELECT 1 FROM t GROUP BY a WITH ROLLUP HAVING MATCH (a) AGAINST ('abc');

# ...and in ORDER BY clause
SELECT 1 FROM t GROUP BY a WITH ROLLUP ORDER BY MATCH (a) AGAINST ('abc');
```

允许在 where 子句中将 match() 与 rollup 一起使用

全文索引分为三种类型：

+ 自然语言搜索
  
  将搜索字符串解释为自然人类语言中的短语（自由文本中的短语）。除双引号（"）字符外，没有特殊运算符。适用分词列表，请参阅[12.10.4 全文分词](./12.10.4.全文分词.md)
  
  如果给出了`IN NATURAL LANGUAGE MODE`修饰符或没有给出修饰符，则全文搜索为自然语言搜索。有关更多信息，请参阅[12.10.1 自然语言全文搜索](./12.10.1.自然语言全文搜索.md)

+ 布尔搜索
  
  使用特殊查询语言的规则解释搜索字符串。该字符串包含要搜索的单词，它还可以包含指定要求的运算符，例如匹配行中必须存在或不存在的单词，或者它的权重应该高于或低于平时。某些常用词（分词）从搜索索引中忽略，如果出现在搜索字符串中，则不匹配。`IN BOOLEAN MODE`修饰符指定布尔搜索，有关更多信息，请参阅[12.10.2.布尔全文搜索](./12.10.2.布尔全文搜索.md)

+ 查询扩展搜索
  
  对自然语言搜索的修改。搜索字符串用于执行自然语言搜索，然后将搜索返回的最相关行中的单词添加到搜索字符串中，然后再次进行搜索，查询返回第二次搜索的行。`IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION`修饰符指定查询扩展搜索。有关更多信息，请参阅[12.10.3.带有查询扩展的全文搜索](./12.10.3.带有查询扩展的全文搜索.md)

有关`FULLTEXT`查询性能的信息，请参阅[第 8.3.5 节，“列索引”](https://dev.mysql.com/doc/refman/8.0/en/column-indexes.html "8.3.5 列索引")。

有关`InnoDB` `FULLTEXT`索引的更多信息，请参阅 [第 15.6.2.4 节，“InnoDB 全文索引”](https://dev.mysql.com/doc/refman/8.0/en/innodb-fulltext-index.html "15.6.2.4 InnoDB 全文索引")。

全文搜索的约束在 [第 12.10.5 节，“全文限制”](./12.10.5.全文搜索限制.md)中列出。

myisam_ftdump 实用程序转储 MyISAM 全文索引的内容， 这可能有助于调试全文查询。请参阅 [第 4.6.3 节，“myisam_ftdump - 显示全文索引信息”](https://dev.mysql.com/doc/refman/8.0/en/myisam-ftdump.html "4.6.3 myisam_ftdump——显示全文索引信息")。
