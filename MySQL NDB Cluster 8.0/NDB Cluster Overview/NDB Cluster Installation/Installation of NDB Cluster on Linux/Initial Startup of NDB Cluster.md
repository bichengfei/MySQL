# 23.3.4 NDB 集群的初始启动

配置好后启动集群并不是很困难。每个集群节点进程必须单独启动，并且在它所在的主机上启动。管理节点应该首先启动，然后是数据节点，最后是任何 SQL 节点：

1. 在管理主机上，直接从 shell (任何目录都可以)发出以下命令以启动管理节点进程：
   
   ```shell
   $> ndb_mgmd --initial -f /var/lib/mysql-cluster/config.ini
   ```
   
   *注意：笔者这里有报以下错误，手动在 /usr/local 下创建 mysql 文件夹即可*
   
   ```shell
   root@dc544097c07c:/usr/local/bin# ndb_mgmd --initial -f /var/lib/mysql-cluster/config.ini
   MySQL Cluster Management Server mysql-8.0.28 ndb-8.0.28
   2022-03-22 11:27:33 [MgmtSrvr] INFO     -- The default config directory '/usr/local/mysql/mysql-cluster' does not exist. Trying to create it...
   2022-03-22 11:27:33 [MgmtSrvr] INFO     -- Failed to create directory '/usr/local/mysql/mysql-cluster', error: 2
   2022-03-22 11:27:33 [MgmtSrvr] ERROR    -- Could not create directory '/usr/local/mysql/mysql-cluster'. Either create it manually or specify a different directory with --configdir=<path>
   root@dc544097c07c:/usr/local/bin#
   ```

2. 在每个数据节点主机上，运行以下命令来启动 [**ndbd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndbd.html "23.5.1 ndbd — NDB 集群数据节点守护进程")进程：
   
   ```shell
   $> ndbd
   ```
   
   *注意：笔者这里有报以下错误，手动在 /usr/local 下创建 mysql/data 文件夹即可*
   
   ```shell
   root@24dd50e1f26c:/usr/local# ndbd
   2022-03-22 11:35:00 [ndbd] INFO     -- Angel connected to '172.17.0.5:1186'
   2022-03-22 11:35:09 [ndbd] INFO     -- Angel allocated nodeid: 2
   2022-03-22 11:35:09 [ndbd] WARNING  -- Cannot change directory to '/usr/local/mysql/data', error: 2
   2022-03-22 11:35:09 [ndbd] ERROR    -- Couldn't start as daemon, error: 'Failed to open logfile '/usr/local/mysql/data/ndb_2_out.log' for write, errno: 2'
   root@24dd50e1f26c:/usr/local#
   ```

3. 如果您使用 RPM 文件在 SQL 节点所在的集群主机上安装 MySQL，您可以（并且应该）使用提供的启动脚本在 SQL 节点上启动 MySQL 服务器进程。（*笔者使用的 .tar.gz，启动方式为`/usr/local/mysql/support-files/myser.server start`*）

如果一切顺利，并且集群设置正确，那么集群现在应该可以运行了。您可以通过调用[**ndb_mgm**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html "23.5.5 ndb_mgm — NDB 集群管理客户端")管理节点客户端对此进行测试。输出应该看起来像这里显示的那样，尽管您可能会看到输出中的一些细微差异，具体取决于您使用的 MySQL 的确切版本：

```shell
$> ndb_mgm
-- NDB Cluster -- Management Client --
ndb_mgm> SHOW
Connected to Management Server at: localhost:1186
Cluster Configuration
---------------------
[ndbd(NDB)]     2 node(s)
id=2    @198.51.100.30  (Version: 8.0.29-ndb-8.0.29, Nodegroup: 0, *)
id=3    @198.51.100.40  (Version: 8.0.29-ndb-8.0.29, Nodegroup: 0)

[ndb_mgmd(MGM)] 1 node(s)
id=1    @198.51.100.10  (Version: 8.0.29-ndb-8.0.29)

[mysqld(API)]   1 node(s)
id=4    @198.51.100.20  (Version: 8.0.29-ndb-8.0.29)
```

注意：不知道是不是笔者本机内存不够的原因，导致了两个数据节点，同时只能启动一个，参考地址 [MySQL :: Forced node shutdown completed. Occured during startphase 0. Initiated by signal 9.](https://forums.mysql.com/read.php?25,506632,506632#msg-506632)

SQL 节点在此处引用为 `[mysqld(API)]`，这反映了 [**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html "4.3.1 mysqld——MySQL 服务器")进程充当 NDB Cluster API 节点的事实。

```textile
笔记
    在 SHOW 的输出中为给定的 NDB Cluster SQL 或其他 API 节点显示的 IP 地址是 SQL 或 API 节点用于连接到集群数据节点而不是任何管理节点的地址。
```

您现在应该准备好使用 NDB Cluster 中的数据库、表和数据。有关简要讨论， 请参阅 [第 23.3.5 节，“带有表和数据的 NDB 集群示例” 。](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-example-data.html "23.3.5 带有表和数据的 NDB Cluster 示例")
