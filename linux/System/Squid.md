## 1.安装

```
$ yum install squid -y
```

## 2.配置

参考配置文件：

```shell
$ vim /etc/squid/squid.conf
logformat   %>a %ui %un [%tl] "%rm %ru HTTP/%rv" %tr %>Hs %<st "%{Referer}>h" "%{User-Agent}>h" %Ss:%Sh
access_log /var/log/squid/access.log combined

#acl noTransactionError src 54.238.69.12/32  13.115.144.153/32
#access_log /var/log/squid/access.log combined noTransactionError

#2.whitelist
acl whitelist src   46.51.236.48/32

http_access allow whitelist
http_access deny  all

http_port 38090
http_port 33090

#　　<cache_dir>; <aufs|ufs>; <目录所在>; <MBytes大小>; <dir1>; <dir2>; 
#　　而后面 dir1, dir2 则是两个次目录的大小，通常 16 256 或 64 64 皆可
cache_dir ufs /var/spool/squid 1024 16 256
#coredump_dir /var/spool/squid
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern (Release|Packages(.gz)*)$      0       20%     2880
refresh_pattern .               0       20%     4320

via off
forwarded_for delete

vary_ignore_expire on
maximum_object_size_in_memory 512 KB  #在内存中单个文件最大缓存大小，超过这个大小将不缓存到内存中
half_closed_clients off
maximum_object_size 65536 KB  #单个文件最大缓存大小，超过这个大小将不缓存
minimum_object_size 0 KB

dns_v4_first on
```

1.允许使用代理的IP地址

```shell
$ vim /etc/squid/squid.conf
# 此处配置10.x.x.x IP段均可使用代理（x代表0~255的一个数字）
acl whitelist src 10.0.0.0/8

# 如果限定单个IP使用，则配置为10.0.0.1/32
acl whitelist src 10.0.0.1/32
```

2.

## 3.使用代理

1.Yum代理

```shell
# 修改 /etc/yum.conf
proxy=http://10.20.11.12:3128
```

2.wget代理

```shell
# 修改 /etc/wgetrc
http_proxy=http://10.20.11.12:3128
https_proxy=http://10.20.11.12:3128
ftp_proxy=http://10.20.11.12:3128
```

3.curl代理

```shell
# 修改 /etc/profile(所有用户) 或 ~/.bashrc(当前用户)
alias curl="curl -x 10.20.11.12:3128"
```

4.全局代理

```shell
# 修改 /etc/profile(所有用户) 或 ~/.bashrc(当前用户)
http_proxy=http://10.20.11.12:3128
https_proxy=http://10.20.11.12:3128
ftp_proxy=http://10.20.11.12:3128
export http_proxy
export https_proxy
export ftp_proxy
```



**代理服务 squid 隐藏真实ip,也就是透明代理**

```shell
修改/etc/squid/squid.config 文件 增加如下 两行;

# Hide client ip
forwarded_for delete

# Turn off via header
via off

# Deny request for original source of a request
follow_x_forwarded_for deny all

See below
request_header_access X-Forwarded-For deny all
```

