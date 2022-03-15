# 23.1 一般信息

MySQL NDB Cluster 使用 MySQL 服务器和 NDB 存储引擎。对 NDB 存储引擎的支持不包含在 Oracle 指定的 MySQL Server 8.0 二进制文件中。（......此处省略 50 字，主要描述当前发行版信息）

```textile
重要的


MySQL NDB Cluster 不支持 InnoDB Cluster，它必须使用 MySQL Server 8.0 和 InnoDB 存储引擎部署，以及其它没有包含在 NDB 集群分布中的应用程序。
MySQL Server 8.0 二进制文件不能与 MySQL NDB Cluster 一起使用。有关部署和使用 InnoDB Cluster 的更多信息，请参阅 MySQL AdminAPI(https://dev.mysql.com/doc/mysql-shell/8.0/en/admin-api-userguide.html)
在 第 23.2.6 节，“使用 InnoDB 的 MySQL 服务器与 NDB Cluster 比较(https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-compared.html)”，讨论了 NDB 和 InnoDB 存储引擎之间的差距
```

### 支持的平台

NDB Cluster 目前在许多平台上可用并受支持。有关操作系统版本、操作系统发行版和硬件平台的特定组合可用的确切支持级别，请参阅 [MySQL :: Supported Platforms: MySQL Cluster](https://www.mysql.com/support/supportedplatforms/cluster.html)

### 可用性

NDB Cluster 二进制和源码包获取地址 [MySQL :: Download MySQL Cluster](https://dev.mysql.com/downloads/cluster/)

### NDB Cluster 版本号

从 MySQL 8.0.13 和 MySQL NDB Cluster 8.0.13 开始，NDB 8.0 遵循与 MySQL Server 8.0 系列版本相同的发布模式。在本手册和其他 MySQL 文档中，我们使用以“NDB”开头的版本号来标识这些和更高版本的 NDB Cluster 版本。此版本号是 NDB 8.0 版本使用的 NDB 存储引擎版本，同样也与  NDB Cluster 8.0 版本所基于的 MySQL 8.0 服务端版本相同。

### NDB  Cluster 软件中使用的版本字符串

MySQl NDB Cluster 发行版提供的 MySQL 客户端显示的版本字符串使用以下格式：

```sql
mysql-mysql_server_version-cluster
```

mysql_server_version 表示 NDB Cluster 版本所基于的 MySQL 服务端的版本。对于所有的 NDB Cluster 8.0 版本，version 是 8.0.n，n 是其中的版本号。从源码构建使用`-DWITH NDBCLUSTER`或者等效地将集群后缀添加到版本字符串(请参阅 [第 23.3.1.4 节，“在 Linux 上从源代码构建 NDB 集群”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-linux-source.html "23.3.1.4 在 Linux 上从源代码构建 NDB 集群")和 [第 23.3.2.2 节，“在 Windows 上从源代码编译和安装 NDB 集群”](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-windows-source.html "23.3.2.2 Compiling and Installing NDB Cluster from Source on Windows"))。你可以在 MySQL 客户端中看到这种格式，如下所示：

```sql
$> mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 8.0.29-cluster Source distribution

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> SELECT VERSION()\G
*************************** 1. row ***************************
VERSION(): 8.0.29-cluster
1 row in set (0.00 sec)
```

使用 MySQL 8.0 的 NDB Cluster 的第一个通用版本是 NDB 8.0.19，使用 MySQL 8.0.19.

不包含在 MySQL 8.0 发行版中的其他 NDB 集群，显示以下格式的版本号

```sql
mysql-mysql_server_version ndb-ndb_engine_version
```

mysql_server_version 表示 NDB 版本所基于的 MySQL 服务端的版本。对于所有的 NDB Cluster 8.0 版本，version 是 8.0.n，n 是版本号。

ncb_engine_version 是该 NDB 集群中使用的 NDB 存储引擎版本。对于所有的 NDB 8.0 版本，此数字与 MySQL 服务端版本相同。你可以在 ndb_mgm 客户端的命令输出中看到这种格式，如下所示：

```sql
ndb_mgm> SHOW
Connected to Management Server at: localhost:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=1    @10.0.10.6  (mysql-8.0.29 ndb-8.0.29, Nodegroup: 0, *)
id=2    @10.0.10.8  (mysql-8.0.29 ndb-8.0.29, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=3    @10.0.10.2  (mysql-8.0.29 ndb-8.0.29)

[mysqld(API)]   2 node(s)
id=4    @10.0.10.10  (mysql-8.0.29 ndb-8.0.29)
id=5 (not connected, accepting connect from any host)
```

### 与标准 MySQL 8.0 版本的兼容性

虽然许多标准 MySQL 模式和应用程序可以使用 NDB 集群运行，但未修改的应用程序和数据库模式在使用 NDB 集群运行时可能会稍微不兼容或性能欠佳（请参阅 [第 23.2.7 节，“NDB Cluster 的已知限制”）](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations.html "23.2.7 NDB Cluster 的已知限制"）。这些问题中的大多数都可以克服，但这也意味着你不太可能切换现有但应用程序数据存储，例如，MyISAM 或者 InnoDB 在不考虑模式、查询、请求更改的情况下使用 NDB 存储引擎。一个不支持 NDB 的 mysqld 编译后（构建时未使用`-DWITH_NDBCLUSTER_STORAGE_ENGINE`），无法替代使用 NDB 构建的 mysqld。

### NDB Cluster 开发源码树

NDB Cluster 开发源码可以从 https://github.com/mysql/mysql-server 访问，使用 GPL 协议。有关使用 Git 获取 MySQL 源代码并自己构建它们的信息，请参阅 [第 2.9.5 节，“使用开发源树安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/installing-development-tree.html "2.9.5 使用开发源码树安装 MySQL")。

```textile
笔记与 MySQL Server 8.0 一样，NDB Cluster 版本是使用 CMake 构建的
```

 NDB Cluster 8.0 从 NDB 8.0.19 开始作为通用版本发布，建议用于新部署。NDB Cluster 7.5 和 7.6 是以前的 GA 版本，仍然支持生产。

有关 NDB Cluster 7.5 的信息，请参考 [NDB Cluster 7.5 中的新增功能](https://dev.mysql.com/doc/refman/5.7/en/mysql-cluster-what-is-new-7-5.html)

有关 NDB Cluster 7.6 的信息，请参考 [NDB Cluster 7.6中的新增功能](https://dev.mysql.com/doc/refman/5.7/en/mysql-cluster-what-is-new-7-6.html)

有关 NDB Cluster 7.3 和 7.4 的信息，请参考 [MySQL NDB Cluster 7.3 和 NDB Cluster 7.4](https://dev.mysql.com/doc/refman/5.6/en/mysql-cluster.html)

随着 NDB Cluster 的不断发展，本章的内容可能会进行修订。有关 NDB Cluster 的其他信息可以在 MySQL 网站 [MySQL :: MySQL Cluster CGE](http://www.mysql.com/products/cluster/)上找到。

### 其他资源

有关 NDB Cluster 的更过信息可以在以下未知找到：

+ 有关 NDB Cluster 的一些常见问题的答案，请参阅 [第 A.10 节，“MySQL 8.0 FAQ：NDB Cluster”](https://dev.mysql.com/doc/refman/8.0/en/faqs-mysql-cluster.html "A.10 MySQL 8.0 常见问题解答：NDB 集群")。

+ NDB Cluster 论坛：https://forums.mysql.com/list.php?25

+ 许多 NDB Cluster 用户可开发人员在博客上讲述了它们使用 NDB Cluster 的经历，并通过 [PlanetMySQL](http://www.planetmysql.org/)提供这些信息
