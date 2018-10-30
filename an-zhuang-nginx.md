# 安装Nginx

## 必要的软件准备

### 安装pcre

 如过已经装了，请跳这一步。

```text
# yum install pcre pcre-devel -y
```

或者使用源码安装

```text
# cd /usr/local/src
# wget -O pcre-8.31.tar.gz http://sourceforge.net/projects/pcre/files/pcre/8.31/pcre-8.31.tar.gz/download
# tar xf pcre-8.31.tar.gz
# cd pcre-8.31
# ./configure
# make && make install
```

PCRE全称Perl Compatible Regular Expressions，中文Perl兼容正则表达式，官方站点为http://www.pcre.org/，安装pcre库是为了使Nginx支持HTTP Rewrite模块。

### 安装openssl

 需要ssl的支持，如果不需要ssl支持，请跳过这一步。

```text
# yum install openssl openssl-devel -y
```

## 安装Nginx

### 添加www用户组和用户

添加www用户组和用户且不允许登录

```text
# useradd www -M -s /sbin/nologin
```

整理中......

