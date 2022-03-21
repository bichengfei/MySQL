# 在 Linux 上安装 NDB 集群二进制版本

本节介绍从 Oracle 提供的预编译二进制文件为每种类型的集群节点安装正确可执行文件所需的步骤。

要使用预编译的二进制文件设置集群，每个集群主机安装过程的第一步是从 [NDB Cluster 下载页面](https://dev.mysql.com/downloads/cluster/) 下载二进制文件（对于最新的 64 位 NDB 8.0 版本，这是 `mysql-cluster-gpl-8.0.28-linux-glibc2.12-x86_64.tar.gz`）。我们假设你已将文件放在每台机器的 /var/tmp 目录中。

如果您需要自定义二进制文件，请参阅 [第 2.9.5 节，“使用开发源树安装 MySQL”](https://dev.mysql.com/doc/refman/8.0/en/installing-development-tree.html "2.9.5 使用开发源码树安装 MySQL")。

```textile
笔记
    安装完成后，不要启动任何二进制文件，我们会向你展示如何在节点配置之后执行此操作（请参阅 第 23.3.3 节，“NDB 集群的初始配置”）。
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
   apt-get install libaio1 -y
   ```
   
   此操作会对 MySQL 的 root 账户生成一个随机密码，如果你不希望生成随机密码，你可以使用 `--initialize-insecure`选项代替`--initialize`。在任何一种情况下，你都应该在执行此不走之前查看[第 2.10.1 节“初始化数据目录”以获取更多信息。](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html "2.10.1 初始化数据目录")另见 [第 4.4.2 节，“mysql_secure_installation - 提高 MySQL 安装安全性”](https://dev.mysql.com/doc/refman/8.0/en/mysql-secure-installation.html "4.4.2 mysql_secure_installation——提高 MySQL 安装安全性")。

4. 
