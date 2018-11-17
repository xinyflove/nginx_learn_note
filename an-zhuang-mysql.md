# 安装MySQL

## 环境简介

Linux：CentOS7

MySQL：MySQL5.7.24

> Mysql5.7安装与mySQL5.1安装还是有一些区别了，需要使用cmake进行安装信息的配置。据说Mysql从5.5以后安装都需要cmake，因为5.5以后的版本里面没有configure，没法通过./configure 进行安装配置，需要自己去生成配置信息。

## 安装前期准备

### 创建mysql用户和mysql组

创建组：`# groupadd mysql`   
创建用户:`# useradd -r -g mysql mysql`

### **创建mysql文件夹于数据存放文件夹data**

```text
# mkdir -p /usr/local/mysql/data
```

###  预先安装依赖软件

因为前面说过，要使用cmake生成配置信息。用yum安装，也可源码安装。

```text
# yum install -y cmake gcc-c++ ncurses-devel perl-Data-Dumper boost boost-doc boost-devel
```

###  创建boost文件

创建boost文件夹并下载boost的tar.gz文件放在此目录下， 否则cmake会报错，本示例中使用的是boost\_1\_59\_0.tar.gz，因为网络下载有点慢，可以用ftp传上去。

如果不确定使用哪个版本的boost，也可以只创建目录，不下载boost文件，在cmake时候，参数 -`DWITH_BOOST=/usr/local/boost` ，会自动下载对应的版本boost文件。

```text
# mkdir /usr/local/boost
```

### 下载mysql

在mysql官网找到合适的版本下载，我使用的是5.7版本，放在/usr/local/src/目录下

```text
# cd /usr/local/src
# wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24.tar.gz
```

## 安装MySQL

### 生成信息配置

```text
# cd usr/local/src/
# tar -zxf mysql-5.7.24.tar.gz
# cd mysql-5.7.24

# cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DSYSCONFDIR=/usr/local/mysql/etc \
-DMYSQL_USER=mysql \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DENABLE_DOWNLOADS=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_DEBUG=0 \
-DMYSQL_MAINTAINER_MODE=0 \
-DWITH_SSL:STRING=bundled \
-DWITH_ZLIB:STRING=bundled \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/usr/local/boost
```

### 安装

配置成功后进行安装

```text
# make && make install
```



