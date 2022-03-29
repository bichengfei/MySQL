# 23.3.5 带有表和数据的 NDB 集群示例

```textile
本节中的信息适用于在 Unix 和 Windows 平台上运行的 NDB 集群
```

在 NDB 集群中处理数据库表和数据与在标准 MySQL 中处理没有太大区别，有两个关键点要记住：

+ 对于要在集群中复制的表，它必须使用 NDB 存储引擎。要指定这一点，请在创建表时使用 engine=NDBCLUSTER 或 engine=NDB 选项：
  
  ```sql
  create table tbl_name(col_name column_definitions) engine=NDBCLUSTER;
  ```
  
  或者，对于使用不同存储引擎的现有表，使用 alter table 将表更改为使用 NDBCLUSTER:
  
  ```sql
  alter table tbl_name engine=NDBCLUSTER;
  ```

+ 每个 NDB 表都有一个主键。如果创建表时用户没有定义主键，NDB 存储引擎会自动生成一个隐藏的主键，就像其它表索引一样，这样的键占用空间。（由于存储这些自动创建的索引导致的内存不足问题并不少见）

如果你使用 mysqldump 的输出从现有数据库导入表，则可以在文本编辑器中打开 SQL 脚本并将 engine 选项添加到任何表创建语句，或替换任何现有 engine 选项。假设在一个不支持 NDB 集群的 MySQL  服务器上有 world 库，并且你想导出 city 表：

```shell
$> mysqldump --add-drop-table world City > city_table.sql
```

生成的 city_table.sql 文件包含此表创建语句（以及导入表数据所需的 insert 语句）：

```sql
DROP TABLE IF EXISTS `City`;
CREATE TABLE `City` (
  `ID` int(11) NOT NULL auto_increment,
  `Name` char(35) NOT NULL default '',
  `CountryCode` char(3) NOT NULL default '',
  `District` char(20) NOT NULL default '',
  `Population` int(11) NOT NULL default '0',
  PRIMARY KEY  (`ID`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

INSERT INTO `City` VALUES (1,'Kabul','AFG','Kabol',1780000);
INSERT INTO `City` VALUES (2,'Qandahar','AFG','Qandahar',237500);
INSERT INTO `City` VALUES (3,'Herat','AFG','Herat',186800);
(remaining INSERT statements omitted)
```

你需要确保 MySQL 为该表使用 NDB 存储引擎。有两种方法可以实现这一点，其中之一是在导入 NDB 集群数据库之前对表定义进行修改。以 city 表为例，修改 engine 定义的选项如下：

```sql
DROP TABLE IF EXISTS `City`;
CREATE TABLE `City` (
  `ID` int(11) NOT NULL auto_increment,
  `Name` char(35) NOT NULL default '',
  `CountryCode` char(3) NOT NULL default '',
  `District` char(20) NOT NULL default '',
  `Population` int(11) NOT NULL default '0',
  PRIMARY KEY  (`ID`)
) ENGINE=NDBCLUSTER DEFAULT CHARSET=latin1;

INSERT INTO `City` VALUES (1,'Kabul','AFG','Kabol',1780000);
INSERT INTO `City` VALUES (2,'Qandahar','AFG','Qandahar',237500);
INSERT INTO `City` VALUES (3,'Herat','AFG','Herat',186800);
(remaining INSERT statements omitted)
```

对于要成为集群数据库的部分，每个表都要进行这一步操作。完成此操作的最简单方法是对包含定义的文件进行搜索和替换，并将 type=engine_name 或 engine=engine_name 的所有实例替换为 engine=NDBCLUSTER。如果你不想修改文件，可以使用未修改的文件创建表，然后使用 alter table 修改其存储引起，细节在本节后面给出。

假设你已经在集群的 SQL 节点上创建了一个名为 world 的数据库，那么你可以使用 mysql 命令行客户端读取 city_table.sql，并按照通常的方式创建并填充相应的表：

```sql
$> mysql world < city_table.sql
```

非常重要的是要记住，前面的命令必须在运行 SQL 节点的主机上执行（在这种情况下，在 IP 地址为 的机器上 `198.51.100.20`）。

要在 SQL 节点上创建整个`world`数据库的副本，请在非集群服务器上使用[**mysqldump**](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html "4.5.4 mysqldump——一个数据库备份程序")将数据库导出到名为 `world.sql`（例如，在 `/tmp`目录中）的文件中。然后按照刚才的描述修改表定义，并将文件导入集群的 SQL 节点，如下所示：

