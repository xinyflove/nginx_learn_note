# Nginx添加到系统服务

## 将 nginx 添加到系统服务，来控制启动与停止

> 使用 nginx 自带的执行文件，来控制启动与停止，也可以编写脚本，做为系统服务来控制 nginx 启动，更加方便和快捷

 nginx 默认的情况下，在 `sbin` 目录下 有一个名为 nginx 的执行文件。其用法，可以使用 `-h` 查看帮助。

```text
[root@instance-vs50oe41 ~]# /usr/local/nginx-1.14.1/sbin/nginx -h
nginx version: nginx/1.14.1
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/nginx-1.14.1/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
```

 此帮助，这是我再简单的提一下：

```text
-h  显示帮助
-v  显示版本
-V  显示版本并且显示编译时的选项及参数
-t  测试配置文件
-s  可以指定信号，可以指定的信号有 stop qiut reopen reload
-p  nginx 路径
-c  指定配置文件，默认为 conf/nginx.conf
```

 一般情况下，可以直接使用 sbin 目录下的 nginx 来控制 nginx 进程：

 启动 nginx

```text
# /usr/local/nginx/sbin/nginx
```

 指定启动时使用的配置文件

```text
# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/cnof/nginx.conf
```

 测试配置文件，检测语法是否有错误，若不 -c 则是测试默认的配置文件

```text
# /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
```

 停止 nginx

```text
# /usr/local/nginx/sbin/nginx -s stop
```

 重新载入配置文件，一般用一更改了配置文件过后，使其生效

```text
/usr/local/nginx/sbin/nginx -s reload
```

 

默认情况下，使用这些也足够，这里提供一下脚本，可以将它加入系统服务，做为系统服务控制，脚本如下：

```text
#! /bin/sh
 
### BEGIN INIT INFO
# Provides:          nginx
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO
 
# Author:   
# website:  
 
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
NAME=nginx
NGINX_BIN=/usr/local/nginx-1.14.1/sbin/$NAME
CONFIGFILE=/usr/local/nginx-1.14.1/conf/$NAME.conf
PIDFILE=/usr/local/nginx-1.14.1/logs/$NAME.pid
 
case "$1" in
    start)
        echo -n "Starting $NAME... "
 
        if netstat -tnpl | grep -q nginx;then
            echo "$NAME (pid `pidof $NAME`) already running."
            exit 1
        fi
 
        $NGINX_BIN -c $CONFIGFILE
 
        if [ "$?" != 0 ] ; then
            echo " failed"
            exit 1
        else
            echo " done"
        fi
        ;;
 
    stop)
        echo -n "Stoping $NAME... "
 
        if ! netstat -tnpl | grep -q nginx; then
            echo "$NAME is not running."
            exit 1
        fi
 
        $NGINX_BIN -s stop
 
        if [ "$?" != 0 ] ; then
            echo " failed. Use force-quit"
            exit 1
        else
            echo " done"
        fi
        ;;
 
    status)
        if netstat -tnpl | grep -q nginx; then
            PID=`pidof nginx`
            echo "$NAME (pid $PID) is running..."
        else
            echo "$NAME is stopped"
            exit 0
        fi
        ;;
 
    force-quit)
        echo -n "Terminating $NAME... "
 
        if ! netstat -tnpl | grep -q nginx; then
            echo "$NAME is not running."
            exit 1
        fi
 
        kill `pidof $NAME`
 
        if [ "$?" != 0 ] ; then
            echo " failed"
            exit 1
        else
            echo " done"
        fi
        ;;
 
    restart)
        $0 stop
        sleep 1
        $0 start
        ;;
 
    reload)
        echo -n "Reload service $NAME... "
 
        if netstat -tnpl | grep -q nginx; then
            $NGINX_BIN -s reload
            echo " done"
        else
            echo "$NAME is not running, can't reload."
            exit 1
        fi
        ;;
 
    configtest)
        echo -n "Test $NAME configure files... "
 
        $NGINX_BIN -t
        ;;
 
    *)
        echo "Usage: $0 {start|stop|force-quit|restart|reload|status|configtest}"
        exit 1
        ;;
esac
```

 在 `/etc/init.d/` 目录下，创建一个文件，名称为 nginx ， 并将以下代码，放到该文件里，然后添加执行权限

```text
# chmod 755 /etc/init.d/nginx
```

 添加到系统服务，并设置开机自动启动。 \(centos redhat 系列\)

```text
# chkconfig --add nginx
# chkconfig --level 2345 nginx on
```

 ubuntu debian 系列，添加自动启动如下

```text
update-rc.d -f nginx defaults
```

 完成后就可以使用如下命令来控制

```text
[root@localhost ~]# /etc/init.d/nginx 
Usage: /etc/init.d/nginx {start|stop|force-quit|restart|reload|status|configtest}
[root@localhost ~]#
```

