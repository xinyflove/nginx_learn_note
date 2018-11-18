# 安装MySQL

## 环境简介

Linux：CentOS7

MySQL：MySQL5.5.62

> Mysql从5.5以后安装都需要cmake，因为5.5以后的版本里面没有configure，没法通过./configure 进行安装配置，需要自己去生成配置信息。
>
> 注：本来是打算安装5.7的版本，但是老是内存不足安装失败。原来MySQL 5.6,5.7及MariaDB 10必须在1G以上内存的更高配置上安装，目前我使用的服务器内存是1G，所以只能安装5.5版本。

## 安装前期准备

### 检查并卸载掉原有MySQL

使用如下命令查看该操作系统上是否已经安装了mysql数据库

```text
# rpm -qa | grep mysql
```

有的话，我们就通过 rpm -e 命令 或者 rpm -e --nodeps 命令来卸载掉

```text
# rpm -e mysql　　         #普通删除模式
# rpm -e --nodeps mysql　　#强力删除模式，如果使用上面命令删除时，提示有依赖的其它文件，则用该命令可以对其进行强力删除在删除完以后我们可以通过 rpm -qa | grep mysql 命令来查看mysql是否已经卸载成功！
```

### 设置MySQL用户和组

创建组：`# groupadd mysql`   
创建用户:`# useradd -r -g mysql mysql`

### **创建mysql文件夹和数据存放文件夹data**

```text
# mkdir -p /usr/local/mysql/data
```

###  安装编译源码所需的工具和库

```text
# yum install -y gcc gcc-c++ ncurses-devel perl
```

###  安装cmake

因为前面说过，要使用cmake生成配置信息。用yum安装，也可源码安装。

```text
# yum install -y cmake
```

###  创建boost文件（5.7版本以上需要）

创建boost文件夹并下载boost的tar.gz文件放在此目录下， 否则cmake会报错，本示例中使用的是boost\_1\_59\_0.tar.gz，因为网络下载有点慢，可以用ftp传上去。

如果不确定使用哪个版本的boost，也可以只创建目录，不下载boost文件，在cmake时候，参数 -`DWITH_BOOST=/usr/local/boost` ，会自动下载对应的版本boost文件。

```text
# mkdir /usr/local/boost
```

### 下载mysql

在mysql官网找到合适的版本下载，我使用的是5.7版本，放在/usr/local/src/目录下

```text
# cd /usr/local/src
# wget https://dev.mysql.com/get/Downloads/MySQL-5.5/mysql-5.5.62.tar.gz
```

如果下载失败，可以先下载下来，用ftp软件传到服务器上。

## 安装MySQL

### 解压并生成信息配置

```text
# cd usr/local/src/
# tar -zxf mysql-5.5.62.tar.gz
# cd mysql-5.5.62

# cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \
-DSYSCONFDIR=/etc \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DMYSQL_TCP_PORT=3306 \
-DENABLE_DOWNLOADS=1
```

{% hint style="danger" %}
注：重新运行配置，需要删除CMakeCache.txt文件
{% endhint %}

如果是5.7版本的需要引入boost，需要添加如下配置

```text
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/usr/local/boost 
```

### 编译和安装

配置成功后进行安装

```text
# make && make install
```

## 配置MySQL

### 修改mysql目录所有者和组

修改mysql安装目录

```text
# cd /usr/local/mysql   
# chown -R mysql:mysql .        #注意后面的小数点，表示当前目录，不能省略。
```

因为我把mysql数据库文件目录放在了安装目录，所以不用在修改mysql数据库文件目录了。

###  **初始化mysql数据库**

```text
# cd /usr/local/mysql   
# scripts/mysql_install_db --user=mysql --datadir=/usr/local/mysql/data
```

###   修改my.cnf配置文件

```text
# vi /etc/my.cnf
```

修改前

```text
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

修改后

```text
[mysqld]
datadir=/usr/local/mysql/data   #安装时定义的数据存放目录
socket=/usr/local/mysql/mysql.sock      #安装时定义的监听套接字路径
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

###  **复制mysql服务启动脚本到 service服务中**

```text
# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
```

### 启动mysql服务

此时我们看看安装的mysql是否能够启动成功

```text
# service mysqld start
#结果如下
Starting MySQL..                                           [  OK  ]
```

说明我们安装成功了。

