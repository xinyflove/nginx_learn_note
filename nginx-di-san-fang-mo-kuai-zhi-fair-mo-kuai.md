Nginx第三方模块之fair模块
https://yq.aliyun.com/articles/73621
模块介绍
The Nginx fair proxy balancer enhances the standard round-robin load balancer provided
with Nginx so that it will track busy back end servers (e.g. Thin, Ebb, Mongrel)
and balance the load to non-busy server processes.

简单翻译一下，fair采用的不是内建负载均衡使用的轮换的均衡算法，而是可以根据页面大小、加载时间长短智能的进行负载均衡。

下载地址：nginx-upstream-fair https://github.com/gnosek/nginx-upstream-fair?spm=a2c4e.11153940.blogcont73621.10.752155b9ugIZJD

解压：

unzip  nginx-upstream-fair-master.zip
模块安装
未安装Nginx
切换到Nginx目录执行一下操作
配置：

./configure --prefix=/usr/local/nginx  --sbin-path=/usr/local/nginx/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid  --add-module=/home/nginx-upstream-fair-master
编译安装

make && make intstall
安装过Nginx
切换到Nginx目录执行一下操作

配置

./configure --prefix=/usr/local/nginx  --sbin-path=/usr/local/nginx/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid  --add-module=/home/nginx-upstream-fair-master
编译

make
复制Nginx

 cp objs/nginx /usr/local/nginx/nginx
配置实现
upstream backserver { 
fair; 
server 192.168.0.14; 
server 192.168.0.15; 
} 
注意事项
已安装Nginx，配置第三方模块时，只需要--add-module=/第三方模块目录，然后make编译一下就可以，不要 make install 安装。编译后复制objs下面的Nginx到指定目录下。

配置中path自行定义即可。
