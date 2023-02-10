## 1.安装必备环境

系统：Ubuntu

Node.js：版本 ≥ 14.x

npm / yarn ：版本 ≥ 6.x

数据库任选一个：

- SQLite：版本  ≥ 3
- PostgreSQL：版本  ≥ 10
- MySQL：版本  ≥ 5.6
- MariaDB：版本  ≥ 10.1
- MongoDB：版本  ≥ 3.6



## 2.安装nodejs

省略，我自己用nvm安装管理

## 3.安装mysql

```shell
# 1、下载mysql数据库
$ sudo apt-get install mysql-server mysql-client

# 2、连接mysql，上一步创建过程中会要求输入密码，如果没有输入，下面登陆就直接回车；
$ sudo mysql -uroot -p 
	# 没有密码直接回车；

# 3、创建数据库
> CREATE DATABASE strapi;

# 4、创建用户(user123)和密码(password123)，用于登陆上面数据库
> CREATE USER 'strapi'@'%' IDENTIFIED BY 'strapi';

# 给用户(user123)设置权限，使得其有全部权限
> GRANT ALL ON strapi.* TO 'strapi'@'%' IDENTIFIED BY 'strapi' WITH GRANT OPTION;
    
# 4、保存所有更新
> FLUSH PRIVILEGES;

# 5、退出
> EXIT;
```

设置root账号密码

```shell
# 1、登录mysql，有密码输入密码，无密码直接回车
$ sudo mysql

# 2、修改root用户的密码为：password
> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password';

# 3、保存所有更改
> FLUSH PRIVILEGES;

# 4、退出
> quit
```



## 4.安装strapi

```shell
$ npx create-strapi-app wsdy-strapi
? Choose your installation type Custom (manual settings)
? Choose your preferred language JavaScript
? Choose your default database client mysql
? Database name: strapi
? Host: 192.168.254.71
? Port: 3306
? Username: strapi
? Password: ******
? Enable SSL connection: (y/N) 


# 启动strapi
$ cd .../wsdy-strapi
$ npm run develop
```

访问报错：http://192.168.254.71:1337/admin

![image-20230111095342311](D:\Tech\linux\Ronnie\assets\image-20230111095342311.png)

解决办法：

```shell
$ npm run build
$ npm run develop
```

再次访问：http://192.168.254.71:1337/admin



## 5.创建Content type

新建content type

![image-20230111095933098](D:\Tech\linux\Ronnie\assets\image-20230111095933098.png)

![image-20230111100134255](D:\Tech\linux\Ronnie\assets\image-20230111100134255.png)

![image-20230111100159847](D:\Tech\linux\Ronnie\assets\image-20230111100159847.png)

右上角save保存：

![image-20230111100244314](D:\Tech\linux\Ronnie\assets\image-20230111100244314.png)

插值操作：

![image-20230111100336078](D:\Tech\linux\Ronnie\assets\image-20230111100336078.png)

![image-20230111100410611](D:\Tech\linux\Ronnie\assets\image-20230111100410611.png)



查看数据库插值是否成功？

```mysql
mysql> select * from a100s;
+----+--------------+----------------------------+----------------------------+--------------+---------------+---------------+
| id | a_100        | created_at                 | updated_at                 | published_at | created_by_id | updated_by_id |
+----+--------------+----------------------------+----------------------------+--------------+---------------+---------------+
|  1 | hello, a100! | 2023-01-11 10:04:18.571000 | 2023-01-11 10:04:18.571000 | NULL         |             1 |             1 |
+----+--------------+----------------------------+----------------------------+--------------+---------------+---------------+
1 row in set (0.00 sec)

```

大功告成！！！



## 6.拉取gitlab代码

```shell
$ git clone ...........
$ cd /data/wsdy-strapi
$ vim .env

$ npm install
# 报错如下：
FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory

升级内存，再次执行 npm install

$ npm run build
$ npm run develop

访问： http://xxxxxxxxxx:1337/admin
```

