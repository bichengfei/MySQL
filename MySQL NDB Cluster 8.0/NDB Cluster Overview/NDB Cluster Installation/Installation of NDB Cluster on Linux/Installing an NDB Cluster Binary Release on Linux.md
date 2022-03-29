# 在 Linux 上安装 NDB 集群二进制版本

本节介绍从 Oracle 提供的预编译二进制文件为每种类型的集群节点安装正确可执行文件所需的步骤。

要使用预编译的二进制文件设置集群，每个集群主机安装过程的第一步是从 [NDB Cluster 下载页面](https://dev.mysql.com/downloads/cluster/) 下载二进制文件（对于最新的 64 位 NDB 8.0 版本，这是 `mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz`）。我们假设你已将文件放在每台机器的 /var/tmp 目录中。

如果您需要自定义二进制文件，请参阅 [第 2.9.5 节，“使用开发源树安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/installing-development-tree.html "2.9.5 使用开发源码树安装 MySQL")。

```textile
笔记
    安装完成后，不要启动任何二进制文件，我们会向你展示如何在节点配置之后执行此操作（请参阅 第 23.3.3 节，“NDB 集群的初始配置”）。
```

```shell
apt-get update
apt-get install sudo -y
apt install net-tools
```



### SQL 节点

以系统用户 root 的身份，在指定托管 SQL 节点的每台机器上执行以下步骤；

1. 检查你的 /etc /passwd 和 /etc/group 文件（或使用操作系统提供的任何工具来管理用户和组）以查看系统上是否已经存在 mysql 组和 mysql 用户。一些操作系统发行版在操作系统安装过程中创建这些。如果它们尚不存在，请创建一个新的 mysql 用户组，然后将 mysql 用户添加到该组：
   
   ```shell
   $> groupadd mysql
   $> useradd -g mysql -s /bin/false mysql
   ```
   
   useradd 和 groupadd 的语法在不同版本的 Unix 可能略有不同，或者它们可能有不同的名称，例如 adduser 和 addgroup.

2. 将位置更改为包含下载文件的目录，解压缩文件，并创建一个名为 mysql 的符号引用
   
   ```shell
   $> cd /var/tmp
   $> tar -C /usr/local -xzvf mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz
   $> ln -s /usr/local/mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64 /usr/local/mysql
   ```

3. 将位置更改为 mysql 目录，并使用 `mysqld --initialize` 设置系统数据库
   
   ```shell
   $> cd /usr/local/mysql
   $> /bin/mysqld --initialize
   ```
   
   注意：笔者使用 ubuntu:18.04 的 docker 版本，遇到以下报错：
   
   ```shell
   mysqld: error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory
   ```
   
   最终通过以下命令解决（执行了很多命令，不知道哪个生效了）：
   
   参考: [mysql - Error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory - Stack Overflow](https://stackoverflow.com/questions/57833459/error-while-loading-shared-libraries-libnuma-so-1-cannot-open-shared-object-fi)
   
   ```shell
   sudo apt-get install libnuma-dev
   apt-get install libaio1 libaio-dev
   ```
   
   此操作会对 MySQL 的 root 账户生成一个随机密码，如果你不希望生成随机密码，你可以使用 `--initialize-insecure`选项代替`--initialize`。在任何一种情况下，你都应该在执行此不走之前查看[第 2.10.1 节“初始化数据目录”以获取更多信息。](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html "2.10.1 初始化数据目录")另见 [第 4.4.2 节，“mysql_secure_installation - 提高 MySQL 安装安全性”](https://dev.mysql.com/doc/refman/8.0/en/mysql-secure-installation.html "4.4.2 mysql_secure_installation——提高 MySQL 安装安全性")。

4. 为 MySQL 服务器和数据目录设置必要的权限：
   
   ```shell
   $> chown -R root .
   $> chown -R mysql data
   $> chgrp -R mysql .
   ```

5. 将 MySQL 启动脚本复制到相应目录，使其可执行，并将其设置为在操作系统启动时启动：
   
   ```shell
   $> cp support-files/mysql.server /etc/rc.d/init.d/
   $> chmod +x /etc/rc.d/init.d/mysql.server
   $> chkconfig --add mysql.server
   ```
   
   （启动脚本目录可能因您的操作系统和版本而异——例如，在某些 Linux 发行版中，它是 `/etc/init.d`.）
   
   这里我们使用 Red Hat 的**chkconfig**来创建启动脚本的链接；在您的平台上使用适合此目的的任何方法，例如 Debian 上的**update-rc.d**。

请记住，必须在 SQL 节点所在的每台机器上重复上述步骤。

### 数据节点

数据节点的安装不需要  mysqld 二进制文件，仅需要  NDB  集群数据节点可执行的 ndbd（单线程）或 ndbmtd（多线程）。这些二进制文件也可以在 .tar.gz 存档中找到，同样，我们假设你已经将此文档放置在 /var/tmp.

使用 root 用户，执行以下步骤以在数据节点主机上安装数据节点二进制文件：

1. 将位置更改为 /var/tmp 目录，并将 ndbd 和 ndbmtd 二进制文件从存档中提取到合适到目录中，例如： /usr/local/bin
   
   ```shell
   $> cd /var/tmp
   $> tar -zxvf mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz
   $> cd mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64
   $> cp bin/ndbd /usr/local/bin/ndbd
   $> cp bin/ndbmtd /usr/local/bin/ndbmtd
   ```
   
   `/var/tmp`（一旦 [**ndb_mgm**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html "23.5.5 ndb_mgm — NDB 集群管理客户端")和[**ndb_mgmd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html "23.5.4 ndb_mgmd — NDB 集群管理服务器守护进程") 被复制到可执行文件目录 ，您就可以安全地删除通过解压下载的存档创建的目录及其包含 的文件。）

2. 将位置更改为你将文件复制到的目录，然后使他们都可以执行
   
   ```shell
   $> cd /usr/local/bin
   $> chmod +x ndb*
   ```

应在每个数据节点主机上重复上述步骤。

尽管运行 NDB 集群数据节点只需要一个数据节点执行文件，但我们在前面的说明中向你展示了如何安装 ndbd 和 ndbmtd。我们建议你在安装或升级 NDB 集群时执行此操作，即使你打算只使用其中一个，因为这样可以在你以后决定从一个更改为另一个时节省时间和麻烦。

```textile
笔记
    托管数据节点的每台机器上的数据目录时 /usr/local/mysql/data。在配置管理节点时，这条信息是必不可少。（请参阅第 23.3.3 节，“NDB 集群的初始配置”。））
```

### 管理节点

管理节点的安装不需要 mysqld 二进制文件。只需要 NDB Cluster 管理服务器ndb_mgmd；您很可能还想安装管理客户端 ndb_mgm。这两个二进制文件也可以在`.tar.gz`存档中找到。同样，我们假设您已将此存档放置在 `/var/tmp`.

作为 system `root`，执行以下步骤在管理节点主机上 安装[**ndb_mgmd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html "23.5.4 ndb_mgmd — NDB 集群管理服务器守护进程")和 [**ndb_mgm ：**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html "23.5.5 ndb_mgm — NDB 集群管理客户端")

1. 将位置更改为`/var/tmp` 目录，并将[**ndb_mgm**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html "23.5.5 ndb_mgm — NDB 集群管理客户端")和 [**ndb_mgmd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html "23.5.4 ndb_mgmd — NDB 集群管理服务器守护进程")从存档中提取到合适的目录中，例如`/usr/local/bin`：
   
   ```shell
   $> cd /var/tmp
   $> tar -zxvf mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz
   $> cd mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64
   $> cp bin/ndb_mgm* /usr/local/bin
   ```
   
   `/var/tmp`（一旦 [**ndb_mgm**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgm.html "23.5.5 ndb_mgm — NDB 集群管理客户端")和[**ndb_mgmd**](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-programs-ndb-mgmd.html "23.5.4 ndb_mgmd — NDB 集群管理服务器守护进程") 被复制到可执行文件目录 ，您就可以安全地删除通过解压下载的存档创建的目录及其包含 的文件。）

2. 将位置更改为您将文件复制到的目录，然后使它们都可执行：
   
   ```shell
   $> cd /usr/local/bin
   $> chmod +x ndb_mgm*
   ```

在[第 23.3.3 节，“NDB 集群的初始配置”中](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-install-configuration.html "23.3.3 NDB Cluster 的初始配置")，我们为示例 NDB 集群中的所有节点创建配置文件。
