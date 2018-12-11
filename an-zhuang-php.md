# 安装PHP

## 必要的软件准备

```text
# yum -y install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
```

暂不解释安装这些软件的用途，后续更新。

## 安装PHP

### 下载PHP

 打开/usr/local/src目录，我们把下载的软件放在此目录下

```text
# cd /usr/local/src
```

 下载Nginx，在PHP官网\(php.net\)获取下载链接，我使用的是5.6版本

```text
# wget http://cn2.php.net/distributions/php-5.6.33.tar.gz
```

### 解压PHP

```text
# tar -zxf php-5.6.33.tar.gz
```

### 编译PHP

```text
# cd php-5.6.33/
# ./configure --prefix=/usr/local/php56 --with-config-file-path=/usr/local/php56/etc --enable-fpm --with-fpm-user=www  --with-fpm-group=www --enable-inline-optimization --disable-debug --disable-rpath --enable-shared  --enable-soap --with-libxml-dir --with-xmlrpc --with-openssl --with-mcrypt --with-mhash --with-pcre-regex --with-sqlite3 --with-zlib --enable-bcmath --with-iconv --with-bz2 --enable-calendar --with-curl --with-cdb --enable-dom --enable-exif --enable-fileinfo --enable-filter --with-pcre-dir --enable-ftp --with-gd --with-openssl-dir --with-jpeg-dir --with-png-dir --with-zlib-dir  --with-freetype-dir --enable-gd-native-ttf --enable-gd-jis-conv --with-gettext --with-gmp --with-mhash --enable-json --enable-mbstring --enable-mbregex --enable-mbregex-backtrack --with-libmbfl --with-onig --enable-pdo --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-zlib-dir --with-pdo-sqlite --with-readline --enable-session --enable-shmop --enable-simplexml --enable-sockets  --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-wddx --with-libxml-dir --with-xsl --enable-zip --enable-mysqlnd-compression-support --with-pear --enable-opcache
```

### 安装PHP

```text
# make && make install
```

### 配置PHP

```text
#复制php配置文件到指定目录(根据编译参数决定)
# cp php.ini-production /usr/local/php56/etc/php.ini
#复制php-fpm启动文件到init.d目录
# cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm56
#复制php-fpm配置文件到指定目录
# cp /usr/local/php56/etc/php-fpm.conf.default /usr/local/php56/etc/php-fpm.conf
#修改文件权限
# chmod 755 /etc/init.d/php-fpm56
```

### 启动PHP

```text
# /etc/init.d/php-fpm56 start
```

 通过`netstat -an | grep 9000`, 可以看到 php-fpm 已经在监听 9000 端口了

### 配置Nginx

 安装了 php 想要对外提供web服务，还需要配合nginx ， php-fpm 启动默认监听端口是 9000, 通过 nginx 的 fastcgi 可以将对http请求转发到 php-fpm 上来处理。

 在你的 nginx 某个虚拟主机配置上添加类似这样的代码

```text
location ~ \.php$ {
    root           html;
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  /$document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```

 这部分的配置根据 nginx 的配置情况略有不同，具体指令请查阅 nginx fastcgi 相关。

### 其他注意事项

#### 提示date.timezone错误

在php.ini配置文件找到date.timezone，去掉分号，修改为

```text
date.timezone = PRC
```

保存重启php-fpm

## 如果服务器上已经安装过了php5 怎么办

这个很简单，如果是文件位置冲突，比如 `/etc/init.d/php-fpm` 已经存在了，你可以将 php7 的脚本改名为`/etc/init.d/php-fpm7`。

如果是端口冲突，可以修改 `/usr/local/php/etc/php-fpm.conf` ，将 `listen = 127.0.0.1:9000`, 修改成另一个端口号，比如 `9001`， 再到nginx 配置中把 fastcgi 部分指定的端口也改成 `9001` 的即可。

这样看来，在一个服务器上启动多个版本的 php 都是没问题的，并且可以同时对外提供服务。

## PHP编译参数解释

```text
--prefix 指定安装目录；
--with-config-file-path 指定 php.ini 配置所在的目录；
--with-fpm-user=www 指定程序运行时的非特权用户
--with-fpm-group=www 指定程序运行时的非特权用户组
```