```shell
$> mysql world < /tmp/world.sql
```

如果您将文件保存到其他位置，请相应地调整前面的说明。

在 SQL 节点上运行[`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.10 SELECT 语句")查询与在 MySQL 服务器的任何其他实例上运行它们没有什么不同。要从命令行运行查询，您首先需要以通常的方式登录 MySQL Monitor（ 在提示符 处指定`root`密码）：`Enter password:`

```shell
$> mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1 to server version: 8.0.29-ndb-8.0.29

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql>
```

我们只是使用 MySQL 服务器的`root` 帐户，并假设您已遵循安装 MySQL 服务器的标准安全预防措施，包括设置强`root`密码。有关更多信息，请参阅 [第 2.10.4 节，“保护初始 MySQL 帐户”](https://dev.mysql.com/doc/refman/8.0/en/default-privileges.html "2.10.4 保护初始 MySQL 帐户")。

值得考虑的是，NDB Cluster 节点 在相互访问时*不*使用 MySQL 特权系统。设置或更改 MySQL 用户帐户（包括`root`帐户）仅影响访问 SQL 节点的应用程序，而不影响节点之间的交互。有关更多信息，请参阅 [第 23.6.18.2 节，“NDB 集群和 MySQL 特权”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-security-mysql-privileges.html "23.6.18.2 NDB 集群和 MySQL 权限")。

如果在导入 SQL 脚本之前没有修改表定义中的子句的 engine，则此时应运行以下语句：

```sql
mysql> USE world;
mysql> ALTER TABLE City ENGINE=NDBCLUSTER;
mysql> ALTER TABLE Country ENGINE=NDBCLUSTER;
mysql> ALTER TABLE CountryLanguage ENGINE=NDBCLUSTER;
```

选择数据库并对该数据库中的表运行**SELECT**查询也以通常的方式完成，就像退出 MySQL Monitor 一样：

```sql
mysql> USE world;
mysql> SELECT Name, Population FROM City ORDER BY Population DESC LIMIT 5;
+-----------+------------+
| Name      | Population |
+-----------+------------+
| Bombay    |   10500000 |
| Seoul     |    9981619 |
| São Paulo |    9968485 |
| Shanghai  |    9696300 |
| Jakarta   |    9604900 |
+-----------+------------+
5 rows in set (0.34 sec)

mysql> \q
Bye

$>
```

使用 MySQL 的应用程序可以使用标准 API 来访问 [`NDB`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html "第23章 MySQL NDB Cluster 8.0")表。请务必记住，您的应用程序必须访问 SQL 节点，而不是管理或数据节点。这个间断的例子展示了我们如何通过 PHP 5.x mysqli 扩展在网络上其它地方的 web 服务器上运行 select 的语句：

```php
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
  "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
  <meta http-equiv="Content-Type"
           content="text/html; charset=iso-8859-1">
  <title>SIMPLE mysqli SELECT</title>
</head>
<body>
<?php
  # connect to SQL node:
  $link = new mysqli('198.51.100.20', 'root', 'root_password', 'world');
  # parameters for mysqli constructor are:
  #   host, user, password, database

  if( mysqli_connect_errno() )
    die("Connect failed: " . mysqli_connect_error());

  $query = "SELECT Name, Population
            FROM City
            ORDER BY Population DESC
            LIMIT 5";

  # if no errors...
  if( $result = $link->query($query) )
  {
?>
<table border="1" width="40%" cellpadding="4" cellspacing ="1">
  <tbody>
  <tr>
    <th width="10%">City</th>
    <th>Population</th>
  </tr>
<?
    # then display the results...
    while($row = $result->fetch_object())
      printf("<tr>\n  <td align=\"center\">%s</td><td>%d</td>\n</tr>\n",
              $row->Name, $row->Population);
?>
  </tbody
</table>
<?
  # ...and verify the number of rows that were retrieved
    printf("<p>Affected rows: %d</p>\n", $link->affected_rows);
  }
  else
    # otherwise, tell us what went wrong
    echo mysqli_error();

  # free the result set and the mysqli connection object
  $result->close();
  $link->close();
?>
</body>
</html>
```

我们假设运行在 Web 服务器上的进程可以到达 SQL 节点的 IP 地址。

以类似的方式，您可以使用 MySQL C API、Perl-DBI、Python-mysql 或 MySQL 连接器来执行数据定义和操作任务，就像您通常使用 MySQL 一样。
