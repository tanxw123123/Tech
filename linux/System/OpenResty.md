# openresty

## 1. OpenResty安装

1.下载安装包: https://openresty.org/en/download.html

2.安装依赖包:

```shell
$ yum -y install pcre pcre-devel openssl openssl-devel gcc gcc-c++ zlib zlib-devel postgresql-devel 
```

3.编译&安装:

```shell
$ mkdir /data/software && cd /data/software
$ wget https://openresty.org/download/openresty-1.11.2.2.tar.gz 
$ tar -xf openresty-1.11.2.2.tar.gz && cd openresty-1.11.2.2

$ ./configure --prefix=/usr/local/openresty   --with-luajit   --with-http_iconv_module   --with-http_postgres_module   --with-http_stub_status_module   --with-stream   --with-stream_ssl_module 
$ make && make install
$ ln -s   /usr/local/openresty/nginx/sbin/nginx  /usr/bin/nginx

```

4.修改配置文件：

配置4层转发并开启日志

```shell
$ vim /usr/local/openresty/nginx/conf/nginx.conf

stream {
    log_format proxy '$remote_addr [$time_local] '
                 '$protocol $status $bytes_sent $bytes_received '
                 '$session_time "$upstream_addr" '
                 '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';
    access_log /usr/local/openresty/nginx/logs/tcp-access.log proxy ;
    open_log_file_cache off;
    # include /etc/nginx/conf.d/*.stream;

    upstream ssh-proxy {
        server 192.168.254.203:22;
    }

    server {
        listen 8019;
        proxy_pass ssh-proxy;
    }
}
```

查看配置文件：

```shell
$ nginx -t   有报错
```

测试发现nginx会等待session结束才会记录到日志文件；

原因分析：  nginx自1.9.0开始提供tcp/udp的反向代理功能，直到1.11.4才开始提供session日志功能。

---



## 2. OpenResty 平滑升级

下载高版本：

```shell
$ wget https://openresty.org/download/openresty-1.13.6.1.tar.gz
```

开始升级：

```shell
$ tar -xf openresty-1.13.6.1.tar.gz
$ cd openresty-1.13.6.1
$ ./configure --prefix=/usr/local/openresty   --with-luajit   --with-http_iconv_module   --with-http_postgres_module   --with-http_stub_status_module   --with-stream   --with-stream_ssl_module

$ make 
$ mv /usr/local/openresty/nginx/sbin/nginx /usr/local/openresty/nginx/sbin/nginx.old
$ cd /data/software/openresty-1.13.6.1/build/nginx-1.13.6/objs/
$ cp nginx /usr/local/openresty/nginx/sbin/
$ nginx -s reload
$ nginx -V

```

