## 1.用户管理

```mysql
- 新建用户
> create user 'username'@'%' identified by 'password';

- 分配权限
> grant privileges on databasename.tablename to 'username'@'host';
```

## 2.基础命令

```mysql
> show variables like '%max_connection%';  查看最大连接数
> select USER , count(*) from information_schema.processlist group by USER;   查看当前连接中各个用户的连接数
> show status like  'Threads%';   查看打开的连接数; Threads_connected ：这个数值指的是打开的连接数.
> SHOW GLOBAL VARIABLES LIKE 'wait_timeout';     查看等待超时时间
> show variables like  '%timeout%';
```

SQL语言共分为四大类：数据定义语言DDL，数据操纵语言DML，数据查询语言DQL，数据控制语言DCL。

```shell
DDL: 数据定义语言  
对象：数据库和表  
关键词： create alter drop truncate  

DML: 数据操纵语言  
对象：记录（行）  
关键词：insert update delete  
truncate和delete的区别：  
truncate是删除表，再重新创建这个表。属于DDL，delete是一条一条删除表中的数据，属于DML。  

DQL: 数据查询语言  
数据查询语言DQL基本结构是由SELECT子句，FROM子句，WHERE子句组成的查询块：  
SELECT <字段名表>  
FROM <表或视图名>  
WHERE <查询条件>  

DCL: 数据控制语言  
数据控制语言DCL用来授予或回收访问数据库的某种特权，并控制数据库操纵事务发生的时间及效果，对数据库实行监视等  
1) GRANT：授权。  
2) ROLLBACK [WORK] TO [SAVEPOINT]：回退到某一点。  
```