###  添加mysql自启动

```text
# chkconfig mysqld on
```

###  配置环境变量

添加到环境变量中后，可以直接使用mysql命令， 修改/etc/profile文件

```text
# vi /etc/profile
```

在文件末尾添加

```text
PATH=/usr/local/mysql/bin:$PATH
export PATH
```

 保存关闭文件，运行下面的命令，让配置立即生效

```text
# source /etc/profile
```

### 修改mysql的root密码

```text
# mysql -uroot  
mysql> SET PASSWORD = password('root');
mysql>flush privileges;  #刷新系统授权表
mysql>quit;    #退出
```

### 重新登录mysql

```text
# mysql -uroot -p
输入密码 root
```

如果登录成功，说明安装完成了，尽情的使用吧！

## cmake参数说明

| 参数 | 说明 |
| :--- | :--- |
|  -DCMAKE\_INSTALL\_PREFIX=dir\_name |  设置mysql安装目录 |
|  -DMYSQL\_UNIX\_ADDR=file\_name |  设置监听套接字路径，这必须是一个绝对路径名。默认为/tmp/mysql.sock |
| -DSYSCONFDIR | 设置mysql配置文件目录 |
|  -DMYSQL\_DATADIR=dir\_name |  设置mysql数据库文件目录 |
|  -DDEFAULT\_CHARSET=charset\_name |  设置服务器的字符集。 缺省情况下，MySQL使用latin1的（CP1252西欧）字符集。cmake/character\_sets.cmake文件包含允许的字符集名称列表。 |
|  -DDEFAULT\_COLLATION=collation\_name |   校验字符 |
|  -DWITH\_INNOBASE\_STORAGE\_ENGINE=1 |  安装 innodb 存储引擎 |
|  -DWITH\_ARCHIVE\_STORAGE\_ENGINE=1 | 安装 archive 存储引擎 |
|  -DWITH\_BLACKHOLE\_STORAGE\_ENGINE=1 | 安装 blackhole 存储引擎 |
|  -DWITH\_PERFSCHEMA\_STORAGE\_ENGINE=1 |   |
|  -DMYSQL\_TCP\_PORT=port\_num |  设置mysql服务器监听端口，默认为3306 |
|  -DENABLE\_DOWNLOADS=bool |  是否要下载可选的文件。例如，启用此选项（设置为1），cmake将下载谷歌所使用的测试套件运行单元测试。 |
| -DWITH\_MYISAM\_STORAGE\_ENGINE=1 | 安装 myisam 存储引擎 |
| -DWITH\_PARTITION\_STORAGE\_ENGINE=1 | 安装数据库分区 |
| -DENABLED\_LOCAL\_INFILE=1 | 允许从本地导入数据 |
| -DWITH\_READLINE=1 | 快捷键功能 |
| -DWITH\_SSL=yes | 支持 SSL |
| -DEXTRA\_CHARSETS=all | 安装所有扩展字符集 |

 **存储引擎的选择（官网文档，我也不是很明白哈）↓**

存储引擎是作为插件构建的。您可以将插件构建为静态模块\(编译到服务器中\)或动态模块\(构建为动态库，在使用之前必须使用 INSTALL PLUGIN 语句或——plugin-load选项将其安装到服务器中\)。一些插件可能不支持静态或动态构建。 InnoDB、MyISAM、MERGE、MEMORY和CSV引擎是必需的\(总是编译到服务器中\)，不需要显式安装。 要静态地将存储引擎编译到服务器，请使用-DWITH\_engine\_STORAGE\_ENGINE=1。一些允许的引engine是ARCHIVE、BLACKHOLE、EXAMPLE、FEDERATED、NDB或NDBCLUSTER \(NDB\)、PARTITION\(分区支持\)和PERFSCHEMA\(性能模式\)。例子: 

```text
-DWITH_ARCHIVE_STORAGE_ENGINE=1
-DWITH_BLACKHOLE_STORAGE_ENGINE=1
-DWITH_PERFSCHEMA_STORAGE_ENGINE=1
```

 要从构建中排除存储引擎，请使用-DWITHOUT\_engine\_STORAGE\_ENGINE=1。例子:

```text
-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1
-DWITHOUT_FEDERATED_STORAGE_ENGINE=1
-DWITHOUT_PARTITION_STORAGE_ENGINE=1
```

