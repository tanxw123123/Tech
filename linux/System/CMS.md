## 1.苹果CMS搭建

环境：ubuntu 20.04

### 1.安装mysql数据库

```shell
$ apt-get update && apt-get upgrade -y -f && apt-get install mysql-server mysql-client
$ mysql -uroot
> create database videocms;
> create user 'video1'@'%' identified by 'videocms';
> grant all on videocms.* to 'video1'@'%';
```

### 2.安装nginx

```shell
$ apt-get install nginx
```

### 3.安装php环境

```shell
$ apt-get update && apt-get install php-fpm php-curl php-mbstring php-mysql php-xml
$ apt-get install php-gd php-zip   # 安装依赖模块
```

修改nginx配置文件：

```shell
$ vim /etc/nginx/sites-available/default

location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    # With php-fpm (or other unix sockets):
    # !!! 注意我们使用 php-fpm 那么这个需要使用  注意 7.2 可能不对
    fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    # With php-cgi (or other tcp sockets):
    #   fastcgi_pass 127.0.0.1:9000;
}

# 上面的 7.2 需要替换为 特定的版本，目前为 7.2

$ nginx -t
$ nginx -s reload
```

### 4.下载苹果cms源代码

```shell
$ git clone https://github.com/magicblack/maccms8.git
# 将苹果CMS的源代码解压缩到 /var/www/html 目录中

$ chown www-data:www-data /var/www/html -R
```

访问：http://ip/index.php

```
admin
mjPZBCQ@F9Ffx^ih
654321

http://8.210.126.138/macms/index.php
http://macms.oball48.com/macms/index.php
```



